
# NestJS + 어댑터 패턴

어댑터 패턴(Adapter Pattern)은 외부 라이브러리나 API와 내부 코드 간의 인터페이스를 정의하여 결합도를 낮추고 유연성을 높이는 데 사용됩니다.  
NestJS에서는 어댑터 패턴을 통해 외부 서비스나 라이브러리를 캡슐화하고, 내부 로직 변경 없이 쉽게 교체 가능하게 설계할 수 있습니다.

---

## 1. 어댑터 패턴이란?

어댑터 패턴은 다음과 같은 상황에서 유용합니다:
- 외부 라이브러리나 API를 애플리케이션에 통합할 때.
- 외부 코드와 내부 코드 간의 결합도를 줄이고 싶을 때.
- 외부 라이브러리가 변경되더라도 내부 코드를 최소한으로 수정하고 싶을 때.

---

## 2. NestJS에서의 어댑터 패턴

NestJS에서는 어댑터 패턴을 구현하여 외부 API와의 통합을 단순화할 수 있습니다.  
다음은 외부 이메일 전송 서비스를 어댑터 패턴으로 캡슐화한 예제입니다.

---

## 3. 이메일 전송 예제

### 3.1. ExternalEmailService
외부 이메일 전송 서비스를 나타내는 클래스입니다.

```typescript
export class ExternalEmailService {
  sendEmail(to: string, body: string): boolean {
    console.log(`Sending email to ${to}: ${body}`);
    return true; // 이메일 전송 성공
  }
}
```

---

### 3.2. EmailAdapter
어댑터는 외부 서비스와 내부 코드 간의 인터페이스 역할을 합니다.

```typescript
import { Injectable } from '@nestjs/common';
import { ExternalEmailService } from './external-email.service';

@Injectable()
export class EmailAdapter {
  private readonly emailService = new ExternalEmailService();

  send(to: string, message: string): boolean {
    return this.emailService.sendEmail(to, message);
  }
}
```

---

### 3.3. NotificationService
EmailAdapter를 사용하는 내부 서비스입니다.

```typescript
import { Injectable } from '@nestjs/common';
import { EmailAdapter } from './email.adapter';

@Injectable()
export class NotificationService {
  constructor(private readonly emailAdapter: EmailAdapter) {}

  notifyUser(email: string, message: string) {
    const result = this.emailAdapter.send(email, message);
    if (result) {
      console.log('Notification sent successfully!');
    } else {
      console.log('Failed to send notification.');
    }
  }
}
```

---

### 3.4. NotificationController
NotificationService를 사용하는 컨트롤러입니다.

```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { NotificationService } from './notification.service';

@Controller('notifications')
export class NotificationController {
  constructor(private readonly notificationService: NotificationService) {}

  @Post()
  sendNotification(@Body() body: { email: string; message: string }) {
    const { email, message } = body;
    this.notificationService.notifyUser(email, message);
    return { message: 'Notification request processed' };
  }
}
```

---

### 3.5. Module
서비스와 어댑터를 등록하는 모듈입니다.

```typescript
import { Module } from '@nestjs/common';
import { NotificationController } from './notification.controller';
import { NotificationService } from './notification.service';
import { EmailAdapter } from './email.adapter';

@Module({
  controllers: [NotificationController],
  providers: [NotificationService, EmailAdapter],
})
export class NotificationModule {}
```

---

## 4. 실행 결과

### 이메일 전송 요청
**POST /notifications**
```json
{
  "email": "test@example.com",
  "message": "Hello, this is a test message!"
}
```

**콘솔 출력:**
```
Sending email to test@example.com: Hello, this is a test message!
Notification sent successfully!
```

**응답:**
```json
{
  "message": "Notification request processed"
}
```

---

## 5. 어댑터 교체

외부 이메일 서비스가 다른 라이브러리로 변경되었다면, 어댑터만 수정하면 됩니다.

### NewExternalEmailService
새로운 이메일 전송 서비스.

```typescript
export class NewExternalEmailService {
  dispatchEmail(recipient: string, content: string): boolean {
    console.log(`Email sent to ${recipient}: ${content}`);
    return true; // 이메일 전송 성공
  }
}
```

### ModifiedEmailAdapter
새로운 서비스를 사용하도록 어댑터를 수정.

```typescript
import { Injectable } from '@nestjs/common';
import { NewExternalEmailService } from './new-external-email.service';

@Injectable()
export class EmailAdapter {
  private readonly emailService = new NewExternalEmailService();

  send(to: string, message: string): boolean {
    return this.emailService.dispatchEmail(to, message);
  }
}
```

**결과:** 내부 서비스(`NotificationService`)와 컨트롤러는 수정 없이 그대로 작동합니다.

---

## 6. 결론

어댑터 패턴을 사용하면:
1. **외부 서비스와의 결합도 감소**: 내부 코드를 수정하지 않고 외부 서비스를 교체 가능.
2. **유연성 증가**: 다양한 외부 서비스를 쉽게 통합.
3. **재사용성 향상**: 공통된 인터페이스로 여러 외부 서비스를 캡슐화.

NestJS와 어댑터 패턴을 결합하여 외부 API와의 통합을 체계적으로 관리하세요!
