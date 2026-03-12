


  

---

  

## 프로젝트 한 줄 소개

  

`FrontQuiz`는 프론트엔드 학습자를 위해 카테고리와 난이도를 선택해 퀴즈를 풀고, 채점 결과와 오답/통계를 관리하며, 커뮤니티에서 질문과 정보를 공유할 수 있게 만든 React 기반 웹 서비스다.

  

---

  

## 30초 소개 버전

  

`FrontQuiz는 React, TypeScript 기반의 프론트엔드 학습용 퀴즈 서비스입니다. 사용자가 카테고리와 난이도를 선택하면 퀴즈를 생성해 풀 수 있고, 채점 결과를 바탕으로 오답 저장과 통계 확인이 가능합니다. 로그인 사용자는 마이페이지에서 학습 기록을 관리할 수 있고, 커뮤니티에서 질문과 정보 글도 작성할 수 있게 구성했습니다. 프론트에서는 TanStack Query로 서버 상태를 관리하고, Zustand persist로 로그인 상태를 유지했으며, Skeleton UI와 Error Boundary를 적용해서 사용자 경험도 개선했습니다.`

  

---

  

## 1분 소개 버전

  

`이 프로젝트는 단순히 문제를 보여주는 데서 끝나는 게 아니라, 학습 흐름 전체를 관리하는 데 집중했습니다. 먼저 사용자가 카테고리와 난이도를 선택하면 퀴즈를 생성하고 답안을 제출할 수 있게 했습니다. 이후 채점 결과를 보여주고, 로그인 사용자의 경우 정답 여부를 통계 데이터에 반영하도록 연결했습니다. 틀린 문제는 별도로 저장해서 마이페이지에서 다시 볼 수 있게 했고, 카테고리와 난이도 기준으로 필터링도 가능하게 만들었습니다. 또한 커뮤니티 기능을 넣어서 사용자가 질문이나 정보를 공유할 수 있게 했고, 검색에는 debounce를 적용해 불필요한 요청을 줄였습니다. 구현 측면에서는 React Router 기반 페이지 구조, TanStack Query 기반 데이터 패칭/캐싱, Zustand persist 기반 사용자 상태 유지, Axios interceptor 기반 토큰 재시도, Skeleton UI와 Error Fallback 같은 예외 처리 구조를 중심으로 설계했습니다.`

  

---

  

## 이력서 기준 핵심 포인트와 코드 연결

  

### 1. 카테고리/난이도 기반 퀴즈 생성

  

면접에서 이렇게 말하면 된다.

  

`사용자가 먼저 주제와 난이도를 고른 뒤 퀴즈 화면으로 이동하도록 플로우를 설계했습니다. 선택값은 전역 상태로 관리해서 다음 페이지에서도 그대로 사용할 수 있게 했고, 퀴즈 화면에 진입하면 그 값을 기준으로 API를 호출해 문제를 생성합니다.`

  

코드 근거

  

- `src/page/OptionSelectPage.tsx`

  - 카테고리와 난이도 선택 UI를 단계적으로 제공한다.

  - 선택이 끝나면 `/quiz`로 이동한다.

- `src/store/useOptionStore.ts`

  - 사용자가 고른 카테고리와 난이도를 전역 상태로 저장한다.

- `src/features/quiz/hooks/useQuiz.ts`

  - `generateQuiz(category, level)`로 문제 생성 API를 호출한다.

- `src/features/quiz/QuizScreen.tsx`

  - 화면 진입 시 `fetchQuiz()`를 호출해 실제 문제를 불러온다.

  

설명 포인트

  

- 선택 UI와 실제 퀴즈 API 호출을 분리해 역할을 나눴다.

- `enabled: false`로 자동 호출을 막고, 필요한 시점에만 `refetch()` 하도록 제어했다.

- 퀴즈를 다시 생성할 때 이전 결과와 사용자 답안을 초기화해 상태 꼬임을 줄였다.

  

예상 질문

  

`왜 useQuery인데 enabled를 false로 두었나요?`

  

답변 예시

  

`이 경우에는 화면 마운트만으로 항상 호출되기보다, 사용자가 선택을 완료한 뒤 명확한 시점에 문제를 불러오는 게 더 중요했습니다. 그래서 TanStack Query의 캐싱 장점은 유지하면서도, 호출 타이밍은 refetch로 제어했습니다.`

  

---

  

### 2. 답안 제출과 채점 결과 처리

  

면접에서 이렇게 말하면 된다.

  

`문제를 가져온 뒤 사용자가 답안을 입력하면 채점 API를 호출하고, 정답 여부와 해설을 결과 상태에 저장해 화면에 반영했습니다. 로그인 사용자의 경우 채점 결과를 통계 API와도 연결해서 학습 데이터가 누적되도록 만들었습니다.`

  

코드 근거

  

- `src/features/quiz/hooks/useQuiz.ts`

  - `handleSubmit()`에서 답안 제출과 채점을 수행한다.

  - 채점이 끝나면 `setResult(response.data)`로 결과를 저장한다.

  - 로그인 사용자일 경우 `handleAddStatistics()`를 호출한다.

- `src/store/useQuizStore.ts`

  - 사용자 답안, 채점 상태, 채점 결과를 저장한다.

  

설명 포인트

  

- 채점 중에는 `setIsGrading(true)`로 중복 요청을 줄였다.

- 통계 반영 후 `invalidateQueries`로 관련 데이터를 최신 상태로 유지했다.

- 에러가 발생하면 Toast로 사용자에게 즉시 피드백을 준다.

  

예상 질문

  

`채점 결과와 통계 반영을 왜 분리했나요?`

  

답변 예시

  

`채점 자체는 비로그인 사용자도 가능하지만, 통계 저장은 로그인 사용자에게만 필요한 기능이었습니다. 그래서 채점 API와 통계 적재 로직을 분리해서 조건부로 동작하게 만들었습니다.`

  

---

  

### 3. 마이페이지 오답 저장과 재학습 흐름

  

면접에서 이렇게 말하면 된다.

  

`학습 서비스에서 중요한 건 한 번 푸는 것보다 틀린 문제를 다시 보는 흐름이라고 생각했습니다. 그래서 채점 결과 중 오답은 별도로 저장하고, 마이페이지에서 카테고리/난이도별로 다시 조회할 수 있게 만들었습니다.`

  

코드 근거

  

- `src/features/myPage/hooks/useIncorrectAnswers.ts`

  - `handleAddInCorrect()`에서 오답 저장 API를 호출한다.

  - 문제, 사용자가 고른 답, 해설, 난이도, 주제를 함께 저장한다.

  - `getIncorrectAnswers()`에서 필터 조건과 cursor를 기준으로 목록을 가져온다.

- `src/features/myPage/IncorrectAnswers/IncorrectAnswer.tsx`

  - `useInfiniteQuery`로 오답 목록을 페이지 단위로 조회한다.

  - 카테고리/난이도 필터와 상세 모달을 제공한다.

- `src/hooks/useInfiniteScroll.ts`

  - Intersection Observer 기반 무한 스크롤을 구현한다.

  

설명 포인트

  

- 오답 저장 시 단순히 문제 id만 저장하지 않고, 사용자의 선택 답과 해설까지 함께 저장해 복습 효율을 높였다.

- `useInfiniteQuery`와 `cursor` 기반 조회를 사용해 긴 목록도 부담 없이 탐색할 수 있게 했다.

- 무한 스크롤 훅을 별도로 분리해 다른 리스트에도 재사용할 수 있는 구조로 만들었다.

  

예상 질문

  

`왜 페이지네이션 대신 무한 스크롤을 썼나요?`

  

답변 예시

  

`오답 복습은 사용자가 연속적으로 훑어보는 경우가 많아서, 페이지 번호를 계속 누르는 것보다 자연스럽게 이어지는 경험이 더 적합하다고 판단했습니다. 대신 커뮤니티는 특정 페이지 이동 니즈가 있어 일반 페이지네이션을 유지했습니다.`

  

---

  

### 4. 사용자 통계 제공

  

면접에서 이렇게 말하면 된다.

  

`문제를 푸는 경험만 제공하면 학습 서비스로서 동기 부여가 약할 수 있다고 생각해서, 정답 수, 오답 수, 정확도, 카테고리별 통계, 난이도별 통계를 시각적으로 확인할 수 있게 했습니다.`

  

코드 근거

  

- `src/features/myPage/hooks/useUserStatistics.ts`

  - 사용자 통계 데이터를 조회한다.

- `src/features/myPage/UserStatistics/UserStatistics.tsx`

  - 전체 통계, 카테고리 통계, 난이도 통계를 각각 렌더링한다.

  - 데이터가 없을 때의 예외 상태도 분리했다.

- `src/utils/checkStatistics.ts`

  - 화면에 필요한 파생 데이터와 empty state 판단을 담당한다.

  

설명 포인트

  

- 서버 데이터는 조회만 하고, 화면에서 필요한 파생 상태는 util 함수로 분리했다.

- 로딩 시 Skeleton, 실패 시 ErrorComp, 빈 데이터일 때 empty state를 각각 다르게 보여준다.

- 통계 화면은 단순 조회 페이지가 아니라 "학습 동기 부여" 역할까지 고려했다.

  

예상 질문

  

`프론트에서 통계를 계산한 이유가 있나요?`

  

답변 예시

  

`서버 원본 데이터를 그대로 쓰기보다, 화면에 필요한 totalCorrect, accuracy, empty state 같은 값은 프론트에서 한 번 정리하는 편이 컴포넌트 가독성이 좋아졌습니다. 서버 응답 형식을 바꾸지 않고도 UI 요구사항을 유연하게 대응할 수 있는 장점도 있었습니다.`

  

---

  

### 5. OAuth 로그인과 로그인 상태 유지

  

면접에서 이렇게 말하면 된다.

  

`로그인 경험에서는 사용자가 새로고침을 하더라도 상태가 유지되는 점이 중요하다고 생각했습니다. 그래서 OAuth 로그인 이후 받은 사용자 정보를 Zustand persist에 저장하고, 인증이 필요한 API는 별도 Axios 인스턴스와 refresh 재시도 로직으로 관리했습니다.`

  

코드 근거

  

- `src/page/KakaoCallback.tsx`

  - 인가 코드를 받아 백엔드에 전달하고, 응답받은 사용자 정보를 저장한다.

- `src/store/useUserStore.ts`

  - 사용자 정보 저장, 초기화, hydration 완료 상태를 관리한다.

  - `persist`를 사용해 로그인 상태를 유지한다.

- `src/apis/privateInstance.ts`

  - 401 발생 시 refresh 요청 후 원래 요청을 재시도한다.

  - refresh 실패 시 사용자 상태를 정리한다.

- `src/components/ProtectedRoute.tsx`

  - 인증이 필요한 페이지를 보호한다.

  

설명 포인트

  

- 사용자 정보 저장과 인증 요청 로직을 분리해 책임을 명확히 했다.

- 새로고침 이후에도 persist 상태가 복원되도록 설계했다.

- 인증 만료 시 interceptor에서 한 번 재시도하게 해 사용자 경험을 끊지 않도록 했다.

  

예상 질문

  

`localStorage 기반 persist를 쓸 때 주의한 점이 있나요?`

  

답변 예시

  

`민감한 토큰 자체를 프론트 상태에 저장하기보다, 사용자 정보와 로그인 상태 유지에 집중했습니다. 실제 인증 흐름은 쿠키 기반 요청과 refresh 재시도로 처리해서 보안상 역할을 분리하려고 했습니다.`

  

---

  

### 6. 커뮤니티 기능과 검색 최적화

  

면접에서 이렇게 말하면 된다.

  

`혼자 퀴즈를 푸는 구조에서 끝나지 않게 하려고 커뮤니티를 추가했습니다. 질문과 정보 카테고리를 나눠 게시글을 조회할 수 있게 했고, 검색 입력에는 debounce를 적용해 타이핑할 때마다 불필요한 요청이 나가지 않도록 했습니다.`

  

코드 근거

  

- `src/features/community/hooks/useCommunity.ts`

  - 페이지, 카테고리, 검색어 기반 게시글 목록을 요청한다.

- `src/features/community/CommunityMain.tsx`

  - 카테고리 필터, 검색 입력, 페이지네이션 UI를 제공한다.

  - `useDebounce`를 적용해 검색 요청 빈도를 줄였다.

- `src/hooks/useDebounce.ts`

  - 일정 시간 동안 입력이 멈춘 뒤 값이 반영되도록 처리한다.

- `src/features/community/CommunityDetail.tsx`

  - 게시글 상세, 댓글 등록, 수정/삭제 관련 동작을 관리한다.

  

설명 포인트

  

- 검색어를 즉시 query key에 넣지 않고 debounce된 값으로 연결했다.

- 커뮤니티 목록은 사용자가 특정 페이지로 이동할 가능성이 높아 페이지네이션을 택했다.

- 상세 페이지는 게시글, 댓글, 수정/삭제 모달 등 상호작용이 많은 구조로 분리했다.

  

예상 질문

  

`검색 최적화에서 debounce를 적용한 이유는 무엇인가요?`

  

답변 예시

  

`검색창은 입력마다 상태가 변하므로 그대로 요청을 보내면 API 호출 수가 과도해질 수 있습니다. 그래서 사용자가 타이핑을 잠시 멈춘 뒤에만 요청이 나가도록 해서 성능과 UX를 모두 개선했습니다.`

  

---

  

### 7. TanStack Query 기반 서버 상태 관리

  

면접에서 이렇게 말하면 된다.

  

`서버에서 가져오는 데이터는 로컬 UI 상태와 성격이 달라서 TanStack Query로 분리 관리했습니다. 특히 퀴즈, 커뮤니티, 통계, 오답 목록처럼 갱신 주기와 캐싱 전략이 다른 데이터를 query key 기준으로 나눠 관리했습니다.`

  

코드 근거

  

- `src/queryKeys.ts`

  - 기능별 query key를 일관성 있게 정의한다.

- `src/features/quiz/hooks/useQuiz.ts`

  - 퀴즈 생성 쿼리와 통계 무효화 처리.

- `src/features/community/CommunityMain.tsx`

  - 게시글 목록 조회.

- `src/features/myPage/IncorrectAnswers/IncorrectAnswer.tsx`

  - 무한 쿼리 기반 오답 목록 조회.

- `src/features/myPage/UserStatistics/UserStatistics.tsx`

  - 사용자 통계 조회.

  

설명 포인트

  

- query key를 파일 하나에서 관리해 충돌과 실수를 줄였다.

- `invalidateQueries`로 데이터 일관성을 유지했다.

- `staleTime`을 기능별로 다르게 설정해 조회 빈도를 조절했다.

  

예상 질문

  

`Zustand로도 서버 데이터를 관리할 수 있는데 왜 TanStack Query를 썼나요?`

  

답변 예시

  

`서버 상태는 로딩, 에러, 캐싱, 재요청, 무효화 같은 요구가 많아서 전역 상태 라이브러리만으로 관리하면 직접 구현해야 할 로직이 많아집니다. TanStack Query는 그 부분을 표준화해줘서 유지보수성이 좋았습니다.`

  

---

  

### 8. UX 개선: Skeleton UI, Error Boundary, Toast

  

면접에서 이렇게 말하면 된다.

  

`프로젝트에서 기능 구현만큼 신경 쓴 부분이 예외 상황과 로딩 경험이었습니다. 데이터를 기다리는 동안에는 Skeleton UI를 보여주고, API 실패나 렌더링 오류가 났을 때는 각각에 맞는 에러 UI를 분리해 사용자 경험이 갑자기 끊기지 않도록 했습니다.`

  

코드 근거

  

- `src/components/GlobalErrorBoundary.tsx`

  - 렌더링 단계의 예외를 잡는다.

- `src/components/GlobalErrorFallback.tsx`

  - 전역 에러 발생 시 대체 UI를 렌더링한다.

- `src/components/ui/Skeleton/QuestionCardSkeleton.tsx`

  - 퀴즈 로딩 시 스켈레톤 표시.

- `src/components/ui/Skeleton/UserStatisticsSkeleton.tsx`

  - 통계 화면 로딩 시 스켈레톤 표시.

- `src/store/useToastStore.ts`

  - 성공/실패/경고 메시지를 전역 Toast로 관리한다.

  

설명 포인트

  

- 로딩, 비즈니스 에러, 렌더링 에러를 같은 문제로 보지 않고 각각 분리했다.

- 단순 alert 대신 Toast를 사용해 사용자 흐름을 방해하지 않도록 했다.

- Skeleton을 사용해 "멈춘 화면"이 아니라 "로딩 중인 화면"처럼 느껴지게 했다.

  

예상 질문

  

`Error Boundary와 API 에러 처리는 어떻게 구분했나요?`

  

답변 예시

  

`Error Boundary는 렌더링 단계에서 발생한 예외를 잡는 역할이고, API 실패는 query 상태나 try/catch로 제어했습니다. 둘은 발생 지점이 다르기 때문에 처리 계층도 나눴습니다.`

  

---

  

## 기술 스택 설명 템플릿

  

### React

  

`컴포넌트 기반으로 화면을 기능 단위로 나눌 수 있어서 퀴즈, 커뮤니티, 마이페이지를 각각 분리해서 관리하기 좋았습니다.`

  

### TypeScript

  

`퀴즈 데이터, 게시글 데이터, 통계 데이터처럼 구조가 다른 응답이 많아서 타입으로 계약을 명확히 하는 것이 유지보수에 도움이 됐습니다.`

  

### React Router

  

`퀴즈 선택, 퀴즈 풀이, 마이페이지, 커뮤니티처럼 페이지 흐름이 분명한 서비스라 라우팅 구조화가 중요했습니다. 보호 라우트도 함께 적용했습니다.`

  

### TanStack Query

  

`서버 상태를 캐싱하고 무효화하기 좋았고, 로딩/에러 상태 분리가 쉬워서 사용자 경험 개선에도 직접 도움이 됐습니다.`

  

### Zustand

  

`사용자 정보, 퀴즈 답안, 옵션 선택, Toast처럼 비교적 가벼운 클라이언트 상태를 빠르게 다루기 좋았습니다. persist도 간단히 붙일 수 있었습니다.`

  

### Axios

  

`공개 API와 인증 API를 인스턴스로 분리했고, 인증 API는 interceptor를 적용해 refresh 재시도 흐름을 넣었습니다.`

  

---

  

## 협업/설계 관점에서 어필할 포인트

  

- 기능별 폴더 구조로 관심사를 분리했다.

- 서버 상태와 클라이언트 상태를 분리해 관리했다.

- 인증이 필요한 API를 `privateInstance`로 분리해 재사용성과 안정성을 높였다.

- query key를 중앙 관리해 캐시 무효화 실수를 줄였다.

- 공통 훅(`useDebounce`, `useInfiniteScroll`)과 공통 UI(`Modal`, `Dropdown`, `Skeleton`)를 분리해 재사용했다.

  

---

  

## 아쉬웠던 점을 물어봤을 때 답변 예시

  

`아쉬웠던 점은 프론트만으로는 성능과 UX 개선을 많이 했지만, 백엔드 응답 구조와 에러 메시지 규격이 더 일관됐다면 프론트 코드도 더 단순해질 수 있었겠다는 부분입니다. 또 테스트 코드까지 충분히 가져가지는 못해서, 다음에는 퀴즈 흐름과 인증 흐름 중심으로 테스트 자동화를 보완하고 싶습니다.`

  

---

  

## 개선 방향을 물어봤을 때 답변 예시

  

- 퀴즈 생성/채점 플로우에 대한 E2E 테스트 추가

- 통계 화면 차트 시각화 강화

- 커뮤니티 검색 조건 고도화

- 오답 노트 기반 재추천 퀴즈 기능 추가

- 접근성과 키보드 사용성 개선

  

---

  

## 꼬리 질문 대비 한 문장 답변 모음

  

`왜 이 프로젝트를 만들었나요?`

  

`프론트엔드 학습 과정에서 이론 암기보다 반복 문제 풀이와 복습 흐름이 중요하다고 느껴서, 학습과 기록 관리를 함께 할 수 있는 서비스로 만들었습니다.`

  

`가장 신경 쓴 부분은 무엇인가요?`

  

`사용자가 문제를 풀고 끝나는 게 아니라, 채점 결과가 오답 저장과 통계까지 연결되는 학습 흐름 전체를 끊기지 않게 만드는 데 가장 신경 썼습니다.`

  

`가장 어려웠던 부분은 무엇인가요?`

  

`서버 상태와 로컬 상태가 섞이면 복잡해지기 쉬워서, TanStack Query와 Zustand의 역할을 분리하는 기준을 잡는 과정이 가장 중요했고 어려웠습니다.`

  

`본인이 잘했다고 생각하는 부분은 무엇인가요?`

  

`기능 구현에 그치지 않고 로그인 유지, 에러 처리, 스켈레톤, 검색 최적화처럼 실제 사용 경험에 영향을 주는 부분까지 함께 챙긴 점입니다.`

  

---

  

## 코드 확인 추천 순서

  

면접 직전에는 아래 순서로 코드만 빠르게 다시 보면 된다.

  

1. `src/page/OptionSelectPage.tsx`

2. `src/features/quiz/hooks/useQuiz.ts`

3. `src/features/quiz/QuizScreen.tsx`

4. `src/store/useUserStore.ts`

5. `src/apis/privateInstance.ts`

6. `src/features/myPage/hooks/useIncorrectAnswers.ts`

7. `src/features/myPage/IncorrectAnswers/IncorrectAnswer.tsx`

8. `src/features/myPage/UserStatistics/UserStatistics.tsx`

9. `src/features/community/CommunityMain.tsx`

10. `src/hooks/useDebounce.ts`

11. `src/hooks/useInfiniteScroll.ts`

12. `src/components/GlobalErrorBoundary.tsx`

  

---

  

## 마지막 면접용 정리 멘트

  

`FrontQuiz는 React 기반으로 퀴즈 생성, 채점, 오답 관리, 통계, 커뮤니티까지 이어지는 학습 서비스를 구현한 프로젝트입니다. 저는 이 프로젝트에서 단순 화면 구현보다도 사용자 흐름이 자연스럽게 이어지도록 상태 관리, 캐싱, 인증 유지, 로딩/에러 UX를 설계한 경험을 강조해서 설명할 수 있습니다.`