
# NestJS + 라이프사이클 이벤트(Lifecycle Events)

NestJS는 애플리케이션과 각 구성 요소의 생명주기(lifecycle)를 관리하고, 개발자가 특정 시점에 작업을 수행할 수 있도록 다양한 라이프사이클 이벤트를 제공합니다.

---

## 1. 라이프사이클 이벤트란?

라이프사이클 이벤트는 NestJS 애플리케이션 또는 개별 프로바이더, 모듈 등이 초기화되거나 종료될 때 호출되는 특수한 메서드입니다.  
이를 통해 다음과 같은 작업을 수행할 수 있습니다:
1. **초기화 작업**:
   - 데이터베이스 연결, 설정값 로드 등.
2. **종료 작업**:
   - 리소스 정리, 연결 종료 등.
3. **모니터링 및 로깅**:
   - 특정 단계에서 상태 확인.

---

## 2. 주요 라이프사이클 메서드

### 2.1. `OnModuleInit`
모듈이 초기화될 때 실행됩니다.

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';

@Injectable()
export class AppService implements OnModuleInit {
  onModuleInit() {
    console.log('AppService module initialized');
  }
}
```

---

### 2.2. `OnApplicationBootstrap`
애플리케이션이 부트스트랩될 때 실행됩니다.

```typescript
import { Injectable, OnApplicationBootstrap } from '@nestjs/common';

@Injectable()
export class AppService implements OnApplicationBootstrap {
  onApplicationBootstrap() {
    console.log('Application has bootstrapped');
  }
}
```

---

### 2.3. `OnModuleDestroy`
모듈이 종료될 때 실행됩니다.

```typescript
import { Injectable, OnModuleDestroy } from '@nestjs/common';

@Injectable()
export class AppService implements OnModuleDestroy {
  onModuleDestroy() {
    console.log('AppService module is being destroyed');
  }
}
```

---

### 2.4. `OnApplicationShutdown`
애플리케이션이 종료될 때 실행됩니다.

```typescript
import { Injectable, OnApplicationShutdown } from '@nestjs/common';

@Injectable()
export class AppService implements OnApplicationShutdown {
  onApplicationShutdown(signal?: string) {
    console.log(`Application is shutting down due to ${signal}`);
  }
}
```

---

### 2.5. `BeforeApplicationShutdown`
애플리케이션 종료 전 호출됩니다.

```typescript
import { Injectable, BeforeApplicationShutdown } from '@nestjs/common';

@Injectable()
export class AppService implements BeforeApplicationShutdown {
  beforeApplicationShutdown(signal?: string) {
    console.log(`Before application shutdown: ${signal}`);
  }
}
```

---

## 3. 모듈 레벨에서의 라이프사이클

### 모듈에서 라이프사이클 메서드 사용

```typescript
import { Module, OnModuleInit, OnModuleDestroy } from '@nestjs/common';

@Module({})
export class AppModule implements OnModuleInit, OnModuleDestroy {
  onModuleInit() {
    console.log('AppModule initialized');
  }

  onModuleDestroy() {
    console.log('AppModule destroyed');
  }
}
```

---

## 4. 애플리케이션 종료 훅

### Graceful Shutdown (정상 종료 처리)

NestJS는 SIGINT, SIGTERM 신호를 처리하여 애플리케이션 종료 시 작업을 수행할 수 있습니다.

#### `main.ts`에서 설정

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Graceful Shutdown 설정
  app.enableShutdownHooks();

  await app.listen(3000);
  console.log('Application is running on http://localhost:3000');
}
bootstrap();
```

#### 서비스에서 종료 처리

```typescript
import { Injectable, OnApplicationShutdown } from '@nestjs/common';

@Injectable()
export class AppService implements OnApplicationShutdown {
  onApplicationShutdown(signal?: string) {
    console.log(`Cleanup on shutdown signal: ${signal}`);
    // 데이터베이스 연결 종료, 캐시 비우기 등
  }
}
```

---

## 5. 실전 예제

### 데이터베이스 연결 관리

#### 데이터베이스 서비스

```typescript
import { Injectable, OnApplicationShutdown, OnModuleInit } from '@nestjs/common';

@Injectable()
export class DatabaseService implements OnModuleInit, OnApplicationShutdown {
  private connection: any;

  async onModuleInit() {
    console.log('Connecting to the database...');
    this.connection = {}; // 데이터베이스 연결 로직
    console.log('Database connected');
  }

  async onApplicationShutdown(signal?: string) {
    console.log(`Closing database connection due to ${signal}`);
    // 연결 종료 로직
    this.connection = null;
    console.log('Database connection closed');
  }
}
```

#### 모듈에 등록

```typescript
import { Module } from '@nestjs/common';
import { DatabaseService } from './database.service';

@Module({
  providers: [DatabaseService],
})
export class AppModule {}
```

---

## 6. 결론

NestJS의 라이프사이클 이벤트는 다음과 같은 장점을 제공합니다:
1. **초기화 및 종료 작업 관리**:
   - 서비스, 모듈, 애플리케이션의 생명주기를 체계적으로 관리 가능.
2. **정상 종료 처리**:
   - 애플리케이션 종료 시 리소스 정리 및 상태 저장.
3. **모니터링 및 로깅**:
   - 특정 단계에서 애플리케이션 상태를 로깅하여 디버깅에 도움.

NestJS의 라이프사이클 메서드를 활용하여 안정적이고 관리 가능한 애플리케이션을 설계하세요!
