
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

## 5. 결론

NestJS의 DI는 다음과 같은 이점을 제공합니다:
1. **객체 생성 및 관리의 자동화**:
   - 객체 생성 로직을 간소화하고, 의존성을 자동으로 관리.
2. **유연한 프로바이더 설정**:
   - 싱글톤, 요청 기반, 또는 트랜지언트 스코프 지원.
3. **모듈화된 설계**:
   - 코드 재사용성을 극대화하고, 유지보수성을 향상.

DI는 NestJS의 핵심 설계 철학 중 하나로, 대규모 애플리케이션을 체계적으로 관리하는 데 필수적인 도구입니다.  
NestJS의 DI 컨테이너를 활용하여 강력하고 유연한 애플리케이션을 구축하세요!
