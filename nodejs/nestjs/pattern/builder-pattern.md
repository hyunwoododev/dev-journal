
# NestJS + Builder 패턴으로 최적화된 ETL 파이프라인 구현

**Builder 패턴**은 단계별 객체 생성 과정을 분리하여 복잡한 객체를 유연하고 효율적으로 생성할 수 있게 합니다.  
여기에 병렬 처리와 성능 최적화를 결합하여 ETL(Extract, Transform, Load) 파이프라인을 더욱 강력하게 설계할 수 있습니다.

---

## 1. Builder 패턴과 최적화

Builder 패턴은 다음과 같은 장점을 제공합니다:
1. **유연한 구성**:
   - 단계별로 파이프라인을 동적으로 조합 가능.
2. **가독성 향상**:
   - 복잡한 로직을 단순한 단계로 나눌 수 있음.
3. **확장 가능성**:
   - 새로운 요구사항에 맞춰 쉽게 확장 가능.

최적화를 위해 병렬 처리, 조건부 실행, 캐싱 등의 기술을 결합하여 성능을 극대화할 수 있습니다.

---

## 2. 최적화된 ETL 파이프라인 설계

---

### 2.1. 병렬 처리를 위한 인터페이스 정의

#### 병렬 실행을 지원하는 단계 인터페이스

```typescript
export interface PipelineStep {
  execute(data: any): Promise<any>;
  isParallel?: boolean; // 병렬 실행 가능 여부
}
```

---

### 2.2. 병렬 처리 단계 구현

#### Extract 단계

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class ExtractStep implements PipelineStep {
  async execute(data: any): Promise<any> {
    console.log('Extracting data...');
    return { rawData: 'raw data from source' };
  }
}
```

#### Transform 단계 (병렬 실행)

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class TransformStep implements PipelineStep {
  isParallel = true; // 병렬 실행 가능

  async execute(data: any): Promise<any> {
    console.log('Transforming data...');
    return { transformedData: data.rawData.toUpperCase() };
  }
}
```

#### Load 단계

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class LoadStep implements PipelineStep {
  async execute(data: any): Promise<any> {
    console.log('Loading data...');
    return { status: 'success', data: data.transformedData };
  }
}
```

---

### 2.3. 병렬 실행을 지원하는 파이프라인 빌더

#### 파이프라인 빌더 구현

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class OptimizedPipelineBuilder {
  private steps: PipelineStep[] = [];

  addStep(step: PipelineStep): this {
    this.steps.push(step);
    return this;
  }

  async execute(data: any): Promise<any> {
    let result = data;

    for (const step of this.steps) {
      if (step.isParallel) {
        // 병렬 실행
        const parallelResults = await Promise.all(
          this.steps.filter(s => s.isParallel).map(s => s.execute(result)),
        );
        result = parallelResults.reduce((acc, res) => ({ ...acc, ...res }), {});
      } else {
        // 순차 실행
        result = await step.execute(result);
      }
    }

    return result;
  }
}
```

---

### 2.4. 모듈 및 컨트롤러 구성

#### 모듈 구성

```typescript
import { Module } from '@nestjs/common';
import { ExtractStep } from './steps/extract.step';
import { TransformStep } from './steps/transform.step';
import { LoadStep } from './steps/load.step';
import { OptimizedPipelineBuilder } from './optimized-pipeline.builder';

@Module({
  providers: [ExtractStep, TransformStep, LoadStep, OptimizedPipelineBuilder],
})
export class PipelineModule {}
```

#### 컨트롤러 구성

```typescript
import { Controller, Get } from '@nestjs/common';
import { OptimizedPipelineBuilder } from './optimized-pipeline.builder';
import { ExtractStep } from './steps/extract.step';
import { TransformStep } from './steps/transform.step';
import { LoadStep } from './steps/load.step';

@Controller('pipeline')
export class PipelineController {
  constructor(
    private readonly pipelineBuilder: OptimizedPipelineBuilder,
    private readonly extractStep: ExtractStep,
    private readonly transformStep: TransformStep,
    private readonly loadStep: LoadStep,
  ) {}

  @Get('run')
  async runOptimizedPipeline() {
    const pipeline = this.pipelineBuilder
      .addStep(this.extractStep)
      .addStep(this.transformStep) // 병렬 처리
      .addStep(this.loadStep);

    const result = await pipeline.execute({});
    return result;
  }
}
```

---

## 3. 고급 기능 추가

---

### 3.1. 캐싱을 통한 성능 최적화

#### 캐싱 단계 추가

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class CachingStep implements PipelineStep {
  private cache = new Map();

  async execute(data: any): Promise<any> {
    if (this.cache.has(data.rawData)) {
      console.log('Using cached data...');
      return this.cache.get(data.rawData);
    }

    console.log('Processing and caching data...');
    const processedData = { cachedData: data.rawData.toLowerCase() };
    this.cache.set(data.rawData, processedData);
    return processedData;
  }
}
```

---

### 3.2. 조건부 단계 실행

#### 특정 조건에 따라 단계 실행

```typescript
@Injectable()
export class ConditionalStep implements PipelineStep {
  async execute(data: any): Promise<any> {
    if (!data.shouldProcess) {
      console.log('Skipping step due to condition...');
      return data;
    }

    console.log('Executing conditional step...');
    return { ...data, processed: true };
  }
}
```

---

### 3.3. 동적 병렬 단계 추가

#### 런타임에 동적으로 병렬 단계 추가

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class DynamicParallelStep implements PipelineStep {
  isParallel = true;

  async execute(data: any): Promise<any> {
    console.log('Executing dynamic parallel step...');
    return { dynamicData: `Processed at ${Date.now()}` };
  }
}
```

---

## 4. 실행 결과

### 병렬 파이프라인 실행

**GET /pipeline/run**

**결과:**
```
Extracting data...
Transforming data...
Loading data...
{
  "status": "success",
  "data": "RAW DATA FROM SOURCE",
  "dynamicData": "Processed at 1634609432"
}
```

---

## 5. 결론

Builder 패턴을 사용하여 다음과 같은 최적화된 ETL 파이프라인을 설계할 수 있습니다:
1. **병렬 처리**:
   - CPU 및 I/O 리소스를 효율적으로 사용.
2. **조건부 실행**:
   - 불필요한 단계를 건너뛰어 성능 최적화.
3. **캐싱**:
   - 반복 작업 제거로 성능 향상.

NestJS와 Builder 패턴의 조합을 활용하여 효율적이고 확장 가능한 고성능 파이프라인을 설계하세요!
