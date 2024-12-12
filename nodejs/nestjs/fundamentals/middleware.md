
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



# NestJS + Middleware Composition

NestJS의 **미들웨어(Middleware)**는 요청(Request)과 응답(Response) 사이에서 특정 작업을 수행하는 데 사용됩니다.  
NestJS는 유연한 미들웨어 구성 방식을 지원하며, 여러 개의 미들웨어를 조합하여 강력한 요청 처리를 설계할 수 있습니다.

---

## 1. Middleware란?

미들웨어는 다음과 같은 작업을 수행할 수 있습니다:
1. **요청 데이터 처리**:
   - 요청 데이터를 검사하거나 수정.
2. **응답 데이터 처리**:
   - 응답 데이터를 변경하거나 추가 작업 수행.
3. **로깅 및 모니터링**:
   - 요청 및 응답 상태 로깅.
4. **인증 및 권한**:
   - 사용자 인증 및 권한 검사.

---

## 2. NestJS에서 Middleware 구현하기

---

### 2.1. 기본 미들웨어 구현

#### 함수형 미들웨어

```typescript
import { Request, Response, NextFunction } from 'express';

export function LoggerMiddleware(req: Request, res: Response, next: NextFunction) {
  console.log(`Request Method: ${req.method}, URL: ${req.url}`);
  next();
}
```

#### 모듈에 등록

```typescript
import { Module, MiddlewareConsumer, RequestMethod } from '@nestjs/common';
import { LoggerMiddleware } from './logger.middleware';

@Module({})
export class AppModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({ path: '*', method: RequestMethod.ALL });
  }
}
```

---

### 2.2. 클래스 기반 미들웨어

```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class AuthMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const token = req.headers['authorization'];
    if (!token || token !== 'valid-token') {
      return res.status(403).send('Forbidden');
    }
    next();
  }
}
```

#### 모듈에 등록

```typescript
import { Module, MiddlewareConsumer, RequestMethod } from '@nestjs/common';
import { AuthMiddleware } from './auth.middleware';

@Module({})
export class AppModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(AuthMiddleware)
      .forRoutes({ path: 'protected', method: RequestMethod.ALL });
  }
}
```

---

## 3. 고급 미들웨어 구성

---

### 3.1. 미들웨어 체인 구성

여러 미들웨어를 조합하여 체인을 구성할 수 있습니다.

#### 두 개의 미들웨어 정의

```typescript
export function MiddlewareOne(req: Request, res: Response, next: NextFunction) {
  console.log('Middleware One');
  next();
}

export function MiddlewareTwo(req: Request, res: Response, next: NextFunction) {
  console.log('Middleware Two');
  next();
}
```

#### 체인으로 등록

```typescript
@Module({})
export class AppModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(MiddlewareOne, MiddlewareTwo)
      .forRoutes({ path: '*', method: RequestMethod.ALL });
  }
}
```

---

### 3.2. 동적 미들웨어 등록

#### 동적 조건에 따른 등록

```typescript
export function ConditionalMiddleware(req: Request, res: Response, next: NextFunction) {
  if (req.headers['x-custom-header']) {
    console.log('Custom Header Detected');
  }
  next();
}

@Module({})
export class AppModule {
  configure(consumer: MiddlewareConsumer) {
    const condition = process.env.USE_CONDITIONAL === 'true';
    if (condition) {
      consumer.apply(ConditionalMiddleware).forRoutes('*');
    }
  }
}
```

---

### 3.3. 미들웨어별 경로 설정

#### 경로별 미들웨어 등록

```typescript
@Module({})
export class AppModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({ path: 'logs', method: RequestMethod.GET });

    consumer
      .apply(AuthMiddleware)
      .forRoutes({ path: 'protected', method: RequestMethod.ALL });
  }
}
```

---

## 4. 글로벌 미들웨어 설정

글로벌 미들웨어는 애플리케이션 전체에서 요청을 처리합니다.

#### 글로벌 미들웨어 등록

`main.ts`:

```typescript
import { LoggerMiddleware } from './logger.middleware';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(LoggerMiddleware); // 글로벌 미들웨어 등록
  await app.listen(3000);
}
bootstrap();
```

---

## 5. 실행 결과

### 요청 예제

**GET /logs**
- 로그 출력:
```
Middleware One
Middleware Two
Request Method: GET, URL: /logs
```

**GET /protected**
- 인증 실패 시:
```
403 Forbidden
```

---

## 6. 결론

NestJS의 미들웨어는 다음과 같은 장점을 제공합니다:
1. **요청 처리 확장**:
   - 요청/응답 흐름에서 다양한 작업 수행 가능.
2. **유연한 구성**:
   - 글로벌, 체인, 조건부 등 다양한 방식으로 미들웨어 구성.
3. **모듈성**:
   - 특정 경로나 요청 메서드에만 미들웨어를 적용 가능.

NestJS의 미들웨어 기능을 활용하여 효율적이고 확장 가능한 애플리케이션을 설계하세요!
