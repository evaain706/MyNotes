


### 1. 프로젝트 폴더 구조 

단순한 계층형 구조가 아닌, **관심사 분리(Separation of Concerns)와 기능 기반(Feature-based) 아키텍처**를 혼합하여 확장성과 유지보수성이 매우 높게 설계되어 있습니다.

- `src/apis/`: Axios 인스턴스, 인터셉터 등 서버 통신을 위한 기본 설정 관리.
- `src/components/`: 애플리케이션 전반에서 재사용되는 공통 UI 요소 (ex: Button, Modal, Input 등).
- `src/constants/`: 전역적으로 사용되는 고정값(Magic number 등)을 한 곳에서 선언하여 일관성을 관리 및 휴먼 에러 방지.
- `src/features/`: **[핵심 폴더]** 프로젝트의 핵심 도메인(`quiz`, `community`, `myPage`)별로 관련된 하위 컴포넌트, 전용 훅, 타입 등을 응집도 있게 묶어서 관리. (특정 기능을 수정/삭제할 때 해당 폴더만 보면 되므로 유지보수성이 획기적으로 향상됨)
- `src/hooks/`: 프로젝트 전역에서 재사용되는 Custom Hooks.
- `src/page/`: React Router와 1:1로 매칭되는 최상단 뷰 컴포넌트. 페이지의 레이아웃과 도메인 기능 컴포넌트를 조립하는 역할.
- `src/queryKeys/`: React Query를 사용할 때 쿼리 키(Query Key)의 오타 방지 및 중앙 집중적 관리를 위한 폴더 (캐시 무효화 등을 쉽게 관리).
- `src/store/`: 클라이언트 전역 상태 관리 (Zustand 등) 로직을 보관.
- `src/types/`: 프로젝트 전역에서 공유되는 명시적인 TypeScript 인터페이스 및 타입 선언.
- `src/utils/`: 날짜 포맷팅, 데이터 가공 등 여러 컴포넌트에서 재사용되는 유틸리티 함수.


---


1. **React 19 & Vite**
    - **도입 이유:** 강력한 CSR(Client-Side Rendering) 컴포넌트 기반 UI 개발을 위해 React를 메인으로 사용하며, Webpack 대비 압도적으로 빠른 HMR(Hot Module Replacement) 속도와 최적화된 빌드를 제공하는 Vite를 채택하여 개발 생산성을 끌어올렸습니다.
2. **TypeScript**
    - **도입 이유:** 컴파일러 단에서 에러를 사전에 잡아내어 런타임 오류 방지. LLM API 응답 데이터나 상태 트리의 형태 등 정교한 타입 관리를 위해 안정성 측면에서 도입되었습니다.

#### 🗄️ 상태 관리 & 데이터 패칭

3. **TanStack React Query (`@tanstack/react-query`)** - _서버 상태 관리_
    - **도입 이유:** 단순한 데이터 패칭을 넘어, 서버 데이터 캐싱/동기화, 로딩 및 에러 상태의 **선언적 리팩토링**을 위해 도입했습니다. 개발자님께서 최근 시도하셨던 _선언적 에러 핸들링(Error Boundary 기반 Fallback UI 노출)_ 구조에 가장 최적화된 도구이며 복잡한 비동기 상태의 단일 진실 공급원(SSOT) 역할을 수행합니다.
4. **Zustand** - _클라이언트 전역 상태 관리_
    - **도입 이유:** Redux처럼 보일러플레이트 코드가 방대하지 않고, 가볍고 직관적인 API를 제공하여 다루기 쉽습니다. UI 전역 상태(모달 on/off, 라이트/다크모드 등)를 매우 적은 코드로 다루기 위해 채택되었습니다.
5. **Axios**
    - **도입 이유:** 내장 `fetch` API에 비해 인스턴스/인터셉터 생성이 직관적이라 API 통신 시 Authorization Header 토큰 자동 주입 및 글로벌 에러 공통 처리 등에 유리합니다.

#### ✍️ 폼 관리

6. **React Hook Form**
    - **도입 이유:** 제어 컴포넌트로 폼을 다룰 때 발생하는 타이핑 시 로스(리렌더링 폭탄)를 없애주는 '비제어 컴포넌트' 기반 라이브러리입니다. `Community Form`의 최적화나 복잡한 폼 유효성 검사를 적은 코드로 구현하고 성능을 향상시키기 위해 도입하셨습니다.

#### 🎨 스타일링 및 UI / UX

7. **Tailwind CSS (@tailwindcss/vite, postcss)**
    - **도입 이유:** CSS/SCSS 파일을 오가며 클래스 네이밍('btn-primary-wrapper' 등)을 고민할 필요가 없는 유틸리티 최우선 CSS 프레임워크입니다. 빠른 퍼블리싱과 컴포넌트 내부에서의 스타일 파악 직관성을 위해 사용되었습니다.
8. **clsx & tailwind-merge**
    - **도입 이유:** Tailwind의 가장 큰 단점인 '클래스 조건부 렌더링 시 코드가 지저분해짐', 그리고 'props로 넘어온 클래스가 우선순위 밀림(충돌)' 현상을 해결하기 위한 조합입니다.
9. **Framer Motion (`motion`)**
    - **도입 이유:** UX를 크게 개선해주는 마이크로 애니메이션이나 레이아웃 전환 애니메이션을 쉽게 적용. 선언적인 구조라서 React와 매우 잘 어울려 사용자 참여를 유도하는 동적인 UI 구성에 채택되었습니다.
10. **Lucide React**
    - **도입 이유:** 일관되고 세련된 디자인의 SVG 기반 경량 아이콘 라이브러리로, 별개의 CSS 폰트 파일 없이 컴포넌트처럼 호출할 수 있어 유지보수가 뛰어납니다.

#### 📝 유틸리티 서비스

11. **react-markdown & rehype-highlight**
    - **도입 이유:** LLM(Llama 등) 모델의 응답이나 해설, 퀴즈 내용이 주로 마크다운 형식으로 내려오는데, 이를 화면에 안전하고 예쁜 HTML로 렌더해주며 내부 코드블럭(Syntax Highlighting) 처리까지 담당하기 위해 채택되었습니다.



---

## QueryKeyFactory패턴




> 프로젝트 초기에는 각 `useQuery` 호출마다 queryKey를 인라인으로 작성했는데, 키를 수정하거나 `invalidateQueries`할 때 휴먼에러발생
> 
> queryKeys.ts에 도메인별로 계층적 팩토리 함수를 모아 **단일 진실 공급원(Single Source of Truth)**으로 관리하면, `queryKeys.community.all`로 커뮤니티 전체를 무효화하거나 `queryKeys.community.list({page, category, search})`로 특정 조건만 무효화하는 것이 가능합니다. TypeScript `as const`를 사용해 자동완성과 타입 안전성도 확보했습니다.





