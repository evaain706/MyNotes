
## Zustand란?

React 상태관리 라이브러리로 Redux등 다른 상태관리 라이브러리와 비교하였을때 손쉬운 사용이 가능하다.

---

#### 매우 작은 보일러 플레이트

Redux와는 다르게 액션,디스패치,리듀서,액션생성자 들이 필요없이 
`store`라는 하나의 상태저장소를 생성하는것으로 충분

```js
import { create } from 'zustand';

// 스토어의 상태와 상태를 변경할 함수의 타입을 정의
interface CountState {
  count: number; 
  increaseCount: () => void; 
  resetCount: () => void;   
}

// 'create' 함수로 스토어를 생성
export const useCountStore = create<CountState>((set) => ({
  // 초기 상태정의
  count: 0,
  
  // 상태를 변경하는 함수들을 정의
  // 'set' 함수는 상태를 업데이트하는 유일한 방법임
  increaseCount: () => set((state) => ({ count: state.count + 1 })),
  resetCount: () => set({ count: 0 }),
}));

```

#### 사용 예시

```jsx
import { useCountStore } from '../../store/CountStore';

const Count = () => {
  const { count, increaseCount, resetCount } = useCountStore();

  return (
    <div>
      <h2>현재 카운트: {count}</h2>

      <button onClick={increaseCount}>카운트증가</button>

      <button onClick={resetCount}>카운트초기화 </button>
    </div>
  );
};

export default Count;

```

---

#### 성능 최적화에 효율적

리렌더링에 있어서 store의 state를 구독하고있는 컴포넌트 그리고
그 컴포넌트가 사용하고있는 상태가 **변경**될때에만 리렌더링이 발생한다.


```jsx
import { useCountStore } from '../../store/CountStore';

const CountDisplay = () => {
// count를 선택해서 가져옴
// count가 변경될때에만 컴포넌트가 리렌더링
  const count = useCountStore((state) => state.count);

  return (
    <div>
      <h2>현재 카운트: {count}</h2>
    </div>
  );
};
export default CountDisplay;

```


```jsx
import { useCountStore } from '../../store/CountStore';

const CountControl = () => {
  // 증가,초기화 함수를 선택해서 가져옴
  // count값이 변해도 이 컴포넌트에서는 count state를 구독하지않기때문에 리렌더링X
  const increaseCount = useCountStore((state) => state.increaseCount);
   const resetCount = useCountStore((state) => state.resetCount);

  return (
    <div>
      <button onClick={increaseCount}>카운트값 증가</button>
      <button onClick={resetCount}>카운트값 초기화</button>
    </div>
  );
};
export default CountControl;
```

`Context API`와 비교하였을때, `Context API`는 Provider내부의 어떤값이든 변경되면 
그 Context를 구독중인 모든 컴포넌트가 리렌더링 되지만 Zustand는 다르다.


`const count = useCountStore((state) => state.count);` 와 같은방식으로 가져오는것을
**`Selector(셀렉터)`** 라고 하는데 이렇게 특정값만 추출하여 가져온뒤
이 셀렉터로 가져온값이 이전과 다를때에만 리렌더링이 발생하여 성능을 최적화한다.


---

#### Provider가 필요없음

`Context API`나 `Redux`는 `<Provider>`컴포넌트로 값을 전달하고자하는 컴포넌트들을 감싸줘야하지만,
`Zustand`는 감싸줄 필요없이 원하는 컴포넌트에서 `store`를통해 불러와서 사용이 가능하다.

---

#### 미들웨어

다양한 미들웨어를 간단하게 사용가능하다.

