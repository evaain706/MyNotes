1. useState
``` jsx
const [state,setState] = useState(0);

.
.
.
<button onClick={() => {setState((prev)=> prev + 1)}}>버튼</button>
```