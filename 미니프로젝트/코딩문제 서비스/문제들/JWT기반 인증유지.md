
# 상황

백엔드에서 **accessToken / refreshToken** 모두 **쿠키**(`httpOnly`, `secure`, `SameSite`)로 발급
- 인증 유지와 관리 편리
- accessToken 만료 시 재발급 로직 간편

프론트에서 로그인 시 쿠키에 토큰 저장
**로그인 상태가 필요한 API 요청** 전용 Axios 인스턴스(`privateInstance`)를 별도로 생성


---

# 목표

- 로그인 상태가 필요한 요청에서 **accessToken이 없거나 만료(401)** 발생 시
- 자동으로 **refreshToken을 사용해 accessToken 재발급**
- 원래 요청을 **토큰 갱신 후 자동 재시도**


---

#  구현 방법

###  Axios 인스턴스 생성

```js
import axios from 'axios';
import { API_HEADERS, API_TIMEOUT } from '../constants/apiConstants';

const privateInstance = axios.create({
  baseURL: 'http://localhost:5000',
  timeout: API_TIMEOUT,
  headers: API_HEADERS.JSON,
  withCredentials: true, // 쿠키 전송 필수
});
```
###  Axios 인터셉터 설정

```js
privateInstance.interceptors.response.use(
  (response) => response, // 정상 응답은 그대로 반환
  async (error) => {
    const originalRequest = error.config;

    // 401 에러 & 재시도하지 않은 경우
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        // refreshToken 쿠키를 사용해 accessToken 재발급 요청
        await axios.post(
          '/api/auth/refresh',
          {},
          { withCredentials: true } // 쿠키 전송 필수
        );

        // 재발급 후 원래 하려던 요청 재시도
        return privateInstance(originalRequest);
      } catch (refreshError) {
        return Promise.reject(refreshError); // refresh 실패 시 에러 리턴
      }
    }

    return Promise.reject(error); // 나머지에러는 그대로 리턴
  }
);

```

---

# 키워드

### withCredentials란?

**서로 다른 도메인(Cross-Origin) 간의 요청에서, 쿠키나 인증 헤더와 같은 인증 정보(Credentials)를 자동으로 포함하여 전송할 수 있도록 해주는 클라이언트 측 옵션**

```js
axios.get('http://localhost:5000/api/user', { withCredentials: true });
```

---

### **동일출처정책(Same-Origin-Policy,SOP)**

기본적으로 브라우저는 보안을 위해 한 출처(예: `http://localhost:5173`)에서 다른 출처(예: `http://localhost:5000`)로 API를 요청할 때, 쿠키와 같은 민감한 정보를 자동으로 안보냄
-> 보안적인 이유때문임

- **동일 출처 (Same-Origin)**: 프로토콜, 호스트, 포트가 모두 같은 경우
   (예시: `내 사이트` -> `내 사이트 API`)

- **교차 출처 (Cross-Origin)**: 셋 중 하나라도 다른 경우
  (예시: `localhost:5173` -> `localhost:5000`)

---

#### CORS (Cross-Origin Resource Sharing / 교차 출처 리소스 공유)

- **정의**: SOP의 제한을 안전하게 우회할 수 있도록 브라우저와 서버가 약속하는 표준 메커니즘

- **원리**: 서버가 응답 헤더를 통해 "이 출처에서 온 요청은 안전하다"라고 브라우저에게 알려주는 방식
