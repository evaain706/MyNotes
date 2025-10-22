

## OAuth란?

**인터넷 사용자들이 비밀번호를 제공하지 않고 다른 웹사이트,서비스 상의 자신들의 정보에 대해 다른 서비스가 접근할수있도록 권한을 부여하는 `접근위임` 을 통한 개방형 표준 프로토콜**

---

## 구현방식

#### 사용된것들

- 프론트엔드(React)
- 백엔드(Express)
- DB(MongoDB)
---

## 1.로그인시작단계

```jsx
const OauthLogin = () => {
  const KAKAO_REST_API_KEY = import.meta.env.VITE_KAKAO_REST_API_KEY;
  const KAKAO_REDIRECT_URI = import.meta.env.VITE_KAKAO_REDIRECT_URI;
  const KAKAO_AUTH_URL = `https://kauth.kakao.com/oauth/authorize?response_type=code&client_id=${KAKAO_REST_API_KEY}&redirect_uri=${KAKAO_REDIRECT_URI}`;

  function handleKakaoLogin() {
    window.location.href = KAKAO_AUTH_URL;
  }

  return (
    <div>
      <button onClick={handleKakaoLogin}>카카오 로그인</button>
    </div>
  );
};

export default OauthLogin;

```

 1. 사용자가 `카카오 로그인` 버튼을 클릭
 2. `handleKakaoLogin()` 에 위해 인증URL로 페이지를 이동

   인증URL(`KAKAO_AUTH_URL`)에는 `REST_API_KEY`와 `REDIRECT_URI`라는 두가지 쿼리가 존재
  - `REST_API_KEY` -> 카카오에서 발급받은 REST API 키
  - `REDIRECT_URI` -> 카카오에서 인증이 성공하면 사용자를 어디로 다시 돌려보낼지의 주소


---

## 2.콜백처리

```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import './index.css';
import Main from './page/Main';
import Layout from './components/Layout';
import OptionSelect from './page/OptionSelect';
import Quiz from './page/Quiz';
import KakaoCallback from './page/KakaoCallback';

function App() {
  const router = createBrowserRouter([
    {
      path: '/',
      element: <Main />,
    },
    {
      path: '/oauth',
      element: <KakaoCallback />,
    },

    {
      element: <Layout />,
      children: [
        {
          path: '/select',
          element: <OptionSelect />,
        },
        {
          path: '/quiz',
          element: <Quiz />,
        },
      ],
    },
  ]);

  return <RouterProvider router={router} />;
}

export default App;

```


```jsx
import React, { useEffect } from 'react';
import { useLocation, useNavigate } from 'react-router-dom';

import useUserStore from '../store/useUserStore';
import { instance } from '../apis/instance';
import KakaoLoading from '../components/KakaoLoading';

const KakaoCallback = () => {
  const location = useLocation();
  const navigate = useNavigate();
  const { setUser } = useUserStore();

  useEffect(() => {
    const code = new URLSearchParams(location.search).get('code');

    if (code) {
      const sendCodeToBackend = async () => {
        try {
          const response = await instance.post('/api/auth/kakao', {
            code: code,
          });

          const { user } = response.data;

          setUser(user);

          navigate('/');
        } catch (error) {
          console.error('카카오 로그인 에러:', error);

          navigate('/');
        }
      };

      sendCodeToBackend();
    }
  }, [location, navigate]);

  return <KakaoLoading />;
};

export default KakaoCallback;

```

유저가 카카오사이트에서 성공적으로 로그인하면 로그인시작단계에서 지정한
`REDIRECT_URI` 로 돌아오게됨

이 페이지에 렌더링되는것이 `KakaoCallback`컴포넌트

**`KakaoCallback`컴포넌트의 역할**
- 리다이렉션된 주소에는 카카오가 발급한 **일회성인가코드**가 쿼리파라미터에 포함됨
- useEffect훅 내부에서 쿼리문자열(`code?=example`)을 가져온뒤 `code`를 추출
- 추출한`code`를 **백엔드 서버 API** `/api/auth/kakao`로 전송


---

## 3. 토큰관리 및 인증유지

```jsx
import axios from 'axios';
import type { AxiosInstance } from 'axios';

import { API_HEADERS, API_TIMEOUT } from '../constants/apiConstants';

const privateInstance: AxiosInstance = axios.create({
  baseURL: 'http://localhost:5000',
  timeout: API_TIMEOUT,
  headers: API_HEADERS.JSON,
  withCredentials: true,
});

privateInstance.interceptors.response.use(
  (res) => res,
  async (error) => {
    const originalRequest = error.config;

    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        await privateInstance.post('/api/auth/refresh', {});

        return privateInstance(originalRequest);
      } catch (refreshError) {
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  },
);

export { privateInstance };

```

백엔드는 `code`를 받아 그것을 통해 카카오와 통신한뒤, 사용자정보(id,nickname등)을 획득
그다음 신규유저일경우 유저등록과 같은 로직들을 거친뒤
`accessToken / refreshToken`을 발급하여 **쿠키**에 담아서 응답하도록함.

