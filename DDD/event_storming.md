
# 이벤트 스토밍 (Event Storming)

---

## 1. 이벤트 스토밍의 정의와 목적

### 정의
- 이벤트 스토밍은 비즈니스 이벤트를 시각적으로 정리하고 시스템 요구사항과 비즈니스 로직을 탐색하기 위한 협업 중심의 워크샵입니다.
- 도메인 전문가와 기술팀(개발자, 설계자 등)이 함께 시스템의 이벤트 중심 관점을 탐구합니다.

### 목적
- 복잡한 도메인을 단순화하고, 시스템의 설계와 요구사항을 명확히 이해.
- 빠진 요구사항이나 비효율적인 설계를 발견.
- 협업을 통해 비즈니스 전문가와 개발자가 동일한 이해를 갖게 함.

---

## 2. 진행 방식

### 2.1 사전 준비

#### 필요한 도구
- 큰 벽 또는 보드, 다양한 색상의 스티커(또는 디지털 툴: Miro, Lucidchart 등).
- 스티커 색상별 역할:
  - **주황색**: 이벤트(Ex: `사용자 생성됨`).
  - **파란색**: 커맨드(명령, Ex: `사용자 생성 요청`).
  - **노란색**: 엔티티 또는 객체(Ex: `사용자`, `계좌`).
  - **빨간색**: 외부 시스템(Ex: SMTP 서버, 외부 API).

#### 참여자
- 비즈니스 도메인 전문가(현업 관계자).
- IT 기술팀(개발자, 설계자).
- 프로세스 조정자(Moderator, Facilitator).

---

### 2.2 워크샵의 단계

1. **도메인 이벤트 나열**
   - 비즈니스에서 발생할 수 있는 모든 이벤트를 팀원들과 함께 브레인스토밍.
   - 이벤트는 과거형, 수동형으로 표현(Ex: `상품이 장바구니에 추가됨`).
   - 왼쪽에서 오른쪽으로 시간의 흐름에 따라 나열.

2. **커맨드(명령) 도출**
   - 이벤트를 발생시키는 원인이 되는 커맨드를 도출.
   - 커맨드는 파란색 스티커로 작성(Ex: `장바구니에 상품 추가`).

3. **엔티티 식별**
   - 이벤트와 커맨드가 연관된 주요 엔티티(또는 객체)를 노란색 스티커로 정의.
   - 예: `상품`, `장바구니`.

4. **외부 시스템 및 의존성 정의**
   - 이벤트와 연관된 외부 시스템을 빨간색 스티커로 추가.
   - 예: 외부 이메일 서버(SMTP)와의 연동.

5. **플로우 검증**
   - 시간 순서대로 나열된 이벤트 플로우를 검토.
   - 누락된 이벤트나 불필요한 이벤트를 발견하고 수정.

---

## 3. 활용 사례와 시나리오

### 3.1 사용자 등록 시나리오
1. 사용자 생성 요청(파란색 커맨드).
2. `사용자 생성됨` 이벤트(주황색 이벤트).
3. `이메일 인증 요청됨` 이벤트 발생.
4. 외부 시스템(빨간색)으로 이메일 인증 요청 전송.

### 3.2 주문 프로세스
1. `장바구니에 상품 추가` 커맨드 발생.
2. `상품이 장바구니에 추가됨` 이벤트.
3. `주문 생성 요청` 커맨드 발생.
4. `주문이 생성됨` 이벤트와 외부 결제 시스템 호출.

---

## 4. 이벤트 스토밍의 주요 특징

### 반복 가능
- 설계 과정에서 필요에 따라 반복적으로 수행하여 개선 가능.

### 시각적이고 직관적
- 스티커를 활용하여 모든 팀원이 이해하기 쉽도록 정보를 공유.

### 도메인 전문가와의 협업 강화
- 비즈니스 전문가가 적극적으로 참여하여 시스템의 요구사항을 명확히 정의.

### 변화 용이성
- 변경 사항이 발생했을 때, 스티커를 이동하거나 수정함으로써 손쉽게 설계를 조정할 수 있음.

---

## 5. 이벤트 스토밍의 장점

1. **효율적 커뮤니케이션**
   - 비즈니스와 기술 팀이 동일한 목표와 이해를 공유.

2. **빠른 요구사항 도출**
   - 초기 단계에서 복잡한 도메인을 시각적으로 표현해 빠르게 요구사항을 수집.

3. **설계 품질 향상**
   - 반복적으로 수행하며 설계를 개선.

4. **변화에 대한 민첩성**
   - 변화하는 요구사항에 손쉽게 대응 가능.

---
