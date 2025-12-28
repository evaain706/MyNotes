
##  로직

```js 
const getIncorrectAnswers = async ({
    category,
    level,
    cursor,
    limit = 10,
  }: {
    category?: string;
    level?: string;
    cursor?: string | null;
    limit?: number;
  }) => {
    const query = new URLSearchParams();

    if (category) query.append('category', category);
    if (level) query.append('level', level);
    if (cursor) query.append('cursor', cursor);
    query.append('limit', limit.toString());

    const response = await privateInstance.get(
      `/api/mypage/incorrect-answers?${query.toString()}`,
    );

    return response.data;
  };
```

``` json

hasNextPage : true
items : [{id: "694eae4867f6321523bd4e4b",…},…]
nextCursor: "2025-11-25T16:30:50.241Z"

```

## useInfiniteQuery란?

 페이지네이션(무한스크롤) 전용 React Query가 제공해주는 훅

---

##  useQuery와의 차이?

- 일반 `useQuery` -> 페이지가 바뀔때마다 데이터가 `교체`됨
- `useInfiniteQuery` -> 페이지가 추가될때 마다 데이터가 `누적`됨

### 예시코드

```js
const {
  data,
  isPending,
  error,
  fetchNextPage,
  hasNextPage,
  isFetchingNextPage,
} = useInfiniteQuery<IncorrectAnswerResponse>({
  queryKey: ['incorrectAnswer', category, level],
  queryFn: ({ pageParam }) =>
    getIncorrectAnswers({
      category,
      level,
      cursor: pageParam as string | null,
    }),
  initialPageParam: null,
  getNextPageParam: (lastPage) => lastPage.nextCursor,
  staleTime: 5 * 60 * 1000,
});
```


## 예시코드에 포함된 반환값

| 값                    | 설명                                       |
| -------------------- | ---------------------------------------- |
| `data`               | `{ pages: [...], pageParams: [...] }` 형태 |
| `fetchNextPage`      | **다음 페이지 요청 함수**                         |
| `hasNextPage`        | 다음 페이지 존재 여부                             |
| `isFetchingNextPage` | 다음 페이지 요청 중인지의 여부                        |
| `isPending`          | 최초 로딩 여부                                 |

---

## `queryFn`

- `pageParam` -> 다음페이지를 요청할 기준값
-  **`queryFn` 안의 `pageParam`은  
    `getNextPageParam`이 반환한 값으로 자동 갱신

첫 호출

`pageParam === null`

 다음 호출

`pageParam === lastPage.nextCursor`


---

###  `initialPageParam`

`initialPageParam: null`

- **첫 페이지 요청 시 cursor 값**
-  cursor 기반 API는 `null` 또는 `undefined`로 시작함

---

###  `getNextPageParam`

```js
getNextPageParam: (lastPage) => lastPage.nextCursor
```

- `lastPage` => 마지막으로 성공적으로 받아온 페이지의 응답데이터

### 내부동작
```js
const nextPageParam = getNextPageParam(lastPage);

if (nextPageParam !== undefined && nextPageParam !== null) {
  hasNextPage = true;
}
```
``
-  `nextCursor`가 있으면 → 다음 페이지 존재  
- 없으면 → 더 이상 요청 안 함

---

### `useInfiniteScroll` 훅과 연결

```js
const targetRef = useInfiniteScroll({
  loading: isFetchingNextPage,
  hasNextPage: !!hasNextPage,
  onLoadMore: fetchNextPage,
});
```


```jsx
  <div className='relative mt-4 flex h-[40rem] p-10 md:h-[60rem]'>
        {isPending && <IncorrectAnswerSkeleton />}

        {!isPending && incorrectAnswers.length === 0 && (
          <div className='absolute top-1/2 left-1/2 -translate-x-1/2 '>
            저장된 문제가 없습니다
          </div>
        )}

        {incorrectAnswers.map((item) => (
          <div key={item.id} onClick={() => handleOpenModal(item)}>
            <AnswerHistoryCard data={item} />
          </div>
        ))}

        <div ref={targetRef} /> %% targetRef연결 하여 뷰포트에 보일시 fetch %%

        {isFetchingNextPage && <IncorrectAnswerSkeleton />}

        {selectedQuiz && (
          <IncorrectModal
            data={selectedQuiz}
            isOpen={open}
            setIsopen={handleCloseModal}
          />
        )}
      </div>
```