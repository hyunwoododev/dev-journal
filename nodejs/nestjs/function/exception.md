
# NestJS + 예외 필터(Exception Filter)

`예외 필터(Exception Filter)`는 NestJS에서 에러를 처리하고, 일관된 에러 응답을 생성하는 데 사용됩니다.  
컨트롤러, 서비스, 또는 기타 처리 로직에서 발생한 예외를 가로채어 사용자 친화적인 메시지를 반환할 수 있습니다.

---

## 1. 예외 필터란?

NestJS의 예외 필터는 다음과 같은 역할을 수행합니다:
1. **예외 처리**:
   - 발생한 예외를 캐치하여 적절한 HTTP 응답으로 변환.
2. **커스터마이징 가능**:
   - 기본 에러 메시지 대신 사용자 정의 메시지를 반환.
3. **전역 또는 특정 범위에 적용 가능**:
   - 애플리케이션 전역, 특정 컨트롤러, 또는 특정 핸들러에만 적용 가능.

---

## 2. 예외 필터 구현 방법

1. **기본 제공 HTTP 예외 필터**:
   - `HttpException` 클래스를 사용하여 HTTP 상태 코드를 반환.
2. **커스텀 예외 필터**:
   - 사용자 정의 예외 필터를 만들어 고유한 에러 처리 로직 추가.

---

## 3. 예외 필터 구현 예제

### 3.1. 기본 제공 HttpException 사용

#### 컨트롤러에서 예외 발생

```typescript
import { Controller, Get, HttpException, HttpStatus } from '@nestjs/common';

@Controller('items')
export class ItemsController {
  @Get()
  getItems() {
    throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);
  }
}
```

**응답**:
```json
{
  "statusCode": 403,
  "message": "Forbidden"
}
```

---

### 3.2. 커스텀 예외 필터

#### 예외 필터 구현

```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Response } from 'express';

@Catch(HttpException)
export class CustomHttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const status = exception.getStatus();
    const exceptionResponse = exception.getResponse();

    response.status(status).json({
      success: false,
      statusCode: status,
      message: exceptionResponse['message'] || exceptionResponse,
      timestamp: new Date().toISOString(),
    });
  }
}
```

#### 컨트롤러에 적용

```typescript
import { Controller, Get, UseFilters, HttpException, HttpStatus } from '@nestjs/common';
import { CustomHttpExceptionFilter } from './custom-http-exception.filter';

@Controller('items')
@UseFilters(CustomHttpExceptionFilter) // 특정 컨트롤러에 적용
export class ItemsController {
  @Get()
  getItems() {
    throw new HttpException('Item not found', HttpStatus.NOT_FOUND);
  }
}
```

**응답**:
```json
{
  "success": false,
  "statusCode": 404,
  "message": "Item not found",
  "timestamp": "2024-01-01T12:00:00.000Z"
}
```

---

### 3.3. 전역 예외 필터

#### 전역 필터 설정

`main.ts`에서 설정:

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { CustomHttpExceptionFilter } from './custom-http-exception.filter';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // 전역 예외 필터 등록
  app.useGlobalFilters(new CustomHttpExceptionFilter());

  await app.listen(3000);
}
bootstrap();
```

---

### 3.4. 커스텀 예외 클래스

#### 커스텀 예외 정의

```typescript
import { HttpException, HttpStatus } from '@nestjs/common';

export class CustomNotFoundException extends HttpException {
  constructor(message: string) {
    super(message, HttpStatus.NOT_FOUND);
  }
}
```

#### 컨트롤러에서 사용

```typescript
import { Controller, Get } from '@nestjs/common';
import { CustomNotFoundException } from './custom-not-found.exception';

@Controller('products')
export class ProductsController {
  @Get()
  getProducts() {
    throw new CustomNotFoundException('Product not found');
  }
}
```

**응답**:
```json
{
  "statusCode": 404,
  "message": "Product not found"
}
```

---

## 4. 고급 예외 필터 구현

### 요청-응답 로깅 추가

```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class LoggingExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    console.error(`[${request.method}] ${request.url}`, exception.message);

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      error: exception.message,
    });
  }
}
```

---

## 5. 결론

NestJS의 예외 필터는 다음과 같은 장점을 제공합니다:
1. **일관된 에러 처리**:
   - 애플리케이션 전역에서 공통된 에러 처리 로직 적용 가능.
2. **사용자 정의 가능**:
   - 비즈니스 로직에 맞는 커스텀 에러 메시지와 구조 제공.
3. **모듈성**:
   - 특정 컨트롤러 또는 라우트에만 적용 가능.

예외 필터를 활용하여 사용자 친화적이고 안전한 애플리케이션을 설계하세요!
