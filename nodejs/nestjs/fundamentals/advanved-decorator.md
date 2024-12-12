
# NestJS + Custom Decorators

NestJS는 TypeScript의 메타프로그래밍 기능을 활용하여 **데코레이터(Decorator)**를 제공합니다.  
개발자는 내장 데코레이터 외에도 **커스텀 데코레이터(Custom Decorators)**를 만들어 비즈니스 로직을 간소화하고 코드를 더욱 읽기 쉽게 설계할 수 있습니다.

---

## 1. 데코레이터(Decorator)란?

데코레이터는 클래스, 메서드, 프로퍼티, 또는 매개변수에 대해 추가적인 메타데이터를 정의하거나 동작을 확장하는 역할을 합니다.

### NestJS에서 제공하는 내장 데코레이터
- `@Controller()`  
- `@Injectable()`  
- `@Get()`  
- `@Body()`

### Custom Decorators의 장점
1. **코드 재사용성**:
   - 반복되는 로직을 데코레이터로 추출하여 재사용 가능.
2. **가독성 향상**:
   - 복잡한 코드를 단순화.
3. **유연한 설계**:
   - 커스텀 로직을 다양한 요소에 적용 가능.

---

## 2. Custom Decorators 구현

---

### 2.1. 메서드 데코레이터

#### 요청 로깅 데코레이터

```typescript
import { SetMetadata } from '@nestjs/common';

export const LogRequest = (message: string) => SetMetadata('logMessage', message);
```

#### 데코레이터 적용

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('logs')
export class LogsController {
  @Get()
  @LogRequest('Fetching logs')
  getLogs() {
    return 'Logs fetched successfully';
  }
}
```

---

### 2.2. 매개변수 데코레이터

#### 사용자 정의 요청 데이터 추출

```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CustomHeader = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.headers[data];
  },
);
```

#### 데코레이터 사용

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('headers')
export class HeadersController {
  @Get()
  getCustomHeader(@CustomHeader('x-custom-header') header: string) {
    return `Custom Header: ${header}`;
  }
}
```

---

### 2.3. 클래스 데코레이터

#### API 버전 관리

```typescript
import { SetMetadata } from '@nestjs/common';

export const ApiVersion = (version: string) => SetMetadata('apiVersion', version);
```

#### 데코레이터 적용

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('users')
@ApiVersion('1.0')
export class UserController {
  @Get()
  getUsers() {
    return 'User list for API v1.0';
  }
}
```

---

## 3. 고급 Custom Decorators

---

### 3.1. 권한 기반 접근 제어

#### Role 데코레이터

```typescript
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
```

#### Guard와 통합

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    if (!roles) return true;

    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return roles.includes(user.role);
  }
}
```

#### 데코레이터 사용

```typescript
import { Controller, Get, UseGuards } from '@nestjs/common';

@Controller('admin')
export class AdminController {
  @Get()
  @UseGuards(RolesGuard)
  @Roles('admin')
  getAdminData() {
    return 'Admin data';
  }
}
```

---

### 3.2. 캐싱 데코레이터

#### Cacheable 데코레이터

```typescript
import { Injectable } from '@nestjs/common';
import { Cache } from 'cache-manager';
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const Cacheable = (key: string, ttl: number) => {
  return (target: any, propertyKey: string, descriptor: PropertyDescriptor) => {
    const originalMethod = descriptor.value;

    descriptor.value = async function (...args: any[]) {
      const cacheManager: Cache = this.cacheManager;
      const cachedData = await cacheManager.get(key);
      if (cachedData) return cachedData;

      const result = await originalMethod.apply(this, args);
      await cacheManager.set(key, result, { ttl });
      return result;
    };
  };
};
```

#### 데코레이터 사용

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class UserService {
  constructor(private readonly cacheManager: Cache) {}

  @Cacheable('userList', 300)
  async getUsers() {
    return ['User1', 'User2'];
  }
}
```

---

## 4. 실행 결과

### 요청 데이터 추출
**GET /headers**

요청 헤더:
```
x-custom-header: my-header-value
```

**응답**:
```
Custom Header: my-header-value
```

---

### 권한 기반 접근 제어
**GET /admin**

- `user.role = 'user'`:
```
403 Forbidden
```

- `user.role = 'admin'`:
```
Admin data
```

---

## 5. 결론

NestJS의 Custom Decorators는 다음과 같은 장점을 제공합니다:
1. **코드의 재사용성 증가**:
   - 중복 로직을 데코레이터로 추출하여 간결하게 유지.
2. **높은 가독성**:
   - 비즈니스 로직을 명확하게 표현.
3. **확장성**:
   - 권한 제어, 캐싱 등 고급 기능을 간편히 구현 가능.

Custom Decorators를 활용하여 더욱 효율적이고 유지보수 가능한 애플리케이션을 설계하세요!
