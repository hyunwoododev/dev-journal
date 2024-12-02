
# NestJS + 파이프(Pipes)

`파이프(Pipes)`는 요청 데이터를 변환하거나, 유효성을 검증하는 데 사용되는 NestJS의 주요 기능 중 하나입니다.  
컨트롤러 핸들러가 실행되기 전에 파이프가 데이터를 가공하거나 검증하며, 잘못된 데이터가 들어오면 요청을 차단할 수 있습니다.

---

## 1. 파이프란?

NestJS의 파이프는 다음 두 가지 주요 역할을 수행합니다:
1. **데이터 변환**:
   - 요청 데이터를 원하는 형식으로 변환합니다. 예를 들어, 문자열을 정수로 변환.
2. **데이터 유효성 검사**:
   - 데이터가 특정 조건을 충족하는지 검증합니다. 유효하지 않은 경우 예외를 발생시킵니다.

---

## 2. 파이프 구현 방법

NestJS에서 파이프는 다음과 같은 방식으로 구현할 수 있습니다:
1. **기본 제공 파이프**:
   - `ValidationPipe`, `ParseIntPipe` 등.
2. **커스텀 파이프**:
   - 사용자 정의 로직을 포함하는 파이프를 구현.

---

## 3. 파이프 구현 예제

### 3.1. 기본 제공 파이프 사용

#### ValidationPipe

`class-validator`와 `class-transformer`를 활용하여 DTO를 기반으로 요청 데이터를 검증합니다.

**DTO 정의**:
```typescript
import { IsString, IsInt, Min, Max } from 'class-validator';

export class CreateUserDto {
  @IsString()
  name: string;

  @IsInt()
  @Min(18)
  @Max(60)
  age: number;
}
```

**컨트롤러에서 사용**:
```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { CreateUserDto } from './create-user.dto';

@Controller('users')
export class UsersController {
  @Post()
  createUser(@Body() createUserDto: CreateUserDto) {
    return { message: 'User created successfully', data: createUserDto };
  }
}
```

**글로벌 ValidationPipe 설정**:
```typescript
import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // ValidationPipe를 전역으로 설정
  app.useGlobalPipes(new ValidationPipe());

  await app.listen(3000);
}
bootstrap();
```

---

### 3.2. 데이터 변환 파이프

#### ParseIntPipe

`ParseIntPipe`는 요청 데이터를 정수로 변환합니다.

**컨트롤러에서 사용**:
```typescript
import { Controller, Get, Param, ParseIntPipe } from '@nestjs/common';

@Controller('items')
export class ItemsController {
  @Get(':id')
  getItem(@Param('id', ParseIntPipe) id: number) {
    return { id, message: 'Item fetched successfully' };
  }
}
```

**요청**:
```
GET /items/123
```

**응답**:
```json
{
  "id": 123,
  "message": "Item fetched successfully"
}
```

---

### 3.3. 커스텀 파이프

#### 간단한 커스텀 파이프

**커스텀 파이프 정의**:
```typescript
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';

@Injectable()
export class UppercasePipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata): string {
    if (typeof value !== 'string') {
      throw new BadRequestException('Validation failed: Value is not a string');
    }
    return value.toUpperCase();
  }
}
```

**컨트롤러에서 사용**:
```typescript
import { Controller, Get, Query, UsePipes } from '@nestjs/common';
import { UppercasePipe } from './uppercase.pipe';

@Controller('greetings')
export class GreetingsController {
  @Get()
  @UsePipes(UppercasePipe)
  getGreeting(@Query('name') name: string) {
    return { message: `Hello, ${name}!` };
  }
}
```

**요청**:
```
GET /greetings?name=world
```

**응답**:
```json
{
  "message": "Hello, WORLD!"
}
```

---

## 4. 커스텀 파이프와 DTO의 조합

#### CustomValidationPipe

**CustomValidationPipe**:
```typescript
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';
import { validate } from 'class-validator';
import { plainToInstance } from 'class-transformer';

@Injectable()
export class CustomValidationPipe implements PipeTransform {
  async transform(value: any, metadata: ArgumentMetadata) {
    const { metatype } = metadata;
    if (!metatype || !this.toValidate(metatype)) {
      return value;
    }

    const object = plainToInstance(metatype, value);
    const errors = await validate(object);

    if (errors.length > 0) {
      throw new BadRequestException('Validation failed');
    }

    return object;
  }

  private toValidate(metatype: Function): boolean {
    const types: Function[] = [String, Boolean, Number, Array, Object];
    return !types.includes(metatype);
  }
}
```

**컨트롤러에서 사용**:
```typescript
import { Controller, Post, Body, UsePipes } from '@nestjs/common';
import { CustomValidationPipe } from './custom-validation.pipe';
import { CreateUserDto } from './create-user.dto';

@Controller('users')
export class UsersController {
  @Post()
  @UsePipes(new CustomValidationPipe())
  createUser(@Body() createUserDto: CreateUserDto) {
    return { message: 'User created successfully', data: createUserDto };
  }
}
```

---

## 5. 결론

NestJS의 파이프는 다음과 같은 장점을 제공합니다:
1. **데이터 유효성 검사**:
   - 잘못된 데이터로 인해 발생할 수 있는 오류를 사전에 방지.
2. **데이터 변환**:
   - 요청 데이터를 컨트롤러 로직에서 쉽게 처리할 수 있도록 변환.
3. **유연성**:
   - 기본 제공 파이프와 커스텀 파이프를 결합하여 다양한 요구사항을 충족.

NestJS의 파이프를 활용하여 안전하고 효율적인 애플리케이션을 설계하세요!
