
# NestJS + 순환 참조(Circular Dependency) 방지

NestJS는 모듈화된 구조와 의존성 주입(DI)을 사용하여 설계됩니다.  
그러나 잘못된 설계로 인해 **순환 참조(Circular Dependency)** 문제가 발생할 수 있습니다.  
본 문서에서는 순환 참조 문제를 이해하고 이를 방지하거나 해결하는 고급 기법을 다룹니다.

---

## 1. 순환 참조란?

순환 참조는 두 개 이상의 모듈이나 클래스가 서로를 직접 또는 간접적으로 참조하는 경우를 말합니다.  
이로 인해 애플리케이션이 예기치 않게 동작하거나 실행 중단이 발생할 수 있습니다.

---

### 예시: 순환 참조 발생

```typescript
// user.service.ts
@Injectable()
export class UserService {
  constructor(private readonly authService: AuthService) {}
}

// auth.service.ts
@Injectable()
export class AuthService {
  constructor(private readonly userService: UserService) {}
}
```

위 예제에서 `UserService`와 `AuthService`는 서로를 참조하며, 순환 참조 문제가 발생합니다.

---

## 2. 순환 참조 방지 및 해결 기법

---

### 2.1. ForwardRef를 활용한 순환 참조 해결

#### ForwardRef로 순환 참조 명시

`@Inject(forwardRef(() => ClassName))`를 사용하여 순환 참조를 해결합니다.

```typescript
// user.service.ts
@Injectable()
export class UserService {
  constructor(@Inject(forwardRef(() => AuthService)) private readonly authService: AuthService) {}
}

// auth.service.ts
@Injectable()
export class AuthService {
  constructor(@Inject(forwardRef(() => UserService)) private readonly userService: UserService) {}
}
```

#### ForwardRef를 사용하는 모듈 구성

```typescript
// user.module.ts
@Module({
  providers: [UserService],
  exports: [UserService],
})
export class UserModule {}

// auth.module.ts
@Module({
  imports: [forwardRef(() => UserModule)],
  providers: [AuthService],
})
export class AuthModule {}
```

---

### 2.2. 이벤트 기반 설계로 순환 참조 제거

#### EventEmitter를 사용한 간접 참조

이벤트를 사용하여 직접 참조 대신 간접적으로 통신.

```typescript
import { Injectable } from '@nestjs/common';
import { EventEmitter2 } from '@nestjs/event-emitter';

@Injectable()
export class AuthService {
  constructor(private readonly eventEmitter: EventEmitter2) {}

  triggerUserEvent(userId: string) {
    this.eventEmitter.emit('user.created', { userId });
  }
}

@Injectable()
export class UserService {
  constructor() {
    EventEmitter2.on('user.created', (event) => {
      console.log(`User created with ID: ${event.userId}`);
    });
  }
}
```

---

### 2.3. 의존성 분리를 통한 구조 개선

#### 새로운 서비스로 공통 의존성 분리

순환 참조되는 공통 로직을 별도의 서비스로 분리.

```typescript
@Injectable()
export class SharedService {
  sharedLogic() {
    return 'Shared logic executed';
  }
}

// user.service.ts
@Injectable()
export class UserService {
  constructor(private readonly sharedService: SharedService) {}

  userLogic() {
    return this.sharedService.sharedLogic();
  }
}

// auth.service.ts
@Injectable()
export class AuthService {
  constructor(private readonly sharedService: SharedService) {}

  authLogic() {
    return this.sharedService.sharedLogic();
  }
}
```

---

### 2.4. 추상화와 인터페이스 활용

#### 인터페이스로 의존성 주입

```typescript
export interface IUserService {
  getUserDetails(id: string): any;
}

// user.service.ts
@Injectable()
export class UserService implements IUserService {
  getUserDetails(id: string) {
    return { id, name: 'John Doe' };
  }
}

// auth.service.ts
@Injectable()
export class AuthService {
  constructor(@Inject('IUserService') private readonly userService: IUserService) {}

  validateUser(userId: string) {
    return this.userService.getUserDetails(userId);
  }
}

// app.module.ts
@Module({
  providers: [
    {
      provide: 'IUserService',
      useClass: UserService,
    },
    AuthService,
  ],
})
export class AppModule {}
```

---

## 3. 고급 기법: Circular Dependency Plugin 활용

#### 플러그인을 사용한 자동 감지 및 해결

`@nestjs/circular-dependency` 플러그인을 설치하여 순환 참조를 감지.

```bash
npm install @nestjs/circular-dependency
```

**main.ts**:
```typescript
import { CircularDependencyPlugin } from '@nestjs/circular-dependency';

const app = await NestFactory.create(AppModule);
app.useLogger(app.get(CircularDependencyPlugin));
await app.listen(3000);
```

---

## 4. 순환 참조 방지 모범 사례

---

### 4.1. 설계 단계에서 의존성 주입 최소화
- 서비스 간 직접 참조를 최소화하고, 공통 기능은 별도로 분리.

---

### 4.2. 계층 구조의 명확화
- 모듈 계층 구조를 명확히 하여 상위와 하위 모듈 간 참조 순환 방지.

---

### 4.3. 의존성 주입 주기 관리
- 필요할 경우 `forwardRef`를 통해 의존성 주기를 명시적으로 설정.

---

## 5. 결론

NestJS 애플리케이션에서 순환 참조는 설계와 유지보수에 큰 영향을 미칠 수 있습니다.  
다음과 같은 방법으로 이를 방지하거나 해결하세요:
1. **ForwardRef 사용**:
   - 순환 참조가 불가피한 경우 명시적으로 해결.
2. **공통 의존성 분리**:
   - 순환되는 로직을 별도의 서비스로 분리.
3. **이벤트 기반 설계**:
   - 간접 참조를 통한 해결.
4. **플러그인 활용**:
   - 자동 감지 및 해결 도구 사용.

NestJS의 구조적 설계 원칙을 준수하여 확장 가능하고 유지보수하기 쉬운 애플리케이션을 설계하세요!
