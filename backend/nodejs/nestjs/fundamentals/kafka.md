
# NestJS + Event Listeners & Kafka Integration

NestJS는 이벤트 기반 아키텍처와 Kafka 같은 분산 메시징 시스템을 통합하여 높은 확장성과 안정성을 가진 애플리케이션을 설계할 수 있도록 지원합니다.  
**Event Listeners**와 **Event Emitters**를 사용하여 서비스 간 통신을 구현하고, Kafka를 통해 분산 환경에서도 효율적인 메시징 시스템을 구축할 수 있습니다.

---

## 1. Event Listeners & Kafka란?

### Event Emitters
- **역할**: 특정 이벤트를 발생시킴.
- **장점**: 느슨한 결합을 통해 서비스 간 통신.

### Kafka
- **역할**: 분산 메시징 시스템으로, 높은 처리량과 확장성을 지원.
- **장점**:
  1. 대규모 데이터 처리.
  2. 여러 서비스 간 안정적인 데이터 교환.
  3. 높은 확장성과 메시지 내구성.

---

## 2. NestJS에서 이벤트 기반 아키텍처와 Kafka 통합하기

---

### 2.1. 설치

```bash
npm install @nestjs/event-emitter @nestjs/microservices kafkajs
```

---

### 2.2. Event Emitter 설정

#### EventEmitterModule 등록

```typescript
import { Module } from '@nestjs/common';
import { EventEmitterModule } from '@nestjs/event-emitter';

@Module({
  imports: [EventEmitterModule.forRoot()],
})
export class AppModule {}
```

---

### 2.3. Kafka 설정

#### KafkaModule 생성

```typescript
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'KAFKA_SERVICE',
        transport: Transport.KAFKA,
        options: {
          client: {
            brokers: ['localhost:9092'],
          },
          consumer: {
            groupId: 'nestjs-group',
          },
        },
      },
    ]),
  ],
  exports: [ClientsModule],
})
export class KafkaModule {}
```

---

### 2.4. 이벤트 발행 및 Kafka 메시지 전송

#### 이벤트 정의

```typescript
export class UserCreatedEvent {
  constructor(public readonly userId: number, public readonly email: string) {}
}
```

#### UserService에서 이벤트 발행

```typescript
import { Injectable } from '@nestjs/common';
import { EventEmitter2 } from '@nestjs/event-emitter';
import { UserCreatedEvent } from './user-created.event';

@Injectable()
export class UserService {
  constructor(private readonly eventEmitter: EventEmitter2) {}

  createUser(email: string) {
    const userId = Math.floor(Math.random() * 1000);
    console.log(`User created with ID: ${userId}`);

    const event = new UserCreatedEvent(userId, email);
    this.eventEmitter.emit('user.created', event);
  }
}
```

#### Kafka 메시지 발행

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';
import { UserCreatedEvent } from './user-created.event';
import { ClientKafka } from '@nestjs/microservices';

@Injectable()
export class KafkaPublisherService implements OnModuleInit {
  constructor(private readonly kafkaClient: ClientKafka) {}

  async onModuleInit() {
    await this.kafkaClient.connect();
  }

  @OnEvent('user.created')
  async handleUserCreatedEvent(event: UserCreatedEvent) {
    console.log(`Publishing event to Kafka: ${event.email}`);
    this.kafkaClient.emit('user.created', JSON.stringify(event));
  }
}
```

---

### 2.5. Kafka 메시지 소비 및 이벤트 리스너

#### Kafka 메시지 소비

```typescript
import { Controller } from '@nestjs/common';
import { EventPattern } from '@nestjs/microservices';

@Controller()
export class KafkaConsumerController {
  @EventPattern('user.created')
  handleUserCreatedMessage(message: any) {
    console.log(`Kafka message received: ${JSON.stringify(message.value)}`);
  }
}
```

---

### 2.6. 모듈 통합

```typescript
import { Module } from '@nestjs/common';
import { UserService } from './user.service';
import { KafkaPublisherService } from './kafka-publisher.service';
import { KafkaConsumerController } from './kafka-consumer.controller';
import { KafkaModule } from './kafka.module';

@Module({
  imports: [KafkaModule],
  providers: [UserService, KafkaPublisherService],
  controllers: [KafkaConsumerController],
})
export class UserModule {}
```

---

## 3. 고급 Kafka 기능

---

### 3.1. Kafka 파티셔닝 및 복제

- **파티셔닝**: 메시지를 특정 파티션으로 분산하여 병렬 처리 가능.
- **복제**: 메시지 복제를 통해 데이터 손실 방지.

#### Kafka 설정에서 파티셔닝 추가

```typescript
options: {
  client: {
    brokers: ['localhost:9092'],
  },
  consumer: {
    groupId: 'nestjs-group',
  },
  producer: {
    partition: 0, // 특정 파티션으로 메시지 전송
  },
}
```

---

### 3.2. 메시지 재시도 및 오류 처리

#### Kafka 재시도 옵션 설정

```typescript
options: {
  client: {
    brokers: ['localhost:9092'],
  },
  consumer: {
    groupId: 'nestjs-group',
    retry: {
      retries: 3, // 최대 재시도 횟수
    },
  },
}
```

#### 메시지 처리 실패 로깅

```typescript
import { EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class KafkaConsumerController {
  @EventPattern('user.created')
  async handleUserCreatedMessage(@Payload() message: any) {
    try {
      console.log(`Kafka message processed: ${message.value}`);
    } catch (error) {
      console.error(`Message processing failed: ${error.message}`);
    }
  }
}
```

---

## 4. 실행 결과

### 이벤트 발행 및 Kafka 메시지 전송

**UserService 호출:**

```typescript
userService.createUser('user@example.com');
```

**콘솔 출력:**
```
User created with ID: 123
Publishing event to Kafka: user@example.com
Kafka message received: {"userId":123,"email":"user@example.com"}
```

---

## 5. 결론

NestJS의 Event Listeners와 Kafka 통합은 다음과 같은 장점을 제공합니다:
1. **확장성**:
   - Kafka의 분산 아키텍처로 대규모 트래픽 처리 가능.
2. **비동기 작업 처리**:
   - 이벤트 기반 메시징으로 비동기 작업 처리 최적화.
3. **유연한 통합**:
   - NestJS의 EventEmitter와 Kafka를 활용하여 강력한 이벤트 기반 시스템 설계.

NestJS와 Kafka를 통합하여 확장 가능하고 안정적인 애플리케이션을 구축하세요!
