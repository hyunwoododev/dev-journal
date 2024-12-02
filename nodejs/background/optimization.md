
# Node.js 성능 최적화

Node.js는 빠르고 효율적인 서버 환경을 제공하지만, 성능을 최적화하기 위해 중요한 설계 및 코딩 기법을 활용해야 합니다.  
본 문서에서는 Node.js 애플리케이션의 성능을 극대화하기 위한 전략과 실전 사례를 다룹니다.

---

## 1. 성능 최적화의 필요성

Node.js는 I/O 작업에 강점이 있지만, 다음과 같은 이유로 성능 최적화가 필요합니다:
1. **싱글 스레드 환경**:
   - CPU 집약적인 작업에서 이벤트 루프가 차단될 수 있음.
2. **메모리 관리**:
   - 대규모 데이터를 처리할 때 메모리 사용을 최소화.
3. **응답 속도 향상**:
   - 사용자 경험 개선 및 확장성 확보.

---

## 2. 성능 최적화를 위한 핵심 전략

---

### 2.1. 비동기 프로그래밍 활용

#### `async/await`와 `Promise`를 적극적으로 사용

```javascript
async function fetchData(url) {
  const response = await fetch(url);
  const data = await response.json();
  return data;
}
```

#### 비동기 작업 병렬 처리

```javascript
async function processInParallel(urls) {
  const results = await Promise.all(urls.map(url => fetchData(url)));
  return results;
}
```

---

### 2.2. 이벤트 루프 차단 방지

#### CPU 집약 작업을 워커 스레드로 분리

```javascript
const { Worker } = require('worker_threads');

function runWorker(file, workerData) {
  return new Promise((resolve, reject) => {
    const worker = new Worker(file, { workerData });
    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', (code) => {
      if (code !== 0) reject(new Error(`Worker stopped with exit code ${code}`));
    });
  });
}

// Usage
runWorker('./worker.js', { task: 'heavy computation' }).then(console.log);
```

---

### 2.3. 데이터 캐싱

#### 메모리 캐싱 (예: Redis)

```javascript
const Redis = require('ioredis');
const redis = new Redis();

async function getCachedData(key, fetchFunction) {
  const cachedData = await redis.get(key);
  if (cachedData) return JSON.parse(cachedData);

  const data = await fetchFunction();
  await redis.set(key, JSON.stringify(data), 'EX', 3600); // 1시간 캐싱
  return data;
}
```

---

### 2.4. 스트림 활용

#### 스트림을 사용하여 대용량 데이터 처리

```javascript
const fs = require('fs');

const readable = fs.createReadStream('large-file.txt');
const writable = fs.createWriteStream('output.txt');

readable.pipe(writable).on('finish', () => {
  console.log('File processing completed.');
});
```

---

### 2.5. 클러스터링과 로드 밸런싱

#### 클러스터를 활용하여 여러 프로세스로 부하 분산

```javascript
const cluster = require('cluster');
const http = require('http');
const os = require('os');

if (cluster.isMaster) {
  const numCPUs = os.cpus().length;
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
} else {
  http.createServer((req, res) => res.end('Hello World')).listen(3000);
}
```

---

## 3. 메모리 최적화

---

### 3.1. 메모리 누수 방지

#### 이벤트 리스너 누수 점검

```javascript
const EventEmitter = require('events');
const emitter = new EventEmitter();
emitter.setMaxListeners(10); // 최대 리스너 수 설정
```

#### 객체 참조 해제

```javascript
let cache = {};
function clearCache(key) {
  delete cache[key];
}
```

---

### 3.2. 대용량 데이터 처리

#### JSON 스트림 파싱

```javascript
const JSONStream = require('JSONStream');
const fs = require('fs');

fs.createReadStream('large.json')
  .pipe(JSONStream.parse('*'))
  .on('data', (data) => console.log(data));
```

---

## 4. 네트워크 최적화

---

### 4.1. HTTP/2 활용

Node.js는 HTTP/2를 기본 지원하며, 다중화(Multiplexing)로 성능을 향상시킬 수 있습니다.

```javascript
const http2 = require('http2');

const server = http2.createSecureServer({
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),
});

server.on('stream', (stream, headers) => {
  stream.respond({ ':status': 200 });
  stream.end('Hello HTTP/2');
});

server.listen(3000);
```

---

### 4.2. Gzip 압축 활성화

```javascript
const express = require('express');
const compression = require('compression');
const app = express();

app.use(compression());
app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(3000);
```

---

## 5. 모니터링 및 디버깅

---

### 5.1. 성능 모니터링

#### PM2와 모니터링 대시보드

```bash
npm install pm2 -g
pm2 start app.js
pm2 monit
```

---

### 5.2. 프로파일링

#### Node.js 내장 프로파일러 사용

```bash
node --inspect app.js
```

Chrome DevTools를 사용하여 실행 중인 애플리케이션을 분석합니다.

---

## 6. 결론

Node.js 애플리케이션의 성능 최적화를 위해 다음과 같은 전략을 활용하세요:
1. **비동기 및 병렬 처리**:
   - 이벤트 루프 차단 방지 및 병렬 실행 활용.
2. **데이터 캐싱**:
   - Redis와 같은 도구로 데이터 로딩 속도 향상.
3. **네트워크 최적화**:
   - HTTP/2와 Gzip 압축으로 전송 속도 최적화.
4. **모니터링**:
   - PM2와 프로파일링 도구로 애플리케이션 성능 분석.

Node.js의 강력한 기능과 최적화 기법을 활용하여 고성능 서버 애플리케이션을 설계하세요!
