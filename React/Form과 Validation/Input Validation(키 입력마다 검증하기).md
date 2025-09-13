


```jsx
import { useState } from 'react';

const LoginForm = () => {
  //    const [email, setEmail] = useState('');
  //   const [password, setPassword] = useState('');

  const [formValues, setFormValues] = useState({
    email: '',
    password: '',
  });

  const [didEdit, setDidEdit] = useState({
    email: false,
    password: false,
  });

  const emailIsInvalid = didEdit.email && !formValues.email.includes('@');

  function handleInputChange(identifier: 'email' | 'password', value: string) {
    setFormValues((prev) => ({
      ...prev,
      [identifier]: value,
    }));

    setDidEdit((prev) => ({
      ...prev,
      [identifier]: false,
    }));
  }

  function handleInputBlur(identifier: 'email' | 'password') {
    setDidEdit((prev) => ({
      ...prev,
      [identifier]: true,
    }));
  }

  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    console.log(`이메일: ${formValues.email} 패스워드: ${formValues.password}`);
  }

  return (
    <form
      onSubmit={handleSubmit}
      className='flex w-full max-w-[40rem] flex-col gap-30 bg-gray-200 p-[2rem]'
    >
      <h2>로그인</h2>

      <div className='flex w-full flex-col gap-10'>
        <div className='relative'>
          <label htmlFor='email'>이메일</label>
          <input
            className='bg-white'
            id='email'
            type='email'
            name='email'
            value={formValues.email}
            onChange={(e) => handleInputChange('email', e.target.value)}
            onBlur={() => handleInputBlur('email')}
          />
          <div className='absolute top-7'>
            {emailIsInvalid && (
              <p className='text-md text-red-500'>
                이메일형식으로 입력해주세요
              </p>
            )}
          </div>
        </div>
        <div>
          <label htmlFor='password'>비밀번호</label>
          <input
            className='bg-white'
            type='password'
            name='password'
            value={formValues.password}
            onChange={(e) => handleInputChange('password', e.target.value)}
            onBlur={() => handleInputBlur('password')}
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