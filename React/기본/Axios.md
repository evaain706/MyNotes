
## Axios란?

브라우저,Nodejs환경 모두에서 사용가능한 `Promise`기반의 HTTP클라이언트 라이브러리
-> 기존의 `XMLHttpRequest`나`fetch`에 비해 간편하고 다양한 부가기능을 제공해줌

---

## 핵심기능

- Promise기반의 비동기 처리
  -> `async/await` 문법과 함께 사용하여 비동기코드를 동기코드처럼 깔끔하게 작성가능
- 자동 JSON데이터 변환
  -> 서버에서 응답받은 `JSON`형식 데이터를 별도의 처리없이 바로 객체로 변환
     보낼때도 객체를 자동으로 `JSON`으로 변환
- 요청,응답시 `인터셉터` 
  -> 요청을 보내기전이나 응답을 받기전에 특정 작업을 처리하는 인터셉터 선언 가능
- 간편한 에러 핸들링
  -> `status`코드가 `2xx`범위를 벗어나면 모두 `catch`블록에서 처리되어 에러핸들링 간편
- 요청,취소 타임아웃 설정
  -> 진행중인 HTTP요청을 취소,일정시간 응답이 없으면 자동중단이나 재시도 설정가능
- 브라우저 호환성
  -> 오래된 브라우저까지 지원
- `axios.create`를 사용한 `instance`생성후 재사용가능
  -> `baseURL`,`headers`,`timeout`,`interceptor`등에대한 설정을 미리 담아 재사용성 증대

---

## Axios vs Fetch

### Axios
- **설치**: `npm install axios` 필요
- **데이터 변환**: JSON 데이터를 자동으로 JavaScript 객체로 변환해 줌
- **에러 핸들링**: 2xx 범위를 벗어난 HTTP 상태 코드를 모두 에러로 처리하여 `.catch()`에서 처리 가능
- **인터셉터**: 요청과 응답을 가로채서 전역적으로 로직(토큰 주입, 에러 처리 등)을 추가하기 용이함
- **기타**: 요청 취소, 타임아웃, 업로드 진행 상태 모니터링 등 다양한 편의 기능 내장
###  Fetch API
- **설치**: 브라우저 내장 기능으로 별도 설치 불필요
- **데이터 변환**: `response.json()`이나 `response.text()`와 같은 메서드를 직접 호출하여 수동으로 변환해야 함
- **에러 핸들링**: 네트워크 장애와 같은 실제 요청 실패 시에만 에러를 발생시킴 (404, 500 에러는 성공 응답으로 간주)
- **인터셉터**: 내장된 인터셉터 기능이 없어 직접 구현해야 함
- **기타**: 기능이 단순하여 가볍지만, 타임아웃 설정 등 부가 기능은 직접 구현해야 하는 번거로움이 있음


---

## axios.create()를 통한 인스턴스 생성예시


```js
import axios, { AxiosInstance } from 'axios';

import { API_HEADERS, API_TIMEOUT } from '../constants/apiConstants';

/* Axios 인스턴스 생성 : 기본 Content type = application/json */
const instance: AxiosInstance = axios.create({
  baseURL: 'http://localhost:5000',
  timeout: API_TIMEOUT,
  headers: API_HEADERS.JSON,
});

/* 에러 메시지 출력 */
const getErrorMessage = (error: unknown): string => {
  if (axios.isAxiosError(error)) {
    const status = error.response?.status;
    const message = error.response?.data?.message;

    if (typeof message === 'string') return message;
    if (status) {
      switch (status) {
        case 400:
          return '🚨 잘못된 요청입니다. (400)';
        case 401:
          return '🚨 인증이 필요합니다. (401)';
        case 403:
          return '🚨 권한이 없습니다. (403)';
        case 404:
          return '🚨 요청한 리소스를 찾을 수 없습니다. (404)';
        case 500:
          return '🚨 서버 내부 오류가 발생했습니다. (500)';
        default:
          return `🚨 요청에 실패했습니다. (Status: ${status})`;
      }
    }
  }

  if (error instanceof Error && error.message === 'Network Error') {
    return '🚨 네트워크 오류가 발생했습니다. 인터넷 연결을 확인해주세요.';
  }

  return '🚨 알 수 없는 오류가 발생했습니다.';
};

// 최대 재시도 횟수
const MAX_RETRY = 3;

// 재시도 횟수 Map (URL별 독립적으로 재시도를 카운트하기 위함)
const retryCounts = new Map<string, number>();

// 응답 인터셉터 - 서버에서 내려주는 message를 그대로 사용하고, 네트워크 오류 시 자동 재시도 (최대 3번)
instance.interceptors.response.use(
  (res) => res,
  async (err) => {
    const config = err.config;

    // config 없다면 재시도 불가능
    if (!config || !config.url) {
      return Promise.reject(new Error(getErrorMessage(err)));
    }

    const currentRetry = retryCounts.get(config.url) || 0;

    // 네트워크 오류 또는 5xx 서버 오류일 때 재시도
    if (
      (err.message === 'Network Error' ||
        (err.response && err.response.status >= 500)) &&
      currentRetry < MAX_RETRY
    ) {
      retryCounts.set(config.url, currentRetry + 1);
      return instance(config); // 재시도
    }

    retryCounts.delete(config.url); // 카운터 Map 정리 (메모리 누수 방지)
    return Promise.reject(new Error(getErrorMessage(err)));
  },
);

export { instance };

```

위와같이 `baseURL`,`headers`,`timeout`,`interceptor`등에대한 설정을 미리 정의해놓은
인스턴스를 생성해놓은뒤 사용한다.

#### instance 사용예시

```jsx
import ErrorBoundary from './components/ErrorBoundary/ErrorBoundary';
import { useState } from 'react';
import { instance } from './apis/instance'; //instance import

function App() {
  const [message, setMessage] = useState('');

  const getMessage = async () => {
    try {
      const response = await instance.get('/api/data'); //instance 사용
      setMessage(response.data.message);
    } catch (err) {
      console.log(err);
    }
  };

  return (
    <div className='mt-50 flex h-300 flex-col items-center justify-center '>
     
      <button onClick={getMessage}>눌러</button>
      <ErrorBoundary fallback={<div>서버에서 받아오는중 에러발생</div>}>
        {message && <div>{message}</div>}
      </ErrorBoundary>

    </div>
  );
}

export default App;

```


