

## Intersection Observer API란?

타겟팅한 요소가 상위요소 or 최상위요소의 화면에 보이고있는지를 감지하는 기능을 제공하는 
`Web API` 
타겟팅한 요소가 뷰포트에 들어오거나(보이거나) 나갈때(안보이게될때) **한번만** 콜백함수를
실행하여 효율적인 무한스크롤 구현이 가능해짐


---

### 커스텀훅으로 구현하기

```jsx
import { useEffect, useRef } from 'react';


interface useInfiniteScrollType {
  
  loading: boolean;
  hasNextPage: boolean;
  threshold?: number;

  onLoadMore: () => void;
}

const useInfiniteScroll = ({
  loading,
  hasNextPage,
  threshold = 150, 
  onLoadMore,
}: useInfiniteScrollType) => {
 
  const targetRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
  
    if (loading || !hasNextPage) return;

    const observerCallback = (entries: IntersectionObserverEntry[]) => {
     
      if (entries[0].isIntersecting) {
        onLoadMore();
      }
    };

    const observer = new IntersectionObserver(observerCallback, {
     
      root: null,
      rootMargin: `0px 0px ${threshold}px 0px`,
      threshold: 0,
    });

    if (targetRef.current) {
      observer.observe(targetRef.current);
    }

    return () => {
      if (targetRef.current) {
        observer.unobserve(targetRef.current);
      }
    };
   
  }, [loading, hasNextPage, onLoadMore, threshold]);


  return targetRef;
};

export default useInfiniteScroll;

```