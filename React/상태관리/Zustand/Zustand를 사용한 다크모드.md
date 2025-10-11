

## Zustand Store

```js
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface ThemeState {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

// 현재 theme state에 따라 html태그의 class를 변경하는 역할
const handleUpdateClass = (theme: 'light' | 'dark') => {
  const root = document.documentElement;
  if (theme === 'dark') {
    root.classList.add('dark');
  } else {
    root.classList.remove('dark');
  }
};

export const useThemeStore = create<ThemeState>()(
// localstorage에 저장/가져오기위한 persist 미들웨어 사용
  persist(
    (set) => ({
      theme: 'light',
      toggleTheme: () =>
        set((state) => ({
          theme: state.theme === 'light' ? 'dark' : 'light',
        })),
    }),
    {
      name: 'theme-storage',
     
     //새로고침시 로컬스토리지에서 theme를 복원(새로고침해도 현재테마 유지)
      onRehydrateStorage: () => (state) => {
        if (state) {
          handleUpdateClass(state.theme);
        }
      },
    },
  ),
);

//theme가 바뀔때마다 실행
useThemeStore.subscribe((state) => {
  handleUpdateClass(state.theme);
});

```



---

## index.css Tailwind 커스텀 접두사 추가

```js
@custom-variant dark (&:where(.dark, .dark *));
```

```js
if (theme === 'dark') {
    root.classList.add('dark');
  }
```
위 store 코드에서 theme이 dark가 되면 루트에 `dark`라는 클래스를 추가하게되는데,
위 커스텀 접두사를 통해 `.dark`클래스가 html에 있을때 적용되는 `varient`를 추가한다.



---

## 사용

```jsx
import { SearchItem } from './useSearch';
import { useThemeStore } from '../../store/themeStore';

interface SearchResultsProps {
  items: SearchItem[];
  loading: boolean;
}

export default function SearchResults({ items, loading }: SearchResultsProps) {

  // useThemeStore에서 toggleTheme를 구조분해할당으로 가져옴
  const { toggleTheme } = useThemeStore();
  
  if (loading) return <div>로딩 중...</div>;

  if (items.length === 0)
    return <div className='mt-4 text-gray-500'>검색 결과가 없습니다</div>;

  return (
    <div className='flex flex-col'>
      <button
        onClick={toggleTheme}
        className='mt-6 rounded bg-blue-500 px-4 py-2 dark:bg-black'
      >
        테마 변경
      </button>
      {items.map((item) => (
        <div
          key={item._id}
          className='border-b border-gray-300 py-2 dark:bg-amber-200'
        >
          <div>{item.name}</div>
          <div className='text-sm text-gray-500'>{item.category}</div>
          <div className='text-sm text-gray-600'>{item.description}</div>
        </div>
      ))}
    </div>
  );
}

```

정의해둔 `useThemeStore`에서 `toggleTheme`을 가져온뒤 버튼과 연결시켜 이를 누르게되면
루트 html에 `dark`라는 클래스가 추가되고, `dark:` 라는 커스텀접두사를 통해
**theme가 'dark'일때** 정의해둔 스타일이 적용된다.