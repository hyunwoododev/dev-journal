
# NestJS + 가드(Guard)

`가드(Guard)`는 요청이 특정 조건을 충족하는지 확인하여 처리 흐름을 제어하는 데 사용됩니다.  
가드는 주로 인증 및 권한 부여 로직을 처리하며, 요청이 컨트롤러 또는 라우트 핸들러에 도달하기 전에 실행됩니다.

---

## 1. 가드란?

NestJS에서 가드는 다음과 같은 역할을 수행합니다:
1. **요청 인증**:
   - 요청 헤더 또는 쿠키에 포함된 토큰을 검증하여 인증된 사용자만 접근 가능하게 합니다.
2. **권한 부여**:
   - 특정 역할(Role)을 가진 사용자만 특정 경로에 접근할 수 있도록 제한합니다.

---

## 2. 가드 구현 방법

NestJS에서는 다음과 같은 방식으로 가드를 구현할 수 있습니다:
1. **클래스 기반 가드**.
2. **전역 가드**.
3. **특정 경로에만 적용하는 가드**.

---

## 3. 가드 구현 예제

### 3.1. 기본 인증 가드

#### 가드 구현

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = request.headers['authorization'];

    if (!token) {
      console.log('No token provided');
      return false; // 요청 차단
    }

    // 토큰 검증 로직
    return token === 'valid-token';
  }
}
```

#### 컨트롤러에 적용

```typescript
import { Controller, Get, UseGuards } from '@nestjs/common';
import { AuthGuard } from './auth.guard';

@Controller('protected')
export class ProtectedController {
  @Get()
  @UseGuards(AuthGuard) // 가드 적용
  getProtectedData() {
    return { message: 'You have access to protected data' };
  }
}
```

---

### 3.2. 역할(Role) 기반 가드

#### 역할(Role) 가드 구현

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private readonly requiredRoles: string[]) {}

  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const user = request.user; // 사용자 정보는 미들웨어나 인증 가드에서 설정한다고 가정

    if (!user || !user.roles) {
      return false;
    }

    return this.requiredRoles.every((role) => user.roles.includes(role));
  }
}
```

#### 데코레이터로 역할 설정

```typescript
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
```

#### 컨트롤러에 적용

```typescript
import { Controller, Get, UseGuards } from '@nestjs/common';
import { RolesGuard } from './roles.guard';
import { Roles } from './roles.decorator';

@Controller('admin')
@UseGuards(new RolesGuard(['admin'])) // 관리자만 접근 가능
export class AdminController {
  @Get()
  @Roles('admin')
  getAdminData() {
    return { message: 'Admin access granted' };
  }
}
```

---

### 3.3. 전역 가드

#### 전역 가드 설정

`main.ts`에서 설정:

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { AuthGuard } from './auth.guard';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // 전역 가드 설정
  app.useGlobalGuards(new AuthGuard());

  await app.listen(3000);
  console.log('Application is running on http://localhost:3000');
}
bootstrap();
```

---

## 4. 커스텀 인증 및 권한 부여 로직

### JWT 기반 인증 가드

#### JWT 가드 구현

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import * as jwt from 'jsonwebtoken';

@Injectable()
export class JwtAuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const token = request.headers['authorization'];

    if (!token) {
      return false;
    }

    try {
      const decoded = jwt.verify(token, 'secret-key'); // JWT 검증
      request.user = decoded; // 사용자 정보를 요청 객체에 저장
      return true;
    } catch (error) {
      return false;
    }
  }
}
```

#### 컨트롤러에 적용

```typescript
import { Controller, Get, UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from './jwt-auth.guard';

@Controller('secure')
@UseGuards(JwtAuthGuard) // JWT 인증 가드 적용
export class SecureController {
  @Get()
  getSecureData() {
    return { message: 'You have access to secure data' };
  }
}
```

---

## 5. 결론

NestJS의 가드는 다음과 같은 장점을 제공합니다:
1. **보안 강화**:
   - 요청 인증 및 권한 부여를 중앙에서 관리.
2. **유연성**:
   - 특정 경로, 컨트롤러, 또는 애플리케이션 전역에 적용 가능.
3. **확장 가능성**:
   - 커스텀 인증 로직, 역할 관리 등 다양한 요구사항에 맞게 구현 가능.

가드는 NestJS 애플리케이션의 보안을 강화하고 요청 흐름을 제어하는 데 중요한 역할을 합니다.  
효율적인 인증 및 권한 부여를 위해 가드를 활용하세요!
