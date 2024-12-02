
# NestJS + 미들웨어(Middleware)

`미들웨어(Middleware)`는 요청과 응답 사이에서 추가적인 처리를 수행하는 함수입니다.  
NestJS에서 미들웨어는 애플리케이션의 요청 흐름을 제어하고, 공통 로직을 재사용할 수 있도록 도와줍니다.

---

## 1. 미들웨어란?

NestJS의 미들웨어는 Express 또는 Fastify와 유사하게 작동하며, 다음과 같은 역할을 수행할 수 있습니다:
1. **요청/응답 로깅**.
2. **요청 본문 또는 헤더 수정**.
3. **인증 및 권한 검사**.
4. **CORS 설정**.
5. **특정 경로에 대한 커스텀 로직 추가**.

---

## 2. 미들웨어 구현 방법

NestJS에서 미들웨어는 다음 세 가지 방식으로 구현할 수 있습니다:
1. **함수 기반 미들웨어**.
2. **클래스 기반 미들웨어**.
3. **전역 미들웨어**.

---

## 3. 미들웨어 구현 예제

### 3.1. 함수 기반 미들웨어

```typescript
import { Request, Response, NextFunction } from 'express';

export function loggerMiddleware(req: Request, res: Response, next: NextFunction) {
  console.log(`Request... ${req.method} ${req.url}`);
  next(); // 다음 미들웨어 또는 컨트롤러로 이동
}
```

**모듈에서 적용:**

```typescript
import { Module, MiddlewareConsumer, RequestMethod } from '@nestjs/common';
import { loggerMiddleware } from './logger.middleware';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(loggerMiddleware) // 미들웨어 등록
      .forRoutes({ path: '*', method: RequestMethod.ALL }); // 모든 경로에 적용
  }
}
```

---

### 3.2. 클래스 기반 미들웨어

```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`Request... ${req.method} ${req.url}`);
    next();
  }
}
```

**모듈에서 적용:**

```typescript
import { Module, MiddlewareConsumer, RequestMethod } from '@nestjs/common';
import { LoggerMiddleware } from './logger.middleware';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware) // 클래스 기반 미들웨어 등록
      .forRoutes({ path: '*', method: RequestMethod.ALL }); // 모든 경로에 적용
  }
}
```

---

### 3.3. 특정 경로에만 적용

```typescript
consumer
  .apply(LoggerMiddleware)
  .forRoutes({ path: 'users', method: RequestMethod.GET }); // 특정 경로에만 적용
```

---

### 3.4. 전역 미들웨어

NestJS에서 전역 미들웨어는 애플리케이션의 모든 요청에 적용됩니다.

#### 전역 미들웨어 설정
`main.ts`에서 설정합니다.

```typescript
import { LoggerMiddleware } from './logger.middleware';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // 전역 미들웨어 등록
  app.use(LoggerMiddleware);

  await app.listen(3000);
  console.log('Application is running on http://localhost:3000');
}
bootstrap();
```

---

## 4. 미들웨어에서 추가적인 작업

### 요청 및 응답 로깅

```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`Request Method: ${req.method}, URL: ${req.url}`);
    res.on('finish', () => {
      console.log(`Response Status: ${res.statusCode}`);
    });
    next();
  }
}
```

---

### 인증 미들웨어

```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class AuthMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const token = req.headers['authorization'];
    if (!token) {
      return res.status(403).send('Forbidden');
    }
    // 토큰 검증 로직 추가 가능
    next();
  }
}
```

**모듈에서 적용:**

```typescript
consumer
  .apply(AuthMiddleware)
  .forRoutes({ path: 'protected/*', method: RequestMethod.ALL });
```

---

## 5. 결론

NestJS의 미들웨어는 다음과 같은 장점을 제공합니다:
1. **요청 흐름 제어**: 요청을 처리하기 전에 공통 로직을 실행 가능.
2. **유연한 적용**: 특정 경로나 HTTP 메서드에만 적용 가능.
3. **확장 가능성**: 사용자 정의 미들웨어를 만들어 애플리케이션 요구사항에 맞춤.

미들웨어는 요청과 응답 사이에서 공통 작업을 처리하기 위한 강력한 도구입니다. NestJS의 미들웨어를 활용하여 효율적인 애플리케이션을 설계하세요!
