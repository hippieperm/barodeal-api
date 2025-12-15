# barodeal-api

쇼핑트렌드 1~100위를 조회할 수 있는 Node.js Express REST API 서버입니다.

## 기능

- 쇼핑트렌드 1~100위 전체 조회
- 쇼핑트렌드 상위 N위 조회 (1~100 사이)
- **매일 한국시간 12시 자동 데이터 갱신**
- 실제 쇼핑트렌드 데이터 제공
- 데이터 캐싱으로 빠른 응답
- 수동 데이터 갱신 기능
- CORS 지원
- JSON 형식 응답

## 설치 및 실행

### 1. Node.js 설치

Node.js 14 이상 버전이 필요합니다. [Node.js 공식 사이트](https://nodejs.org/)에서 다운로드하세요.

### 2. 의존성 설치

```bash
npm install
```

### 3. 서버 실행

```bash
npm start
```

또는 개발 모드 (nodemon 사용):

```bash
npm run dev
```

서버는 기본적으로 `http://localhost:5000`에서 실행됩니다.

### 4. 앱 연동을 위한 설정

**로컬 네트워크에서 접속 (같은 WiFi):**

1. 서버 컴퓨터의 IP 주소 확인:

   - macOS/Linux: `ifconfig` 또는 `ip addr`
   - Windows: `ipconfig`
   - 예: `192.168.0.100`

2. 환경 변수로 호스트와 포트 설정:

```bash
# 기본 설정 (모든 인터페이스에서 접속 가능)
HOST=0.0.0.0 PORT=5000 npm start

# 특정 IP 주소 사용
HOST=192.168.0.100 PORT=5000 npm start

# 포트만 변경
PORT=8000 npm start
```

3. 앱에서 접속할 주소:

```
http://192.168.0.100:5000/api/trends
```

**프로덕션 환경 (도메인 사용):**

```bash
HOST=0.0.0.0 PORT=80 npm start
```

또는 `.env` 파일 사용 (dotenv 패키지 설치 필요):

```bash
# .env 파일
HOST=0.0.0.0
PORT=5000
NAVER_CLIENT_ID=your_client_id
NAVER_CLIENT_SECRET=your_client_secret
```

## API 엔드포인트

### 1. 루트 엔드포인트

```
GET /
```

API 정보를 반환합니다.

### 2. 쇼핑트렌드 전체 조회 (1~100위)

```
GET /api/trends
```

**응답 예시:**

```json
{
  "success": true,
  "data": [
    {
      "rank": 1,
      "keyword": "노트북",
      "search_count": 9950,
      "trend_change": "stable",
      "category": "쇼핑",
      "updated_at": "2024-01-01T12:00:00+09:00"
    },
    ...
  ],
  "count": 100,
  "last_update": "2024-01-01T12:00:00+09:00",
  "timestamp": "2024-01-01T12:05:00+09:00"
}
```

### 3. 쇼핑트렌드 상위 N위 조회

```
GET /api/trends/:limit
```

**파라미터:**

- `limit`: 조회할 상위 순위 (1~100)

**예시:**

```
GET /api/trends/10
```

상위 10위까지 조회합니다.

### 4. 트렌드 데이터 수동 갱신

```
POST /api/refresh
```

트렌드 데이터를 즉시 갱신합니다. (스케줄러를 기다리지 않고)

**응답 예시:**

```json
{
  "success": true,
  "message": "트렌드 데이터가 갱신되었습니다.",
  "last_update": "2024-01-01T12:00:00+09:00",
  "timestamp": "2024-01-01T12:05:00+09:00"
}
```

### 5. 헬스 체크

```
GET /api/health
```

서버 상태와 스케줄러 상태를 확인합니다.

**응답 예시:**

```json
{
  "status": "healthy",
  "scheduler_running": true,
  "last_update": "2024-01-01T12:00:00+09:00",
  "cache_count": 100,
  "timestamp": "2024-01-01T12:05:00+09:00"
}
```

## 사용 예시

### cURL

```bash
# 전체 트렌드 조회
curl http://localhost:5000/api/trends

# 상위 10위 조회
curl http://localhost:5000/api/trends/10

# 수동 갱신
curl -X POST http://localhost:5000/api/refresh
```

### JavaScript/Node.js

```javascript
const axios = require("axios");

// 전체 트렌드 조회
const response = await axios.get("http://localhost:5000/api/trends");
console.log(response.data);

// 상위 10위 조회
const response2 = await axios.get("http://localhost:5000/api/trends/10");
console.log(response2.data);
```

### Python

```python
import requests

# 전체 트렌드 조회
response = requests.get('http://localhost:5000/api/trends')
data = response.json()
print(data)

# 상위 10위 조회
response = requests.get('http://localhost:5000/api/trends/10')
data = response.json()
print(data)
```

## 자동 업데이트 스케줄

- **업데이트 시간**: 매일 한국시간(KST) 12:00
- **업데이트 방식**: node-cron을 통해 자동 실행
- **데이터 캐싱**: 메모리에 캐시되어 빠른 응답 제공

서버 시작 시 자동으로 초기 데이터를 로드하며, 이후 매일 정오에 자동으로 갱신됩니다.

## 실제 데이터 연동

현재는 네이버 쇼핑 트렌드를 가져오도록 구현되어 있습니다. 더 정확한 데이터를 원하시면 다음 방법을 사용할 수 있습니다:

### 네이버 API 키 설정

네이버 API 키는 코드에 기본값으로 설정되어 있지만, 환경 변수로도 설정할 수 있습니다:

```bash
export NAVER_CLIENT_ID="your_client_id"
export NAVER_CLIENT_SECRET="your_client_secret"
```

또는 `.env` 파일 사용:

```bash
NAVER_CLIENT_ID=your_client_id
NAVER_CLIENT_SECRET=your_client_secret
```

### 쿠팡 파트너스 API 사용

1. [쿠팡 파트너스](https://partners.coupang.com/)에서 API 키 발급
2. 환경 변수 설정:

```bash
export COUPANG_ACCESS_KEY="your_access_key"
export COUPANG_SECRET_KEY="your_secret_key"
```

3. `app.js`의 `fetchAlternativeTrends()` 함수에서 API 호출 구현

## 개발 참고사항

- 데이터는 메모리에 캐시되므로 서버 재시작 시 초기 데이터가 자동으로 로드됩니다
- 스케줄러는 node-cron을 사용하여 백그라운드에서 실행됩니다
- 한국시간(KST) 기준으로 매일 12시에 자동 업데이트됩니다
- 개발 모드에서는 `npm run dev`를 사용하면 파일 변경 시 자동으로 재시작됩니다 (nodemon)

## 프로젝트 구조

```
barodeal-api/
├── app.js              # 메인 서버 파일
├── package.json        # Node.js 의존성 및 스크립트
├── README.md          # 프로젝트 문서
└── node_modules/      # 설치된 패키지 (자동 생성)
```

## 라이선스

MIT
