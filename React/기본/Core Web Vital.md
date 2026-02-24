

Core Web Vitals는 사용자 경험(UX)을 정량화하기 위한 **Google의 핵심 성능 지표 3가지** 
각각은 로딩 속도, 상호작용 반응성, 시각적 안정성을 측정

# 1. LCP (Largest Contentful Paint)

> **로딩 성능 지표**

- 뷰포트 내에서 가장 큰 콘텐츠 요소(이미지, 비디오, 블록 텍스트 등)가 렌더링되는 시점
    
- “사용자가 주요 콘텐츠를 볼 수 있게 된 시간”
    

### 기준

- ✅ Good: **2.5초 이하**
    
- ⚠️ Needs Improvement: 2.5 ~ 4.0초
    
- ❌ Poor: 4.0초 초과
    

### 프론트엔드 관점 개선 방법

- Above-the-fold 이미지 `priority` (Next.js Image)
    
- 서버 응답 시간 단축 (TTFB 개선)
    
- 렌더 차단 JS/CSS 제거
    
- Critical CSS 인라인 처리
    
- 불필요한 CSR 지연 제거
    

---

# 2. INP (Interaction to Next Paint)

> **상호작용 반응성 지표**

- 사용자의 클릭, 탭, 키 입력 등 상호작용 이후 다음 화면 업데이트까지 걸리는 시간
    
- 2024년부터 FID를 대체
    

### 기준

- ✅ Good: **200ms 이하**
    
- ⚠️ Needs Improvement: 200 ~ 500ms
    
- ❌ Poor: 500ms 초과
    

### 프론트엔드 관점 개선 방법

- 메인 스레드 블로킹 JS 감소
    
- 긴 작업 분할 (Long Task 분리)
    
- `useMemo`, `useCallback` 남용 제거 (불필요 계산 방지)
    
- 가상화(react-window 등)
    
- 이벤트 핸들러 내부 무거운 로직 제거
    

👉 React에서 과도한 리렌더링이 있으면 INP가 악화됩니다.

---

# 3. CLS (Cumulative Layout Shift)

> **시각적 안정성 지표**

- 예상치 못한 레이아웃 이동의 누적 점수
    

예:

- 이미지 로딩 후 밀림
    
- 광고 영역 삽입
    
- 폰트 로딩 후 텍스트 이동
    

### 기준

- ✅ Good: **0.1 이하**
    
- ⚠️ Needs Improvement: 0.1 ~ 0.25
    
- ❌ Poor: 0.25 초과
    

### 프론트엔드 관점 개선 방법

- 이미지 width/height 명시
    
- Skeleton UI 적용
    
- 폰트 `font-display: swap` 사용
    
- 동적 삽입 요소에 고정 영역 확보




- LCP → 서버 응답 + 이미지 최적화
    
- INP → 리렌더링 최소화
    
- CLS → Skeleton + 고정 레이아웃