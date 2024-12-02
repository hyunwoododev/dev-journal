
# WebSocket + NestJS: 상세 가이드

WebSocket은 클라이언트와 서버 간의 **실시간 양방향 통신**을 가능하게 하는 프로토콜입니다.  
NestJS는 WebSocket을 효율적으로 구현할 수 있는 강력한 도구와 라이브러리를 제공합니다.

---

## 1. WebSocket이란?

WebSocket은 다음과 같은 특징을 갖습니다:
1. **양방향 통신**:
   - 서버와 클라이언트가 서로 데이터를 주고받을 수 있습니다.
2. **연결 유지**:
   - 초기 연결 이후 지속적으로 통신 가능.
3. **효율성**:
   - HTTP 요청/응답 모델보다 적은 네트워크 오버헤드.

---

## 2. NestJS에서 WebSocket 사용하기

NestJS는 WebSocket 통신을 위한 다양한 라이브러리를 지원합니다:
1. `socket.io` (기본 지원).
2. `ws` (경량 WebSocket 라이브러리).
3. `@nestjs/websockets`와 `@nestjs/platform-socket.io`.

---

## 3. 주요 WebSocket 라이브러리 비교

| 라이브러리         | 특징                                                        | 사용 사례 |
|--------------------|------------------------------------------------------------|-----------|
| `socket.io`        | **풍부한 기능** (네임스페이스, 룸 등), 브라우저 지원, CORS 설정 가능 | 다중 클라이언트 지원 |
| `ws`               | 경량 WebSocket 라이브러리, 표준 WebSocket API 준수           | 단순한 통신 시 |
| `@nestjs/websockets` | NestJS 전용 WebSocket 추상화 계층 제공                      | NestJS 통합 |

---

## 4. WebSocket 설치 및 기본 설정

### 4.1. 기본 설치

```bash
npm install @nestjs/websockets @nestjs/platform-socket.io socket.io
```

---

### 4.2. WebSocket 기본 설정

#### WebSocket 게이트웨이 구현

```typescript
import {
  WebSocketGateway,
  SubscribeMessage,
  MessageBody,
  ConnectedSocket,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Socket } from 'socket.io';

@WebSocketGateway(3001, { cors: true }) // 포트 및 CORS 설정
export class ChatGateway implements OnGatewayConnection, OnGatewayDisconnect {
  handleConnection(client: Socket) {
    console.log(`Client connected: ${client.id}`);
  }

  handleDisconnect(client: Socket) {
    console.log(`Client disconnected: ${client.id}`);
  }

  @SubscribeMessage('message')
  handleMessage(@MessageBody() message: string, @ConnectedSocket() client: Socket) {
    console.log(`Message received: ${message} from ${client.id}`);
    return { event: 'message', data: `Hello, ${client.id}` };
  }
}
```

---

## 5. 고급 WebSocket 기능

### 5.1. 네임스페이스 및 룸 사용

#### 네임스페이스 생성

```typescript
@WebSocketGateway({ namespace: '/chat' })
export class ChatNamespaceGateway {
  @SubscribeMessage('message')
  handleNamespaceMessage(@MessageBody() message: string) {
    console.log(`Message on /chat: ${message}`);
  }
}
```

#### 룸 사용

```typescript
@WebSocketGateway()
export class ChatRoomGateway {
  @WebSocketServer()
  server: Server;

  joinRoom(client: Socket, room: string) {
    client.join(room);
    this.server.to(room).emit('room-joined', { room });
  }

  sendMessageToRoom(room: string, message: string) {
    this.server.to(room).emit('room-message', { message });
  }
}
```

---

### 5.2. WebSocket과 Redis 통합 (Socket.IO-Redis)

#### Redis 어댑터 설치

```bash
npm install @socket.io/redis-adapter redis
```

#### Redis 어댑터 설정

```typescript
import { WebSocketGateway, WebSocketServer } from '@nestjs/websockets';
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';
import { Server } from 'socket.io';

@WebSocketGateway()
export class RedisChatGateway {
  @WebSocketServer()
  server: Server;

  async onModuleInit() {
    const pubClient = createClient({ url: 'redis://localhost:6379' });
    const subClient = pubClient.duplicate();

    await Promise.all([pubClient.connect(), subClient.connect()]);

    this.server.adapter(createAdapter(pubClient, subClient));
    console.log('Redis adapter connected');
  }
}
```

---

## 6. 클라이언트 예제

### 6.1. JavaScript 클라이언트

```javascript
const socket = io('http://localhost:3001');

socket.on('connect', () => {
  console.log('Connected to server');
  
  socket.emit('message', 'Hello Server!');

  socket.on('message', (data) => {
    console.log('Received:', data);
  });
});
```

---

## 7. WebSocket 기반 인증

#### 토큰 기반 인증 구현

```typescript
import { WebSocketGateway, WebSocketServer, OnGatewayConnection } from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway()
export class AuthGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  handleConnection(client: Socket) {
    const token = client.handshake.auth?.token;
    if (token !== 'valid-token') {
      client.disconnect();
      console.log('Invalid token, client disconnected');
    } else {
      console.log(`Client connected: ${client.id}`);
    }
  }
}
```

---

## 8. 결론

NestJS의 WebSocket 통합은 다음과 같은 장점을 제공합니다:
1. **실시간 양방향 통신**:
   - 서버와 클라이언트 간의 빠르고 안정적인 데이터 전송.
2. **확장 가능성**:
   - 네임스페이스와 룸, Redis 어댑터를 활용한 확장.
3. **유연한 통합**:
   - 다양한 WebSocket 라이브러리와의 통합 지원.

NestJS의 WebSocket 모듈을 활용하여 실시간 애플리케이션을 효율적으로 구축하세요!
