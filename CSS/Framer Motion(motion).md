
## 설명

React 오픈소스 애니메이션 라이브러리로 복잡한 CSS나 JS코드없이도 
선언적인 props를 통해 간단하고 직관적으로 애니메이션을 적용가능하도록해준다.


---

## 사용법

```
pnpm install motion
```


#### 기본

```jsx
import { motion } from 'motion/react';

const MotionComp = () => {
  return (
   <motion.div>
     <motion.p>하이</motion.p>
    </motion.div>;

)};

export default MotionComp;

```
- `motion`을 import한뒤 적용하고싶은 `HTML태그`나 `SVG태그`앞에 `motion.`을 붙이기
---

#### `animate` 와 `initial` Props

- `initial` : 컴포넌트가 처음 렌더링될때의 상태지정
- `animate` : 애니메이션을 끝났을 때의 상태를 지정
  -> 위와같은 두개의 Props에 CSS속성을 객체형태로 전달하면 애니메이션을 생성해줌

  ```jsx
import { motion } from 'motion/react';

const MotionComp = () => {
  return (
    <motion.div>
      <motion.p
      // 초기에는 투명도0 크기0.3에서  투명도1,크기1로 변하는 애니메이션
        initial={{ opacity: 0, scale: 0.3 }}
        animate={{ opacity: 1, scale: 1 }}
      >
        하이
      </motion.p>
    </motion.div>
  );
};

export default MotionComp;

  ```

---

##### `trainsition` Prop

애니메이션의 진행과정을 세밀하게 제어하고싶을때 사용함
`지속시간(duration)`  `딜레이(delay)` `반복(repeat)` 나 `물리효과(type:'spring')` 등을설정가능


```jsx
import { motion } from 'motion/react';

const MotionComp = () => {
  return (
    <motion.div>
      <motion.p //x축으로 200px이동하고 360도 회전
        animate={{ x: 200, opacity: 1, scale: 1, rotate: 360 }} 
        
        transition={{
          type: 'spring', //물리기반 스프링효과
          stiffness: 50, // 스프링 강도
          duration: 10, // 지속시간(초단위)
          repeat: Infinity, //무한반복
          repeatType: 'reverse', //반복방식
        }}
      >
        하이
      </motion.p>
    </motion.div>
  );
};

export default MotionComp;

```

---

##### `varients`

여러 애니메이션 프리셋을 미리 정의해둔뒤, 이름을 붙여서 재사용할수있도록 할수있음
복잡한 연쇄 애니메이션등을 만들때도 사용

```jsx
import { motion } from 'motion/react';

// 객체에 애니메이션 상태들을 미리 정의함(이름은 자유)
const pTagVarients = {
  start: { scale: 0, y: 50 }, //initial에 들어갈 애니메이션
  end: { scale: 1, y: 0, transition: { duration: 0.8 } }, //animate
  exit: { scale: 0, y: -50, opacity: 0 },//exit에 들어갈 애니메이션
};

const MotionComp = () => {
  return (
    <motion.div> 
      <motion.p
       variants={pTagVarients} // varient? prop에 객체 전달
       //initial과 animate에 객체의 key를 전달
        initial='start'
        animate='end'>
        하이
      </motion.p>
    </motion.div>
  );
};

export default MotionComp;

```
---

##### `whileHover` 와 `whileTap`

각각 마우스를 올렸을때와 클릭했을때 나타날 애니메이션을 만들수있음
- `whileDrag` `whileFocus` `whileInView`등도 존재함

```jsx
import { motion } from 'motion/react';

const pTagVarients = {
  start: { scale: 0, y: 50 },
  end: { scale: 1, y: 0, transition: { duration: 0.8 } },
  hover: { scale: 2, transition: { duration: 0.7 } },
  tap: { scale: 3, rotation: 360, transition: { duration: 0.7 } },
  exit: { scale: 0, y: -50, opacity: 0 },
};

const MotionComp = () => {
  return (
    <motion.div>
      <motion.p
        variants={pTagVarients}
        initial='start'
        animate='end'
        whileHover='hover'
        whileTap='tap'
      >
        하이
      </motion.p>
    </motion.div>
  );
};

export default MotionComp;

```


---


##### `AnimatePresence`

**컴포넌트가 DOM에서 사라질때의 애니메이션을 설정가능하도록해줌**

React에서는 상태가 변경되어 조건부렌더링되는 컴포넌트가 사라져야할때 마로 `unmount`되지만
`<AnimatePresence>`로 감싸게된다면 내부의 컴포넌트는 정의된 `exit` 애니메이션이
끝날때까지 기다리게됨


``` jsx
import { motion, AnimatePresence } from "framer-motion";
import { useState } from "react";

function PresenceExample() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>Open Modal</button>
      {/* AnimatePresence로 조건부 렌더링되는 컴포넌트를 감싸기. */}
      <AnimatePresence>
        {isOpen && (
          <motion.div
            style={{ position: "fixed",width: 200, height: 150}}
            initial={{ opacity: 0, y: -50 }}
            animate={{ opacity: 1, y: 0 }}
            // exit: 컴포넌트가 사라질 때 적용될 애니메이션
            exit={{ opacity: 0, y: 50 }}
            onClick={() => setIsOpen(false)}
          >
           예시 모달
          </motion.div>
        )}
      </AnimatePresence>
    </div>
  );
}
```












