
# NestJS + 인터셉터(Interceptor)

`인터셉터(Interceptor)`는 요청(Request)과 응답(Response)을 가로채어 추가 작업을 수행할 수 있는 NestJS의 강력한 기능입니다.  
컨트롤러 전후에 동작하며, 로깅, 데이터 변환, 캐싱, 성능 모니터링 등 다양한 작업에 유용합니다.

---

## 1. 인터셉터란?

NestJS의 인터셉터는 다음과 같은 역할을 수행할 수 있습니다:
1. **요청 및 응답 변환**:
   - 요청 데이터를 가공하거나, 응답 데이터를 변환합니다.
2. **로깅 및 모니터링**:
   - 컨트롤러 실행 전후에 실행 시간을 측정하거나 로그를 남깁니다.
3. **캐싱**:
   - 동일한 요청에 대해 캐싱된 데이터를 반환합니다.

---

## 2. 인터셉터 구현 방법

NestJS에서는 다음과 같은 방식으로 인터셉터를 구현할 수 있습니다:
1. **클래스 기반 인터셉터**.
2. **전역 인터셉터**.

---

## 3. 인터셉터 구현 예제

### 3.1. 클래스 기반 인터셉터

#### 간단한 로깅 인터셉터

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
    console.log('Before controller execution');
    const now = Date.now();

    return next.handle().pipe(
      tap(() =>
        console.log(`After controller execution... ${Date.now() - now}ms`),
      ),
    );
  }
}
```

**컨트롤러에 적용:**

```typescript
import { Controller, Get, UseInterceptors } from '@nestjs/common';
import { LoggingInterceptor } from './logging.interceptor';

@Controller('example')
@UseInterceptors(LoggingInterceptor) // 특정 컨트롤러에만 적용
export class ExampleController {
  @Get()
  getExample() {
    return { message: 'Hello, Interceptor!' };
  }
}
```

**출력**:
```
Before controller execution
After controller execution... 5ms
```

---

### 3.2. 응답 데이터 변환 인터셉터

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map((data) => ({
        success: true,
        data,
      })),
    );
  }
}
```

**컨트롤러에 적용:**

```typescript
import { Controller, Get, UseInterceptors } from '@nestjs/common';
import { TransformInterceptor } from './transform.interceptor';

@Controller('example')
@UseInterceptors(TransformInterceptor)
export class ExampleController {
  @Get()
  getExample() {
    return { message: 'Hello, Interceptor!' };
  }
}
```

**응답**:
```json
{
  "success": true,
  "data": {
    "message": "Hello, Interceptor!"
  }
}
```

---

### 3.3. 전역 인터셉터

`main.ts`에서 전역으로 인터셉터를 설정할 수 있습니다.

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { LoggingInterceptor } from './logging.interceptor';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // 전역 인터셉터 등록
  app.useGlobalInterceptors(new LoggingInterceptor());

  await app.listen(3000);
  console.log('Application is running on http://localhost:3000');
}
bootstrap();
```

---

### 3.4. 캐싱 인터셉터

#### 간단한 캐싱 인터셉터

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable, of } from 'rxjs';

const cache = new Map<string, any>();

@Injectable()
export class CachingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const key = `${request.method}:${request.url}`;

    if (cache.has(key)) {
      console.log(`Cache hit for ${key}`);
      return of(cache.get(key)); // 캐싱된 데이터 반환
    }

    return next.handle().pipe(
      tap((data) => {
        console.log(`Caching response for ${key}`);
        cache.set(key, data); // 응답 캐싱
      }),
    );
  }
}
```

---

## 4. 결론

NestJS의 인터셉터는 다음과 같은 장점을 제공합니다:
1. **요청 및 응답 흐름 제어**:
   - 컨트롤러 실행 전후에 작업을 수행할 수 있습니다.
2. **재사용성**:
   - 특정 컨트롤러 또는 전역적으로 적용 가능.
3. **다양한 활용**:
   - 로깅, 데이터 변환, 캐싱, 인증 등 다양한 작업에 활용 가능.

NestJS의 인터셉터를 활용하여 더 유연하고 확장 가능한 애플리케이션을 설계하세요!
