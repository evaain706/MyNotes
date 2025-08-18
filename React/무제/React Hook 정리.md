
## #useState

- 상태를 관리하기 위해 사용하는 훅  
- 비동기적으로 업데이트되며 여러 개의 state 호출은 batching(묶음 처리)되어 최신 값이 보장되지 않을 수 있음  
- 최신 값 보장이 필요한 경우 함수형 업데이트 `(prev) => ...` 를 사용하는 것이 안전  
- 상태 변경 시 UI가 리렌더링됨  
- 현재 상태값 `state`와 변경 함수 `setState`를 반환  
- 반드시 `setState`를 통해 변경해야 함  
``` jsx
const [state,setState] = useState(0);

.
.
.
<button onClick={() => {setState((prev)=> prev + 1)}}>버튼</button>
```

## #useEffect

- 컴포넌트가 렌더링될 때마다 부수효과(side effect)를 수행하기 위한 훅
    
- 대표적 용도: 데이터 페칭, 이벤트 등록/해제, DOM 조작 등
    
- 의존성 배열(deps)에 따라 실행 시점이 달라짐
    
    - deps 없음: 매 렌더링마다 실행
        
    - deps 빈 배열 `[]`: 마운트 시 한 번만 실행
        
    - deps에 값 존재: 해당 값이 변경될 때 실행
        
- `return`에서 cleanup 함수 작성 가능 (언마운트 시 실행)

``` jsx
useEffect(() => {
  const id = setInterval(() => console.log("타이머 실행"), 1000);
  return () => clearInterval(id); // 클린업 함수
}, []);
```

## #useContext

- Context API와 함께 사용하여 전역적으로 상태를 공유할 수 있는 훅
    
- props drilling 없이 데이터를 하위 컴포넌트에 전달 가능

``` jsx 
const ThemeContext = createContext("light");

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Child />
    </ThemeContext.Provider>
  );
}

function Child() {
  const theme = useContext(ThemeContext);
  return <p>현재 테마: {theme}</p>;
}
```

## #useReducer

- 복잡한 상태 관리 로직이 필요한 경우 `useState` 대신 사용하는 훅
    
- `state`와 `dispatch`를 반환하며, reducer 함수에 따라 상태를 업데이트
    
- Redux와 유사한 패턴

``` jsx
function reducer(state, action) {
  switch (action.type) {
    case "increment": return { count: state.count + 1 };
    case "decrement": return { count: state.count - 1 };
    default: return state;
  }
}

const [state, dispatch] = useReducer(reducer, { count: 0 });

<button onClick={() => dispatch({ type: "increment" })}>+1</button>

```

## #useRef

- DOM 요소나 변수를 직접 참조할 때 사용하는 훅
    
- 값이 변경되어도 컴포넌트를 리렌더링하지 않음
    
- 주로 포커스 제어, 애니메이션, 이전 값 기억 등에 활용

``` jsx

const inputRef = useRef(null);

const focusInput = () => {
  inputRef.current.focus();
};

<input ref={inputRef} />
<button onClick={focusInput}>포커스</button>
```

## #useMemo

- 연산 비용이 큰 계산의 결과를 메모이제이션하여 불필요한 재계산 방지
    
- 의존성 배열의 값이 변경될 때만 다시 계산

``` jsx

const getMaxValue = useMemo(() => CalNum(num), [num]);
```


## #useCallback

- 함수를 메모이제이션하여 불필요한 재생성 방지
    
- 자식 컴포넌트에 props로 함수를 넘길 때 성능 최적화에 유용

``` jsx

const handleClick = useCallback(() => {
  console.log("클릭됨");
}, []);
```