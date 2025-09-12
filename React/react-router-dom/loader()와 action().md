
## loader()

라우트가 렌더링되기전에 데이터를 가져오는 함수
- Promise를 반환 가능하여 async/await 사용가능
- 라우트 단위로 데이터를 미리 로딩
- 로딩된 데이터는 useLoaderData()로 접근

루트 (ex App.js)에서 라우트 정의에 직접 정의할수도 있지만
보통  export 를 통해 내보낸뒤 import 하여 사용
```jsx
import { useLoaderData } from 'react-router-dom';

import EventsList from '../components/EventsList';

  
function EventsPage() {

  const data = useLoaderData();

  const events = data.events;

  return (
    <>
      <EventsList events={events} />
    </>
  );
}
export default EventsPage;

// loader => 서버가 아닌 브라우저에서 실행됨(어떤 브라우저API든 사용가능 ex 로컬스토리지,쿠키등)

export async function loader() {

  const response = await fetch('http://localhost:8080/events');

  if (!response.ok) {
    // return { isError: true, message: '이벤트 불러오기 실패' };
    throw new Response(JSON.stringify({ message: '이벤트 불러오기 실패' }), {
      status: 500,
    });
    //  라우터의 errorElement를 사용하여 throw시 에러 컴포넌트가 나타나게함
  } else {
    return response;
    //loader함수 => response에서 수작업으로 데이터를 추출해서 return할 필요없이 Promise객체인 response를 바로 return가능
  }
}
```


```js
import EventsPage, { loader as eventsLoader } from './pages/EventsPage';

 {

            index: true,

            element: <EventsPage />,

            loader: eventsLoader,

            // 필요한 컴포넌트에 loader함수를정의한뒤 불러와서 사용

          },
```


### 상세페이지같은 컴포넌트에서 데이터를 불러올때

컴포넌트 내부가 아니기때문에 useParmas와 같은 훅을 사용할수는 없지만

```jsx
export async function loader({ requset, params }) {
 const id = params.id //eventId
 const method = requset.method //get
}
```
(event/:eventId) -> eventId값을 꺼내오기 가능

```jsx
export async function loader({ req, params }) {

  const id = params.eventId;
  // useParams훅을 사용할수는 없지만 createBrowserRouter에 정의된
  // { path: ':eventId', element: <EventDetailPage/>}에서 eventId를 가져올수있음
  const response = await fetch(`http://localhost:8080/events/${id}`);
  if (!response.ok) {

    throw new Response(

      JSON.stringify({ message: '이벤트 상세 데이터 불러오기 실패' }),
      {
        status: 500,
      }
    );
  } else {
    return response;
  }
}


```




## useLoaderData()

해당 라우트의 loader에서 반환된 데이터를 가져오는 훅

```jsx
import { useLoaderData } from 'react-router-dom';

import EventsList from '../components/EventsList';

  
function EventsPage() {

  const data = useLoaderData(); //loader에서 반환된 데이터를 가져옴

  const events = data.events;

  return (
    <>
      <EventsList events={events} />
    </>
  );
}
export default EventsPage;


```
✅ **장점**

- 데이터 로딩과 UI 렌더링을 분리
    
- 페이지가 렌더링될 때 이미 데이터가 준비되어 있어 사용자 경험이 좋아짐
    
- ErrorBoundary나 Suspense와 함께 사용 가능


## action()

라우트에서 폼 제출이나 데이터변경요청을 처리하는 함수

- 라우트 단위로 정의됨.
- `Form`과 함께 사용하면 자동으로 action으로 데이터가 전달됨.
- Promise를 반환할 수 있고, 반환값은 `useActionData`로 가져올 수 있음.

loader와 같이 루트 (ex App.js)에서 라우트 정의에 직접 정의할수도 있지만
보통  export 를 통해 내보낸뒤 import 하여 사용

```jsx
// prop으로 method를 받음 (post,patch)
function EventForm({ method, event }) {
  const navigate = useNavigate();
  //  useActionData()
  // - 현재 라우트의 action 함수가 반환(return)한 값을 가져옴
  // - 일반적으로 폼 유효성 검사 에러 메시지나 실패 응답 데이터를 UI에 표시할 때 사용
  // 가장가까운 Action에 대한 액세스를 제공
  const data = useActionData();
  //  useNavigation: 라우터가 제공하는 훅
  // - 현재 내비게이션 상태를 알려줌
  // - state 값으로 "idle" | "submitting" | "loading" 등을 return
  // - 이걸 사용하여 Form 전송 중이거나 라우트 이동 중일 때 로딩 상태 UI를 만들 수 있음
  const navigation = useNavigation();
  // 현재 상태가 'submitting'이면 전송 중으로 판단
  const isSubmitting = navigation.state === 'submitting'

  function cancelHandler() {
    navigate('..');
  }

  return (
    // react-router-dom이 제공하는 Form 사용
    // 폼 제출 시 페이지 전체 리로드 없이 action 함수로 데이터를 전달
    // 제출시 method prop의 값을 통해 FormData에 POST or PATCH가 들어감
    <Form method={method} className={classes.form}>
      {/* response.status === 422 인 경우, response 객체가 data에 들어감 */}
      {data && data.errors && (
        <ul>
          {Object.values(data.errors).map((err) => (
            <li key={err}>{err}</li>
          ))}
        </ul>
      )}
      <p>
        <label htmlFor='title'>제목</label>
        <input
          id='title'
          type='text'
          name='title'
          required
          defaultValue={event ? event.title : ''}
        />
      </p>
      <p>

        <label htmlFor='image'>이미지</label>

        <input
          id='image'
          type='url'
          name='image'
          required
          defaultValue={event ? event.image : ''}
        />

      </p>

      <p>

        <label htmlFor='date'>날짜</label>

        <input

          id='date'

          type='date'

          name='date'

          required

          defaultValue={event ? event.date : ''}

        />

      </p>

      <p>

        <label htmlFor='description'>상세</label>

        <textarea

          id='description'

          name='description'

          rows='5'

          required

          defaultValue={event ? event.description : ''}

        />

      </p>

      <div className={classes.actions}>

        <button type='button' onClick={cancelHandler} disabled={isSubmitting}>
          취소
        </button>
        {/* isSubmitting이 true일 땐 버튼을 비활성화해서 중복 제출 방지 */}
        <button disabled={isSubmitting}>
          {isSubmitting ? '전송중...' : '저장'}
        </button>
      </div>
    </Form>
  );
}
```

``` jsx
export async function action({ request, params }) {

  // 1. request.formData()를 통해 Form에서 전송된 데이터를 가져옴

  const data = await request.formData();

  const eventData = {
    title: data.get('title'),
    image: data.get('image'),
    date: data.get('date'),
    description: data.get('description'),
  };

  // 2. 기본 URL 세팅: 새 이벤트 생성 시

  let url = 'http://localhost:8080/events';

  // 3. 클라이언트 요청의 method 확인

  // request.method는 Form에서 지정한 method(post,patch,delete등)를 가지고 있음

  // → 이를 통해 같은 action에서 post 와 patch를 구분

  if (request.method === 'PATCH') {

    const eventId = params.eventId; // URL param으로 eventId 가져오기

    url = 'http://localhost:8080/events/' + eventId; // 수정 대상 URL

  }

  // 4. 서버에 요청 보내기

  const response = await fetch(url, {
    method: request.method, // Form에서 지정한 method 사용
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(eventData),
  });

  // 5. 유효성 검증 실패(422) 시 response 그대로 반환

  // → react-router는 action 반환값을 useActionData로 전달

  if (response.status === 422) {
    return response;
  }
  // 6. 기타 서버 에러

  if (!response.ok) {

    throw new Response(JSON.stringify({ message: '이벤트 추가 실패' }), {
      status: 500,
    });
  }

  // 7. 성공 시 이벤트 목록 페이지로 리다이렉트

  return redirect('/events');

}
```




## loader() vs action()


- **loader** → 읽기 전용 데이터 페칭
    
- **action** → 서버로 데이터 보내기, 수정/등록/삭제