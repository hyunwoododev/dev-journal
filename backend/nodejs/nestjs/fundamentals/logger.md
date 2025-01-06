
# NestJS + Logger 클래스

`Logger` 클래스는 NestJS에서 애플리케이션의 로그를 기록하고 관리하는 데 사용되는 기본 제공 클래스입니다.  
효율적인 디버깅과 운영 관리를 위해 필수적인 기능을 제공합니다.

---

## 주요 역할

1. **로그 기록**:
   - 다양한 로그 레벨(`log`, `error`, `warn`, `debug`, `verbose`)을 사용하여 메시지를 출력합니다.

2. **범위(Scope) 지정**:
   - 각 로그 메시지에 특정 범위를 설정하여 로그의 출처를 명확히 구분합니다.

3. **커스터마이징**:
   - 사용자 정의 로거를 만들어 원하는 형식과 동작으로 로그를 기록할 수 있습니다.

---

## Logger 클래스의 주요 메서드

1. **`log(message: string, context?: string)`**
   - 일반 정보를 로그로 기록합니다.

2. **`error(message: string, trace?: string, context?: string)`**
   - 에러 메시지와 스택 트레이스를 기록합니다.

3. **`warn(message: string, context?: string)`**
   - 경고 메시지를 기록합니다.

4. **`debug(message: string, context?: string)`**
   - 디버깅 메시지를 기록합니다.

5. **`verbose(message: string, context?: string)`**
   - 상세한 정보를 기록합니다.

---

## 사용 예제

### 1. 기본 Logger 사용

#### 서비스에서 로그 기록

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class AppService {
  private readonly logger = new Logger(AppService.name); // 범위 지정

  handleRequest(): void {
    this.logger.log('Handling request'); // 일반 로그
    this.logger.warn('This is a warning'); // 경고 로그
    this.logger.error('An error occurred', 'Error Stack'); // 에러 로그
  }
}
```

**출력**:
```
[Nest] 12345   - 2024-01-01 12:00:00   [AppService] Handling request
[Nest] 12345   - 2024-01-01 12:00:00   [AppService] This is a warning
[Nest] 12345   - 2024-01-01 12:00:00   [AppService] An error occurred
Error Stack
```

---

### 2. 글로벌 로거 설정

`Logger`를 글로벌 로거로 설정하여 애플리케이션 전역에서 사용할 수 있습니다.

#### `main.ts`

```typescript
import { Logger } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // 글로벌 로거 설정
  app.useLogger(new Logger());

  await app.listen(3000);
  Logger.log('Application is running on http://localhost:3000');
}
bootstrap();
```

---

### 3. 사용자 정의 Logger

NestJS의 기본 `Logger`를 확장하여 커스터마이징할 수 있습니다.

#### 사용자 정의 로거 생성

```typescript
import { ConsoleLogger } from '@nestjs/common';

export class CustomLogger extends ConsoleLogger {
  log(message: string, context?: string) {
    super.log(`Custom Log: ${message}`, context);
  }

  error(message: string, trace?: string, context?: string) {
    super.error(`Custom Error: ${message}`, trace, context);
  }
}
```

#### 사용자 정의 로거 적용

```typescript
import { Logger } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { CustomLogger } from './custom-logger.service';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // 사용자 정의 로거 설정
  app.useLogger(new CustomLogger());

  await app.listen(3000);
  Logger.log('Custom Logger is running on http://localhost:3000');
}
bootstrap();
```

---

### 4. 환경별 로깅 레벨 설정

#### 개발 환경과 운영 환경에서 다른 로깅 레벨 사용

```typescript
import { Logger, LogLevel } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: process.env.NODE_ENV === 'production'
      ? ['error', 'warn']
      : ['log', 'error', 'warn', 'debug', 'verbose'], // 로깅 레벨 설정
  });

  await app.listen(3000);
  Logger.log('Application is running on http://localhost:3000');
}
bootstrap();
```

---

## 5. Logger와 서비스 통합

#### 요청별 로그 기록

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class UserService {
  private readonly logger = new Logger(UserService.name);

  getUser(id: string) {
    this.logger.log(`Fetching user with ID: ${id}`);
    // 로직 처리
    return { id, name: 'John Doe' };
  }
}
```

#### 컨트롤러에서 로그 기록

```typescript
import { Controller, Get, Param } from '@nestjs/common';
import { UserService } from './user.service';
import { Logger } from '@nestjs/common';

@Controller('users')
export class UserController {
  private readonly logger = new Logger(UserController.name);

  constructor(private readonly userService: UserService) {}

  @Get(':id')
  getUser(@Param('id') id: string) {
    this.logger.log(`Received request for user ID: ${id}`);
    return this.userService.getUser(id);
  }
}
```

**출력**:
```
[Nest] 12345   - 2024-01-01 12:00:00   [UserController] Received request for user ID: 1
[Nest] 12345   - 2024-01-01 12:00:00   [UserService] Fetching user with ID: 1
```

---

## 6. 결론

NestJS의 `Logger` 클래스는:
1. **다양한 로그 레벨 지원**: `log`, `error`, `warn`, `debug`, `verbose`.
2. **범위 기반 로깅**: 특정 서비스나 컨트롤러에서 발생한 로그를 명확히 식별 가능.
3. **커스터마이징 용이**: 애플리케이션 요구사항에 맞게 사용자 정의 로거 구현 가능.
4. **환경별 로깅 제어**: 개발 환경과 운영 환경에 따라 로깅 수준 조정.

NestJS의 `Logger`를 통해 애플리케이션의 디버깅과 운영 관리를 효율적으로 수행하세요!
