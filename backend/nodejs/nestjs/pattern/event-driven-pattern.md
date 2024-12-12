
# NestJS + Event-driven 패턴

Event-driven 패턴은 서비스 간의 느슨한 결합을 제공하고 비동기 작업 처리를 효율적으로 수행할 수 있도록 합니다.  
NestJS는 CQRS 패키지의 이벤트 시스템을 활용하여 이벤트 기반 아키텍처를 쉽게 구현할 수 있습니다.

---

## 1. Event-driven 패턴이란?

이벤트 기반 아키텍처는 특정 동작이 발생했을 때 해당 이벤트를 처리하는 핸들러가 동작하도록 설계됩니다:
- **발행자(Publisher)**: 이벤트를 생성 및 발행.
- **구독자(Subscriber)**: 발행된 이벤트를 처리.

**장점**:
- 서비스 간 느슨한 결합.
- 비동기 작업 처리 용이.
- 확장성과 유지보수성 향상.

---

## 2. NestJS에서의 Event-driven 패턴

NestJS는 `@nestjs/cqrs` 패키지를 사용하여 이벤트를 쉽게 처리할 수 있도록 지원합니다.  
다음은 사용자 생성 시 이메일 알림을 보내는 예제를 통해 Event-driven 패턴을 설명합니다.

---

## 3. 사용자 생성과 알림 전송 예제

### 3.1. Event: UserCreatedEvent
사용자가 생성되었을 때 발행되는 이벤트 클래스입니다.

```typescript
export class UserCreatedEvent {
  constructor(public readonly userId: number, public readonly email: string) {}
}
```

---

### 3.2. UserService
사용자를 생성하고, 이벤트를 발행합니다.

```typescript
import { Injectable } from '@nestjs/common';
import { EventBus } from '@nestjs/cqrs';
import { UserCreatedEvent } from './user-created.event';

@Injectable()
export class UserService {
  constructor(private readonly eventBus: EventBus) {}

  async createUser(email: string): Promise<void> {
    const userId = Math.floor(Math.random() * 1000); // 가짜 사용자 ID 생성
    console.log(`User created with ID: ${userId}`);
    this.eventBus.publish(new UserCreatedEvent(userId, email));
  }
}
```

---

### 3.3. Event Listener: EmailNotificationHandler
이벤트를 처리하여 이메일 알림을 전송합니다.

```typescript
import { EventsHandler, IEventHandler } from '@nestjs/cqrs';
import { UserCreatedEvent } from './user-created.event';

@EventsHandler(UserCreatedEvent)
export class EmailNotificationHandler implements IEventHandler<UserCreatedEvent> {
  handle(event: UserCreatedEvent): void {
    console.log(`Sending email to ${event.email} for user ${event.userId}`);
    // 이메일 전송 로직 추가 가능
  }
}
```

---

### 3.4. Controller
UserService를 호출하여 이벤트를 발행합니다.

```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { UserService } from './user.service';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  async createUser(@Body() body: { email: string }) {
    const { email } = body;
    await this.userService.createUser(email);
    return { message: 'User created and notification sent' };
  }
}
```

---

### 3.5. Module
핸들러와 서비스를 등록하는 모듈입니다.

```typescript
import { Module } from '@nestjs/common';
import { CqrsModule } from '@nestjs/cqrs';
import { UserController } from './user.controller';
import { UserService } from './user.service';
import { EmailNotificationHandler } from './email-notification.handler';

@Module({
  imports: [CqrsModule],
  controllers: [UserController],
  providers: [UserService, EmailNotificationHandler],
})
export class UserModule {}
```

---

## 4. 실행 결과

### 사용자 생성 요청
**POST /users**
```json
{
  "email": "user@example.com"
}
```

**콘솔 출력**:
```
User created with ID: 123
Sending email to user@example.com for user 123
```

**응답**:
```json
{
  "message": "User created and notification sent"
}
```

---

## 5. 추가적인 확장

### 5.1. 이벤트를 파일로 로깅
새로운 이벤트 리스너를 추가하여 이벤트를 파일로 기록할 수 있습니다.

#### FileLoggerHandler
```typescript
import { EventsHandler, IEventHandler } from '@nestjs/cqrs';
import { UserCreatedEvent } from './user-created.event';
import * as fs from 'fs';

@EventsHandler(UserCreatedEvent)
export class FileLoggerHandler implements IEventHandler<UserCreatedEvent> {
  handle(event: UserCreatedEvent): void {
    const log = `User Created: ID=${event.userId}, Email=${event.email}
`;
    fs.appendFileSync('user-events.log', log);
    console.log('Event logged to file');
  }
}
```

**모듈에 추가**:
```typescript
@Module({
  imports: [CqrsModule],
  controllers: [UserController],
  providers: [UserService, EmailNotificationHandler, FileLoggerHandler],
})
export class UserModule {}
```

---

## 6. 결론

NestJS에서 Event-driven 패턴은 다음과 같은 장점을 제공합니다:
1. **서비스 간 느슨한 결합**: 이벤트 발행자와 리스너가 독립적으로 동작.
2. **확장성**: 새로운 이벤트 핸들러를 추가하여 기능 확장 가능.
3. **비동기 처리**: 대규모 이벤트를 효율적으로 처리.

Event-driven 패턴은 마이크로서비스 아키텍처 및 대규모 애플리케이션에서 필수적인 패턴입니다.  
NestJS의 이벤트 시스템을 활용하여 확장 가능하고 유지보수하기 쉬운 애플리케이션을 구축하세요!
