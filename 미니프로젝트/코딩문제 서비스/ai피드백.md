
# 프로젝트 상세 분석 리포트

  

## 1. 🔍 코드 퀄리티 & 패턴 분석

  

### ✅ 잘 구현된 부분

  

#### 1.1 타입 안전성

- **Zustand 스토어 타입 정의**: `useQuizStore`, `useOptionStore`, `useUserStore` 모두 명확한 타입 정의

- **커스텀 훅 타입**: `useDebounce`, `useInfiniteScroll` 등 인터페이스 정의가 명확함

- **API 응답 타입**: `PostResponse`, `Quiz`, `Result` 등 도메인 타입이 잘 분리됨

  

#### 1.2 상태 관리 패턴

- **Zustand 활용**: 전역 상태를 Zustand로 깔끔하게 관리

- **React Query 통합**: 서버 상태와 클라이언트 상태를 적절히 분리

- **Persist 미들웨어**: 사용자 정보와 옵션을 로컬스토리지에 저장하여 새로고침 후에도 유지

  

#### 1.3 컴포넌트 구조

- **Feature 기반 폴더 구조**: `features/Community`, `features/quizComp` 등 도메인별로 잘 분리

- **재사용 가능한 컴포넌트**: `Button`, `Input`, `Modal` 등 UI 컴포넌트가 잘 추상화됨

- **컴포넌트 합성**: `Dropdown`, `Modal` 등 Compound Component 패턴 활용

  

#### 1.4 에러 처리

- **ErrorBoundary**: 퀴즈 영역에 에러 바운더리 적용

- **Axios Interceptor**: 전역 에러 처리 및 재시도 로직 구현

- **토스트 시스템**: 사용자 액션에 대한 피드백 제공

  

### ⚠️ 개선이 필요한 부분

  

#### 1.1 TypeScript `any` 사용 (중요도: 높음)

  

**문제점:**

```typescript

// src/features/Community/components/communityDetail/CommentForm.tsx:11

mutate: any;

  

// src/features/Community/hooks/useCommunity.ts:72, 90, 119, 138

onError: (error: any) => { ... }

```

  

**문제:**

- 타입 안전성 상실

- IDE 자동완성 및 타입 체크 불가

- 런타임 에러 가능성 증가

  

**개선 방안:**

```typescript

// CommentForm.tsx

import type { UseMutationResult } from '@tanstack/react-query';

  

interface CommentFormProps {

  postId: string;

  mutate: UseMutationResult<

    unknown,

    Error,

    { postId: string; nickname: string; content: string; password?: string }

  >;

  isLoading: boolean;

}

  

// useCommunity.ts

onError: (error: Error) => {

  const message = error instanceof Error

    ? error.message

    : '알 수 없는 오류가 발생했습니다.';

  addToast('error', message);

}

```

  

#### 1.2 오타 및 네이밍 (중요도: 낮음)

  

**문제점:**

```typescript

// src/features/quizComp/QuestionCard.tsx:24

const markdownContnet = quiz?.question; // 'Contnet' -> 'Content'

```

  

**문제:**

- 오타로 인한 가독성 저하

- 일관성 없는 네이밍

  

#### 1.3 React.memo 사용 시 의존성 배열 부재 (중요도: 중간)

  

**문제점:**

```typescript

// src/features/quizComp/QuestionCard.tsx:35

export default React.memo(QuestionCard);

  

// src/features/quizComp/OptionCard.tsx:132

export default React.memo(OptionsCard);

```

  

**문제:**

- `React.memo`에 비교 함수가 없어 props가 얕은 비교만 수행

- 객체나 함수 props가 매번 새로 생성되면 메모이제이션 효과 없음

  

**개선 방안:**

```typescript

// QuestionCard는 props가 원시 타입이므로 현재도 괜찮지만,

// OptionsCard는 useQuizStore를 직접 사용하므로 메모이제이션 효과 제한적

// OptionsCard 내부에서 zustand selector를 사용하는 것이 더 효과적

```

  

#### 1.4 Zustand Store에서 불필요한 렌더링 가능성 (중요도: 중간)

  

**문제점:**

```typescript

// src/store/useQuizStore.ts

export const useQuizStore = create<QuizState>((set) => ({

  quiz: null,

  userAnswer: '',

  result: null,

  isLoading: false,

  isGrading: false,

  // ...

}));

  

// 사용 예시

const quiz = useQuizStore((s) => s.quiz);

const userAnswer = useQuizStore((s) => s.userAnswer);

const setUserAnswer = useQuizStore((s) => s.setUserAnswer);

```

  

**문제:**

- 여러 개의 selector를 사용하면 각각 구독이 생성됨

- 하나의 액션으로 여러 상태가 변경되면 여러 번 리렌더링 가능

  

**개선 방안:**

```typescript

// 여러 상태를 한 번에 선택

const { quiz, userAnswer, setUserAnswer } = useQuizStore((s) => ({

  quiz: s.quiz,

  userAnswer: s.userAnswer,

  setUserAnswer: s.setUserAnswer,

}));

  

// 또는 shallow 비교 사용

import { shallow } from 'zustand/shallow';

const { quiz, userAnswer } = useQuizStore(

  (s) => ({ quiz: s.quiz, userAnswer: s.userAnswer }),

  shallow

);

```

  

#### 1.5 Axios Interceptor에서 Zustand 직접 호출 (중요도: 높음)

  

**문제점:**

```typescript

// src/apis/privateInstance.ts:29

useUserStore.getState().clearUser();

```

  

**문제:**

- 인터셉터는 React 컴포넌트 외부에서 실행됨

- React의 생명주기와 무관한 곳에서 스토어 접근

- 테스트 어려움 및 사이드 이펙트 예측 불가

  

**개선 방안:**

```typescript

// 인터셉터는 순수 함수로 유지하고,

// 에러 발생 시 특정 상태 코드를 반환하거나

// 이벤트를 발생시켜 컴포넌트에서 처리하도록 변경

  

// 또는 별도의 authService 생성

// src/services/authService.ts

export const authService = {

  clearUser: () => useUserStore.getState().clearUser(),

};

```

  

#### 1.6 ErrorBoundary에서 throw error 패턴 (중요도: 중간)

  

**문제점:**

```typescript

// src/features/quizComp/QuestionCard.tsx:12-14

if (error) {

  throw error;

}

```

  

**문제:**

- ErrorBoundary는 React 렌더링 중 발생한 에러만 캐치

- 의도적으로 throw하는 것은 일반적이지 않은 패턴

- 에러 상태를 props로 전달하는 것이 더 명확

  

**개선 방안:**

```typescript

// ErrorBoundary 없이 에러 상태를 직접 처리

if (error) {

  return <ErrorDisplay error={error} onRetry={fetchQuiz} />;

}

```

  

#### 1.7 useScroll 훅의 의존성 배열 문제 (중요도: 중간)

  

**문제점:**

```typescript

// src/hooks/useScroll.ts:36

}, [options]);

```

  

**문제:**

- `options` 객체가 매번 새로 생성되면 useEffect가 계속 실행됨

- IntersectionObserver가 불필요하게 재생성될 수 있음

  

**개선 방안:**

```typescript

useEffect(() => {

  // ...

}, [

  options?.threshold,

  options?.rootMargin,

  options?.freezeOnceVisible,

  options?.root,

]);

```

  

#### 1.8 Retry 로직의 메모리 누수 가능성 (중요도: 낮음)

  

**문제점:**

```typescript

// src/apis/instance.ts:47

const retryCounts = new Map<string, number>();

```

  

**문제:**

- Map이 계속 쌓일 수 있음 (성공한 요청의 카운트는 삭제되지만)

- 장기 실행 시 메모리 사용량 증가 가능

  

**개선 방안:**

```typescript

// LRU 캐시 사용 또는 최대 크기 제한

// 또는 성공/실패 여부와 관계없이 일정 시간 후 자동 삭제

```

  

---

  

## 2. 🎤 꼬리물기 식 기술 면접 질문

  

### 질문 1: Zustand와 React Query를 함께 사용한 이유는?

  

**면접관 질문:**

> "이 프로젝트에서 Zustand와 React Query를 함께 사용하셨는데, 각각의 역할을 어떻게 구분하셨나요? 그리고 `useQuizStore`는 왜 Zustand로 관리하셨나요? React Query의 `useMutation`으로도 충분하지 않았을까요?"

  

**모범 답안:**

```

Zustand는 클라이언트 상태(UI 상태, 폼 상태)를, React Query는 서버 상태(API 데이터)를 관리하기 위해 구분했습니다.

  

useQuizStore의 경우:

- quiz, userAnswer, result는 퀴즈 화면 내에서만 사용되는 일시적인 상태입니다

- 페이지를 벗어나면 초기화되어야 하는 임시 상태이므로 서버에 저장할 필요가 없습니다

- React Query는 서버 데이터의 캐싱, 동기화에 최적화되어 있어, 이런 임시 UI 상태에는 과한 도구입니다

  

반면 커뮤니티 게시글은:

- 서버에서 가져오는 데이터이므로 React Query 사용

- 캐싱, 자동 리프레시, 무한 스크롤 등 React Query의 기능 활용

  

트레이드오프:

- Zustand만 사용하면 캐싱, 리프레시 로직을 직접 구현해야 함

- React Query만 사용하면 클라이언트 상태까지 서버처럼 관리하게 되어 복잡도 증가

```

  

**핵심 키워드:**

- 서버 상태 vs 클라이언트 상태

- 캐싱 전략

- 상태 관리 도구 선택 기준

- 관심사의 분리 (Separation of Concerns)

  

---

  

### 질문 2: ErrorBoundary를 퀴즈 컴포넌트에만 적용한 이유는?

  

**면접관 질문:**

> "ErrorBoundary를 `QuizScreen`의 `QuestionCard`에만 적용하셨는데, 다른 페이지에는 왜 적용하지 않으셨나요? 그리고 `QuestionCard`에서 `if (error) throw error`로 에러를 던지는 패턴은 일반적이지 않은데, 이렇게 구현한 특별한 이유가 있나요?"

  

**모범 답안:**

```

퀴즈 컴포넌트에만 적용한 이유:

- 퀴즈는 사용자가 직접 문제를 요청하는 동적 콘텐츠이므로 에러 발생 가능성이 높습니다

- 다른 페이지(커뮤니티, 통계)는 이미 로드된 데이터를 보여주므로 에러 처리가 다릅니다

- 퀴즈에서만 "다시 시도" 기능이 의미가 있습니다

  

throw error 패턴의 문제점:

- 실제로는 이 패턴이 적절하지 않았습니다

- ErrorBoundary는 예상치 못한 에러를 캐치하는 용도이고,

  예상 가능한 에러는 조건부 렌더링으로 처리하는 것이 더 명확합니다

  

개선 방안:

- error 상태를 props로 받아서 ErrorDisplay 컴포넌트를 렌더링

- 또는 React Query의 error 상태를 직접 활용

```

  

**핵심 키워드:**

- ErrorBoundary의 목적과 사용 시점

- 예상 가능한 에러 vs 예상치 못한 에러

- 에러 처리 전략

- 사용자 경험 (UX) 고려

  

---

  

### 질문 3: Axios Interceptor에서 Zustand Store를 직접 호출한 이유는?

  

**면접관 질문:**

> "`privateInstance.ts`의 인터셉터에서 `useUserStore.getState().clearUser()`를 직접 호출하셨는데, 이게 React 컴포넌트 외부에서 실행되는데 문제가 없나요? 그리고 이렇게 하면 테스트는 어떻게 하셨나요?"

  

**모범 답안:**

```

현재 구현의 문제점:

- 인터셉터는 React 생명주기와 무관하게 실행되므로,

  컴포넌트 외부에서 스토어를 직접 조작하는 것은 사이드 이펙트가 예측하기 어렵습니다

- 테스트 시 인터셉터를 모킹하기 어렵고, 스토어 의존성이 숨겨져 있습니다

  

왜 이렇게 했는지:

- 401 에러 발생 시 즉시 사용자 상태를 초기화해야 했고,

  가장 간단한 방법이었습니다

  

개선 방안:

1. 인터셉터는 에러만 반환하고, 컴포넌트에서 처리

2. 별도의 authService 레이어 생성

3. 이벤트 기반 아키텍처 (EventEmitter) 사용

  

실제로는 React Query의 onError에서 처리하거나,

전역 에러 핸들러를 만드는 것이 더 React스러운 방법입니다.

```

  

**핵심 키워드:**

- 사이드 이펙트 관리

- 테스트 가능성 (Testability)

- 관심사의 분리

- React의 생명주기

- 의존성 주입 (Dependency Injection)

  

---

  

### 질문 4: useDebounce를 커스텀 훅으로 만든 이유와 구현 방식

  

**면접관 질문:**

> "검색 기능에 debounce를 적용하셨는데, 왜 라이브러리(예: lodash)를 사용하지 않고 직접 구현하셨나요? 그리고 `useDebounce` 훅의 `delay` 파라미터를 의존성 배열에 넣은 이유는?"

  

**모범 답안:**

```

직접 구현한 이유:

1. 번들 사이즈: lodash는 전체 라이브러리가 크고, debounce만 필요했습니다

2. 의존성 최소화: 간단한 기능이므로 직접 구현이 더 가볍습니다

3. 타입 안전성: TypeScript로 명확하게 타입을 정의할 수 있습니다

  

delay를 의존성 배열에 넾은 이유:

- delay 값이 변경되면 새로운 타이머를 설정해야 하므로

- 하지만 실제로 delay는 거의 변경되지 않으므로,

  useRef로 delay를 저장하고 의존성에서 제외하는 것도 방법입니다

  

개선 가능한 부분:

- delay를 useRef로 저장하여 불필요한 재실행 방지

- 또는 delay를 옵션 객체로 받아서 useMemo로 메모이제이션

```

  

**핵심 키워드:**

- 번들 사이즈 최적화

- 의존성 관리

- 커스텀 훅 설계

- 성능 최적화

- 트레이드오프 (번들 사이즈 vs 개발 속도)

  

---

  

### 질문 5: React.memo를 사용했지만 실제로 최적화 효과가 있는가?

  

**면접관 질문:**

> "`QuestionCard`와 `OptionsCard`에 `React.memo`를 적용하셨는데, `OptionsCard`는 내부에서 `useQuizStore`를 여러 번 호출하고 있습니다. 이 경우 `React.memo`가 실제로 최적화 효과가 있나요? 그리고 어떤 상황에서 `React.memo`가 효과적일까요?"

  

**모범 답안:**

```

현재 구현의 문제점:

- OptionsCard는 props가 아닌 zustand store를 직접 구독합니다

- React.memo는 props 변경만 체크하므로, store 변경 시 리렌더링됩니다

- 따라서 React.memo의 효과가 제한적입니다

  

React.memo가 효과적인 경우:

1. 부모 컴포넌트가 자주 리렌더링되지만 props가 변하지 않는 경우 

2. 무거운 렌더링 로직이 있는 컴포넌트

3. props가 원시 타입이거나 안정적인 참조를 가진 경우

  

개선 방안:

1. OptionsCard 내부에서 zustand selector를 하나로 합치기

2. React.memo 제거하고 zustand의 selector 최적화에 집중

3. 또는 OptionsCard를 더 작은 컴포넌트로 분리

  

실제로는:

- zustand의 selector 최적화가 React.memo보다 더 중요합니다

- shallow 비교나 selector 최적화가 선행되어야 합니다

```

  

**핵심 키워드:**

- React.memo의 동작 원리

- 메모이제이션 전략

- 불필요한 리렌더링 방지

- 성능 측정 및 프로파일링

- 조기 최적화의 함정 (Premature Optimization)

  

---

  

## 3. 🔧 리팩토링이 필요한 부분

  

### 3.1 CommentForm의 타입 안전성 개선 (우선순위: 높음)

  

**현재 코드:**

```typescript

// src/features/Community/components/communityDetail/CommentForm.tsx

const CommentForm = ({

  postId,

  mutate,

  isLoading,

}: {

  postId: string;

  mutate: any; // ❌

  isLoading: boolean;

}) => { ... }

```

  

**개선 코드:**

```typescript

import type { UseMutationResult } from '@tanstack/react-query';

import type { AddCommentParams } from '../../hooks/useCommunity';

  

interface CommentFormProps {

  postId: string;

  mutate: UseMutationResult<

    unknown,

    Error,

    AddCommentParams

  >;

  isLoading: boolean;

}

  

const CommentForm = ({ postId, mutate, isLoading }: CommentFormProps) => {

  // ...

};

```

  

**왜 이렇게 하면 좋은가:**

- 타입 안전성 확보로 컴파일 타임에 에러 발견

- IDE 자동완성 지원으로 개발 생산성 향상

- 런타임 에러 가능성 감소

  

---

  

### 3.2 Zustand Store Selector 최적화 (우선순위: 중간)

  

**현재 코드:**

```typescript

// src/features/quizComp/OptionCard.tsx

const quiz = useQuizStore((s) => s.quiz);

const userAnswer = useQuizStore((s) => s.userAnswer);

const setUserAnswer = useQuizStore((s) => s.setUserAnswer);

const result = useQuizStore((s) => s.result);

```

  

**개선 코드:**

```typescript

import { shallow } from 'zustand/shallow';

  

const { quiz, userAnswer, result, setUserAnswer } = useQuizStore(

  (s) => ({

    quiz: s.quiz,

    userAnswer: s.userAnswer,

    result: s.result,

    setUserAnswer: s.setUserAnswer,

  }),

  shallow

);

```

  

**왜 이렇게 하면 좋은가:**

- 여러 selector를 하나로 합쳐 구독 수 감소

- shallow 비교로 불필요한 리렌더링 방지

- React.memo보다 더 효과적인 최적화

  

---

  

### 3.3 useScroll 훅의 의존성 배열 개선 (우선순위: 중간)

  

**현재 코드:**

```typescript

// src/hooks/useScroll.ts

useEffect(() => {

  // ...

}, [options]); // ❌ options 객체가 매번 새로 생성되면 계속 실행됨

```

  

**개선 코드:**

```typescript

useEffect(() => {

  const node = targetRef.current;

  if (!node) return;

  

  const observer = new IntersectionObserver(

    ([entry]) => {

      setIntersecting(entry.isIntersecting);

      if (options?.freezeOnceVisible && entry.isIntersecting) {

        observer.unobserve(node);

      }

    },

    {

      threshold: options?.threshold || 0.1,

      rootMargin: options?.rootMargin || '0px',

      root: options?.root || null,

    },

  );

  

  observer.observe(node);

  return () => observer.disconnect();

}, [

  options?.threshold,

  options?.rootMargin,

  options?.freezeOnceVisible,

  options?.root,

]);

```

  

**왜 이렇게 하면 좋은가:**

- options 객체의 참조가 아닌 실제 값 변경만 감지

- 불필요한 IntersectionObserver 재생성 방지

- 성능 향상 및 메모리 효율성 개선

  

---

  

### 3.4 ErrorBoundary 패턴 개선 (우선순위: 중간)

  

**현재 코드:**

```typescript

// src/features/quizComp/QuestionCard.tsx

if (error) {

  throw error; // ❌ 의도적으로 에러를 던지는 것은 일반적이지 않음

}

```

  

**개선 코드:**

```typescript

// QuestionCard.tsx

interface QuestionCardProps {

  error: Error | null;

  isLoading: boolean;

  quiz: Quiz | null;

  onRetry?: () => void;

}

  

const QuestionCard = ({ error, isLoading, quiz, onRetry }: QuestionCardProps) => {

  if (error) {

    return (

      <div className='flex flex-col items-center gap-4 p-10 text-white'>

        <p className='text-2xl'>퀴즈 불러오기 실패</p>

        <p className='text-lg text-red-400'>{error.message}</p>

        {onRetry && (

          <Button onClick={onRetry}>다시 시도</Button>

        )}

      </div>

    );

  }

  // ... 나머지 코드

};

  

// QuizScreen.tsx

<QuestionCard

  error={error}

  isLoading={isLoading}

  quiz={quiz}

  onRetry={fetchQuiz}

/>

```

  

**왜 이렇게 하면 좋은가:**

- 예상 가능한 에러는 조건부 렌더링으로 처리하는 것이 더 명확

- ErrorBoundary는 예상치 못한 에러를 위한 안전망

- 코드 의도가 더 명확해짐

  

---

  

### 3.5 privateInstance의 스토어 접근 개선 (우선순위: 높음)

  

**현재 코드:**

```typescript

// src/apis/privateInstance.ts

privateInstance.interceptors.response.use(

  (res) => res,

  async (error) => {

    // ...

    if (error.response?.status === 401 && !originalRequest._retry) {

      // ...

      try {

        await instance.post('/api/auth/refresh', {}, { withCredentials: true });

        return privateInstance(originalRequest);

      } catch (refreshError) {

        useUserStore.getState().clearUser(); // ❌

        return Promise.reject(refreshError);

      }

    }

    return Promise.reject(error);

  },

);

```

  

**개선 코드:**

```typescript

// src/services/authService.ts

import { useUserStore } from '@/store/useUserStore';

  

export const authService = {

  clearUser: () => {

    useUserStore.getState().clearUser();

  },

  // 다른 인증 관련 로직도 여기로 이동

};

  

// src/apis/privateInstance.ts

import { authService } from '@/services/authService';

  

privateInstance.interceptors.response.use(

  (res) => res,

  async (error) => {

    // ...

    if (error.response?.status === 401 && !originalRequest._retry) {

      // ...

      try {

        await instance.post('/api/auth/refresh', {}, { withCredentials: true });

        return privateInstance(originalRequest);

      } catch (refreshError) {

        authService.clearUser(); // ✅ 서비스 레이어를 통해 접근

        return Promise.reject(refreshError);

      }

    }

    return Promise.reject(error);

  },

);

```

  

**왜 이렇게 하면 좋은가:**

- 관심사의 분리: 인증 로직을 별도 서비스로 분리

- 테스트 용이성: authService를 모킹하기 쉬움

- 유지보수성: 인증 로직 변경 시 한 곳만 수정

- 사이드 이펙트 명확화

  

---

  

### 3.6 React.memo 제거 또는 개선 (우선순위: 낮음)

  

**현재 코드:**

```typescript

// src/features/quizComp/OptionCard.tsx

export default React.memo(OptionsCard);

```

  

**개선 방안 1: React.memo 제거**

```typescript

// OptionsCard는 zustand store를 직접 구독하므로

// React.memo의 효과가 제한적

// 제거하고 selector 최적화에 집중

export default OptionsCard;

```

  

**개선 방안 2: 비교 함수 추가**

```typescript

export default React.memo(OptionsCard, (prevProps, nextProps) => {

  // OptionsCard는 props가 없으므로 항상 true 반환

  // 하지만 실제로는 store 변경 시 리렌더링되므로 의미 없음

  return true;

});

```

  

**왜 이렇게 하면 좋은가:**

- 불필요한 최적화 제거로 코드 단순화

- 실제 최적화 효과가 있는 부분에 집중 (zustand selector)

- 조기 최적화의 함정 회피

  

---

  

## 📊 종합 평가

  

### 강점

1. ✅ **타입 안전성**: 대부분의 코드에서 TypeScript를 잘 활용

2. ✅ **상태 관리**: Zustand와 React Query의 역할 분리가 명확

3. ✅ **컴포넌트 구조**: Feature 기반 폴더 구조로 유지보수성 좋음

4. ✅ **에러 처리**: 전역 에러 처리 및 사용자 피드백 시스템 구축

5. ✅ **UX 고려**: 로딩 상태, 스켈레톤 UI, 토스트 등 사용자 경험 고려

  

### 개선 필요 사항

1. ⚠️ **타입 안전성**: `any` 타입 사용 지양 필요

2. ⚠️ **성능 최적화**: React.memo 사용 전 selector 최적화 필요

3. ⚠️ **아키텍처**: 인터셉터에서의 스토어 접근 방식 개선 필요

4. ⚠️ **에러 처리**: ErrorBoundary 사용 패턴 개선 필요

  

### 우선순위별 리팩토링 계획

1. **높음**: CommentForm 타입 개선, privateInstance 스토어 접근 개선

2. **중간**: Zustand selector 최적화, useScroll 의존성 배열, ErrorBoundary 패턴

3. **낮음**: React.memo 제거, 오타 수정, 네이밍 개선

  

---

  

## 🎯 면접 대비 체크리스트

  

### 기술적 깊이

- [ ] Zustand와 React Query의 차이와 사용 시점 설명 가능

- [ ] ErrorBoundary의 동작 원리와 사용 시나리오 이해

- [ ] React.memo와 useMemo의 차이 및 최적화 전략

- [ ] Axios Interceptor의 동작 원리와 React와의 통합 방법

  

### 트레이드오프 이해

- [ ] 상태 관리 도구 선택 기준

- [ ] 번들 사이즈 vs 개발 속도

- [ ] 조기 최적화의 함정

- [ ] 타입 안전성 vs 개발 속도

  

### 실무 경험

- [ ] 실제 프로젝트에서 겪은 문제와 해결 과정

- [ ] 코드 리뷰 경험 및 개선 사례

- [ ] 성능 최적화 경험