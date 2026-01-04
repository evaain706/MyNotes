

| **특징**                  | **var (ES5)**          | **let (ES6+)**          | **const (ES6+)**        |
| ----------------------- | ---------------------- | ----------------------- | ----------------------- |
| **스코프 (Scope)**         | 함수 레벨 (Function Scope) | 블록 레벨 (Block Scope)     | 블록 레벨 (Block Scope)     |
| **재선언 (Redeclaration)** | 가능하지만 (위험)             | 불가능                     | 불가능                     |
| **재할당 (Reassignment)**  | 가능                     | 가능                      | 불가능 (단, 객체 내부 값 변경은 가능) |
| **호이스팅 (Hoisting)**     | 발생 (초기화: `undefined`)  | 발생 (초기화 안 됨 -> **TDZ**) | 발생 (초기화 안 됨 -> **TDZ**) |

---

### 2. 상세 동작 

#### A. 스코프 (Scope) : 유효 범위의 차이

- **`var`**: 함수(`function`) 몸체만 스코프로 인정됨. `if`, `for` 문 블록은 무시하고 전역 변수처럼 탈출
    
- **`let`, `const`**: 중괄호 `{}`로 감싸진 모든 블록(Block)을 스코프로 인정


``` js
function scopeTest() {
  if (true) {
    var varVal = "나는 var";
    let letVal = "나는 let";
    const constVal = "나는 const";
  }

  // 1. var는 함수 스코프라 if문 밖에서도 살아있음
  console.log(varVal); // "나는 var"

  // 2. let, const는 블록 스코프라 if문이 끝나는 순간 메모리에서 해제됨 (접근 불가)
  // console.log(letVal); // ReferenceError: letVal is not defined
}
```


#### B. 호이스팅(Hoisting)과 TDZ -> 핵심임

`let const 는 호이스팅이 되지만 초기화가 안됨`

- **`var`**: 선언 단계와 초기화(`undefined`) 단계가 동시에 진행. 그래서 선언 전에도 에러 없이 접근 가능합니다 -> 이게 문제

- **`let`, `const`**: 선언 단계는 이루어지지만, 실제 코드 라인에 도달하기 전까지 초기화 X
- 이 구간을 **TDZ (Temporal Dead Zone, 일시적 사각지대)**라고하며, 이때 접근하면 에러

``` js
// --- var의 경우 ---
console.log(oldVar); // undefined
// (에러가 안 남. 코드가 실행되기 전 엔진이 최상단으로 끌어올려 undefined로 초기화해뒀기 때문)
var oldVar = 10;

// --- let의 경우 ---
// console.log(modernLet); // ReferenceError: Cannot access 'modernLet' before initialization
// (호이스팅되어 메모리 공간은 확보했으나, 아직 값이 할당되지 않은 TDZ 구간임)
let modernLet = 20;
```


#### C. 재할당과 불변성 (`const`의 함정)

`const`는 재할당이 불가능하지만, **"값 자체가 불변(Immutable)"이라는 뜻은 아님.** 객체나 배열 같은 참조 타입의 경우, 주소값만 고정될 뿐 내부 속성은 바꿀 수 있음


``` js
const user = {
  name: "React",
  level: 1
};

// 1. 재할당 시도 -> 에러 발생
// user = { name: "Vue" }; // TypeError: Assignment to constant variable.
// (user가 가리키는 메모리 주소 자체를 바꾸려고 했기 때문)

// 2. 내부 속성 변경 -> 가능!
user.level = 99; 
// (user가 가리키는 힙 메모리 주소는 그대로이고, 그 주소 안의 데이터만 바뀐 것이라 허용됨)
console.log(user.level); // 99

```

---

### 3.  사용 및 결론

1. **기본적으로 `const`를 사용.**
    
    - React 컴포넌트, 함수, 설정값, Hook 등 변하지 않는 참조가 대부분이기 때문.
        
    - 예기치 않은 값 변경을 막아 버그를 줄여줌.
        
2. **값이 바뀌어야 하는 경우에만 `let`을 사용.**
    
    - `for`문의 카운터 변수(`i`), 토글 상태 변수 등.
        
3. **`var`는 절대 사용X**
    
    - 레거시 코드가 아닌 이상, 모던 자바스크립트/리액트에서 `var`를 쓸 이유는 없음.
     why?
     1. 중복 선언이 가능
     2. 스코프의 범위가 전역/함수로 제한됨
     3. 나중에 선언된 변수가 호이스팅으로 인해 사용이 가능