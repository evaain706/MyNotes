
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




