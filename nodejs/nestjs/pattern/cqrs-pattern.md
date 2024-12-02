

# CQRS + NestJS 패턴

CQRS(Command Query Responsibility Segregation) 패턴은 데이터 읽기와 쓰기 작업을 명확히 분리하여 복잡한 비즈니스 로직을 간결하고 확장 가능하게 관리할 수 있도록 합니다. NestJS는 CQRS 패턴을 쉽게 적용할 수 있는 도구와 구조를 제공합니다.

---

## 1. CQRS란?

CQRS는 **Command**와 **Query**를 분리하여 설계합니다:
- **Command**: 데이터의 상태를 변경하는 작업(Create, Update, Delete).
- **Query**: 데이터를 읽는 작업(Read).

이 패턴을 사용하면 다음과 같은 장점이 있습니다:
- 비즈니스 로직과 읽기 작업을 분리하여 코드 복잡도를 줄임.
- 성능 최적화 가능 (읽기와 쓰기 작업을 독립적으로 확장 가능).
- 데이터 무결성과 읽기 전용 데이터 모델의 분리.

---

## 2. NestJS에서의 CQRS

NestJS는 `@nestjs/cqrs` 패키지를 통해 CQRS 패턴을 구현할 수 있습니다. 다음은 NestJS에서 CQRS를 적용한 사용자 관리 예제입니다.

---

## 3. 사용자 관리 예제

### 3.1. Command: CreateUserCommand
사용자 데이터를 생성하기 위한 Command 클래스입니다.

```typescript
export class CreateUserCommand {
  constructor(public readonly name: string, public readonly email: string) {}
}
```

### 3.2. CommandHandler: CreateUserHandler
Command를 처리하여 사용자를 생성하는 핸들러입니다.

```typescript
import { CommandHandler, ICommandHandler } from '@nestjs/cqrs';
import { UserRepository } from './user.repository';

@CommandHandler(CreateUserCommand)
export class CreateUserHandler implements ICommandHandler<CreateUserCommand> {
  constructor(private readonly userRepository: UserRepository) {}

  async execute(command: CreateUserCommand): Promise<void> {
    const { name, email } = command;
    await this.userRepository.save({ name, email });
    console.log(`User created: ${name} (${email})`);
  }
}
```

---

### 3.3. Query: GetUsersQuery
사용자 목록을 조회하는 Query 클래스입니다.

```typescript
export class GetUsersQuery {}
```

### 3.4. QueryHandler: GetUsersHandler
Query를 처리하여 사용자 목록을 반환하는 핸들러입니다.

```typescript
import { QueryHandler, IQueryHandler } from '@nestjs/cqrs';
import { UserRepository } from './user.repository';

@QueryHandler(GetUsersQuery)
export class GetUsersHandler implements IQueryHandler<GetUsersQuery> {
  constructor(private readonly userRepository: UserRepository) {}

  async execute(): Promise<any[]> {
    return this.userRepository.find();
  }
}
```

---

### 3.5. Repository: UserRepository
사용자 데이터를 관리하는 리포지토리입니다. 실제 데이터베이스 연결을 처리합니다.

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class UserRepository {
  private readonly users: any[] = []; // 메모리 데이터베이스

  async save(user: { name: string; email: string }): Promise<void> {
    this.users.push(user);
  }

  async find(): Promise<any[]> {
    return this.users;
  }
}
```

---

### 3.6. Controller
Command와 Query를 실행하기 위한 컨트롤러입니다.

```typescript
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CommandBus, QueryBus } from '@nestjs/cqrs';
import { CreateUserCommand } from './create-user.command';
import { GetUsersQuery } from './get-users.query';

@Controller('users')
export class UserController {
  constructor(
    private readonly commandBus: CommandBus,
    private readonly queryBus: QueryBus,
  ) {}

  @Post()
  async createUser(@Body() body: { name: string; email: string }) {
    const { name, email } = body;
    await this.commandBus.execute(new CreateUserCommand(name, email));
    return { message: 'User created successfully' };
  }

  @Get()
  async getUsers() {
    return this.queryBus.execute(new GetUsersQuery());
  }
}
```

---

### 3.7. Module
CQRS 패턴을 적용한 모듈입니다.

```typescript
import { Module } from '@nestjs/common';
import { CqrsModule } from '@nestjs/cqrs';
import { UserController } from './user.controller';
import { CreateUserHandler } from './create-user.handler';
import { GetUsersHandler } from './get-users.handler';
import { UserRepository } from './user.repository';

@Module({
  imports: [CqrsModule],
  controllers: [UserController],
  providers: [CreateUserHandler, GetUsersHandler, UserRepository],
})
export class UserModule {}
```

---

## 4. 실행 결과

### 사용자 생성 요청
**POST /users**
```json
{
  "name": "John Doe",
  "email": "john.doe@example.com"
}
```

**응답:**
```json
{
  "message": "User created successfully"
}
```

### 사용자 목록 조회
**GET /users**

**응답:**
```json
[
  {
    "name": "John Doe",
    "email": "john.doe@example.com"
  }
]
```

---

## 5. 결론

NestJS에서 CQRS 패턴을 사용하면 다음과 같은 이점을 얻을 수 있습니다:
1. **읽기와 쓰기 로직의 명확한 분리**: 유지보수와 코드 가독성 향상.
2. **비즈니스 로직의 독립성**: Command와 Query 핸들러는 각각의 작업에 집중.
3. **확장성**: 데이터 소스 변경 또는 복잡한 비즈니스 로직 추가 시 기존 코드 영향 최소화.

CQRS는 복잡한 애플리케이션 구조를 체계적으로 관리하는 데 매우 유용한 패턴입니다. NestJS의 강력한 CQRS 지원을 활용하여 확장 가능한 애플리케이션을 설계하세요!
