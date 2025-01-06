
# NestJS + Rate Limiting

**Rate Limiting(속도 제한)**은 서버의 리소스를 보호하고 악의적인 요청이나 과도한 요청으로부터 서버를 방어하기 위해 사용됩니다.  
NestJS는 `@nestjs/throttler` 패키지를 제공하여 쉽게 속도 제한을 구현할 수 있습니다.

---

## 1. Rate Limiting이란?

Rate Limiting은 다음과 같은 이점을 제공합니다:
1. **서버 보호**:
   - 요청 과부하로부터 서버를 보호.
2. **악의적인 요청 차단**:
   - 특정 클라이언트의 과도한 요청을 방지.
3. **공정한 리소스 분배**:
   - 여러 클라이언트 간의 리소스 균등 분배.

---

## 2. NestJS에서 Rate Limiting 구현하기

NestJS는 `@nestjs/throttler` 패키지를 사용하여 쉽게 구현할 수 있습니다.

---

## 3. 구현 예제

### 3.1. 설치

```bash
npm install @nestjs/throttler
```

---

### 3.2. 기본 설정

#### 모듈에 ThrottlerModule 등록

```typescript
import { Module } from '@nestjs/common';
import { ThrottlerModule } from '@nestjs/throttler';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [
    ThrottlerModule.forRoot({
      ttl: 60, // 시간 창(초)
      limit: 10, // 시간 창 내 허용 요청 수
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

---

### 3.3. 컨트롤러에서 속도 제한 적용

#### 글로벌 속도 제한 적용

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('products')
export class ProductsController {
  @Get()
  findAll() {
    return { message: 'This is a rate-limited route' };
  }
}
```

---

### 3.4. 특정 라우트에만 적용

#### ThrottlerGuard 사용

```typescript
import { Controller, Get, UseGuards } from '@nestjs/common';
import { ThrottlerGuard } from '@nestjs/throttler';

@Controller('orders')
export class OrdersController {
  @Get()
  @UseGuards(ThrottlerGuard) // 특정 라우트에만 속도 제한 적용
  getOrder() {
    return { message: 'This route is rate-limited' };
  }
}
```

---

### 3.5. 사용자 정의 속도 제한 설정

#### 커스텀 Guard로 제한값 변경

```typescript
import { Injectable } from '@nestjs/common';
import { ThrottlerGuard } from '@nestjs/throttler';

@Injectable()
export class CustomThrottlerGuard extends ThrottlerGuard {
  protected getTracker(req: Record<string, any>): string {
    // IP 기반 추적
    return req.ip;
  }

  protected getLimit(): number {
    return 5; // 사용자 정의 요청 제한
  }

  protected getTtl(): number {
    return 30; // 사용자 정의 시간 창(초)
  }
}
```

#### 컨트롤러에서 사용

```typescript
import { Controller, Get, UseGuards } from '@nestjs/common';
import { CustomThrottlerGuard } from './custom-throttler.guard';

@Controller('custom')
export class CustomController {
  @Get()
  @UseGuards(CustomThrottlerGuard)
  getCustomRateLimitedRoute() {
    return { message: 'Custom rate-limited route' };
  }
}
```

---

## 4. 고급 기능

### 4.1. Redis를 사용한 Rate Limiting

#### Redis 설치

```bash
npm install cache-manager cache-manager-redis-store
```

#### Redis 기반 설정

```typescript
import { Module } from '@nestjs/common';
import { ThrottlerModule } from '@nestjs/throttler';
import * as redisStore from 'cache-manager-redis-store';

@Module({
  imports: [
    ThrottlerModule.forRootAsync({
      useFactory: () => ({
        ttl: 60,
        limit: 10,
        storage: redisStore({
          host: 'localhost',
          port: 6379,
        }),
      }),
    }),
  ],
})
export class AppModule {}
```

---

### 4.2. 로그 기반 제한

#### 요청과 제한 로그 기록

```typescript
import { Injectable, ExecutionContext } from '@nestjs/common';
import { ThrottlerGuard } from '@nestjs/throttler';

@Injectable()
export class LoggingThrottlerGuard extends ThrottlerGuard {
  protected async handleRequest(
    context: ExecutionContext,
    limit: number,
    ttl: number,
  ): Promise<boolean> {
    const canProceed = await super.handleRequest(context, limit, ttl);
    if (!canProceed) {
      console.log('Rate limit exceeded');
    }
    return canProceed;
  }
}
```

---

## 5. 실행 결과

### 허용된 요청

**GET /products**

```json
{
  "message": "This is a rate-limited route"
}
```

### 제한 초과 시

**GET /products**

```json
{
  "statusCode": 429,
  "message": "Too Many Requests"
}
```

---

## 6. 결론

NestJS의 Rate Limiting은 다음과 같은 장점을 제공합니다:
1. **서버 보호**:
   - 요청 과부하로부터 서버를 보호.
2. **유연성**:
   - 다양한 경로별 제한 설정 및 커스터마이징 가능.
3. **확장성**:
   - Redis와 통합하여 분산 서버 환경에서도 안정적인 속도 제한 가능.

NestJS의 Rate Limiting 기능을 사용하여 안전하고 효율적인 API를 설계하세요!
