
# NestJS + 의존성 주입(Dependency Injection)

`의존성 주입(Dependency Injection, DI)`는 객체 간의 결합도를 줄이고 코드의 유연성과 재사용성을 높이는 설계 패턴입니다.  
NestJS는 DI를 핵심 아키텍처로 채택하여 모듈화된 애플리케이션 개발을 지원합니다.

---

## 1. 의존성 주입이란?

DI는 클래스의 의존성을 외부에서 주입하여 객체 생성 및 관리를 애플리케이션 컨테이너가 책임지도록 하는 패턴입니다.  
NestJS의 DI 컨테이너는 다음을 지원합니다:
1. **의존성 관리**:
   - 객체를 자동으로 생성하고 관리.
2. **결합도 감소**:
   - 코드 간 결합도를 줄여 테스트와 유지보수성을 향상.
3. **재사용성 증가**:
   - 모듈화된 설계로 의존성을 쉽게 재사용 가능.

---

## 2. NestJS에서 DI 사용 방법

1. **서비스 정의 및 제공**:
   - `@Injectable()` 데코레이터를 사용하여 DI 컨테이너에 등록.
2. **의존성 주입**:
   - 생성자를 통해 다른 클래스에 주입.

---

## 3. DI 구현 예제

### 3.1. 기본적인 DI

#### 서비스 정의

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello, World!';
  }
}
```

#### 컨트롤러에서 주입

```typescript
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

---

### 3.2. 여러 의존성 주입

#### 서비스 정의

```typescript
@Injectable()
export class UserService {
  getUser(): string {
    return 'John Doe';
  }
}

@Injectable()
export class OrderService {
  getOrder(): string {
    return 'Order #1234';
  }
}
```

#### 컨트롤러에서 주입

```typescript
import { Controller, Get } from '@nestjs/common';
import { UserService } from './user.service';
import { OrderService } from './order.service';

@Controller()
export class AppController {
  constructor(
    private readonly userService: UserService,
    private readonly orderService: OrderService,
  ) {}

  @Get('user')
  getUser(): string {
    return this.userService.getUser();
  }

  @Get('order')
  getOrder(): string {
    return this.orderService.getOrder();
  }
}
```

---

### 3.3. 커스텀 프로바이더

NestJS에서는 사용자 정의 로직을 가진 프로바이더를 정의할 수 있습니다.

#### 커스텀 프로바이더 생성

```typescript
export const CustomProvider = {
  provide: 'GREETING',
  useValue: 'Hello from Custom Provider!',
};
```

#### 모듈에서 등록

```typescript
import { Module } from '@nestjs/common';

@Module({
  providers: [CustomProvider],
})
export class AppModule {}
```

#### 컨트롤러에서 사용

```typescript
import { Controller, Get, Inject } from '@nestjs/common';

@Controller()
export class AppController {
  constructor(@Inject('GREETING') private readonly greeting: string) {}

  @Get()
  getGreeting(): string {
    return this.greeting;
  }
}
```

---

### 3.4. 비동기 프로바이더

#### 비동기 초기화 프로바이더

```typescript
export const AsyncProvider = {
  provide: 'ASYNC_VALUE',
  useFactory: async (): Promise<string> => {
    const asyncValue = await new Promise((resolve) =>
      setTimeout(() => resolve('Async Value Loaded'), 1000),
    );
    return asyncValue;
  },
};
```

#### 모듈에서 등록

```typescript
import { Module } from '@nestjs/common';

@Module({
  providers: [AsyncProvider],
})
export class AppModule {}
```

#### 컨트롤러에서 사용

```typescript
import { Controller, Get, Inject } from '@nestjs/common';

@Controller()
export class AppController {
  constructor(@Inject('ASYNC_VALUE') private readonly asyncValue: string) {}

  @Get()
  getAsyncValue(): string {
    return this.asyncValue;
  }
}
```

---

## 4. 스코프(Scope) 관리

NestJS에서 프로바이더의 생명주기는 다음과 같이 관리됩니다:

1. **싱글톤 스코프 (기본값)**:
   - 애플리케이션 전역에서 하나의 인스턴스만 생성.
   - 기본 설정.

2. **트랜지언트 스코프**:
   - 요청 시마다 새로운 인스턴스 생성.

3. **요청 스코프**:
   - 각 요청(Request)마다 새로운 인스턴스 생성.

#### 요청 스코프 사용 예

```typescript
import { Injectable, Scope } from '@nestjs/common';

@Injectable({ scope: Scope.REQUEST })
export class RequestScopedService {
  getRequestScopedData(): string {
    return 'Request Scoped Data';
  }
}
```

---

## 4. 고급 의존성 주입 패턴

---

### 4.1. 팩토리 프로바이더

#### 동적 값 생성

```typescript
import { Module } from '@nestjs/common';

@Module({
  providers: [
    {
      provide: 'DYNAMIC_VALUE',
      useFactory: () => {
        return Math.random(); // 동적으로 생성된 값
      },
    },
  ],
})
export class AppModule {}
```

#### 사용

```typescript
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class DynamicService {
  constructor(@Inject('DYNAMIC_VALUE') private readonly value: number) {}

  getValue(): number {
    return this.value;
  }
}
```

---

### 4.2. Value 프로바이더

#### 정적 값 주입

```typescript
import { Module } from '@nestjs/common';

@Module({
  providers: [
    {
      provide: 'CONFIG',
      useValue: { apiKey: '123456' },
    },
  ],
})
export class AppModule {}
```

---

### 4.3. Class 프로바이더

#### 클래스를 직접 주입

```typescript
import { Injectable, Module } from '@nestjs/common';

@Injectable()
export class RealService {
  doSomething(): string {
    return 'Real implementation';
  }
}

@Module({
  providers: [
    {
      provide: 'SERVICE_INTERFACE',
      useClass: RealService,
    },
  ],
})
export class AppModule {}
```

---

### 4.4. 인터셉터와 의존성 주입

#### 예제: 요청 로깅

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');
    return next.handle().pipe(tap(() => console.log('After...')));
  }
}
```

---

## 5. 도구와 철학

---

### 철학
1. **단일 책임 원칙(Single Responsibility Principle)**:
   - 각 클래스는 하나의 책임만 가져야 함.
2. **유연성**:
   - 서비스와 컨트롤러 간의 강한 결합을 제거.
3. **테스트 가능성**:
   - Mock 서비스를 사용하여 의존성을 쉽게 대체.

---

### 도구
1. **InversifyJS**:
   - TypeScript용 DI 컨테이너.
2. **TypeORM**:
   - NestJS와 완벽히 통합된 ORM, DI 지원.
3. **Mocking Libraries**:
   - 테스트 시 의존성을 쉽게 대체.

---

## 6. 실제 사용 사례: 다중 데이터베이스 연결

### 시나리오
- 여러 데이터베이스에 연결하여 동적으로 사용할 서비스 주입.

#### 다중 데이터베이스 서비스

```typescript
@Module({
  providers: [
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: async () => {
        const connection = await createConnection();
        return connection;
      },
    },
  ],
})
export class DatabaseModule {}
```

---

## 7. 결론

NestJS의 의존성 주입은 애플리케이션 설계에서 필수적인 요소로, 모듈성, 유연성, 테스트 가능성을 극대화합니다.  
고급 패턴과 철학을 활용하여 확장 가능하고 유지보수 가능한 시스템을 설계하세요.
