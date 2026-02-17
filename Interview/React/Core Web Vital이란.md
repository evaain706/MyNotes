
#### 실제 사용자 체감 성능을 측정하기위한 Google의 핵심 성능 지표 3가지로 구성



## 1.LCP

`뷰포트내에서 가장 큰요소가 화면에 Paint 될때까지 걸리는시간` 
보통 `img` `video` `bg` 큰 `text-block` 

### 프론트엔드 관점 원인

- render blocking resource (CSS, sync JS)

- 느린 TTFB -> 브라우저가 페이지를 요청한 시점과 서버로부터 첫 번째 정보 바이트를 수신한 시점 사이의 시간

- 이미지 최적화 미흡

- hydration 지연


## 개선법? 

- critical CSS 분리 / preload

- `next/image` 또는 responsive image를 사용한 이미지 최적화

- 서버 응답 시간 단축 (SSR 캐싱, CDN)

- above-the-fold JS 최소화 
  -> `사용자가 페이지 진입 직후 스크롤 없이 보이는 영역을 렌더링하는 데 필요한 최소한의 JS`


---


## 2.INP




## 3.CLS
