

## 메모이제이션이란?

계산 결과를 메모리에 저장해두고 동일한 계산이 요청되었을경우, 다시 계산하지않고 메모리에 
저장되었던 계산결과를 반환하는 최적화 기법

`useMemo` 와 `useCallback` 모두 성능 최적화를 위한 **메모이제이션** 을 해주는 Hook이지만
**무엇을 메모이제이션 하느냐** 가 차이점이라할수있다.

- **`useMemo`**: 복잡한 연산의 **결과 값(value)**을 메모이제이션

- **`useCallback`**: 함수의 **정의 자체(function)**를 메모이제이션

---

## 왜필요할까?

React 컴포넌트는 자신의`state`나 부모로보터 건네받은 `props`가 변경될때마다
`리렌더링`이 일어남

`리렌더링` 이 일어날때마다 내부의 변수선언,함수등의 모든 코드가 다시 실행된다.

``` jsx
// 부모 컴포넌트
function App() {
  const [count, setCount] = useState(0);

  // App 컴포넌트가 리렌더링 될 때마다 매번 새로 생성되는 함수
  const handleLog = () => {
    console.log("버튼 클릭!");
  };

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        카운트 증가: {count}
      </button>
      {/* Child 컴포넌트에 props로 handleLog 함수를 전달 */}
      <ChildComponent onLog={handleLog} />
    </div>
  );
}

// 자식 컴포넌트
function ChildComponent({ onLog }) {
  console.log("자식 컴포넌트가 렌더링되었습니다.");
  return <button onClick={onLog}>로그 기록</button>;
}
```
카운트 증가 버튼을 누를때마다 count`state`가 변경되며 리렌더링이 발생하는데,
`handleLog`함수도 매번 새로 생성이되게된다.

자식 컴포넌트는 count`state`와는 아무런 연관이없는데도 불구하고 
`handleLog`함수가 새로 생성되기때문에 같이 리렌더링이 발생하게된다.

이런 문제를 막기위해 `useMemo`와 `useCallback` 이라는 최적화를 위한 훅이 필요하다.

---

## useCallback -> 함수를 메모이제이션

`useCallback`은 **함수의 정의 자체**를 메모이제이션한다.
자식 컴포넌트에 `props`로 함수를 전달할때, 불필요한 리렌더링을 방지할수있다.

```jsx
const memoizedCallback = useCallback(() => {
  // 실행할 로직
  exampleFunc(dep1, dep2);
}, [dep1, dep2]); // dep1 또는 dep2가 변경될 때만 함수가 새로 생성됨
```


---

## useMemo -> 값을 메모이제이션

`useMemo` 는 계산비용이 큰 **결과값**을 저장하고, 의존성배열의 값이 변경될때에만 
해당 연산을 다시 수행한다.

```jsx
const memoizedValue = useMemo(() => {
  // 연산 비용이 큰 작업을 수행
  return ExampleValue(dep1, dep2);
}, [dep1, dep2]); // dep1 또는 dep2가 변경될 때만 함수가 다시 실행됨
```

---

## 상단의 코드를 useMemo, useCallback을 쓴다면?


```jsx
import React, { useState, useCallback } from 'react';

// React.memo: props가 변경되지 않으면 리렌더링을 스킵하는 고차 컴포넌트
const ChildComponent = React.memo(({ onLog }: { onLog: () => void }) => {
  console.log("자식 컴포넌트가 렌더링되었습니다");
  return <button onClick={onLog}>로그 기록</button>;
});

function App() {
  const [count, setCount] = useState(0);

  // useCallback으로 함수 정의를 메모이제이션
  // 의존성 배열이 비어었기때문에 컴포넌트의 생애주기 동안 한번만생성
  const handleLog = useCallback(() => {
    console.log("버튼 클릭!");
  }, []); 

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        카운트 증가: {count}
      </button>
      <ChildComponent onLog={handleLog} />
    </div>
  );
}
```

- `useCallback` 덕분에 App컴포넌트에서 `count`값이 변경되어도 `handleLog`함수는 재생성X
- `React.memo` - 컴포넌트를 `React.memo`로 감싸면`props`에변경이 없을경우 리렌더링을 스킵

---


## 주의할점

`useMemo`와 `useCallback`은 난사한다고 좋은것이 아니다.
이 훅을 사용하는데도 비용이 발생하기때문에 **필요한부분**에만 적절하게 사용해야한다.

- 성능저하가 실제로 눈에 들어나는 부분에만
- 계산비용이 정말 비쌀때(복잡한 반복문등)
