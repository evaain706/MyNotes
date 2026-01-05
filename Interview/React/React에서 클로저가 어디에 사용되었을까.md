

### React의 원리와 엮어서

리액트의 근간인 `컴포넌트` 는 `함수` 인데, 리렌더링될 때마다 함수가 `다시 호출` 된다.  
그런데도 `state` 값은 이전 값을 기억하고 있는 것처럼 동작한다.  
이게 처음엔 마치 “함수가 안 죽는 것처럼(실행컨텍스트에 남아있는것처럼)” 느껴질 수 있다.

하지만 실제로 컴포넌트 함수 자체는 매번 **완전히 새로 실행**된다.  
그럼에도 불구하고 state가 유지되는 이유는,  
**state 값은 `컴포넌트 함수 안에 저장`되는 게 아니라 `React 내부에 저장`되기 때문**이다.

`useState`를 호출하면, React는 해당 컴포넌트에 연결된 **state 저장소**를 만들고  
그 값을 반환할 때 현재 렌더링된 함수 스코프 안으로 **클로저 형태로 주입**한다.


- 컴포넌트 함수는 매번 새로 실행되지만
- `useState`가 반환한 값과 setter는
- React가 관리하는 state를 **참조하는 클로저**를 들고 있다


이 때문에 함수가 다시 호출되어도  
이전 렌더링에서 생성된 값이 아니라,  
**React가 기억하고 있는 최신 state를 바라보게 된다.**

---


```js
function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(count + 1);
  };

  return <button onClick={handleClick}>{count}</button>;
}

```

여기서 `handleClick` 함수는 렌더링 시점마다 새로 만들어지지만,  
그 안에서 참조하는 `count` 는  
**해당 렌더링 시점의 state를 캡처한 클로저**다.

그래서 중요한 특징이 하나 생긴다

` 클로저는 생성된 시점의 값을 기억한다.`

이게 바로 React에서 자주 마주치는  
**stale closure(오래된 클로저)** 문제의 근원이다.

``` js
setCount(count + 1);

```
이 코드는 “항상 최신 count”를 쓰는 것처럼 보이지만,  
실제로는 **이 렌더링 시점의 count**를 기준으로 동작한다.

그래서 React는 이를 해결하기 위해  
**함수형 업데이트**를 제공한다.

``` js
setCount(prev => prev + 1);

```

이 방식은 클로저에 의존하지 않고,  
React가 보장하는 **가장 최신 state**를 인자로 받아 처리한다.

정리하면 React에서의 클로저는

- 컴포넌트 함수가 다시 호출되더라도
- state가 유지되는 것처럼 보이게 해주는 핵심 메커니즘이자
- 동시에 stale state 버그를 만들 수 있는 원인이기도 하다

---

## 오래된 클로저문제?


`useEffect`에서 의존성 배열을 잘못 설정했을 때 발생하는 버그의 원인이 바로 클로저임

``` js
import React, { useState, useEffect } from 'react';

const Timer = () => {
  const [seconds, setSeconds] = useState<number>(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      // [문제 발생 지점]
      // 이 콜백 함수는 첫 렌더링(Mount) 시점에 딱 한 번 생성
      // 그 당시의 seconds는 0
      // 따라서 이 함수는 영원히 "console.log(0 + 1)"만 반복
      console.log(seconds + 1); 
      setSeconds(seconds + 1); // 화면의 숫자가 1에서 멈춤
    }, 1000);

    return () => clearInterval(intervalId);
  }, []); // 의존성 배열이 비어있음 -> 클로저가 갱신되지 않음

  return <div>{seconds}</div>;
};
```

- `useEffect`의 의존성 배열이 `[]`이므로 이 이펙트는 처음에만 실행됩니다.

- `setInterval` 내부의 콜백 함수는 **첫 렌더링 시점의 `seconds` (값: 0)**를 클로저로 캡처
- 
- 1초마다 실행되지만, 이 함수가 기억하는 `seconds`는 계속 0

- **해결책:** 의존성 배열에 `[seconds]`를 넣거나, 함수형 업데이트 `setSeconds(prev => prev + 1)`를 사용해야 함


## useState 내부구현 원리
``` js
// React 내부 동작의 단순화된 의사 코드 (Pseudo-code)
const React = (function() {
  let _val; // 이 변수가 클로저 공간에 숨겨져있음

  function useState(initVal) {
    const state = _val || initVal; // _val이 있으면 그것을 쓰고, 없으면 초기값
    
    const setState = (newVal) => {
      _val = newVal; // 클로저 공간에 있는 _val을 업데이트
      // 이후 리렌더링을 트리거함
    };
    
    return [state, setState];
  }

  return { useState };
})();
```

- `useState`가 호출되고 실행이 끝나도, `_val`이라는 변수는 사라지지 않고 메모리에 남아있음
- 
- 이것이 컴포넌트가 리렌더링(함수 재호출)되어도 상태값이 초기화되지 않고 **유지되는 이유**


---


- **스냅샷:** React 컴포넌트 내부의 모든 함수(핸들러, Effect 등)는 **"자신이 생성된 렌더링 시점"**의 변수들만 볼 수 있음 -> 클로저의 특성
  -> React가 메모리에 별도로 보관해서 값이 유지

- **의존성 배열:** `useEffect`나 `useCallback`의 의존성 배열(`deps`)은 **"언제 이 클로저를 깨고 새로운 값을 기억하는 새 함수를 만들 것인가"**를 결정하는 역할을 함

- **함수형 업데이트:** `setCount(prev => prev + 1)`을 쓰면 클로저로 캡처된 옛날 값(`count`) 대신, React 내부의 최신 상태값(`prev`)을 인자로 받아올 수 있어 오래된 클로저 문제를 피할 수 있음