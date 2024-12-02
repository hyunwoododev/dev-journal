
# Node.js 보안 모범 사례

Node.js 애플리케이션의 보안은 중요하며, 안전한 애플리케이션을 구축하기 위해 다양한 모범 사례를 따를 필요가 있습니다.  
본 문서에서는 Node.js 애플리케이션에서 흔히 발생하는 보안 문제와 이를 방지하기 위한 전략을 다룹니다.

---

## 1. 보안이 중요한 이유

Node.js 애플리케이션은 클라이언트 요청을 처리하고 민감한 데이터를 관리하는 데 사용되기 때문에 다음과 같은 보안 문제를 방지해야 합니다:
1. **데이터 유출**:
   - 민감한 정보가 노출될 위험.
2. **애플리케이션 공격**:
   - SQL Injection, XSS, CSRF와 같은 일반적인 공격.
3. **서비스 중단**:
   - DDoS와 같은 과도한 요청으로 인한 서비스 중단.

---

## 2. Node.js 보안 모범 사례

---

### 2.1. 종속성 관리

#### 최신 버전 유지

Node.js 및 사용 중인 라이브러리의 최신 버전을 유지하여 알려진 취약점을 방지합니다.

```bash
npm outdated
npm update
```

#### npm audit 활용

`npm audit`로 패키지 보안 상태를 점검하고 취약점을 수정합니다.

```bash
npm audit
npm audit fix
```

---

### 2.2. 환경 변수 사용

#### 민감한 정보 보호

민감한 정보를 코드에 직접 포함하지 말고 환경 변수로 관리합니다.

**.env 파일**:
```plaintext
DB_PASSWORD=securepassword
```

**코드에서 사용**:
```javascript
require('dotenv').config();
const dbPassword = process.env.DB_PASSWORD;
```

---

### 2.3. 입력 데이터 검증

#### 데이터 검증을 통해 SQL Injection 방지

```javascript
const express = require('express');
const app = express();
app.use(express.json());

app.post('/user', (req, res) => {
  const { username } = req.body;

  if (!/^[a-zA-Z0-9]+$/.test(username)) {
    return res.status(400).send('Invalid username');
  }

  res.send('Username is valid');
});
```

---

### 2.4. 보안 헤더 설정

#### Helmet을 활용한 보안 헤더 적용

```bash
npm install helmet
```

```javascript
const helmet = require('helmet');
const express = require('express');
const app = express();

app.use(helmet());
app.listen(3000);
```

---

### 2.5. XSS 공격 방지

#### 입력 데이터의 HTML 이스케이프 처리

```bash
npm install xss
```

```javascript
const xss = require('xss');

app.post('/comment', (req, res) => {
  const sanitizedComment = xss(req.body.comment);
  res.send(`Your comment: ${sanitizedComment}`);
});
```

---

### 2.6. CSRF 방지

#### CSRF 토큰 적용

```bash
npm install csurf
```

```javascript
const csurf = require('csurf');
const express = require('express');
const app = express();

const csrfProtection = csurf({ cookie: true });
app.use(csrfProtection);

app.get('/form', (req, res) => {
  res.send(`<form action="/process" method="POST">
              <input type="hidden" name="_csrf" value="${req.csrfToken()}">
              <button type="submit">Submit</button>
            </form>`);
});
```

---

### 2.7. 민감한 데이터 암호화

#### 비밀번호 해싱

```bash
npm install bcrypt
```

```javascript
const bcrypt = require('bcrypt');

const password = 'userpassword';
const saltRounds = 10;

bcrypt.hash(password, saltRounds, (err, hash) => {
  console.log(`Hashed password: ${hash}`);
});
```

---

### 2.8. HTTPS 사용

#### HTTPS 서버 구성

```javascript
const https = require('https');
const fs = require('fs');
const express = require('express');
const app = express();

https
  .createServer(
    {
      key: fs.readFileSync('server-key.pem'),
      cert: fs.readFileSync('server-cert.pem'),
    },
    app,
  )
  .listen(3000, () => {
    console.log('HTTPS server running on port 3000');
  });
```

---

### 2.9. Rate Limiting 적용

#### Rate Limiting으로 DDoS 방지

```bash
npm install express-rate-limit
```

```javascript
const rateLimit = require('express-rate-limit');
const express = require('express');
const app = express();

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15분
  max: 100, // 각 IP당 100개의 요청
});

app.use(limiter);
```

---

## 3. 모니터링 및 알림 설정

---

### 3.1. 로그 관리

#### Winston으로 로깅 설정

```bash
npm install winston
```

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
  ],
});

logger.info('Application is running');
```

---

### 3.2. 실시간 모니터링

#### PM2로 모니터링 설정

```bash
npm install pm2 -g
pm2 start app.js
pm2 monit
```

---

## 4. 결론

Node.js 애플리케이션의 보안을 강화하려면 다음을 실천하세요:
1. **입력 데이터 검증**:
   - SQL Injection, XSS 방지.
2. **보안 헤더 적용**:
   - Helmet 등 도구를 활용.
3. **HTTPS 사용**:
   - 데이터 전송 중 암호화.
4. **Rate Limiting**:
   - DDoS 방지 및 요청 제한.

Node.js 보안 모범 사례를 실천하여 안전하고 신뢰할 수 있는 애플리케이션을 설계하세요!
