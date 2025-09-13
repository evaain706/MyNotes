

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



  