
# NestJS + 싱글톤(Singleton) 패턴

싱글톤 패턴은 애플리케이션 전역에서 하나의 인스턴스만 생성되도록 보장하는 디자인 패턴입니다.  
NestJS는 DI(Dependency Injection)를 활용하여 싱글톤 패턴을 쉽게 구현할 수 있습니다.

---

## 1. 싱글톤 패턴이란?

싱글톤 패턴은 다음과 같은 상황에서 유용합니다:
- 애플리케이션 전체에서 공통된 상태를 공유하거나 관리할 때.
- 고정된 자원(예: 캐시, 설정값 등)을 전역적으로 사용해야 할 때.
- 객체 생성 비용이 크기 때문에 인스턴스를 재사용하고 싶을 때.

**특징**:
- 특정 클래스의 인스턴스가 단 하나만 생성됨.
- 전역적으로 접근 가능한 인터페이스 제공.

---

## 2. NestJS에서 싱글톤 패턴

NestJS의 DI 시스템은 기본적으로 서비스 인스턴스를 싱글톤으로 관리합니다.  
이를 활용하여 애플리케이션 전역에서 상태를 공유하거나 리소스를 관리할 수 있습니다.

---

## 3. 캐시 서비스 예제

### 3.1. CacheService
애플리케이션 전역에서 공유되는 캐시 서비스를 구현합니다.

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class CacheService {
  private readonly cache: Map<string, any> = new Map();

  set(key: string, value: any): void {
    this.cache.set(key, value);
    console.log(`Cache set: ${key} = ${JSON.stringify(value)}`);
  }

  get(key: string): any {
    const value = this.cache.get(key);
    console.log(`Cache get: ${key} = ${JSON.stringify(value)}`);
    return value;
  }

  clear(): void {
    this.cache.clear();
    console.log('Cache cleared');
  }
}
```

---

### 3.2. Controller에서 CacheService 사용

```typescript
import { Controller, Get, Post, Body, Delete } from '@nestjs/common';
import { CacheService } from './cache.service';

@Controller('cache')
export class CacheController {
  constructor(private readonly cacheService: CacheService) {}

  @Post()
  setCache(@Body() body: { key: string; value: any }) {
    const { key, value } = body;
    this.cacheService.set(key, value);
    return { message: 'Cache set successfully' };
  }

  @Get()
  getCache(@Body() body: { key: string }) {
    const { key } = body;
    const value = this.cacheService.get(key);
    return { key, value };
  }

  @Delete()
  clearCache() {
    this.cacheService.clear();
    return { message: 'Cache cleared successfully' };
  }
}
```

---

### 3.3. Module 설정

```typescript
import { Module } from '@nestjs/common';
import { CacheService } from './cache.service';
import { CacheController } from './cache.controller';

@Module({
  providers: [CacheService], // 싱글톤 서비스로 등록
  controllers: [CacheController],
})
export class CacheModule {}
```

---

## 4. 실행 결과

### 캐시 설정 요청
**POST /cache**
```json
{
  "key": "username",
  "value": "john_doe"
}
```

**응답:**
```json
{
  "message": "Cache set successfully"
}
```

**콘솔 출력:**
```
Cache set: username = "john_doe"
```

---

### 캐시 조회 요청
**GET /cache**
```json
{
  "key": "username"
}
```

**응답:**
```json
{
  "key": "username",
  "value": "john_doe"
}
```

**콘솔 출력:**
```
Cache get: username = "john_doe"
```

---

### 캐시 삭제 요청
**DELETE /cache**

**응답:**
```json
{
  "message": "Cache cleared successfully"
}
```

**콘솔 출력:**
```
Cache cleared
```

---

## 5. 결론

NestJS에서 싱글톤 패턴은 다음과 같은 장점을 제공합니다:
1. **중앙 집중식 상태 관리**: 여러 모듈과 컨트롤러에서 동일한 인스턴스를 공유.
2. **효율적인 리소스 사용**: 고정된 자원을 효율적으로 재사용.
3. **NestJS DI 시스템과의 자연스러운 통합**: 별도의 구현 없이 기본적으로 싱글톤 관리 가능.

싱글톤 패턴은 캐시, 설정 관리, 연결 풀링과 같은 전역 상태 관리에 매우 유용합니다.  
NestJS의 DI 시스템을 활용하여 효율적인 애플리케이션을 설계하세요!
