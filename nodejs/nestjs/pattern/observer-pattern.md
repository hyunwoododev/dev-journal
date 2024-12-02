
# NestJS + Observer 패턴

**Observer 패턴**은 객체의 상태 변화를 관찰하여 관련 동작을 자동으로 수행하는 디자인 패턴입니다.  
NestJS의 `EventEmitter`를 활용하면 효율적이고 확장 가능한 이벤트 기반 시스템을 설계할 수 있습니다.

---

## 1. Observer 패턴이란?

Observer 패턴은 한 객체의 상태가 변경될 때, 해당 변경을 의존하는 다른 객체들에게 알리고 자동으로 반응하도록 하는 패턴입니다.

### 주요 특징:
1. **느슨한 결합**:
   - 발행자와 구독자 간의 강한 의존성을 제거.
2. **확장성**:
   - 새로운 구독자를 쉽게 추가 가능.
3. **비동기 처리**:
   - 이벤트 기반 비동기 작업 처리 지원.

---

## 2. NestJS에서 Observer 패턴 구현

---

### 2.1. 이벤트 정의

#### 이벤트 데이터 모델

```typescript
// events/user-created.event.ts
export class UserCreatedEvent {
  constructor(public readonly userId: string, public readonly email: string) {}
}
```

---

### 2.2. 이벤트 발행자 (Subject)

```typescript
import { Injectable } from '@nestjs/common';
import { EventEmitter2 } from '@nestjs/event-emitter';
import { UserCreatedEvent } from './events/user-created.event';

@Injectable()
export class UserService {
  constructor(private readonly eventEmitter: EventEmitter2) {}

  async createUser(email: string): Promise<void> {
    const userId = Math.floor(Math.random() * 1000).toString();
    console.log(`User created with ID: ${userId}`);

    const event = new UserCreatedEvent(userId, email);
    this.eventEmitter.emit('user.created', event); // 이벤트 발행
  }
}
```

---

### 2.3. 이벤트 리스너 (Observer)

#### 사용자 알림 처리 리스너

```typescript
import { Injectable } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';
import { UserCreatedEvent } from './events/user-created.event';

@Injectable()
export class NotificationService {
  @OnEvent('user.created')
  handleUserCreated(event: UserCreatedEvent): void {
    console.log(`Sending notification to ${event.email}`);
  }
}
```

---

#### 사용자 감사 로그 처리 리스너

```typescript
import { Injectable } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';
import { UserCreatedEvent } from './events/user-created.event';

@Injectable()
export class AuditService {
  @OnEvent('user.created')
  handleUserCreated(event: UserCreatedEvent): void {
    console.log(`Logging creation of user ID: ${event.userId}`);
  }
}
```

---

### 2.4. 모듈 통합

```typescript
import { Module } from '@nestjs/common';
import { EventEmitterModule } from '@nestjs/event-emitter';
import { UserService } from './user.service';
import { NotificationService } from './notification.service';
import { AuditService } from './audit.service';

@Module({
  imports: [EventEmitterModule.forRoot()],
  providers: [UserService, NotificationService, AuditService],
})
export class UserModule {}
```

---

## 3. 고급 Observer 패턴 기능

---

### 3.1. 동적 이벤트 등록 및 해제

#### 이벤트 등록/해제

```typescript
import { Injectable } from '@nestjs/common';
import { EventEmitter2 } from '@nestjs/event-emitter';

@Injectable()
export class DynamicEventService {
  constructor(private readonly eventEmitter: EventEmitter2) {}

  registerDynamicEvent() {
    const handler = (data: any) => {
      console.log('Dynamic Event Received:', data);
    };

    this.eventEmitter.on('dynamic.event', handler);

    return () => {
      this.eventEmitter.off('dynamic.event', handler);
    };
  }
}
```

---

### 3.2. 이벤트 필터링

#### 조건부 이벤트 처리

```typescript
import { Injectable } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';
import { UserCreatedEvent } from './events/user-created.event';

@Injectable()
export class FilteredNotificationService {
  @OnEvent('user.created', { filter: (event: UserCreatedEvent) => +event.userId > 500 })
  handleFilteredEvent(event: UserCreatedEvent): void {
    console.log(`Notification sent to user ID greater than 500: ${event.email}`);
  }
}
```

---

### 3.3. 비동기 이벤트 처리

#### 비동기 로직 추가

```typescript
import { Injectable } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';
import { UserCreatedEvent } from './events/user-created.event';

@Injectable()
export class AsyncNotificationService {
  @OnEvent('user.created')
  async handleAsyncEvent(event: UserCreatedEvent): Promise<void> {
    await new Promise(resolve => setTimeout(resolve, 1000));
    console.log(`Asynchronous notification sent to: ${event.email}`);
  }
}
```

---

## 4. 실행 결과

### 사용 예시

**사용자 생성 요청:**

```typescript
userService.createUser('user@example.com');
```

**결과:**
```
User created with ID: 123
Sending notification to user@example.com
Logging creation of user ID: 123
```

---

### 동적 이벤트 처리

**동적 이벤트 등록 및 호출:**

```typescript
const unregister = dynamicEventService.registerDynamicEvent();
eventEmitter.emit('dynamic.event', { key: 'value' });
unregister(); // 이벤트 해제
```

**결과:**
```
Dynamic Event Received: { key: 'value' }
```

---

## 5. 결론

NestJS의 Observer 패턴은 다음과 같은 장점을 제공합니다:
1. **느슨한 결합**:
   - 발행자와 리스너 간의 독립성 유지.
2. **확장성**:
   - 새로운 리스너를 손쉽게 추가.
3. **비동기 처리**:
   - 비동기 작업을 통한 성능 최적화.

Observer 패턴과 NestJS의 EventEmitter를 활용하여 이벤트 중심의 효율적이고 확장 가능한 애플리케이션을 설계하세요!
