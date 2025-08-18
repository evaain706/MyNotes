 #useState
   - 상태를 관리하기위해 사용하는 훅
   - 비동기적으로 업데이트되며  여러개의 state호출은 묶음처리되어 최신값보장이 되지않을수도있음
   - 따라서 최신값보장이 필요한 상태의 경우 함수형 업데이트((prev) => prev +1)를 사용하는것이 안전
   - 상태변경시 UI가 리렌더링됨
   - 현재 상태값인 state와 변경함수인 setState가있음
   - 반드시 setState를 통해 변경해야함
``` jsx
const [state,setState] = useState(0);

.
.
.
<button onClick={() => {setState((prev)=> prev + 1)}}>버튼</button>
```