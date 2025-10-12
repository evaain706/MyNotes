

## key

`key`는 React에서 배열의 각요소를 식별하기위해 사용되는 문자열속성
어떤 항목을 추가,삭제,변경할지 식별하는데 사용

---

## 왜 필요함?

위의 설명대로 배열을통해 여러요소들을 렌더링할때 각 요소들을 구별할 **식별자**가 필요한데,
`key`가 이 역할을 담당한다.

`key`가 없다면 배열의 순서가 변경되거나, 요소가 추가/삭제될때 어떤 요소가 변경되었는지
정확하게 식별하기 어려움.


## 코드예시

```jsx
import React from 'react';

interface User {
  id: number;
  name: string;
}

const userList: User[] = [
  { id: 1, name: 'Kim' },
  { id: 2, name: 'Lee' },
  { id: 3, name: 'Maeng' },
];

function UserProfileList() {
  return (
    <ul>
     
      {userList.map((user) => (
         //고유한 값인 id를 key로 지정
        <li key={user.id}>
          {user.name}
        </li>
      ))}
    </ul>
  );
}

export default UserProfileList;
```

- 배열요소에서 고유한 값인 `id`를 `key`로 사용하여 각 요소들을 구별할 식별자로 사용

---

## 주의할점

#### 1. `index`를 `key`값으로 사용하는것을 피하기

  ```jsx

userList.map((user, index) => (
  <li key={index}>{user.name}</li>
));
  ```
  
   배열의 순서가 변경되거나 항목이 추가/삭제 되면 각 요소의 `index`가 변경되는데
   `key`를 `index`로 설정하였으므로 요소와 `key`가 일치하지않게된다.

   - 추가/삭제,순서 변경이 일어날일이없는 계산이 필요없는 정적인 배열을 렌더링 할때
   - 고유한 ID가 없을때 -> **고유한 ID가 있는것이 가장좋음**
   - 재정렬,필터링 될일이 없을때
   
위 3가지 경우에서는 `index`를 `key`로 사용해도 괜찮다


**-> 데이터 자체에 포함된 `id`와 같은 예측가능하고 고유한값을 `key`로 사용하는것이 가장이상적**

