
# Node.js에서 싱글 스레드로 병렬 처리 구현하기

Node.js는 **싱글 스레드** 환경에서 동작하지만, 비동기 프로그래밍과 이벤트 루프(Event Loop) 덕분에 병렬 처리가 가능합니다.  
이를 통해 효율적으로 I/O 작업과 CPU 집약적인 작업을 처리할 수 있습니다.

---

## 1. Node.js에서 싱글 스레드의 동작 원리

Node.js는 **비동기 논블로킹 I/O 모델**을 기반으로 설계되었습니다.  
다음은 주요 구성 요소들입니다:

### 1.1. 이벤트 루프(Event Loop)
- 이벤트 루프는 Node.js의 핵심입니다.  
- 작업(함수 호출, 요청 등)을 큐에 넣고, 완료되면 콜백을 실행합니다.
- 이 과정은 논블로킹으로 동작하므로 스레드를 차단하지 않습니다.

### 1.2. 백그라운드 작업
- **libuv** 라이브러리를 사용하여 파일 시스템 작업, 네트워크 요청 등 I/O 작업을 스레드 풀에서 처리합니다.
- 싱글 스레드가 아닌 **스레드 풀**을 활용하여 병렬 작업을 처리합니다.

---

## 2. 병렬 실행을 가능하게 하는 주요 기술

### 2.1. I/O 작업 병렬 처리
Node.js의 **libuv**는 파일 읽기, 데이터베이스 쿼리, API 호출과 같은 I/O 작업을 스레드 풀에서 병렬로 처리합니다.

```javascript
const fs = require('fs');

console.time('readFiles');
fs.readFile('file1.txt', () => console.log('File 1 read'));
fs.readFile('file2.txt', () => console.log('File 2 read'));
console.timeEnd('readFiles');
```

---

### 2.2. `Promise`와 `Promise.all`을 활용한 병렬 처리

`Promise.all`을 사용하면 여러 비동기 작업을 병렬로 실행할 수 있습니다.

```javascript
async function fetchData() {
  const results = await Promise.all([
    fetch('https://api.example.com/data1'),
    fetch('https://api.example.com/data2'),
  ]);
  return results.map(res => res.json());
}
```

---

## 3. CPU 집약 작업을 병렬 처리하는 방법

Node.js는 싱글 스레드에서 CPU 집약적인 작업을 처리하면 이벤트 루프가 차단될 수 있습니다.  
이를 해결하기 위해 다음 두 가지 접근법을 사용할 수 있습니다.

### 3.1. Worker Threads (워커 스레드)

워커 스레드는 Node.js 10 이상에서 사용 가능하며, 독립적인 실행 환경을 제공합니다.

#### 워커 스레드 구현

**main.js**:
```javascript
const { Worker } = require('worker_threads');

const worker = new Worker('./worker-script.js');
worker.on('message', (message) => console.log('Received:', message));
worker.postMessage('Start');
```

**worker-script.js**:
```javascript
const { parentPort } = require('worker_threads');

parentPort.on('message', (message) => {
  console.log('Worker received:', message);
  parentPort.postMessage('Task completed!');
});
```

---

### 3.2. 클러스터링 (Clustering)

Node.js의 **Cluster 모듈**은 프로세스를 복제하여 병렬로 작업을 처리합니다.

#### 클러스터링 예제

```javascript
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
} else {
  http.createServer((req, res) => res.end('Hello World')).listen(3000);
}
```

---

## 4. 병렬 처리를 위한 주의점과 한계

### 한계
1. **싱글 스레드의 한계**:
   - JavaScript 코드 자체는 여전히 단일 스레드에서 실행됩니다.
2. **블로킹 작업**:
   - CPU 집약적인 작업은 이벤트 루프를 차단할 수 있습니다.

### 주의점
1. **적절한 작업 분배**:
   - CPU 작업은 워커 스레드나 클러스터링을 활용.
2. **비동기 오류 처리**:
   - `Promise` 또는 콜백의 오류를 적절히 처리하여 안정성 확보.

---

## 5. 결론

Node.js는 **비동기 작업을 싱글 스레드 환경에서 병렬로 처리하는 환상적인 도구**입니다.  
I/O 작업은 이벤트 루프와 스레드 풀을 통해 처리하고, CPU 집약적인 작업은 워커 스레드나 클러스터링을 활용하여 처리할 수 있습니다.

### 주요 기술 요약
- **I/O 병렬 처리**:
  - libuv와 Promise를 활용.
- **CPU 작업 분산**:
  - 워커 스레드와 클러스터링으로 분산 처리.
- **확장성과 안정성**:
  - 병렬 처리 기법을 통해 고성능 애플리케이션 구축.

Node.js의 병렬 처리 기술을 활용하여 확장 가능하고 효율적인 애플리케이션을 설계하세요!
