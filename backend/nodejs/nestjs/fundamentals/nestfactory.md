
# NestJS + NestFactory 클래스

`NestFactory`는 NestJS 애플리케이션의 생성 및 초기화를 담당하는 핵심 클래스입니다.  
이를 사용하여 애플리케이션을 부트스트랩하고, HTTP 서버 또는 다른 종류의 애플리케이션을 실행할 수 있습니다.

---

## 주요 역할

1. **애플리케이션 생성**:
   - NestJS 애플리케이션 인스턴스를 생성합니다.
   - HTTP 기반 서버, WebSocket 서버, 또는 Microservice를 실행할 수 있습니다.

2. **모듈 초기화**:
   - 루트 모듈을 전달받아 의존성 주입 컨테이너를 구성하고 초기화합니다.

3. **서버 설정**:
   - 글로벌 미들웨어, 필터, 파이프, 가드 등을 설정할 수 있습니다.

---

## NestFactory의 주요 메서드

### 1. `create()`
NestJS 애플리케이션을 생성합니다.

- **기본 HTTP 서버 생성**:
  ```typescript
  const app = await NestFactory.create(AppModule);
  ```

- **기본 옵션 설정**:
  ```typescript
  import { Logger } from '@nestjs/common';

  const app = await NestFactory.create(AppModule, {
    logger: ['error', 'warn'], // 로그 수준 설정
  });
  ```

---

### 2. `createMicroservice()`
마이크로서비스 애플리케이션을 생성합니다.

```typescript
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';

const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
  transport: Transport.TCP,
  options: { port: 3001 },
});
app.listen(() => console.log('Microservice is listening'));
```

---

### 3. `createApplicationContext()`
HTTP 서버 없이 애플리케이션 컨텍스트를 생성합니다.  
백그라운드 작업이나 CLI 도구에서 유용합니다.

```typescript
import { NestFactory } from '@nestjs/core';

const appContext = await NestFactory.createApplicationContext(AppModule);
// 애플리케이션 컨텍스트를 활용한 비즈니스 로직
await appContext.close();
```

---

## HTTP 애플리케이션 생성 예제

### 1. 기본 HTTP 서버 생성

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
  console.log('Application is running on http://localhost:3000');
}
bootstrap();
```

---

### 2. 글로벌 미들웨어 및 설정

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // 글로벌 파이프 설정
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true, // DTO에서 정의되지 않은 속성 제거
      forbidNonWhitelisted: true, // 정의되지 않은 속성이 들어오면 예외 발생
    }),
  );

  // CORS 설정
  app.enableCors();

  await app.listen(3000);
  console.log('Application is running on http://localhost:3000');
}
bootstrap();
```

---

### 3. WebSocket 서버와 통합

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { IoAdapter } from '@nestjs/platform-socket.io';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useWebSocketAdapter(new IoAdapter(app));
  await app.listen(3000);
  console.log('WebSocket server is running on http://localhost:3000');
}
bootstrap();
```

---

## 마이크로서비스 애플리케이션 생성

NestFactory를 사용하여 마이크로서비스를 생성하고 설정할 수 있습니다.

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.TCP,
    options: {
      port: 3001,
    },
  });

  await app.listen();
  console.log('Microservice is running on port 3001');
}
bootstrap();
```

---

## 유틸리티 애플리케이션 생성

HTTP 서버 없이 애플리케이션 컨텍스트만 생성할 수도 있습니다.  
이를 통해 CLI 도구나 테스트 환경에서 NestJS를 활용할 수 있습니다.

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const appContext = await NestFactory.createApplicationContext(AppModule);

  const myService = appContext.get(MyService);
  await myService.performTask();

  await appContext.close();
}
bootstrap();
```

---

## 결론

`NestFactory`는 NestJS 애플리케이션의 초기화 및 실행에 있어 중요한 역할을 합니다:
1. 다양한 애플리케이션 유형(HTTP, WebSocket, Microservice)을 지원.
2. 전역 설정 및 초기화 작업 처리.
3. 테스트와 CLI 유틸리티를 위한 컨텍스트 생성 가능.

NestFactory를 통해 다양한 종류의 애플리케이션을 유연하고 효과적으로 설계하세요!
