

## 목표

커뮤니티 페이지에서의 게시글 페이지네이션 구현

1. 오프셋 기반 페이지네이션
2. 카테고리 필터링 포함

---

## 구현

#### 백엔드

``` js
router.get('/getPost', async (req, res) => {
  try {
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 10;
    const category = req.query.category;

    const skip = (page - 1) * limit;

    const filter = {};
    if (category) filter.category = category;

    const totalCount = await CommunityPost.countDocuments(filter);

    const posts = await CommunityPost.find(filter)
      .sort({ createdAt: -1 })
      .skip(skip)
      .limit(limit);

    res.json({
      totalCount,
      totalPages: Math.ceil(totalCount / limit),
      currentPage: page,
      posts,
    });
  } catch (error) {
    res.status(500).json({ message: '서버 오류', error });
  }
});

```
- skip을 사용한 offset구현
- filter로 카테고리 걸러내기

---

## 프론트

``` js
const fetchPosts = async (
    page: number,
    limit: number = 5,
    category?: string,
  ) => {
    const query = new URLSearchParams({
      page: String(page),
      limit: String(limit),
    });

    if (category) {
      query.append('category', category);
    }

    const response = await instance.get(`/api/community/getPost?${query}`);
    return response.data;
  };
```

쿼리문자열을 쉽게 다룰수있도록해주는 내장 API인
URLSearchParams로 쿼리 문자열을 깔끔하게 생성하도록함

category를 optional로 설정하여 category가 있을시 query에 추가하여 쿼리문자열 생성
`api/community/getPost?page=1&limit=5&category=question`



#### 커뮤니티페이지 메인 컴포넌트

```jsx
const [page, setPage] = useState(1);
  const [category, setCategory] = useState<string | null>(null);
  const { fetchPosts } = useCommunity();

  const { data, isPending, error } = useQuery({
    queryKey: ['post', page, category],
    queryFn: () => fetchPosts(page, 5, category || undefined),
  });

  if (isPending) return <>로딩중..</>;
  if (error) return <>에러발생: {error.message}</>;

```

- useQuery사용
- page와 category를 쿼리키로 사용하여 캐싱처리




## 추가구현사항

- 게시글 상세페이지 이동후 뒤로가기시 페이지네이션 `page`유지