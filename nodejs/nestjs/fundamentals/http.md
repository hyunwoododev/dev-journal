
# NestJS + HTTP Client 고급 및 확장 사용법

NestJS는 **HttpModule**과 `Axios`를 기반으로 다양한 HTTP 요청 처리 기능을 제공합니다.  
본 문서에서는 고급 요청 처리, 대규모 애플리케이션에서 확장 가능한 HTTP 통신 설계 및 철학을 다룹니다.

---

## 1. HTTP 클라이언트란?

HTTP 클라이언트는 외부 서비스와 통신하기 위한 도구입니다.  
NestJS의 `HttpModule`은 다음과 같은 기능을 제공합니다:
1. **비동기 요청 처리**:
   - `async/await` 기반의 비동기 호출.
2. **요청/응답 인터셉터**:
   - 요청 또는 응답의 데이터 조작 및 로깅.
3. **RxJS 통합**:
   - 스트림 기반 요청 처리 및 제어.

---

## 2. HttpModule 설정 및 기본 사용법

---

### 2.1. 설치 및 설정

```bash
npm install @nestjs/axios axios
```

#### HttpModule 등록

```typescript
import { Module } from '@nestjs/common';
import { HttpModule } from '@nestjs/axios';

@Module({
  imports: [HttpModule],
})
export class AppModule {}
```

---

### 2.2. 기본 HTTP 요청 처리

#### GET 요청 예제

```typescript
import { Injectable } from '@nestjs/common';
import { HttpService } from '@nestjs/axios';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class ApiService {
  constructor(private readonly httpService: HttpService) {}

  async fetchData(): Promise<any> {
    const response = await firstValueFrom(this.httpService.get('https://api.example.com/data'));
    return response.data;
  }
}
```

#### POST 요청 예제

```typescript
async postData(payload: any): Promise<any> {
  const response = await firstValueFrom(
    this.httpService.post('https://api.example.com/create', payload),
  );
  return response.data;
}
```

---

## 3. 고급 HTTP 요청 패턴

---

### 3.1. 요청 구성: Headers 및 Params

#### Query Parameters 및 Headers 사용

```typescript
async fetchWithQuery(): Promise<any> {
  const response = await firstValueFrom(
    this.httpService.get('https://api.example.com/search', {
      params: { keyword: 'nestjs', limit: 5 },
      headers: { Authorization: 'Bearer token' },
    }),
  );
  return response.data;
}
```

---

### 3.2. 인터셉터와 전역 요청 로깅

#### 요청 로깅 인터셉터

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('HTTP Request sent');
    return next.handle().pipe(tap(() => console.log('HTTP Response received')));
  }
}
```

#### 전역 설정

```typescript
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
  ],
})
export class AppModule {}
```

---

### 3.3. 재시도 및 실패 처리

#### RxJS `retry` 연산자 활용

```typescript
import { Injectable } from '@nestjs/common';
import { HttpService } from '@nestjs/axios';
import { firstValueFrom, catchError, retry } from 'rxjs';

@Injectable()
export class ApiService {
  constructor(private readonly httpService: HttpService) {}

  async fetchWithRetry(): Promise<any> {
    return firstValueFrom(
      this.httpService.get('https://api.example.com/data').pipe(
        retry(3), // 3회 재시도
        catchError(error => {
          throw new Error('요청이 3회 실패하였습니다.');
        }),
      ),
    );
  }
}
```

---

### 3.4. 동적 요청 구성

#### 동적 헤더 및 URL

```typescript
async dynamicRequest(url: string, headers: Record<string, string>): Promise<any> {
  const response = await firstValueFrom(
    this.httpService.get(url, {
      headers,
    }),
  );
  return response.data;
}
```

---

## 4. 확장 가능한 HTTP 설계

---

### 4.1. HttpService 확장

#### 커스텀 HTTP 서비스 생성

```typescript
import { Injectable } from '@nestjs/common';
import { HttpService } from '@nestjs/axios';

@Injectable()
export class CustomHttpService {
  constructor(private readonly httpService: HttpService) {}

  async get(url: string, options?: any): Promise<any> {
    return this.httpService.get(url, options).toPromise();
  }

  async post(url: string, data: any, options?: any): Promise<any> {
    return this.httpService.post(url, data, options).toPromise();
  }
}
```

---

### 4.2. 모듈화된 API 설계

#### API별 모듈 구성

```typescript
@Module({
  imports: [HttpModule],
  providers: [ApiService],
  exports: [ApiService],
})
export class ApiModule {}
```

#### 다른 모듈에서 사용

```typescript
@Module({
  imports: [ApiModule],
})
export class UserModule {}
```

---

### 4.3. API Gateway 통합

#### 요청 라우팅

```typescript
import { Controller, Get, Query, Res } from '@nestjs/common';
import { ApiService } from './api.service';

@Controller('gateway')
export class ApiGatewayController {
  constructor(private readonly apiService: ApiService) {}

  @Get('search')
  async routeSearch(@Query('q') query: string, @Res() res): Promise<any> {
    const data = await this.apiService.fetchWithQuery();
    res.send(data);
  }
}
```

---

## 5. 실시간 데이터와 WebSocket 연동

---

### 실시간 데이터 스트리밍

#### HTTP 데이터 WebSocket 전송

```typescript
import { Injectable } from '@nestjs/common';
import { HttpService } from '@nestjs/axios';
import { WebSocketGateway, WebSocketServer } from '@nestjs/websockets';
import { Server } from 'socket.io';
import { firstValueFrom } from 'rxjs';

@Injectable()
@WebSocketGateway()
export class RealtimeService {
  @WebSocketServer()
  server: Server;

  constructor(private readonly httpService: HttpService) {}

  async sendRealtimeData() {
    const response = await firstValueFrom(this.httpService.get('https://api.example.com/data'));
    this.server.emit('data', response.data);
  }
}
```

---

## 6. 철학과 도구

---

### 설계 철학
1. **단일 책임 원칙**:
   - 각 서비스가 하나의 작업만 담당하도록 설계.
2. **모듈화된 구조**:
   - HTTP 요청 처리 로직을 모듈로 분리하여 재사용 가능.
3. **유연성**:
   - 인터셉터와 커스텀 서비스로 확장성 확보.

---

### 주요 도구
1. **Axios**:
   - HTTP 요청 처리의 핵심 라이브러리.
2. **RxJS**:
   - 비동기 흐름 제어.
3. **Postman**:
   - API 테스트 및 디버깅.

---

## 7. 결론

NestJS의 HTTP 클라이언트는 간단한 요청부터 복잡한 비동기 흐름 관리까지 다양한 기능을 제공합니다.  
커스텀 서비스와 인터셉터를 활용하여 확장성과 유지보수성을 겸비한 네트워크 요청 처리를 설계하세요.
