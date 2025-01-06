
# NestJS + gRPC 통합

`gRPC`는 Google에서 개발한 고성능 원격 프로시저 호출(Remote Procedure Call) 프레임워크로, NestJS에서 마이크로서비스를 구현하는 데 사용됩니다.  
NestJS는 gRPC를 지원하여 프로토콜 버퍼(Protocol Buffers)를 기반으로 효율적인 서비스 간 통신을 제공합니다.

---

## 1. gRPC란?

`gRPC`는 다음과 같은 특징을 제공합니다:
1. **고성능**:
   - Protocol Buffers를 사용하여 데이터 직렬화 및 네트워크 전송 최적화.
2. **다양한 언어 지원**:
   - 여러 언어에서 클라이언트와 서버를 구현 가능.
3. **양방향 스트리밍**:
   - 요청과 응답 모두에서 스트리밍 지원.

---

## 2. NestJS에서 gRPC 통합하기

NestJS는 `@nestjs/microservices` 모듈을 사용하여 gRPC를 통합합니다.  
서비스 간 통신을 위해 `.proto` 파일을 사용합니다.

---

## 3. gRPC 통합 예제

### 3.1. 설치

```bash
npm install @nestjs/microservices @grpc/grpc-js @grpc/proto-loader
```

---

### 3.2. gRPC 설정

#### `proto` 파일 정의

`proto/hero.proto`:
```proto
syntax = "proto3";

service HeroService {
  rpc FindOne (HeroById) returns (Hero);
}

message HeroById {
  int32 id = 1;
}

message Hero {
  int32 id = 1;
  string name = 2;
}
```

---

#### gRPC 서비스 구현

```typescript
import { Injectable } from '@nestjs/common';
import { Hero, HeroById } from './proto/hero';

@Injectable()
export class HeroService {
  private readonly heroes = [
    { id: 1, name: 'Superman' },
    { id: 2, name: 'Batman' },
  ];

  findOne(data: HeroById): Hero {
    return this.heroes.find(hero => hero.id === data.id);
  }
}
```

---

#### gRPC 서버 설정

```typescript
import { Controller } from '@nestjs/common';
import { GrpcMethod } from '@nestjs/microservices';
import { HeroService } from './hero.service';

@Controller()
export class HeroController {
  constructor(private readonly heroService: HeroService) {}

  @GrpcMethod('HeroService', 'FindOne')
  findOne(data: { id: number }) {
    return this.heroService.findOne(data);
  }
}
```

---

### 3.3. 서버 모듈 설정

```typescript
import { Module } from '@nestjs/common';
import { HeroController } from './hero.controller';
import { HeroService } from './hero.service';

@Module({
  controllers: [HeroController],
  providers: [HeroService],
})
export class HeroModule {}
```

---

### 3.4. gRPC 마이크로서비스 설정

`main.ts`:
```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { join } from 'path';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
    transport: Transport.GRPC,
    options: {
      package: 'hero',
      protoPath: join(__dirname, './proto/hero.proto'),
    },
  });

  await app.listen();
  console.log('gRPC microservice is running');
}
bootstrap();
```

---

### 3.5. 클라이언트 구현

#### Node.js 클라이언트

```javascript
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');
const packageDefinition = protoLoader.loadSync('./proto/hero.proto');
const HeroService = grpc.loadPackageDefinition(packageDefinition).hero.HeroService;

const client = new HeroService('localhost:5000', grpc.credentials.createInsecure());

client.FindOne({ id: 1 }, (err, response) => {
  if (err) throw err;
  console.log('Hero:', response);
});
```

---

## 4. 고급 gRPC 기능

### 4.1. 양방향 스트리밍

#### `proto` 파일 수정

```proto
syntax = "proto3";

service HeroService {
  rpc Chat (stream ChatMessage) returns (stream ChatMessage);
}

message ChatMessage {
  string sender = 1;
  string message = 2;
}
```

#### 스트리밍 구현  

```typescript
@GrpcMethod('HeroService', 'Chat')
chat(stream: any) {
  stream.on('data', (message) => {
    console.log('Received:', message);
    stream.write({ sender: 'Server', message: `Echo: ${message.message}` });
  });

  stream.on('end', () => {
    stream.end();
  });
}
```

---

## 5. 실행 결과

### 요청

**Node.js 클라이언트**
```javascript
client.FindOne({ id: 1 }, (err, response) => {
  console.log('Hero:', response);
});
```

### 서버 로그

```
HeroService.FindOne called with: { id: 1 }
```

### 클라이언트 출력

```
Hero: { id: 1, name: 'Superman' }
```

---

## 6. 결론

NestJS의 gRPC 통합은 다음과 같은 장점을 제공합니다:
1. **고성능 통신**:
   - Protocol Buffers 기반 데이터 직렬화.
2. **언어 간 호환성**:
   - 다양한 언어에서 gRPC 클라이언트/서버 구현 가능.
3. **유연한 통합**:
   - 양방향 스트리밍, Unary 호출 등 다양한 패턴 지원.

gRPC를 사용하여 효율적이고 확장 가능한 마이크로서비스를 설계하세요!
