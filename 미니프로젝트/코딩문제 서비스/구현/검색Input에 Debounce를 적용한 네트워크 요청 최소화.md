


``` js
export const useDebounce = (value: string, delay: number = 300) => {
  const [debouncedValue, setDebouncedValue] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debouncedValue;
};
```



```js
const [search, setSearch] = useState('');
const debounceSearch = useDebounce(search, 300);
...
const { data, isPending, error } = useQuery<PostResponse, Error>({
  queryKey: ['post', page, category, debounceSearch],
  queryFn: () => fetchPosts(page, 5, category || undefined, debounceSearch),
  staleTime: 5 * 60 * 1000,
});
```