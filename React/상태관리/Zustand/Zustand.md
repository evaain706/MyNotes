
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

### persist

상태를 `localStorage`나`sessionStorage`에 자동으로 저장한뒤 새로고침할때 그값을 복원
-> 다크모드나 사용자정보유지등에 유용


```jsx
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

export type User = {
  id: number;
  email: string;
  nickname: string;
  teamId: string;
  createdAt: string;
  updatedAt: string;
  image: string | null;
};

type UserStore = {
  user: User | null;
  hasHydrated: boolean;
  setUser: (user: User | null) => void;
  clearUser: () => void;
  setHasHydrated: (state: boolean) => void;
};

const useUserStore = create<UserStore>()(
  devtools(
    persist(
      (set) => ({
        user: null,
        hasHydrated: false,
        setUser: (user: User | null) => set({ user }),
        clearUser: () => set({ user: null }),
        setHasHydrated: (state: boolean) => set({ hasHydrated: state }),
      }),
      {
        name: 'user-storage',
        onRehydrateStorage: () => (state) => {
          state?.setHasHydrated(true);
        },
      },
    ),
  ),
);

export default useUserStore;

```
위 코드를 보면 `persist`를 사용해서 state를 `user-storage`라는 이름의 로컬스토리지에 
저장하고있음을 볼수있다.

또한 `onRehydratedStorage`라는것도 사용하고있는데,
이는 **스토리지에 저장한 상태를 불러와 스토어에 적용(Rehydration)되었을때** 특정작업을 실행하도록 해주는 역할이다. -> '새로고침이 된뒤 `로컬스토리지에서` 값을 가져오는것을 감지'와 비슷

위 코드에서의 예시를 들자면 로그인이 된상태에서 헤더에 유저정보가 뜨도록 되어있다한다면
새로고침시 로컬스토리지에서 값을 불러오는 동안 헤더에 유저정보가 아닌 비로그인UI가
나타나는 `플리커링`현상이 발생하는데, 이를 해결하기위해

값을 가져오는 `Hydration`과정이 완료되면 그때 값을 `true`로 바꾸도록하여

사용하는 컴포넌트에서 

```jsx
  //플리커링 방지를 위해 하이드레이션이 완료되기 전에는 이부분을 렌더링
  if (!hasHydrated) {
    return (
      <header className='mt-7 h-[5rem] animate-pulse rounded-2xl bg-[#101318] text-white md:h-[7rem]'>
        <div className='flex h-full items-center justify-between px-[4rem] md:px-[8rem]'>
          <div className='flex space-x-6' />
        </div>
      </header>
    );
  }
```
이런식으로 로그인버튼이 아닌 대체UI가 나타나도록하여 플리커링을 방지하는데 사용되었다.

---

#### subscribe

store의 상태의 변경을 `구독(감지)`하여 상태가 변경될때마다 특정한 콜백함수를 실행하도록해주는 기능

React컴포넌트의 렌더링과 관계없이 상태변경에 반응하는 로직을 구현하고싶을때 사용

- **로깅(Logging)**: 상태가 변경될 때마다 콘솔에 로그를 남기거나 분석 도구로 이벤트를 보낼 때

- **브라우저 API 연동**: 상태 값에 따라 `document.ㅇㅇ`을 바꾸거나, `localStorage`에 수동으로 값을 저장할 때

- **외부 라이브러리 연동**: React와 직접적인 관련이 없는 다른 라이브러리에 변경된 상태를 전달해야 할 때

  등에 사용된다.

```jsx
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface ThemeState {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const handleUpdateClass = (theme: 'light' | 'dark') => {
  const root = document.documentElement;
  if (theme === 'dark') {
    root.classList.add('dark');
  } else {
    root.classList.remove('dark');
  }
};

export const useThemeStore = create<ThemeState>()(
  persist(
    (set) => ({
      theme: 'light',
      toggleTheme: () =>
        set((state) => ({
          theme: state.theme === 'light' ? 'dark' : 'light',
        })),
    }),
    {
      name: 'theme-storage',

      onRehydrateStorage: () => (state) => {
        if (state) {
          handleUpdateClass(state.theme);
        }
      },
    },
  ),
);

useThemeStore.subscribe((state) => {
  handleUpdateClass(state.theme);
});

```
예시 -> 다크모드를 위한 `store`에서  `useThemeStore`를 구독한뒤 값이변경될때마다
루트 클래스에 `dark`를 추가하거나 제거하는 콜백함수를 호출하는데 사용


---





