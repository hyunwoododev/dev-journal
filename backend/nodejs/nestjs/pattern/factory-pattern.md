
# NestJS + 팩토리(Factory) 패턴

팩토리 패턴(Factory Pattern)은 객체 생성 로직을 별도의 클래스 또는 메서드로 분리하여 코드의 유연성과 확장성을 높이는 디자인 패턴입니다.  
NestJS에서 팩토리 패턴은 동적으로 객체를 생성하거나, 조건에 따라 다른 객체를 반환할 때 매우 유용합니다.

---

## 1. 팩토리 패턴이란?

팩토리 패턴은 다음과 같은 상황에서 유용합니다:
- 객체 생성 로직이 복잡하거나 동적으로 변경되어야 할 때.
- 객체 생성 코드를 클라이언트 코드와 분리하여 결합도를 줄이고 싶을 때.
- 생성할 객체의 유형이 런타임에 결정되는 경우.

---

## 2. NestJS에서의 팩토리 패턴

NestJS에서 팩토리 패턴은 객체 생성의 책임을 팩토리 클래스에 위임함으로써 다음과 같은 이점을 제공합니다:
- 의존성 주입(DI)과 조합하여 객체 생성 로직을 캡슐화.
- 동적으로 다른 객체를 반환하거나, 복잡한 초기화 로직 처리 가능.

---

## 3. 팩토리 패턴을 사용한 예제

### 예제: 알림(Notification) 시스템 구현

**시나리오**: 이메일, SMS, 푸시 알림 등 다양한 알림 방식을 지원하며, 클라이언트 요청에 따라 적절한 알림 서비스를 생성합니다.

---

### 3.1. Notification 인터페이스
모든 알림 서비스는 `Notification` 인터페이스를 구현합니다.

```typescript
export interface Notification {
  send(message: string): string;
}
```

---

### 3.2. 구체적인 알림 서비스
이메일, SMS, 푸시 알림 서비스를 구현합니다.

```typescript
import { Injectable } from '@nestjs/common';
import { Notification } from './notification.interface';

@Injectable()
export class EmailNotification implements Notification {
  send(message: string): string {
    return `Email sent: ${message}`;
  }
}

@Injectable()
export class SmsNotification implements Notification {
  send(message: string): string {
    return `SMS sent: ${message}`;
  }
}

@Injectable()
export class PushNotification implements Notification {
  send(message: string): string {
    return `Push Notification sent: ${message}`;
  }
}
```

---

### 3.3. NotificationFactory
팩토리 클래스는 요청에 따라 적절한 알림 서비스를 생성합니다.

```typescript
import { Injectable } from '@nestjs/common';
import { EmailNotification } from './email-notification.service';
import { SmsNotification } from './sms-notification.service';
import { PushNotification } from './push-notification.service';
import { Notification } from './notification.interface';

@Injectable()
export class NotificationFactory {
  constructor(
    private readonly emailNotification: EmailNotification,
    private readonly smsNotification: SmsNotification,
    private readonly pushNotification: PushNotification,
  ) {}

  createNotification(type: string): Notification {
    switch (type) {
      case 'email':
        return this.emailNotification;
      case 'sms':
        return this.smsNotification;
      case 'push':
        return this.pushNotification;
      default:
        throw new Error(`Unsupported notification type: ${type}`);
    }
  }
}
```

---

### 3.4. NotificationService
NotificationFactory를 사용하여 동적으로 알림 서비스를 생성합니다.

```typescript
import { Injectable } from '@nestjs/common';
import { NotificationFactory } from './notification-factory.service';

@Injectable()
export class NotificationService {
  constructor(private readonly notificationFactory: NotificationFactory) {}

  sendNotification(type: string, message: string): string {
    const notification = this.notificationFactory.createNotification(type);
    return notification.send(message);
  }
}
```

---

### 3.5. NotificationController
NotificationService를 호출하여 클라이언트 요청에 따라 알림을 전송합니다.

```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { NotificationService } from './notification.service';

@Controller('notifications')
export class NotificationController {
  constructor(private readonly notificationService: NotificationService) {}

  @Post()
  send(@Body() body: { type: string; message: string }): string {
    const { type, message } = body;
    return this.notificationService.sendNotification(type, message);
  }
}
```

---

### 3.6. NotificationModule
알림 관련 모든 서비스를 모듈로 정의합니다.

```typescript
import { Module } from '@nestjs/common';
import { NotificationController } from './notification.controller';
import { NotificationService } from './notification.service';
import { NotificationFactory } from './notification-factory.service';
import { EmailNotification } from './email-notification.service';
import { SmsNotification } from './sms-notification.service';
import { PushNotification } from './push-notification.service';

@Module({
  controllers: [NotificationController],
  providers: [
    NotificationService,
    NotificationFactory,
    EmailNotification,
    SmsNotification,
    PushNotification,
  ],
})
export class NotificationModule {}
```

---

## 4. 실행 결과

### 이메일 알림 전송 요청
**POST /notifications**
```json
{
  "type": "email",
  "message": "Welcome to our service!"
}
```

**응답:**
```json
"Email sent: Welcome to our service!"
```

---

### SMS 알림 전송 요청
**POST /notifications**
```json
{
  "type": "sms",
  "message": "Your verification code is 1234"
}
```

**응답:**
```json
"SMS sent: Your verification code is 1234"
```

---

### 푸시 알림 전송 요청
**POST /notifications**
```json
{
  "type": "push",
  "message": "You have a new message!"
}
```

**응답:**
```json
"Push Notification sent: You have a new message!"
```

---

### 지원하지 않는 알림 타입 요청
**POST /notifications**
```json
{
  "type": "fax",
  "message": "This won't work"
}
```

**응답:**
```json
{
  "statusCode": 500,
  "message": "Unsupported notification type: fax",
  "error": "Internal Server Error"
}
```

---

## 5. 결론

NestJS에서 팩토리 패턴은 다음과 같은 이점을 제공합니다:
1. **객체 생성 로직의 캡슐화**: 클라이언트 코드가 객체 생성 방식을 알 필요가 없습니다.
2. **유연성 향상**: 새로운 알림 서비스 추가 시 기존 코드 수정 최소화.
3. **유지보수성 향상**: 객체 생성 로직이 중앙 집중화되어 관리가 용이.

팩토리 패턴은 다양한 상황에서 객체 생성과 관리의 복잡성을 줄이고, 확장 가능한 애플리케이션 설계에 도움을 줍니다!
