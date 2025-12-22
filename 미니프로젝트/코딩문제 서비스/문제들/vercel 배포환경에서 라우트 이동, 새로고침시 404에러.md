



## 왜?

Vercel은 서버 기준 라우트를 찾는데, React 같은 SPA는 라우트를 브라우저에서만 처리하기 때문


## 해결법

`vercel.json`에서 모든경로에대해 `index.html` 을 반환하도록함

```js
{

  "rewrites": [{ "source": "/(.*)", "destination": "/" }]

}
```