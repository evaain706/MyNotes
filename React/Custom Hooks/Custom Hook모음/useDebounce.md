

## 정의

- `디바운스(Debounce)`는 짧은 시간동안 동일한 요청(동작)이 반복되면, 마지막 동작만 실행하도록 지연시키는 기법

- 사용자가 연속으로 이벤트를 발생시켜도 지정해놓은 시간 내에서는 실행하지않고, 추가적인 이벤트가 없을때 최종적으로 한번만 실행시키도록한다.

---
## 상황예시

1. 사용자가 SNS에서 좋아요 버튼을 여러번 누름
2. 버튼을 누를때마다 서버에 요청을 보내게된다면 API호출이 대량발생
##### => Debounce를 사용하여 최종적으로 눌렀을때의 1회만 요청이 가도록하기

---
## 흐름

1. 사용자가 좋아요버튼을 클릭할시, 로컬상태만 먼저 업데이트하여 시각적인 피드백을 제공
2. `setTimeout`을 사용하여 좋아요 요청관련 함수 호출을 지연시킴
3. `setTImeout`으로 지정한 시간내에 추가적인 사용자의 좋아요 버튼 클릭이 없을시 요청관련함수 실행
4. 추가적인 클릭이 있었을경우 동작을 취소하고 다시 대기

---

## useDebounce 예시

```jsx
import { useState, useEffect } from "react";

function useDebounce<T>(value: T, delay: number) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(handler); 
  }, [value, delay]);

  return debouncedValue;
}

```

---
## 좋아요 버튼 로직에서의 사용예시

```jsx
function LikeButton() {
  const [likes, setLikes] = useState(0);
  const debouncedLikes = useDebounce(likes, 500); // 0.5초 대기

  useEffect(() => {
    if (debouncedLikes > 0) {
      console.log("서버 요청: 좋아요 반영", debouncedLikes);
      // 실제 서버 API 호출
      // api.post("/like", { count: debouncedLikes });
    }
  }, [debouncedLikes]);

  return (
    <button onClick={() => setLikes(likes + 1)}>
      좋아용:{likes}
    </button>
  );
}

```


---
## 이외의 사용예시

##### 사용자이벤트가 빠르게 반복되는 부분에 주로사용

- 검색창 자동완성
- 입력값 validation
- 스크롤 이벤트 최적화
- 브라우저창 리사이징 이벤트 최적화
- 이외의 여러 키입력,클릭 이벤트등