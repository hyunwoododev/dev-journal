
# NestJS + Caching (캐싱) 통합 및 확장

캐싱(Caching)은 데이터를 임시로 저장하여 애플리케이션 성능을 최적화하고 서버 부하를 줄이는 중요한 기법입니다.  
NestJS는 `@nestjs/cache-manager`를 기본으로 제공하며, Redis와 같은 고성능 캐시 저장소와 쉽게 통합할 수 있습니다.

---

## 1. Caching의 주요 장점

1. **성능 향상**:
   - 데이터베이스 쿼리와 같은 느린 작업을 줄이고 응답 시간을 단축.
2. **서버 부하 감소**:
   - 동일한 데이터 요청에 대해 캐시된 데이터를 반환하여 부하를 줄임.
3. **확장성**:
   - Redis, Memcached와 같은 고성능 저장소와 통합 가능.

---

## 2. NestJS에서 캐싱 구현하기

NestJS는 기본적으로 `@nestjs/cache-manager`를 사용하며, 다양한 캐시 저장소를 지원합니다.

---

## 3. 기본 캐싱 구현

### 3.1. 설치

```bash
npm install @nestjs/cache-manager
```

---

### 3.2. 기본 설정

#### CacheModule 등록

```typescript
import { Module } from '@nestjs/common';
import { CacheModule } from '@nestjs/cache-manager';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [
    CacheModule.register({
      ttl: 10, // 캐시 TTL (초)
      max: 10, // 최대 캐시 항목 수
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

---

### 3.3. 캐싱 서비스 사용

```typescript
import { Injectable } from '@nestjs/common';
import { Cache } from 'cache-manager';

@Injectable()
export class AppService {
  constructor(private readonly cacheManager: Cache) {}

  async getCachedData(key: string): Promise<string> {
    const cachedData = await this.cacheManager.get<string>(key);
    if (cachedData) {
      return `Cache hit: ${cachedData}`;
    }

    const newData = `Data for ${key}`;
    await this.cacheManager.set(key, newData, { ttl: 10 });
    return `Cache miss: ${newData}`;
  }
}
```

---

## 4. Redis를 사용한 캐싱

Redis는 고성능 캐시 및 데이터 저장소로 널리 사용됩니다.

### 4.1. Redis 설치

```bash
npm install cache-manager-redis-store redis
```

---

### 4.2. Redis 통합 설정

#### CacheModule 등록

```typescript
import { Module } from '@nestjs/common';
import { CacheModule } from '@nestjs/cache-manager';
import * as redisStore from 'cache-manager-redis-store';

@Module({
  imports: [
    CacheModule.register({
      store: redisStore,
      host: 'localhost',
      port: 6379,
      ttl: 60, // TTL 설정
    }),
  ],
})
export class AppModule {}
```

---

### 4.3. Redis 캐싱 서비스 구현

```typescript
import { Injectable } from '@nestjs/common';
import { Cache } from 'cache-manager';

@Injectable()
export class RedisCacheService {
  constructor(private readonly cacheManager: Cache) {}

  async getOrSet(key: string, valueFunction: () => Promise<string>): Promise<string> {
    const cachedData = await this.cacheManager.get<string>(key);
    if (cachedData) {
      console.log('Cache hit');
      return cachedData;
    }

    console.log('Cache miss');
    const newValue = await valueFunction();
    await this.cacheManager.set(key, newValue, { ttl: 60 });
    return newValue;
  }
}
```

---

## 5. 고급 캐싱 기능

### 5.1. 조건부 캐싱

```typescript
if (!(await this.cacheManager.get('key'))) {
  await this.cacheManager.set('key', 'value', { ttl: 300 });
}
```

---

### 5.2. 캐시 삭제

#### 특정 키 삭제

```typescript
await this.cacheManager.del('key');
```

#### 전체 캐시 삭제

```typescript
await this.cacheManager.reset();
```

---

### 5.3. 동적 TTL 설정

```typescript
await this.cacheManager.set('key', 'value', { ttl: dynamicTtl });
```

---

## 6. Memcached를 사용하는 캐싱

### 6.1. Memcached 설치

```bash
npm install cache-manager-memcached-store memcached
```

---

### 6.2. Memcached 설정

#### CacheModule 등록

```typescript
import { Module } from '@nestjs/common';
import { CacheModule } from '@nestjs/cache-manager';
import * as memcachedStore from 'cache-manager-memcached-store';

@Module({
  imports: [
    CacheModule.register({
      store: memcachedStore,
      options: {
        hosts: ['127.0.0.1:11211'],
        ttl: 60, // TTL 설정
      },
    }),
  ],
})
export class AppModule {}
```

---

## 7. 실행 결과

### 요청 예제

**GET /cache/sample**

**결과 (첫 번째 요청)**:
```
Cache miss: Data for sample
```

**결과 (두 번째 요청)**:
```
Cache hit: Data for sample
```

---

## 8. 결론

NestJS의 캐싱 기능은 다음과 같은 장점을 제공합니다:
1. **다양한 저장소 지원**:
   - Redis, Memcached, 기본 메모리 등 여러 저장소와 통합 가능.
2. **유연성**:
   - 동적 TTL, 조건부 캐싱 등 다양한 요구사항 충족.
3. **성능 최적화**:
   - 중복된 작업을 줄이고 서버 부하를 줄임.

NestJS의 캐싱 기능을 활용하여 확장성과 성능을 겸비한 애플리케이션을 설계하세요!
