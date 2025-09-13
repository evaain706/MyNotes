

## Form

-  데이터를 서버로 제출하는기능
- 폼 제출 === 새로운 요청이라고 인식
- Form 내부의 type='submit'의 버튼등의 요소를 클릭하면
   1. 브라우저가 폼데이터를 수집하고
   2. URL로 새요청을 보내기때문에 결과적으로 페이지가 새로 로드됨

 - React에서는 폼 기본동작이 불필요하기때문에 onSubmit에서 기본동작을 막고 직접처리
  ``` jsx
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault(); // 기본 새로고침/쿼리스트링 전송 막음
  console.log("데이터 전송 로직 직접 처리");
};

<form onSubmit={handleSubmit}>
  <input name="username" />
  <button type="submit">전송</button>
</form>

  
  ```


# 폼다루기 방법 1 #useState 

```jsx
import { useState } from 'react';

const LoginForm = () => {
  //   const [email, setEmail] = useState('');
  //   const [password, setPassword] = useState('');

  const [formValues, setFormValues] = useState({
    email: '',
    password: '',
  });

  function handleInputChange(identifier: 'email' | 'password', value: string) {

    setFormValues((prev) => ({

      ...prev,

      [identifier]: value,

    }));

  }

  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    console.log(`이메일: ${formValues.email} 패스워드: ${formValues.password}`);

  }

  return (
    <form
      onSubmit={handleSubmit}
      className='flex w-full max-w-[40rem] flex-col gap-20 bg-gray-200 p-[2rem]'
    >
      <h2>로그인</h2>

      <div className='flex w-full flex-col gap-5'>
        <div>
          <label htmlFor='email'>이메일</label>
          <input
            className='bg-white'
            id='email'
            type='email'
            name='email'
            value={formValues.email}
            onChange={(e) => handleInputChange('email', e.target.value)}
          />
        </div>
        <div>
          <label htmlFor='password'>비밀번호</label>
          <input
            className='bg-white'
            id='password'
            type='password'
            name='password'
            value={formValues.password}
            onChange={(e) => handleInputChange('password', e.target.value)}
          />
        </div>
      </div>

      <p className='flex justify-between'>
        <button className='cursor-pointer'>초기화</button>
        <button className='cursor-pointer'>로그인</button>
      </p>
    </form>
  );
};

export default LoginForm;
```

  #useState 를 사용하여 각 input에 value와 onchange를 설정하여 값을 입력하고 상태에 저장함


# 폼 다루기 방법2 #useRef 

```jsx
import { useRef } from 'react';

const LoginForm = () => {
  const email = useRef<HTMLInputElement>(null);
  const password = useRef<HTMLInputElement>(null);

  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();

    const emailValue = email.current?.value;
    const passwordValue = password.current?.value;

    console.log(`이메일: ${emailValue} 패스워드: ${passwordValue}`);
  }

  return (
    <form
      onSubmit={handleSubmit}
      className='flex w-full max-w-[40rem] flex-col gap-20 bg-gray-200 p-[2rem]'
    >
      <h2>로그인</h2>

      <div className='flex w-full flex-col gap-5'>
        <div>
          <label htmlFor='email'>이메일</label>
          <input
            className='bg-white'
            id='email'
            type='email'
            name='email'
            ref={email}
          />
        </div>

        <div>
          <label htmlFor='password'>비밀번호</label>
          <input
            className='bg-white'
            id='password'
            type='password'
            name='password'
            ref={password}
          />
        </div>
      </div>

      <p className='flex justify-between'>
        <button className='cursor-pointer'>초기화</button>
        <button className='cursor-pointer'>로그인</button>
      </p>
    </form>
  );
};

export default LoginForm;

```
#useRef 를 사용한 방법
- 요소가 많아지면 하나하나 참조를 해줘야하고 값초기화가 어려움



# 폼다루기 방법3 FormData사용

```jsx
export default function Signup() {
  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();

    //formData를 통해 값을 가져오고싶으면 html속성들에 'name'이 필수임
    const formData = new FormData(e.target as HTMLFormElement);

    const acquisitionChannel = formData.getAll('acquisition');

    //폼의 모든 입력창들과 그의 값들에 대한 배열을 제공
    const data = Object.fromEntries(formData.entries());

    data.acquisition = acquisitionChannel;

    console.log(data);
  }

  return (
    <form
      onSubmit={handleSubmit}
      className='mx-auto max-w-xl space-y-6 rounded-2xl bg-white p-8 shadow-lg'
    >
      <h1 className='text-2xl font-bold text-gray-600'>
        회원가입 컴포넌트 예시
      </h1>

      <div className='flex flex-col space-y-1'>
        <label htmlFor='email' className='text-sm font-medium text-gray-700'>
          이메일
        </label>
        <input
          id='email'
          type='email'
          name='email'
          className='w-full rounded-lg border border-gray-300 px-3 py-2 text-gray-800 outline-none focus:border-blue-500 focus:ring-2 focus:ring-blue-200'
        />
      </div>

      <div className='grid grid-cols-1 gap-4 md:grid-cols-2'>
        <div className='flex flex-col space-y-1'>
          <label
            htmlFor='password'
            className='text-sm font-medium text-gray-700'
          >
            비밀번호
          </label>
          <input
            id='password'
            type='password'
            name='password'
            className='w-full rounded-lg border border-gray-300 px-3 py-2 text-gray-800 outline-none focus:border-blue-500 focus:ring-2 focus:ring-blue-200'
          />
        </div>

        <div className='flex flex-col space-y-1'>
          <label
            htmlFor='confirm-password'
            className='text-sm font-medium text-gray-700'
          >
            비밀번호 확인
          </label>
          <input
            id='confirm-password'
            type='password'
            name='confirm-password'
            className='w-full rounded-lg border border-gray-300 px-3 py-2 text-gray-800 outline-none focus:border-blue-500 focus:ring-2 focus:ring-blue-200'
          />
        </div>
      </div>

      <hr className='border-gray-200' />

      <div className='grid grid-cols-1 gap-4 md:grid-cols-2'>
        <div className='flex flex-col space-y-1'>
          <label
            htmlFor='first-name'
            className='text-sm font-medium text-gray-700'
          >
            성
          </label>
          <input
            type='text'
            id='first-name'
            name='first-name'
            className='w-full rounded-lg border border-gray-300 px-3 py-2 text-gray-800 outline-none focus:border-blue-500 focus:ring-2 focus:ring-blue-200'
          />
        </div>

        <div className='flex flex-col space-y-1'>
          <label
            htmlFor='last-name'
            className='text-sm font-medium text-gray-700'
          >
            이름
          </label>
          <input
            type='text'
            id='last-name'
            name='last-name'
            className='w-full rounded-lg border border-gray-300 px-3 py-2 text-gray-800 outline-none focus:border-blue-500 focus:ring-2 focus:ring-blue-200'
          />
        </div>
      </div>

      <div className='flex flex-col space-y-1'>
        <label htmlFor='role' className='text-sm font-medium text-gray-700'>
          직업
        </label>
        <select
          id='role'
          name='role'
          className='w-full rounded-lg border border-gray-300 bg-white px-3 py-2 text-gray-800 outline-none focus:border-blue-500 focus:ring-2 focus:ring-blue-200'
        >
          <option value='student'>학생</option>
          <option value='teacher'>교사</option>
          <option value='other'>기타</option>
        </select>
      </div>

      <fieldset className='space-y-2 border-2 border-gray-300 p-5'>
        <legend className='text-sm font-medium text-gray-700'>방문계기</legend>
        <div className='flex items-center space-x-2'>
          <input
            type='checkbox'
            id='google'
            name='acquisition'
            value='google'
          />
          <label htmlFor='google' className='text-sm text-gray-700'>
            인터넷
          </label>
        </div>
        <div className='flex items-center space-x-2'>
          <input
            type='checkbox'
            id='friend'
            name='acquisition'
            value='friend'
          />
          <label htmlFor='friend' className='text-sm text-gray-700'>
            지인의 추천
          </label>
        </div>
        <div className='flex items-center space-x-2'>
          <input type='checkbox' id='other' name='acquisition' value='other' />
          <label htmlFor='other' className='text-sm text-gray-700'>
            기타
          </label>
        </div>
      </fieldset>

      <div className='flex items-center space-x-2'>
        <input
          type='checkbox'
          id='terms-and-conditions'
          name='terms'
          className='h-4 w-4'
        />
        <label htmlFor='terms-and-conditions' className='text-sm text-gray-700'>
          약관정보 동의
        </label>
      </div>

      <div className='flex justify-end space-x-4 pt-4'>
        <button
          type='reset'
          className='rounded-lg border border-gray-300 px-4 py-2 text-gray-600 hover:bg-gray-100'
        >
          초기화
        </button>
        <button
          type='submit'
          className='rounded-lg bg-blue-600 px-4 py-2 font-medium text-white hover:bg-blue-700 focus:ring-2 focus:ring-blue-300'
        >
          가입하기
        </button>
      </div>
    </form>
  );
}

```


