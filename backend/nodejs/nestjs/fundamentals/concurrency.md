
# NestJS + 동시성 관리

동시성(Concurrency)은 애플리케이션의 확장성과 효율성을 설계하는 데 필수적인 요소입니다.  
NestJS는 Node.js의 비동기 특성을 기반으로 동시성을 관리할 수 있는 강력한 도구를 제공합니다.

---

## 1. 동시성이란?

동시성은 시스템이 여러 작업을 동시에 처리할 수 있는 능력을 의미합니다.  
웹 애플리케이션에서는 주로 다수의 클라이언트 요청을 처리하는 데 사용됩니다.

### 주요 동시성 개념:
1. **비동기 프로그래밍**:
   - `async/await`, Promise, 콜백을 사용하여 블로킹 없는 작업 처리.
2. **이벤트 루프(Event Loop)**:
   - Node.js는 단일 스레드 이벤트 루프를 사용하여 I/O 작업을 효율적으로 처리.
3. **스레드 풀(Thread Pool)**:
   - CPU 집중 작업을 처리하기 위해 `libuv`를 통해 스레드 풀 사용.

---

## 2. NestJS의 동시성 관리

NestJS는 Node.js의 비동기 특성을 활용하여 동시성을 효과적으로 처리할 수 있는 다양한 기능을 제공합니다.

---

### 2.1. 서비스에서 비동기 작업 처리

#### 비동기 서비스 메서드

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class DataService {
  async fetchData(): Promise<string> {
    return new Promise(resolve =>
      setTimeout(() => resolve('데이터 가져오기 완료'), 1000),
    );
  }
}
```

#### 컨트롤러에서 비동기 작업 처리

```typescript
import { Controller, Get } from '@nestjs/common';
import { DataService } from './data.service';

@Controller('data')
export class DataController {
  constructor(private readonly dataService: DataService) {}

  @Get()
  async getData() {
    return this.dataService.fetchData();
  }
}
```

---

## 3. NestJS의 고급 동시성 패턴

---

### 3.1. 병렬 처리

`Promise.all`을 사용하여 여러 작업을 동시에 처리.

#### 병렬 작업 서비스 구현

```typescript
@Injectable()
export class BatchService {
  async fetchMultiple(): Promise<string[]> {
    const tasks = [
      this.fetchData('작업 1'),
      this.fetchData('작업 2'),
      this.fetchData('작업 3'),
    ];
    return Promise.all(tasks);
  }

  private async fetchData(task: string): Promise<string> {
    return new Promise(resolve =>
      setTimeout(() => resolve(`${task} 완료`), 1000),
    );
  }
}
```

---

### 3.2. 작업 제한 및 속도 조절

#### 세마포어 패턴을 사용한 동시 작업 제한

```typescript
@Injectable()
export class ThrottledService {
  private semaphore = 2; // 동시에 2개의 작업만 실행

  async processTasks(tasks: (() => Promise<void>)[]) {
    const queue = [...tasks];

    const runTask = async () => {
      if (queue.length === 0) return;

      const task = queue.shift();
      await task();

      await runTask(); // 다음 작업 실행
    };

    const workers = Array.from({ length: this.semaphore }, runTask);
    await Promise.all(workers);
  }
}
```

---

### 3.3. 작업 큐(Task Queue)

작업 큐는 장기 실행 또는 백그라운드 작업을 관리하는 데 유용합니다.

#### Bull 작업 큐 통합

```bash
npm install @nestjs/bull bull
```

```typescript
import { Module } from '@nestjs/common';
import { BullModule } from '@nestjs/bull';
import { QueueService } from './queue.service';

@Module({
  imports: [
    BullModule.forRoot({
      redis: {
        host: 'localhost',
        port: 6379,
      },
    }),
    BullModule.registerQueue({
      name: 'task-queue',
    }),
  ],
  providers: [QueueService],
})
export class QueueModule {}
```

---

### 3.4. 상태 관리와 뮤텍스(Mutex)

#### 뮤텍스를 사용한 작업 충돌 방지

```typescript
@Injectable()
export class MutexService {
  private isLocked = false;

  async executeTask(task: () => Promise<void>) {
    while (this.isLocked) {
      await new Promise(resolve => setTimeout(resolve, 100)); // 대기
    }

    this.isLocked = true;
    try {
      await task();
    } finally {
      this.isLocked = false;
    }
  }
}
```

---

## 4. 철학과 도구

---

### 철학
1. **상태 유지 최소화**:
   - 가능한 한 상태를 분리하여 동시성 문제를 줄임.
2. **불변성 유지**:
   - 공유 데이터를 변경하지 않고 작업 수행.
3. **적절한 도구 활용**:
   - 작업 큐, 캐시 등을 활용해 확장성과 안정성을 강화.

---

### 도구
1. **Bull**:
   - 작업 큐 관리 라이브러리.
2. **RxJS**:
   - 복잡한 비동기 흐름 처리.
3. **Redis**:
   - 작업 큐와 세마포어를 지원하는 고속 저장소.

---

## 5. 실제 사용 사례: 파일 업로드 병렬 처리

### 시나리오
- 여러 파일을 동시에 업로드하되, 동시 작업 수를 제한.

#### 구현

```typescript
@Injectable()
export class FileProcessingService {
  private semaphore = 5;

  async processFiles(files: string[]): Promise<void> {
    const tasks = files.map(file => () => this.processFile(file));
    await this.processWithSemaphore(tasks);
  }

  private async processFile(file: string): Promise<void> {
    console.log(`파일 처리 중: ${file}`);
    await new Promise(resolve => setTimeout(resolve, 1000));
    console.log(`파일 처리 완료: ${file}`);
  }

  private async processWithSemaphore(tasks: (() => Promise<void>)[]) {
    const queue = [...tasks];

    const runTask = async () => {
      if (queue.length === 0) return;

      const task = queue.shift();
      await task();

      await runTask(); // 다음 작업 처리
    };

    const workers = Array.from({ length: this.semaphore }, runTask);
    await Promise.all(workers);
  }
}
```

---

## 6. 결론

NestJS의 동시성 관리는 비동기 프로그래밍, 작업 큐, 세마포어 및 뮤텍스와 같은 고급 개념을 활용하여 확장 가능한 애플리케이션을 구축할 수 있도록 돕습니다.  
이러한 기법들을 적절히 활용하면 효율적이고 안정적인 애플리케이션 설계가 가능합니다.
