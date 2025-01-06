
# NestJS + Task Scheduling

NestJS는 반복 작업, 예약 작업, 또는 특정 시점에 실행되어야 하는 작업을 효율적으로 처리하기 위한 **Task Scheduling** 기능을 제공합니다.  
이를 위해 주로 `@nestjs/schedule` 패키지가 사용됩니다.

---

## 1. Task Scheduling이란?

**Task Scheduling(작업 예약)**은 다음과 같은 작업을 처리할 때 유용합니다:
1. **주기적인 작업**:
   - 특정 간격으로 데이터 처리.
2. **예약 작업**:
   - 정해진 시간에 실행.
3. **일회성 작업**:
   - 특정 조건이 충족될 때 실행.

---

## 2. NestJS에서 Task Scheduling 구현하기

NestJS는 `@nestjs/schedule` 패키지를 제공하여 크론 작업(Cron Jobs)과 반복 작업(Interval/Timeout)을 쉽게 구현할 수 있습니다.

---

## 3. Task Scheduling 구현 예제

### 3.1. 설치

```bash
npm install @nestjs/schedule
```

---

### 3.2. 기본 설정

#### ScheduleModule 등록

```typescript
import { Module } from '@nestjs/common';
import { ScheduleModule } from '@nestjs/schedule';
import { AppService } from './app.service';

@Module({
  imports: [ScheduleModule.forRoot()],
  providers: [AppService],
})
export class AppModule {}
```

---

### 3.3. 크론 작업(Cron Job)

#### 크론 작업 구현

```typescript
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class AppService {
  @Cron(CronExpression.EVERY_10_SECONDS)
  handleCron() {
    console.log('Cron job executed at', new Date().toISOString());
  }
}
```

**결과**:
- 매 10초마다 작업 실행.

---

### 3.4. 반복 작업(Interval)

#### Interval 작업 구현

```typescript
import { Injectable } from '@nestjs/common';
import { Interval } from '@nestjs/schedule';

@Injectable()
export class AppService {
  @Interval(5000)
  handleInterval() {
    console.log('Interval job executed every 5 seconds');
  }
}
```

**결과**:
- 5초마다 작업 실행.

---

### 3.5. 시간 초과 작업(Timeout)

#### Timeout 작업 구현

```typescript
import { Injectable } from '@nestjs/common';
import { Timeout } from '@nestjs/schedule';

@Injectable()
export class AppService {
  @Timeout(10000)
  handleTimeout() {
    console.log('Timeout job executed after 10 seconds');
  }
}
```

**결과**:
- 애플리케이션 시작 후 10초 뒤에 한 번만 실행.

---

### 3.6. 동적 작업 추가

#### 동적 작업 등록

```typescript
import { Injectable } from '@nestjs/common';
import { SchedulerRegistry } from '@nestjs/schedule';
import { CronJob } from 'cron';

@Injectable()
export class AppService {
  constructor(private schedulerRegistry: SchedulerRegistry) {}

  addDynamicCronJob(name: string, schedule: string) {
    const job = new CronJob(schedule, () => {
      console.log(`Dynamic cron job '${name}' executed at`, new Date().toISOString());
    });

    this.schedulerRegistry.addCronJob(name, job);
    job.start();
  }
}
```

#### 작업 추가 엔드포인트

```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { AppService } from './app.service';

@Controller('tasks')
export class TaskController {
  constructor(private readonly appService: AppService) {}

  @Post('add-cron')
  addCronJob(@Body() body: { name: string; schedule: string }) {
    this.appService.addDynamicCronJob(body.name, body.schedule);
    return { message: 'Cron job added' };
  }
}
```

---

## 4. 고급 기능

### 4.1. 작업 삭제

#### 작업 삭제 구현

```typescript
import { Injectable } from '@nestjs/common';
import { SchedulerRegistry } from '@nestjs/schedule';

@Injectable()
export class AppService {
  constructor(private schedulerRegistry: SchedulerRegistry) {}

  removeCronJob(name: string) {
    this.schedulerRegistry.deleteCronJob(name);
    console.log(`Cron job '${name}' deleted`);
  }
}
```

---

### 4.2. 작업 목록 조회

```typescript
import { Injectable } from '@nestjs/common';
import { SchedulerRegistry } from '@nestjs/schedule';

@Injectable()
export class AppService {
  constructor(private schedulerRegistry: SchedulerRegistry) {}

  listCronJobs() {
    const jobs = this.schedulerRegistry.getCronJobs();
    jobs.forEach((job, name) => {
      console.log(`Job '${name}':`, job.nextDates().toISOString());
    });
  }
}
```

---

## 5. 실행 결과

### 크론 작업

- 매 10초마다 실행 로그:
```
Cron job executed at 2024-01-01T12:00:00.000Z
```

### 동적 작업 추가

**POST /tasks/add-cron**
```json
{
  "name": "dynamicJob",
  "schedule": "*/5 * * * * *"
}
```

**결과**:
```
Dynamic cron job 'dynamicJob' executed at 2024-01-01T12:00:05.000Z
```

---

## 6. 결론

NestJS의 Task Scheduling은 다음과 같은 장점을 제공합니다:
1. **효율적인 예약 작업 관리**:
   - 주기적인 작업, 예약 작업 등을 손쉽게 구현 가능.
2. **유연성**:
   - 동적으로 작업 추가 및 제거 가능.
3. **통합성**:
   - NestJS 모듈과 완벽히 통합.

NestJS의 Task Scheduling 기능을 활용하여 복잡한 예약 작업을 간편하게 관리하세요!
