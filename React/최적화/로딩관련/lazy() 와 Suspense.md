
## lazy 정의

React.lazy는 컴포넌트가 처음 렌더링 될때까지 해당 컴포넌트의 코드를 로딩하는것을 지연시키며
필요할때 동적으로 불러오게할수있도록 하는 함수이다.

당장 화면에 보일필요가없는 컴포넌트를 나중에 불러오도록하여 초기로딩속도를 높일수있다.

---

## lazy 사용법

```jsx
import {lazy} from 'react';

//이렇게 선언된 lazyComp는 실제로 필요해지기전까지는 로드되지않음
const lazyComp = lazy(() => import('./lazyComp.tsx'));
```

- lazy는 import()를 사용하는 함수를 인자로 받음
- import()는 `Promise`를 반환해야하며, 그 `Promise`는 `export default`로 내보내진
  컴포넌트를 `resolve(Promise 객체에서의 요청성공을 의미)`해야한다.

---

## lazy를 Suspense와 같이 사용해야하는 이유

 lazy로 불러오게되는 컴포넌트는 Promise를 반환해야하기때문에
코드를 비동기적으로 가져오게된다. 그 사이의 시간동안 사용자에게는 
깨지거나 비어있는 화면이 나타나게될수있는데,
이것을 위해서 `lazy`로 불러온 컴포넌트는 *반드시* `Suspense` 의 자식으로 렌더링되어야한다.

`Suspense`는 `lazy`컴포넌트가 로드되어 렌더되는 시간동안 `FallBackUI`를 보여주는 역할을한다

---
## Suspense란?

컴포넌트가 아직 렌더링될수없는 상태일때(데이터 로딩등)
`FallBackUI`를 선언적으로 보여줄수있도록 해주는 기능

#### Suspense 사용법


```jsx
import {Suspense} from 'react';

<Suspense fallback={<p>로딩중...</p>}>
	 <예시컴포넌트/>
 </Suspense>
```

- `fallback={}` 에 대신 화면에 표시될 요소를 넣는다.
- 내부에있는 컴포넌트가 로딩을 유발할시 지정해둔 `FallBackUI`가 나타나고 로딩이 완료되는
  시점에 자식컴포넌트가 렌더링된다.


### Suspense의 사용처

1. lazy와 함께 사용하여 코드스플리팅을 통한 초기페이지 로딩속도 최적화
2. 데이터페칭(Tanstack Query등과 같은 라이브러리등과도 호환이좋음)


---

## lazy + Suspense 예시


```jsx
// App.tsx
import React, { Suspense,lazy } from 'react';
import Spinner from './components/Spinner'; // 로딩 중에 보여줄 fallback스피너

// lazyComp 컴포넌트를 초기 로딩에서 제외하고 싶다고 가정함
const lazyComp = lazy(() => import('./components/lazyComp'));

function App() {
  return (
    <div>
      <h1>메인 페이지</h1>
      <p>이쪽내용은 즉시 렌더링됨</p>
      
      {/* lazyComp 코드를 불러오는 동안 스피너 컴포넌트 렌더링 */}
      <Suspense fallback={<Spinner />}>
        {/* lazyComp컴포넌트가 실제로 렌더링되어야 할 시점에 로딩시작 */}
        <ChatWindow />
      </Suspense>
      
    </div>
  );
}
```
