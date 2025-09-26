

#### useEffect란?

**React 컴포넌트에서 '부수효과(Side Effect)'를 수행하기위해 사용되는 `Hook`**

- 부수효과란?
  -> 데이터페칭,수동DOM조작등 렌더링 결과에 직접적인 영향을 주지않는 작업

---

#### 기본구조

```jsx
import {useState,useEffcet} from 'react';

function ExampleComp() {

const [count,setCount] = useState(0);

  //기본형태
  useEffect(() => {
    //1.effect함수
   document.title = `${count}번 클릭했습니다`;
   console.log('컴포넌트 렌더링됨');
  
   // 클린업함수(선택사항) 
   return () => {
    console.log('컴포넌트가 언마운트되거나 다음 useEffect가 실행되기전 호출됨)
   };
  
    //2.의존성배열
  },[count])


return (
 <div>
	 <p>{count}번 클릭했습니다</p>
	 <button onClick={() => setCount(count + 1)}>
	   눌러
	 </button>
 </div>
);
}

export default ExampleComp;
```

**1. Effect함수 : 컴포넌트가 `렌더링된후` 실행될 코드를 담는 함수**

**2. 의존성배열 : `useEffect`가 언제실행될지 결정하는 배열**

 ***이 배열안에 포함된 값이 변경될때마다 effect함수가 다시 실행됨***
   
 의존성배열을 넣지않으면 리렌더링될때마다 실행됨
   ``` jsx
   useEffect(() => {
  // 컴포넌트가 렌더링될 때마다 실행
  console.log('매번 렌더링될 때마다 실행');
});
   ```
   
의존성 배열에 `[]빈배열` 을 넣을경우 처음화면에 마운트될때 딱 한번만 실행됨

```jsx
useEffect(() => {
  // 컴포넌트가 처음 마운트될 때 딱 한 번만 실행
  console.log('처음 한 번만 실행');
}, []);
```
   
##### 의존성 배열?

React는 렌더링이 렌더링이 발생한후 이전 시점의 의존성배열의 값과 현재시점의 값을 비교,
의존성 배열의 값중 하나라도 다르다면 effect함수를 재실행
-> 불필요한 함수의 재실행을 의존성배열을 통해 방지할수있음 -> 성능최적화

---
##### 클린업 함수?

`useEffect` 내에서 함수를 `반환(return)` 하면 이 함수를 `클린업함수(CleanUp function)`이라함

1. **컴포넌트가 화면에서 언마운트될때**
2. **다음 effect함수가 실행되기 직전(의존성 배열의 값이 변경되어 리렌더링될때)
  의 두가지 시점에서 실행됨

```jsx
useEffect(() => {
  // 1초마다 count를 1씩 증가시키는 타이머 설정
  const intervalId = setInterval(() => {
    console.log('시간재는중');
  }, 1000);

  // 클린업 함수
  return () => {
    // 컴포넌트가 언마운트되거나, 의존성이 변경되어 effect가 다시 실행되기 전에
    // 기존의 타이머를 제거
    clearInterval(intervalId);
    console.log('클린업 실행됨');
  };
}, []); 
```

 - 이벤트 리스너 제거
 - 타이머, 인터벌 정리
 - 비동기 작업 정리
   와같은 컴포넌트가 언마운트되더라도 백그라운드에 누적실행될만한 요소들을
   클린업함수를 통해 정리하여 메모리누수를 방지하고,
   이전요소들을 정리하여 예기치않는 동작을 막는 역할을한다.

---

### `useEffect` 가 자주쓰이는부분

- API 데이터 페칭
- 이벤트 리스너 등록 및 해제(`resize` `scroll` 등)
- 타이머 설정 및 해제(`setTimeout` `setInterval`)
- 외부 라이브러리 연동
- 특정 `props` 나 `state` 의 값변경에 따른 부가작업


