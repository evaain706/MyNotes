

## Error Boundary를 사용한 핸들링


React에서 하나의 부모컴포넌트 내부에 <Comp1/>과 <Comp2/>라는 두개의 자식컴포넌트가
렌더링 되는 구조가 있다고 생각해보자

```jsx
<Parent>
<>
	<Comp1/>
	<Comp2/>
	
</>
</Parent>
```

만약 두개의 자식컴포넌트중 하나라도 렌더링과정에서 에러가 발생한다면 해당에러는 
부모컴포넌트로 계속 전파가되며 결국은 앱 전체가 멈추는 상황이 초래된다.

*이런 상황을 방지하기위해 `ErrorBoundary`라는 것을 적용한다.

이름 그대로 에러에대한 경계를 만들어서 특정 컴포넌트에서 발생한 에러가 
상위로 전파되지않도록 막아주는 역할을 하도록한다.

---

## 작동방식


`ErrorBoundary`는 에러가 발생한 컴포넌트 트리 전체를 감싸서 에러를 포착한뒤, 에러가 잡히면 앱 전체가 멈추는 대신 미리 정의해둔 대체 UI(Fallback UI)를 보여준다

이때 `ErrorBoundary`를 어떻게 배치하느냐에 따라 렌더링 결과가 달라진다.


###  1. 두 컴포넌트를 함께 감싸는 경우

하나의 `ErrorBoundary`로 `<Comp1 />`과 `<Comp2 />`를 모두 감싸면, 둘 중 하나에서 에러가 발생했을 때 두 컴포넌트 모두 화면에서 사라지고 Fallback UI가 나타남


```jsx
<Parent>
  <ErrorBoundary fallback={<p>문제발생</p>}>
    <Comp1 />
    <Comp2 />
  </ErrorBoundary>
</Parent>
```

- **상황:** `<Comp1 />`에서 렌더링 에러가 발생
- **결과:** `ErrorBoundary`가 에러를 감지하고 자신의 자식컴포넌트 모두 렌더링하는 것을 중단
  대신, `fallback`으로 지정된 `<p>` 태그가 화면에 렌더링됨. 결과적으로 `<Comp2 />`는 정상임에도 불구하고 화면에 나타나지 않음


### 2. 각 컴포넌트를 개별적으로 감싸는경우

하나의 컴포넌트에서 에러발생시 다른컴포넌트의 렌더링에는 관여되지않도록 하고싶을경우,
각각의 컴포넌트에 `ErrorBoundary`로 감싸준다.

```jsx
 <Parent>
  <ErrorBoundary fallback={<p>Comp1 렌더링 실패</p>}>
    <Comp1 />
  </ErrorBoundary>
  <ErrorBoundary fallback={<p>Comp2 렌더링 실패</p>}>
    <Comp2 />
  </ErrorBoundary>
</Parent>
```
- **상황:** `<Comp1 />`에서 렌더링 에러가 발생

- **결과:**
- 첫 번째 `ErrorBoundary`가 에러를 감지하고, `<Comp1 />` 대신 FallBack UI를 렌더링
- 두 번째 `ErrorBoundary`와 그 자식인 `<Comp2 />`는 영향을 받지 않았으므로 정상 렌더링


---
### Error Boundary 구현

```jsx
import { Component, ErrorInfo, ReactNode } from "react";

// Props 타입 정의
interface Props {
  fallback: ReactNode; // 에러 발생 시 보여줄 UI
  children: ReactNode; // 에러를 감시할 자식 컴포넌트
}

// State 타입 정의
interface State {
  hasError: boolean; // 에러 발생 여부
}

class ErrorBoundary extends Component<Props, State> {
  // 1. 초기 state 설정
  public state: State = {
    hasError: false,
  };

  // 2. 렌더링 단계에서 에러가 발생했을 때 호출됨
  // 이 함수는 state를 업데이트하여 다음 렌더링에서 대체 UI를 보여주도록 하는역할
  public static getDerivedStateFromError(): State {
    return { hasError: true };
  }

  // 3. 자식 컴포넌트에서 에러가 발생했을 때 호출됨
  public componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error("Uncaught error:", error, errorInfo);
  }

  // 4. 렌더링
  public render() {
    // hasError state가 true이면, fallback UI를 렌더링
    if (this.state.hasError) {
      return this.props.fallback;
    }

    // 에러가 없으면 자식 컴포넌트를 그대로 렌더링
    return this.props.children;
  }
}

export default ErrorBoundary;
```

 `getDerivedStateFromError` `componentDidCatch`와 같은 생명주기 메서드를 사용하기위해
 클래스형 컴포넌트로 구현된다.

함수형 컴포넌트에서 `useEffect`와 같은 훅은 컴포넌트가 `렌더링을 마친뒤`에 실행되므로
적합하지 못하다.

---

### 사용예제


#### 에러 발생용 자식컴포넌트
```jsx
import { useState } from 'react';

const ErrorComp = ({ number }: { number: number }) => {
  const [hasError, setHasError] = useState(false);

  const handleErrorClick = () => {
    setHasError(true); // 버튼을 누르면 에러 상태로 변경
  };

  if (hasError) {
    throw new Error(`자식 컴포넌트${number}에서 에러 발생`);
  }
//가독성을 위해 tailwind 스타일링 정의 임의 삭제함
  return (
    <div className='flex h-100 w-100 items-center  bg-amber-200'>
      <div className='flex flex-col items-center justify-center gap-10'>
        <h2 className='text-2xl font-bold'>
          자식 컴포넌트{number}: 정상렌더링
        </h2>
        <button
          className='rounded-md bg-red-500 px-5 py-5 text-xl font-bold'
          onClick={handleErrorClick}
        >
          에러 발생시키기
        </button>
      </div>
    </div>
  );
};
export default ErrorComp;

```
테스트를 위한 에러발생용 자식컴포넌트를 간단하게 정의한다.
#### App.tsx 적용테스트
```jsx
import Dropdown from './components/DropDown';
import ErrorBoundary from './components/ErrorBoundary/ErrorBoundary';
import ErrorComp from './components/ErrorBoundary/ErrorComp';

function App() {
  return (
  //가독성을 위해 tailwind 스타일링 정의 임의 삭제함
    <div className=' justify-center gap-10 border-2 border-black'>
      <h2 className='text-4xl font-bold'>부모 컴포넌트</h2>
      <ErrorBoundary
        fallback={
          <div className='h-100 w-100 bg-red-500'>컴포넌트 1 에러발생</div>
        }
      >
        <ErrorComp number={1} />
      </ErrorBoundary>

      <ErrorBoundary
        fallback={
          <div className='h-100 w-100 bg-red-500'>컴포넌트 2 에러발생</div>
        }
        <div>
          <ErrorComp number={2} />
        </div>
      </ErrorBoundary>
    </div>
  );
}

export default App;

```
---

#### 결과

![[beforeSetError.png]]


![[afterSetError.png]]

###### 원하던대로 부모컴포넌트에게 전파되지않고 에러가 발생한 해당 컴포넌트에 FallBack UI가 나타나고있다.
