
## 1. 중앙정렬

```jsx
<div class="relative w-48 h-48 bg-slate-200">
  <div class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2">
</div>
</div>
```

#### top-1/2 left-1/2는 알겠는데 -translate는 왜?

 => top,left-1/2는 요소의 `중심`이 아닌 요소의 `왼쪽 위 꼭짓점`을 중앙에 위치시키기때문

- `top,bottom,left,right` => **부모**를 기준으로 하는 이동
- `translate` => **자기자신**을 기준으로 하는 이동

 따라서 `top,left`를 통해 요소의 `왼쪽위 꼭짓점`을 부모의 중앙으로 보낸뒤
 `-translate-x/y-1/2`로 자기크기의 절반만큼 왼쪽/위로 다시 당겨 오차를 없애 중앙정렬

 
