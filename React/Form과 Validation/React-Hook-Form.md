

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

#### 비제어형인데 제어형처럼 다루는것이란?

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

## 기본구조

```jsx
import { useForm } from 'react-hook-form';

const ExampleForm = () => {
  const { register, handleSubmit } = useForm();

  const onSubmit = (data) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('username')} placeholder='이름' />
      <input {...register('email')} placeholder='이메일' />
      <button type='submit'>제출</button>
    </form>
  );
};

```

#### 위 코드에서 `data`의 형태

```js
{
  username: '닉네임123',
  email: 'test123@test.com'
}
```

---
## 핵심요소

## 1. register

 - `input`을 react-hook-form에 등록하여 값과 유효성검사규칙등을 관리
 - `onChange()`등의 이벤트핸들러에 자동으로 연결
 - `required,pattern`등의 요소로 유효성검사규칙 정의가능
 
```jsx
import { useForm } from 'react-hook-form';

const RegisterExample = () => {
  const { register, handleSubmit } = useForm();

  const onSubmit = (data: any) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('email', {
          required: '이메일은 필수입니다.',
          pattern: {
            value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
            message: '올바른 이메일 형식이 아닙니다.',
          },
        })}
        placeholder='이메일을 입력하세요'
      />
      <button>제출</button>
    </form>
  );
};

```

- `register('email')` → 이 input의 이름을 `email`로 정의함
- `required`, `pattern` 등으로 유효성 검사 규칙 정의
- 입력값은  `data.email`로 접근 가능
## 2. handleSubmit

   - form의 `onSubmit`에 들어가는 함수 
   - 유효성 검사를 실행한뒤 모든 필드가 통과되면 제출
   
 ```jsx
 const { register, handleSubmit } = useForm();

const onSubmit = (data: any) => {
  console.log('제출된 데이터:', data);
};

<form onSubmit={handleSubmit(onSubmit)}>
  <input {...register('username', { required: true })} placeholder='이름' />
  <button type='submit'>제출</button>
</form>;

 ```
 - 
## 3. errors

- 유효성 검사 실패시 발생하는 에러 정보를 담은 객체

```jsx
import { useForm } from 'react-hook-form';

const ErrorExample = () => {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm();

  const onSubmit = (data: any) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('nickname', { required: '닉네임은 필수입니다.' })}
        placeholder='닉네임'
      />
      {errors.nickname && (
        <p className='text-red-500 text-sm'>{errors.nickname.message}</p>
      )}
      <button>작성</button>
    </form>
  );
};

```

## 4. reset

- 폼의 모든 필드의 값을 초기화, 특정값으로 다시 설정할때사용

 ```jsx
 import { useForm } from 'react-hook-form';

const ResetExample = () => {
  const { register, handleSubmit, reset } = useForm();

  const onSubmit = (data: any) => {
    console.log('전송된 데이터:', data);
    reset(); // 모든 필드 초기화
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('title')} placeholder='제목' />
      <input {...register('content')} placeholder='내용' />
      <button>제출</button>
      <button
        type='button'
        // 필드의 값을 특정값으로 초기화
        onClick={() => reset({ title: '제목', content: '내용' })}
        className='ml-2 border px-2 py-1'
      >
         되돌리기
      </button>
    </form>
  );
};

 ```

## 5. watch

- 특정필드나 모든필드의 값을 실시간으로 감시할때 사용

```jsx
import { useForm } from 'react-hook-form';
import { useEffect } from 'react';

const WatchExample = () => {
  const { register, watch, handleSubmit } = useForm();

  const password = watch('password'); // password 필드 값 감시
  const confirmPassword = watch('confirmPassword');

  useEffect(() => {
    if (password && confirmPassword && password !== confirmPassword) {
      console.log('비밀번호가 일치하지 않습니다');
    }
  }, [password, confirmPassword]);

  const onSubmit = (data: any) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('password')}
        placeholder='비밀번호'
        type='password'
      />
      <input
        {...register('confirmPassword')}
        placeholder='비밀번호 확인'
        type='password'
      />
      <button>제출</button>
    </form>
  );
};

```

- `watch(필드)` → 해당 필드 값이 변경될 때마다 자동으로 최신 값 반환
- `useEffect()`와 함께 사용하여 실시간 검증로직 구현 가능
- `watch()` 와 같이 인자없이 호출시 → 모든 필드의 현재 값 반환


---


##### `Zod` 라이브러리와 함께 사용하여 유효성검사를 더 효율적으로 처리하기도함

