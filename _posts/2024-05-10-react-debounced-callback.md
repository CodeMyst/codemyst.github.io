---
title: Debounced callback hook in React (TypeScript)
description: A simple useDebouncedCallback hook in React with TypeScript
date: 2024-05-10
permalink: react-debounced-callback
---

---

This one is a bit different than your usual `useDebounce` hook, instead of debouncing the change of some value, this instead debounces the call to some function. So here's a simple debounced callback hook in React with TypeScript types:

```tsx
type UnaryVoidFunction<T> = (arg: T) => void;

const useDebouncedCallback = <T,>(func: UnaryVoidFunction<T>, wait: number) => {
  const timeout = useRef<number>();

  const debouncedFunc = (arg: T) => {
    clearTimeout(timeout.current);
    timeout.current = setTimeout(() => func(arg), wait);
  };

  return useCallback(debouncedFunc, [func, wait]);
};

const App: FunctionComponent = () => {
  const [searchQuery, setSearchQuery] = useState("");

  const debouncedSearch = useDebouncedCallback(setSearchQuery, 500);

  return (
    <div>
      <input onChange={(e) => debouncedSearch(e.target.value)} />
      <div>{searchQuery}</div>
    </div>
  );
};
```
