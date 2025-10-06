

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



## 적용

```jsx
import React, { useState, useEffect, useCallback } from 'react';
import { instance } from '../../apis/instance';
import useInfiniteScroll from './useInfiniteScroll';

interface Item {
  id: number;
  name: string;
  description: string;
  createdAt: string;
}

interface ApiResponse {
  items: Item[];
  nextCursor: number | null;
  hasNextPage: boolean;
}

const PaginationParent: React.FC = () => {
  const [items, setItems] = useState<Item[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [nextCursor, setNextCursor] = useState<number | null>(null);
  const [hasNextPage, setHasNextPage] = useState(true);

  const fetchItems = useCallback(
    async (cursor: number | null) => {
      if (isLoading) return;
      setIsLoading(true);

      try {
        const params = {
          limit: 10,
          cursor: cursor ?? undefined,
        };

        const response = await instance.get<ApiResponse>('/api/items-cursor', {
          params,
        });
        const data = response.data;

        setItems((prevItems) =>
          cursor === null ? data.items : [...prevItems, ...data.items],
        );
        setNextCursor(data.nextCursor);
        setHasNextPage(data.hasNextPage);
      } catch (error) {
        console.error('데이터를 불러오는 중 오류가 발생했습니다.', error);
      } finally {
        setIsLoading(false);
      }
    },
    [isLoading],
  );

  const loadMore = useCallback(() => {
    if (hasNextPage && nextCursor !== null) {
      fetchItems(nextCursor);
    }
  }, [hasNextPage, nextCursor, fetchItems]);

  const targetRef = useInfiniteScroll({
    loading: isLoading,
    hasNextPage: hasNextPage,
    onLoadMore: loadMore,
  });

  useEffect(() => {
    fetchItems(null);
  }, []);

  return (
    <div className='mx-auto my-8 max-w-[520px] p-4 font-sans'>
      <h1 className='mb-6 text-center text-3xl font-bold text-gray-800'>
        아이템 목록 (무한 스크롤)
      </h1>

      <div className='flex-1 gap-10 space-y-4'>
        {items.map((item) => (
          <div
            key={item.id}
            className='rounded-lg border border-gray-200 bg-white p-4 shadow-sm transition-shadow duration-200 hover:shadow-md'
          >
            <h2 className='text-xl font-semibold text-gray-900'>
              {item.name} (ID: {item.id})
            </h2>
            <p className='mt-1 text-gray-600'>{item.description}</p>
            <p className='mt-2 text-xs text-gray-400'>
              생성일: {new Date(item.createdAt).toLocaleString('ko-KR')}
            </p>
          </div>
        ))}
      </div>

      <div className='mt-8 text-center'>
        {hasNextPage && <div ref={targetRef} style={{ height: '50px' }} />}
        {isLoading && <p className='text-gray-500'>로딩 중...</p>}
        {!hasNextPage && items.length > 0 && (
          <p className='text-gray-500'>마지막 페이지입니다.</p>
        )}
      </div>
    </div>
  );
};

export default PaginationParent;

```