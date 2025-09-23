
## 언제?

React에서 배열형태의 데이터를 사용해서 리스트형태의 UI를 화면에 그리고싶을때 `map`을사용
서버에서 받아온 배열형태의 데이터를 받아와 화면에 렌더링할때 주로사용됨

ex)상품목록,댓글,게시글 등

---


## 예시

#### 예시 express api 정의

```js
app.get('/api/items', (req, res) => {
 
  const items = []; 

  for (let i = 1; i <= 20; i++) {
    items.push({
      id: i, //  고유 ID
      name: `아이템 ${i}`, //  이름
      description: `이것은 서버에서 보낸 ${i}번째 아이템에 대한 설명입니다.`, //  설명
    });
  }
  res.json(items);
});
```
`id` `name` `description` 이라는 3개의 필드가 있는 데이터들을 20개 생성하여 배열에 담아 
응답을 보낸다.

---

#### 프론트 코드

```jsx
import ErrorBoundary from './components/ErrorBoundary/ErrorBoundary';
import { useState } from 'react';
import { instance } from './apis/instance';


interface Item {
  id: string;
  name: string;
  description: string;
}

function App() {

  const [items, setItems] = useState<Item[]>([]);

  const getItems = async () => {
  
    try {
      const response = await instance.get<Item[]>('/api/items');
      setItems(response.data);
    } catch (err) {
      console.log(err);
    }
  };

  return (
    <div className='mt-50 flex h-300 flex-col items-center justify-center gap-10 border-2 border-black'>
 
      <button onClick={getItems}>눌러</button>
      <ErrorBoundary fallback={<div>서버에서 받아오는중 에러발생</div>}>
        {items &&
          items.map((item) => (
            <div key={item.id}>
		      <p>이름:{item.name}</p>
              <p>설명:{item.description}</p>
            </div>
          ))}
      </ErrorBoundary>
    </div>
  );
}

export default App;

```

---

#### 프론트 코드 뜯어보기

##### 타입선언
```jsx
interface Item {
  id: string;
  name: string;
  description: string;
}
```
Item이라는 TS 인터페이스를 정의함

역할
- 서버에서 오는 데이터의 구조를 `약속`함

*React 컴포넌트 내부에서 `items`라는 state를 사용할때, 내부의 `id` `name` `description`이
존재하는지/ 어떤타입인지를 컴파일단계에서 보장가능*

---

##### state 와 getItems()

```jsx
const [items, setItems] = useState<Item[]>([]);

//비동기 함수 선언
  const getItems = async () => {
   
    try { // 요청성공시
      const response = await instance.get<Item[]>('/api/items');
      setItems(response.data);
    } catch (err) { //실패시
      console.log(err);
    }
  };
```

###### State 설명

- `useState`는 제네릭타입을 받을수있음
- 여기서는 위에서 선언한 Item타입의 배열이 들어올것이라고 명시함

  `useState`는 초기값이 있으면 타입을 자동추론하지만 `빈 배열[]`이나 `null`값은 모호한값일경우에는 `never[]` / `null` 만 추론하기때문에 에러가 발생할수있음
  
   -> 따라서 이런경우에는 `이러한타입을 가진 값이 들어올것이다` 라는것을 알리는 
     제네릭을 통한 타입명시가 필요


###### getItems()

- 예시 코드는 버튼클릭시 데이터를 요청하도록 할것이기때문에 `useEffect()` 를 사용X

 - **`getItems()`라는 비동기 함수 선언**
    
    - `async` 키워드를 붙여서 **Promise를 반환하는 함수**로 만듬
        
    - 내부에서 `await`를 사용해 네트워크 요청을 동기적으로 다루기 쉽게 함
        
- **에러 핸들링을 위해 `try-catch`문 사용**
    
    - `try`: 요청이 성공했을 때 동작 정의
        
    - `catch`: 요청 실패 시 실행되는 로직을 정의함
        
- **Axios Instance로 GET 요청 전송**
    
    - `const response = await instance.get('/api/items')`
        
    - `response`에는 axios의 응답 객체가 담김
        
        - `response.data`: 실제 서버 응답 데이터(JSON 배열)
            
        - `response.status`: HTTP 상태 코드 (200, 404 등)
            
- **서버 응답 데이터를 상태에 저장**
    
    - `setItems(response.data)` 실행
        
    - `useState`로 관리되는 `items` 상태가 업데이트됨
        
- **상태 업데이트 → 리렌더링 발생**

    - React는 `items` 값이 바뀌면 컴포넌트를 다시 렌더링
    
    - 이때 `items.map(...)`이 실행되어 새로운 UI가 그려짐
  
---

##### JSX

```jsx
return (
  <div className='mt-50 flex h-300 flex-col items-center justify-center gap-10 border-2 border-black'>
    {/* 버튼(클릭시 getItems()실행) */}
    <button onClick={getItems}>눌러</button>

    {/* 에러 바운더리 */}
    <ErrorBoundary fallback={<div>서버에서 받아오는중 에러발생</div>}>

      {/* 리스트 렌더링 */}
      {items &&
        items.map((item) => (
          <div key={item.id}>
            <p>이름: {item.name}</p>
            <p>설명: {item.description}</p>
          </div>
        ))}
    </ErrorBoundary>
  </div>
);

```

- **조건부 렌더링**
    
    - `items && ...` → `items`가 falsy(예: `null` or `undefined`)면 아무것도 렌더링하지 않게
        
    - truthy일 때만 `.map()` 을 실행함
        
- **map 렌더링**
    
    - `items.map`으로 배열을 순회하며 각각의 `Item` UI를 생성
        
    - `key={item.id}`를 사용해 React가 각 요소를 고유하게 인식하도록함 (리렌더링 최적화)

---

 1. 컴포넌트가 처음 렌더링될 때는 `items`가 빈 배열 → 리스트 없음
 2. 버튼 클릭 → `getItems()` 실행
 3. 서버 응답이 오면 `setItems`로 상태 업데이트
 4. 상태 변경 → 리렌더링 발생
 5. `items.map(...)`이 실행되어 UI에 리스트가 출력됨
 6.  렌더링 도중 에러가 발생하면 `ErrorBoundary`가 fallback UI를 보여줌



 