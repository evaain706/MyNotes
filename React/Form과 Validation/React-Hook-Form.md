

## React Hook Form이란?

React의 useState를 사용하지 않고도 `제어형` 폼처럼 동작하는 `비제어형` 폼을 구현하는데
도움을 주는 라이브러리

###### 제어형? vs 비제어형?

| 구분       | 제어형(Controlled)            | 비제어형(Uncontrolled)         |
| -------- | -------------------------- | -------------------------- |
| 값 관리     | React의 `useState`로 관리      | DOM의 내부 상태에 맡김             |
| 입력값 변경 시 | 매번 `setState` 호출 → 리렌더링 발생 | DOM 자체에서 값 관리 → 리렌더링 거의 없음 |
| 장점       | 상태를 직접 추적 가능               | 성능이 좋음, 간단                 |
| 단점       | 리렌더링 많음, 코드 길어질 우려         | 값 추적과 검증 불편                |

---

#### `비제어형인데 제어형처럼 다루는것이란?`

- 내부적으로는 `DOM조작`을 통한 `비제어형input`을 사용해서 렌더링을 최소화
- 외부에서는 `register`라는것을 통해 한번으로 `제어형`처럼 값,검증,제출을 관리

```jsx
const { register, handleSubmit } = useForm();

<form onSubmit={handleSubmit(onSubmit)}>
  <input {...register('username')} />
</form>
```
 위 코드가 내부적으로는 밑의 코드
```jsx
ref={inputRef} onChange={...} value={...}
```

---

### 성능이 좋은 이유?

- `useState`를 사용한 제어형 기반 폼은 입력할때마다 컴포넌트가 리렌더링

  ```jsx
  const [name, setName] = useState('');
<input value={name} onChange={(e) => setName(e.target.value)} />
 -> 글자하나를 입력할때마다 setName -> 리렌더링 발생
  ```

- `react-hook-form` 사용시 `ref`로 DOM을 직접추적하여 상태변경X -> 리렌더링 발생 X

 ```jsx
 <input {...register('name')} />
 ```

---
