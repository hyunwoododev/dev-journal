
# NestJS + Dynamic Module Creation

NestJS의 동적 모듈(Dynamic Module) 기능은 런타임에 모듈의 동작을 변경하거나 설정을 기반으로 동적으로 모듈을 생성할 수 있도록 합니다.  
대규모 애플리케이션에서 설정 기반 모듈 구성이나 재사용 가능한 모듈을 설계할 때 매우 유용합니다.

---

## 1. Dynamic Module이란?

Dynamic Module은 NestJS에서 다음과 같은 역할을 수행합니다:
1. **런타임 설정**:
   - 애플리케이션 실행 시 모듈 동작을 설정에 따라 변경.
2. **재사용성 증가**:
   - 여러 환경에서 동일한 모듈을 유연하게 사용 가능.
3. **유연한 설계**:
   - 모듈을 동적으로 생성하고 의존성을 런타임에 주입.

---

## 2. Dynamic Module 구현하기

---

### 2.1. 기본 동적 모듈 구현

#### DynamicModule 반환

```typescript
import { Module, DynamicModule } from '@nestjs/common';

@Module({})
export class ConfigurableModule {
  static forRoot(apiKey: string): DynamicModule {
    return {
      module: ConfigurableModule,
      providers: [
        {
          provide: 'API_KEY',
          useValue: apiKey,
        },
      ],
      exports: ['API_KEY'],
    };
  }
}
```

#### 모듈 사용

```typescript
import { Module } from '@nestjs/common';
import { ConfigurableModule } from './configurable.module';

@Module({
  imports: [ConfigurableModule.forRoot('my-api-key')],
})
export class AppModule {}
```

---

### 2.2. 설정 기반 동적 모듈

#### ConfigService 정의

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class ConfigService {
  constructor(private readonly config: Record<string, any>) {}

  get(key: string): any {
    return this.config[key];
  }
}
```

#### 동적 모듈 생성

```typescript
import { Module, DynamicModule } from '@nestjs/common';
import { ConfigService } from './config.service';

@Module({})
export class ConfigModule {
  static forRoot(config: Record<string, any>): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: ConfigService,
          useValue: new ConfigService(config),
        },
      ],
      exports: [ConfigService],
    };
  }
}
```

#### 모듈 사용

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from './config.module';

@Module({
  imports: [ConfigModule.forRoot({ apiKey: 'my-api-key' })],
})
export class AppModule {}
```

---

### 2.3. 비동기 설정 동적 모듈

#### 비동기 설정 로직 구현

```typescript
import { Module, DynamicModule } from '@nestjs/common';
import { ConfigService } from './config.service';

@Module({})
export class AsyncConfigModule {
  static forRootAsync(): DynamicModule {
    return {
      module: AsyncConfigModule,
      providers: [
        {
          provide: ConfigService,
          useFactory: async () => {
            const config = await new Promise((resolve) =>
              setTimeout(() => resolve({ apiKey: 'async-api-key' }), 1000),
            );
            return new ConfigService(config);
          },
        },
      ],
      exports: [ConfigService],
    };
  }
}
```

#### 모듈 사용

```typescript
import { Module } from '@nestjs/common';
import { AsyncConfigModule } from './async-config.module';

@Module({
  imports: [AsyncConfigModule.forRootAsync()],
})
export class AppModule {}
```

---

## 3. 고급 기능

### 3.1. 동적 모듈 의존성 관리

#### 외부 모듈 의존성 주입

```typescript
import { Module, DynamicModule } from '@nestjs/common';

@Module({})
export class ExternalConfigModule {
  static forFeature(options: { apiKey: string }): DynamicModule {
    return {
      module: ExternalConfigModule,
      providers: [
        {
          provide: 'EXTERNAL_API_KEY',
          useValue: options.apiKey,
        },
      ],
      exports: ['EXTERNAL_API_KEY'],
    };
  }
}
```

#### 사용

```typescript
import { Module } from '@nestjs/common';
import { ExternalConfigModule } from './external-config.module';

@Module({
  imports: [ExternalConfigModule.forFeature({ apiKey: 'external-api-key' })],
})
export class AppModule {}
```

---

### 3.2. 환경별 동적 모듈 설정

#### 환경에 따라 다른 설정 로드

```typescript
import { Module, DynamicModule } from '@nestjs/common';

@Module({})
export class EnvironmentConfigModule {
  static forRoot(env: 'dev' | 'prod'): DynamicModule {
    const config =
      env === 'dev'
        ? { dbHost: 'localhost', dbPort: 5432 }
        : { dbHost: 'prod-db-host', dbPort: 5432 };

    return {
      module: EnvironmentConfigModule,
      providers: [
        {
          provide: 'DB_CONFIG',
          useValue: config,
        },
      ],
      exports: ['DB_CONFIG'],
    };
  }
}
```

#### 사용

```typescript
import { Module } from '@nestjs/common';
import { EnvironmentConfigModule } from './environment-config.module';

@Module({
  imports: [EnvironmentConfigModule.forRoot(process.env.NODE_ENV as 'dev' | 'prod')],
})
export class AppModule {}
```

---

## 4. 실행 결과

### 설정 주입

**ConfigService**를 사용:
```typescript
import { Injectable } from '@nestjs/common';
import { ConfigService } from './config.service';

@Injectable()
export class AppService {
  constructor(private readonly configService: ConfigService) {}

  getConfig(key: string): any {
    return this.configService.get(key);
  }
}
```

---

## 5. 결론

NestJS의 동적 모듈은 다음과 같은 장점을 제공합니다:
1. **런타임 유연성**:
   - 실행 시 설정을 기반으로 모듈 동작을 변경 가능.
2. **재사용 가능**:
   - 설정과 비즈니스 로직을 분리하여 모듈화된 설계.
3. **확장성**:
   - 비동기 초기화와 외부 의존성 주입으로 복잡한 요구사항 충족.

NestJS의 Dynamic Module 기능을 활용하여 확장성과 유연성을 겸비한 애플리케이션을 설계하세요!
