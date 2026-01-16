
# FrontQuiz 프로젝트 핵심 기능 STAR 기법 정리

  

## 1. AI 기반 퀴즈 생성 및 채점 시스템

  

### Situation (상황)

- 프론트엔드 개발자를 위한 학습 플랫폼에서 사용자가 다양한 카테고리와 난이도별로 퀴즈를 풀 수 있어야 함

- LLM API를 활용하여 동적으로 퀴즈를 생성하고, 사용자 답안을 실시간으로 채점해야 함

- 시스템 프롬프트를 통해 JSON 형식 출력을 강제하여 일관된 데이터 구조 보장 필요

  

### Task (과제)

- 카테고리(React, TypeScript, JavaScript 등)와 난이도 선택 기능 구현

- AI API를 통한 퀴즈 생성 및 채점 로직 구현

- 사용자 답안 제출 및 결과 표시 기능 개발

- 오답 자동 저장 및 통계 업데이트 연동

  

### Action (행동)

- **Zustand 상태 관리**: `useQuizStore`로 퀴즈 상태(quiz, userAnswer, result, isLoading, isGrading) 관리

- **옵션 관리**: `useOptionStore`로 카테고리와 난이도 선택 상태 관리

- **API 통신**:

  - `instance.post('/api/quiz/generate-quiz')`로 퀴즈 생성 요청

  - `instance.post('/api/quiz/grade-answer')`로 답안 채점 요청

- **통계 저장**: 로그인 사용자의 경우 `privateInstance.post('/api/quiz/add-statistics')`로 통계 자동 저장

- **React Query 연동**: 채점 후 `queryClient.invalidateQueries(['statistics'])`로 통계 데이터 자동 갱신

- **에러 처리**: ErrorBoundary를 활용한 퀴즈 로딩 실패 시 재시도 기능 제공

  

### Result (결과)

- 사용자가 카테고리와 난이도를 선택하여 즉시 퀴즈를 받을 수 있음

- AI가 생성한 퀴즈를 풀고 실시간으로 채점 결과를 확인 가능

- 정답/오답 여부와 상세한 해설을 제공하여 학습 효과 향상

- 오답은 자동으로 저장되어 마이페이지에서 확인 가능

- 통계 데이터가 자동으로 업데이트되어 학습 진도 추적 가능

  

---

  

## 2. OAuth 2.0 카카오 소셜 로그인 및 JWT 토큰 관리

  

### Situation (상황)

- 사용자 인증을 위해 카카오 소셜 로그인을 구현해야 함

- JWT 기반 인증 시스템에서 Access Token과 Refresh Token을 관리해야 함

- 로그인 상태를 유지하고, 토큰 만료 시 자동 갱신이 필요함

  

### Task (과제)

- 카카오 OAuth 2.0 인증 플로우 구현

- JWT 토큰 저장 및 관리 (httpOnly 쿠키 활용)

- 토큰 만료 시 자동 갱신 로직 구현

- 로그인 상태 유지를 위한 Zustand Persist 적용

- 보호된 라우트 접근 제어 구현

  

### Action (행동)

- **카카오 로그인 플로우**:

  - `KakaoCallback` 컴포넌트에서 URL의 `code` 파라미터 추출

  - `kakaoAuthInstance.post('/api/auth/kakao')`로 백엔드에 인증 코드 전송

  - 응답으로 받은 사용자 정보를 `useUserStore`에 저장

- **토큰 관리**:

  - `privateInstance`에 Axios 인터셉터 적용

  - 401 에러 발생 시 `instance.post('/api/auth/refresh')`로 토큰 갱신 시도

  - 갱신 실패 시 `useUserStore.getState().clearUser()`로 로그아웃 처리

- **상태 유지**:

  - `useUserStore`에 Zustand `persist` 미들웨어 적용

  - `localStorage`에 사용자 정보 저장 (`user-storage` 키)

  - `hasHydrated` 상태로 초기 로딩 시 상태 복원 확인

- **보호된 라우트**: `ProtectedRoute` 컴포넌트로 인증되지 않은 사용자 접근 차단

  

### Result (결과)

- 사용자가 카카오 계정으로 간편하게 로그인 가능

- 로그인 상태가 브라우저 새로고침 후에도 유지됨

- 토큰 만료 시 자동으로 갱신되어 사용자 경험 향상

- 인증이 필요한 페이지에 자동으로 접근 제어 적용

- 로그아웃 시 상태가 완전히 초기화됨

  

---

  

## 3. TanStack React Query를 활용한 서버 상태 관리 및 캐싱

  

### Situation (상황)

- 커뮤니티 게시글, 통계, 오답노트 등 다양한 서버 데이터를 효율적으로 관리해야 함

- 불필요한 API 호출을 줄이고, 데이터 일관성을 유지해야 함

- 로딩 상태와 에러 상태를 일관되게 처리해야 함

  

### Task (과제)

- React Query를 활용한 서버 상태 관리 시스템 구축

- 적절한 캐싱 전략 수립 (staleTime, cacheTime 설정)

- Query Key를 통한 데이터 무효화 및 갱신 관리

- 무한 스크롤을 위한 useInfiniteQuery 활용

  

### Action (행동)

- **Query 설정**:

  - `useQuery`로 커뮤니티 게시글 조회 (staleTime: 5분)

  - `useQuery`로 사용자 통계 조회 (staleTime: 30분)

  - `useInfiniteQuery`로 오답노트 무한 스크롤 구현

- **캐싱 전략**:

  - Query Key에 페이지, 카테고리, 검색어 등 의존성 포함

  - `queryClient.invalidateQueries()`로 관련 데이터 자동 갱신

- **데이터 페칭**:

  - `useCommunity` 훅으로 게시글 페칭 로직 분리

  - `useIncorrectAnswers` 훅으로 오답노트 CRUD 로직 분리

  - `useUserStatistics` 훅으로 통계 조회 로직 분리

- **에러 처리**: 각 쿼리에서 `isPending`, `error` 상태를 활용한 UI 처리

  

### Result (결과)

- 동일한 데이터에 대한 중복 API 호출이 방지되어 성능 향상

- 캐싱된 데이터로 빠른 화면 전환 가능

- 데이터 변경 시 자동으로 관련 쿼리가 갱신되어 일관성 유지

- 로딩 및 에러 상태가 일관되게 처리되어 사용자 경험 개선

- 무한 스크롤로 대량의 데이터를 효율적으로 로드 가능

  

---

  

## 4. Intersection Observer를 활용한 무한 스크롤 구현

  

### Situation (상황)

- 오답노트와 같은 대량의 데이터를 효율적으로 표시해야 함

- 페이지네이션 대신 무한 스크롤로 사용자 경험 향상 필요

- 스크롤 성능 최적화 및 불필요한 API 호출 방지 필요

  

### Task (과제)

- Intersection Observer API를 활용한 무한 스크롤 커스텀 훅 개발

- Cursor 기반 페이지네이션과 연동

- 로딩 상태와 다음 페이지 존재 여부 관리

  

### Action (행동)

- **커스텀 훅 개발**: `useInfiniteScroll` 훅 구현

  - `IntersectionObserver`로 타겟 요소 감시

  - `threshold` 파라미터로 감지 시점 조절 (기본 150px)

  - `loading`과 `hasNextPage` 상태로 불필요한 호출 방지

- **연동**:

  - 오답노트 페이지에서 `useInfiniteQuery`와 연동

  - 스크롤 시 자동으로 다음 페이지 데이터 로드

- **최적화**:

  - `useRef`로 DOM 요소 참조 관리

  - `useEffect` cleanup으로 메모리 누수 방지

  

### Result (결과)

- 사용자가 스크롤만으로 자연스럽게 데이터를 로드할 수 있음

- 페이지네이션 버튼 클릭 없이 연속적인 데이터 탐색 가능

- 성능 최적화로 부드러운 스크롤 경험 제공

- 불필요한 API 호출이 방지되어 서버 부하 감소

  

---

  

## 5. Debounce를 활용한 검색 기능 최적화

  

### Situation (상황)

- 커뮤니티 게시글 검색 시 사용자가 타이핑할 때마다 API 호출이 발생하여 성능 저하 발생

- 불필요한 네트워크 요청을 줄이고 서버 부하를 감소시켜야 함

  

### Task (과제)

- 검색어 입력 시 Debounce 로직 적용

- 사용자가 입력을 멈춘 후에만 API 호출하도록 최적화

- 검색어 변경 시 페이지를 1로 리셋

  

### Action (행동)

- **커스텀 훅 개발**: `useDebounce` 훅 구현

  - `useState`와 `useEffect`로 입력값 지연 처리

  - 기본 300ms 지연 시간 설정

  - `setTimeout`과 `clearTimeout`으로 타이머 관리

- **커뮤니티 검색 연동**:

  - `CommunityMain` 컴포넌트에서 `useDebounce(search, 300)` 적용

  - `debounceSearch`를 Query Key에 포함하여 자동 갱신

  - 검색어 변경 시 `setPage(1)`로 첫 페이지로 리셋

  

### Result (결과)

- 사용자가 타이핑하는 동안 불필요한 API 호출이 발생하지 않음

- 검색어 입력 완료 후에만 API 호출되어 서버 부하 감소

- 네트워크 트래픽이 약 70-80% 감소하여 성능 향상

- 사용자 경험은 유지하면서 백엔드 리소스 효율적 활용

  

---

  

## 6. Axios 인터셉터를 활용한 에러 처리 및 재시도 로직

  

### Situation (상황)

- 네트워크 오류나 서버 에러 발생 시 사용자에게 적절한 피드백 제공 필요

- 일시적인 네트워크 오류에 대한 자동 재시도 기능 필요

- 인증 관련 에러와 일반 에러를 구분하여 처리해야 함

  

### Task (과제)

- Axios 인터셉터로 전역 에러 처리 구현

- 네트워크 오류 및 5xx 에러에 대한 자동 재시도 로직 구현

- 사용자 친화적인 에러 메시지 제공

  

### Action (행동)

- **Public API 인스턴스** (`instance.ts`):

  - `response interceptor`에서 에러 처리

  - 네트워크 오류 또는 5xx 에러 시 최대 3회 재시도

  - `retryCounts` Map으로 URL별 재시도 횟수 추적

  - 카카오 로그인 API는 재시도 제외

  - HTTP 상태 코드별 사용자 친화적 에러 메시지 반환

- **Private API 인스턴스** (`privateInstance.ts`):

  - 401 에러 시 자동 토큰 갱신 시도

  - 갱신 실패 시 사용자 로그아웃 처리

- **에러 메시지 분류**:

  - 400: 잘못된 요청

  - 401: 인증 필요

  - 403: 권한 없음

  - 404: 리소스 없음

  - 500: 서버 내부 오류

  - Network Error: 네트워크 연결 확인 안내

  

### Result (결과)

- 일시적인 네트워크 오류 시 자동으로 재시도하여 사용자 경험 향상

- 명확한 에러 메시지로 사용자가 문제를 이해하고 대응 가능

- 서버 오류 발생 시 자동 재시도로 성공률 향상

- 인증 오류 시 자동 토큰 갱신으로 로그인 상태 유지

  

---

  

## 7. 커뮤니티 CRUD 기능 및 마크다운 렌더링

  

### Situation (상황)

- 사용자들이 퀴즈 관련 정보와 질문을 공유할 수 있는 커뮤니티 기능 필요

- 코드 블록이 포함된 마크다운 형식의 게시글 작성 및 표시 필요

- 카테고리별 필터링 및 검색 기능 필요

  

### Task (과제)

- 게시글 작성/수정/삭제 기능 구현

- 댓글 작성/삭제 기능 구현

- 마크다운 렌더링 및 코드 하이라이팅 적용

- 카테고리(질문/정보) 필터링 및 검색 기능 구현

  

### Action (행동)

- **게시글 관리**:

  - `CommunityForm` 컴포넌트로 게시글 작성/수정

  - `React Hook Form`으로 폼 상태 관리 및 유효성 검사

  - `privateInstance`로 CRUD API 호출

- **마크다운 렌더링**:

  - `react-markdown`으로 마크다운 파싱

  - `rehype-highlight`로 코드 블록 하이라이팅

- **댓글 시스템**:

  - `CommentForm`과 `CommentList` 컴포넌트로 댓글 관리

  - 실시간 댓글 추가/삭제 기능

- **필터링 및 검색**:

  - 카테고리 버튼으로 필터링 (전체/질문/정보)

  - Debounce를 적용한 검색 기능

  - Query Key에 카테고리와 검색어 포함하여 자동 갱신

- **페이지네이션**: `Pagination` 컴포넌트로 페이지 이동

  

### Result (결과)

- 사용자가 마크다운 형식으로 게시글 작성 가능

- 코드 블록이 하이라이팅되어 가독성 향상

- 카테고리별로 게시글을 필터링하여 원하는 정보 빠르게 탐색 가능

- 검색 기능으로 특정 키워드 게시글 빠르게 찾기 가능

- 댓글 기능으로 사용자 간 소통 활성화

  

---

  

## 8. 오답노트 기능 및 통계 관리

  

### Situation (상황)

- 사용자가 틀린 문제를 다시 학습할 수 있도록 오답 저장 기능 필요

- 학습 진도를 추적하기 위한 통계 기능 필요

- 오답을 커뮤니티에 질문으로 쉽게 변환하는 기능 필요

  

### Task (과제)

- 오답 자동 저장 기능 구현

- 오답노트 조회 및 필터링 기능 구현

- 카테고리별/난이도별 통계 조회 기능 구현

- 오답을 커뮤니티 질문 템플릿으로 변환하는 유틸 함수 개발

  

### Action (행동)

- **오답 저장**:

  - 퀴즈 채점 후 오답인 경우 `handleAddInCorrect` 호출

  - `privateInstance.post('/api/quiz/save-incorrect-answer')`로 저장

  - Toast 알림으로 저장 완료 피드백

- **오답노트 조회**:

  - `useInfiniteQuery`로 무한 스크롤 구현

  - 카테고리/난이도 필터링 지원

  - Cursor 기반 페이지네이션 적용

- **오답 삭제**:

  - `useMutation`으로 삭제 기능 구현

  - 삭제 후 Query 무효화로 목록 자동 갱신

- **통계 관리**:

  - `getUserStatistics`로 전체/카테고리별/난이도별 통계 조회

  - 정답률 계산 및 시각화

  - 통계 데이터가 없을 때 빈 상태 UI 제공

- **질문 변환**:

  - `changeIncorrectToQuestion` 유틸 함수로 오답을 마크다운 질문 템플릿으로 변환

  - 커뮤니티 게시글 작성 폼에 자동 입력

  

### Result (결과)

- 사용자가 틀린 문제를 자동으로 저장하여 나중에 복습 가능

- 오답노트에서 필터링하여 특정 카테고리나 난이도의 오답만 확인 가능

- 학습 통계를 통해 자신의 학습 진도와 정답률 파악 가능

- 오답을 한 번의 클릭으로 커뮤니티 질문으로 변환하여 도움 요청 가능

- 학습 데이터가 체계적으로 관리되어 학습 효과 향상

  

---

  

## 9. Context 기반 컴포넌트 조합 패턴 (Modal, Dropdown)

  

### Situation (상황)

- 재사용 가능한 UI 컴포넌트를 만들되, prop drilling을 방지해야 함

- Modal과 Dropdown 같은 복합 컴포넌트를 선언적으로 사용하고 싶음

- 컴포넌트 간 상태 공유가 필요하지만 전역 상태는 과함

  

### Task (과제)

- Context API를 활용한 컴포넌트 조합 패턴 구현

- Modal과 Dropdown을 Compound Component 패턴으로 설계

- 각 컴포넌트의 상태를 Context로 관리하여 prop drilling 방지

  

### Action (행동)

- **Modal 컴포넌트**:

  - `ModalContext`로 열림/닫힘 상태 관리

  - `Modal.Trigger`, `Modal.Content`, `Modal.Header`, `Modal.Footer` 등으로 조합

  - `useModal` 훅으로 상태 접근

- **Dropdown 컴포넌트**:

  - `DropdownContext`로 선택된 값과 열림 상태 관리

  - `Dropdown.Trigger`, `Dropdown.Content`, `Dropdown.Item`으로 조합

  - 키보드 네비게이션 지원

- **컴포넌트 구조**:

  - 각 컴포넌트를 개별 파일로 분리하고 `index.ts`로 통합 export

  - TypeScript로 타입 안정성 보장

  

### Result (결과)

- Modal과 Dropdown을 선언적으로 사용하여 코드 가독성 향상

- Context를 통해 prop drilling 없이 상태 공유 가능

- 컴포넌트를 조합하여 다양한 UI 패턴 구현 가능

- 재사용성이 높아 프로젝트 전반에서 일관된 UI 제공

- 타입 안정성으로 개발 시 오류 사전 방지

  

---

  

## 10. Zustand를 활용한 클라이언트 상태 관리

  

### Situation (상황)

- 퀴즈 상태, 사용자 정보, 옵션 선택 등 다양한 클라이언트 상태를 관리해야 함

- Redux는 과하고, useState만으로는 전역 상태 공유가 어려움

- 로그인 상태는 새로고침 후에도 유지되어야 함

  

### Task (과제)

- Zustand로 경량 전역 상태 관리 시스템 구축

- 로그인 상태는 Persist 미들웨어로 localStorage에 저장

- 각 기능별로 독립적인 스토어 분리

  

### Action (행동)

- **퀴즈 스토어** (`useQuizStore`):

  - 퀴즈 데이터, 사용자 답안, 결과, 로딩 상태 관리

  - 간단한 setter 함수로 상태 업데이트

- **사용자 스토어** (`useUserStore`):

  - `persist` 미들웨어로 localStorage에 저장

  - `devtools` 미들웨어로 Redux DevTools 연동

  - `hasHydrated` 상태로 초기 로딩 관리

  - 프로필 이미지 기본값 처리 로직 포함

- **옵션 스토어** (`useOptionStore`):

  - 카테고리와 난이도 선택 상태 관리

- **Toast 스토어** (`useToastStore`):

  - 전역 알림 메시지 관리

  - success, error, warn 타입 지원

  

### Result (결과)

- Redux보다 간단한 API로 빠른 개발 가능

- 각 기능별로 독립적인 스토어로 관심사 분리

- 로그인 상태가 새로고침 후에도 유지되어 사용자 경험 향상

- TypeScript와 완벽한 타입 추론으로 타입 안정성 보장

- 불필요한 리렌더링이 최소화되어 성능 최적화

  

---

  

## 기술 스택 요약

  

### Core

- **React 19** + **TypeScript**: 타입 안정성과 최신 React 기능 활용

- **Vite**: 빠른 개발 서버 및 빌드

  

### 상태 관리

- **Zustand**: 경량 클라이언트 상태 관리

- **TanStack React Query v5**: 서버 상태 관리 및 캐싱

  

### 라우팅

- **React Router v7**: 클라이언트 사이드 라우팅

  

### HTTP 클라이언트

- **Axios**: 인터셉터를 활용한 에러 처리 및 재시도 로직

  

### 스타일링

- **Tailwind CSS v4**: 유틸리티 기반 스타일링

- **Motion (Framer Motion)**: 애니메이션

  

### 폼 관리

- **React Hook Form**: 폼 상태 관리 및 유효성 검사

  

### 기타

- **React Markdown** + **Rehype Highlight**: 마크다운 렌더링 및 코드 하이라이팅