# **NestJS 기반 Saga 패턴 MSA 구현 가이드**

이 문서는 **NestJS**를 사용하여 **Saga 패턴**을 구현하는 마이크로서비스 아키텍처(MSA) 예제의 흐름을 설명합니다. Kafka를 사용한 이벤트 기반 통신과 트랜잭션 정합성을 보장하는 방법에 중점을 둡니다.

---

## **1. 개요**
Saga 패턴은 **분산 트랜잭션**을 관리하는 패턴으로, 트랜잭션을 여러 단계의 **로컬 트랜잭션**으로 나누어 관리합니다. 중앙 집중식 조율자인 **Saga Orchestrator**가 각 단계를 순차적으로 조율하고, 실패 시 롤백 이벤트를 실행합니다.

---

## **2. 시나리오**
사용자가 주문을 생성하면 다음 트랜잭션이 순차적으로 진행됩니다:

1. **Order Service**: 주문 생성 및 상태 관리
2. **Payment Service**: 결제 처리
3. **Inventory Service**: 재고 확인 및 차감
4. **Shipping Service**: 배송 예약

### **실패 시나리오**
- **결제 실패**: 주문 취소 (Order 상태를 "CANCELED"로 업데이트)
- **재고 부족**: 결제 취소 및 주문 취소
- **배송 실패**: 재고 롤백 및 결제 취소 후 주문 취소

이 모든 흐름은 **Kafka Topics**을 통해 이벤트 기반으로 관리됩니다.

---

## **3. Kafka Topics 정의**
이벤트를 구분하고 각 서비스 간의 흐름을 조율하기 위해 다음과 같은 Kafka Topics를 사용합니다:

| **Topic 이름**        | **발행 이벤트**                           | **발행 서비스**      |
|-----------------------|---------------------------------------|---------------------|
| `order-events`        | ORDER_CREATED, ORDER_CANCELED          | Order Service       |
| `payment-events`      | PAYMENT_SUCCESS, PAYMENT_FAILED        | Payment Service     |
| `inventory-events`    | INVENTORY_SUCCESS, INVENTORY_FAILED    | Inventory Service   |
| `shipping-events`     | SHIPPING_SUCCESS, SHIPPING_FAILED      | Shipping Service    |

이 토픽들은 **Saga Orchestrator**가 중앙에서 구독하여 이벤트 흐름을 관리합니다.

---

## **4. 서비스별 역할**

### **4.1 Order Service**
- **역할**: 주문을 생성하고 이벤트를 발행합니다.
- **이벤트 발행**:
  - `ORDER_CREATED` → 주문 생성 후 발행
  - `ORDER_CANCELED` → 주문 실패 시 발행

### **4.2 Payment Service**
- **역할**: 결제를 처리하고 결과를 발행합니다.
- **이벤트 발행**:
  - `PAYMENT_SUCCESS` → 결제 성공 시
  - `PAYMENT_FAILED` → 결제 실패 시

### **4.3 Inventory Service**
- **역할**: 재고를 확인하고 차감합니다.
- **이벤트 발행**:
  - `INVENTORY_SUCCESS` → 재고 차감 성공 시
  - `INVENTORY_FAILED` → 재고 부족 시

### **4.4 Shipping Service**
- **역할**: 배송을 예약하고 결과를 발행합니다.
- **이벤트 발행**:
  - `SHIPPING_SUCCESS` → 배송 예약 성공 시
  - `SHIPPING_FAILED` → 배송 예약 실패 시

### **4.5 Saga Orchestrator**
- **역할**: 중앙 조율자로 이벤트를 구독하고, 다음 단계의 이벤트를 발행합니다.
- **주요 로직**:
  - `ORDER_CREATED` → `PAYMENT_REQUESTED` 이벤트 발행
  - `PAYMENT_SUCCESS` → `INVENTORY_REQUESTED` 이벤트 발행
  - `INVENTORY_SUCCESS` → `SHIPPING_REQUESTED` 이벤트 발행
  - 실패 발생 시 롤백 이벤트 발행

---

## **5. 전체 트랜잭션 흐름**

### **5.1 성공 흐름**
1. **Order Service** → `order-events`에 `ORDER_CREATED` 발행
2. **Saga Orchestrator** → `PAYMENT_REQUESTED` 이벤트 발행
3. **Payment Service** → `PAYMENT_SUCCESS` 이벤트 발행
4. **Saga Orchestrator** → `INVENTORY_REQUESTED` 이벤트 발행
5. **Inventory Service** → `INVENTORY_SUCCESS` 이벤트 발행
6. **Saga Orchestrator** → `SHIPPING_REQUESTED` 이벤트 발행
7. **Shipping Service** → `SHIPPING_SUCCESS` 이벤트 발행

### **5.2 실패 및 롤백 흐름**
- **결제 실패**:
  - Payment Service → `PAYMENT_FAILED` 발행
  - Saga Orchestrator → `ORDER_CANCELED` 발행

- **재고 부족**:
  - Inventory Service → `INVENTORY_FAILED` 발행
  - Saga Orchestrator → `PAYMENT_CANCEL` 및 `ORDER_CANCELED` 발행

- **배송 실패**:
  - Shipping Service → `SHIPPING_FAILED` 발행
  - Saga Orchestrator → `INVENTORY_ROLLBACK`, `PAYMENT_CANCEL`, `ORDER_CANCELED` 발행

---

## **6. 코드 예시**
**흐름을 이해하기 위한 예시 코드입니다**.

### **6.1 Order Service**
```typescript
await this.producer.send({
  topic: 'order-events',
  messages: [{ value: JSON.stringify({ orderId, status: 'ORDER_CREATED' }) }],
});
```

### **6.2 Payment Service**
```typescript
if (paymentSuccess) {
  await this.producer.send({
    topic: 'payment-events',
    messages: [{ value: JSON.stringify({ orderId, status: 'PAYMENT_SUCCESS' }) }],
  });
} else {
  await this.producer.send({
    topic: 'order-events',
    messages: [{ value: JSON.stringify({ orderId, status: 'PAYMENT_FAILED' }) }],
  });
}
```

### **6.3 Saga Orchestrator**
```typescript
if (event.status === 'PAYMENT_FAILED') {
  console.log(`Payment failed for Order ${event.orderId}. Cancelling order.`);
  await this.producer.send({
    topic: 'order-events',
    messages: [{ value: JSON.stringify({ orderId: event.orderId, status: 'ORDER_CANCELED' }) }],
  });
}
```

---

## **7. 결론**
이 예제는 **Orchestration 기반의 Saga 패턴**을 통해 분산 시스템에서 트랜잭션의 정합성을 유지하는 방법을 보여줍니다.

### **핵심 포인트**
1. **Kafka Topics**를 통해 서비스 간 이벤트 전달
2. **Saga Orchestrator**를 사용한 단계별 트랜잭션 조율
3. 실패 시 이벤트 기반 롤백 처리

이 문서를 통해 Saga 패턴의 흐름과 구현 원리를 이해할 수 있습니다. 각 서비스의 역할과 이벤트 흐름을 잘 정의하면 트랜잭션의 정합성을 유지하면서 분산 시스템을 안정적으로 운영할 수 있습니다.