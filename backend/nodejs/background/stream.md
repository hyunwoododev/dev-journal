
# Node.js + Streams 고급 사용법

Node.js의 **Streams**는 대용량 데이터를 효율적으로 처리하기 위한 핵심 도구입니다.  
스트림을 사용하면 메모리 사용을 최소화하면서 파일, 네트워크, 데이터베이스 등의 데이터를 처리할 수 있습니다.

---

## 1. Node.js의 스트림이란?

스트림(Stream)은 **연속적으로 전달되는 데이터**의 흐름을 처리하기 위한 추상화입니다.

### 주요 특징:
1. **데이터 청크 처리**:
   - 데이터를 작은 청크 단위로 처리하여 메모리 효율을 극대화.
2. **비동기 처리**:
   - 스트림의 데이터 처리는 비동기로 이루어짐.
3. **파이프라인**:
   - 스트림 간 연결(Piping)을 통해 효율적인 데이터 흐름 관리.

---

## 2. 스트림의 종류

Node.js는 다음과 같은 스트림을 제공합니다:
1. **Readable Stream**:
   - 데이터를 읽을 수 있는 스트림 (예: 파일 읽기, HTTP 응답).
2. **Writable Stream**:
   - 데이터를 쓸 수 있는 스트림 (예: 파일 쓰기, HTTP 요청).
3. **Duplex Stream**:
   - 데이터를 읽고 쓸 수 있는 스트림 (예: 소켓 연결).
4. **Transform Stream**:
   - 데이터를 변환할 수 있는 스트림 (예: 압축, 암호화).

---

## 3. 스트림 사용 예제

---

### 3.1. Readable Stream: 파일 읽기

```javascript
const fs = require('fs');

const readable = fs.createReadStream('input.txt', { encoding: 'utf-8' });
readable.on('data', (chunk) => {
  console.log('Data chunk received:', chunk);
});
readable.on('end', () => {
  console.log('No more data.');
});
```

---

### 3.2. Writable Stream: 파일 쓰기

```javascript
const fs = require('fs');

const writable = fs.createWriteStream('output.txt');
writable.write('Hello, Node.js Streams!
');
writable.end('Goodbye, Streams!');
writable.on('finish', () => {
  console.log('All data has been written.');
});
```

---

### 3.3. Pipe: 데이터 스트림 연결

#### 파일 복사

```javascript
const fs = require('fs');

const readable = fs.createReadStream('input.txt');
const writable = fs.createWriteStream('output.txt');
readable.pipe(writable);
```

---

## 4. Transform Stream: 데이터 변환

---

### 4.1. 데이터 변환 예제

#### 대문자로 변환하는 Transform Stream

```javascript
const { Transform } = require('stream');

const uppercaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  },
});

process.stdin.pipe(uppercaseTransform).pipe(process.stdout);
```

---

### 4.2. 데이터 압축: Gzip

```javascript
const fs = require('fs');
const zlib = require('zlib');

const readable = fs.createReadStream('input.txt');
const writable = fs.createWriteStream('input.txt.gz');
const gzip = zlib.createGzip();

readable.pipe(gzip).pipe(writable);
```

---

## 5. 고급 스트림 사용법

---

### 5.1. 스트림 병렬 처리

#### 여러 스트림을 동시에 처리

```javascript
const { pipeline } = require('stream');
const fs = require('fs');
const zlib = require('zlib');

pipeline(
  fs.createReadStream('input.txt'),
  zlib.createGzip(),
  fs.createWriteStream('input.txt.gz'),
  (err) => {
    if (err) {
      console.error('Pipeline failed:', err);
    } else {
      console.log('Pipeline succeeded.');
    }
  },
);
```

---

### 5.2. 스트림의 백프레셔(Backpressure)

#### 백프레셔 제어

스트림의 읽기/쓰기 속도를 조절하여 메모리 과부하를 방지.

```javascript
const { Readable, Writable } = require('stream');

const readable = new Readable({
  read(size) {
    setTimeout(() => {
      this.push('Data chunk');
    }, 100);
  },
});

const writable = new Writable({
  write(chunk, encoding, callback) {
    console.log('Writing:', chunk.toString());
    setTimeout(callback, 200); // 인위적인 지연
  },
});

readable.pipe(writable);
```

---

## 6. 스트림 활용 사례

---

### 6.1. HTTP 응답 스트림 처리

#### 스트리밍 응답

```javascript
const http = require('http');

http.createServer((req, res) => {
  const readable = fs.createReadStream('big-file.txt');
  readable.pipe(res);
}).listen(3000, () => {
  console.log('Server running on port 3000');
});
```

---

### 6.2. 실시간 데이터 처리

#### WebSocket을 통한 스트림 데이터 전송

```javascript
const WebSocket = require('ws');
const { Transform } = require('stream');

const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
  const transformStream = new Transform({
    transform(chunk, encoding, callback) {
      this.push(chunk.toString().toUpperCase());
      callback();
    },
  });

  ws.on('message', (message) => {
    const readable = Readable.from(message);
    readable.pipe(transformStream).pipe(process.stdout);
  });
});
```

---

## 7. 스트림 사용 시 주의점

---

### 7.1. 메모리 관리
- 대용량 데이터를 처리할 때 메모리 사용을 최적화.

### 7.2. 에러 처리
- 스트림의 `error` 이벤트를 반드시 처리.

```javascript
readable.on('error', (err) => {
  console.error('Error:', err);
});
```

---

## 8. 결론

Node.js의 스트림은 대용량 데이터 처리에서 메모리 효율성과 성능을 극대화할 수 있는 강력한 도구입니다.  
1. **Pipe를 활용한 데이터 흐름 관리**:
   - 읽기, 쓰기, 변환을 연결하여 간단하게 처리.
2. **백프레셔로 메모리 과부하 방지**:
   - 데이터 읽기/쓰기 속도 조절.
3. **실시간 데이터 처리**:
   - WebSocket 등과 결합하여 스트림 데이터를 실시간으로 처리.

Node.js 스트림을 활용하여 효율적이고 확장 가능한 애플리케이션을 설계하세요!
