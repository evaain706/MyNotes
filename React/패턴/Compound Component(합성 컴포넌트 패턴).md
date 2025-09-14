

## 합성컴포넌트란?

컴포넌트를 여러개의 부모컴포넌트와 여러개의 하위 컴포넌트로 나눠서(ex:Trigger,Content,Item)
부모컴포넌트가 상태,로직들을 Context를 통해 제공하고 
하위컴포넌트들은 그것을 활용하여 컴포넌트를 구성하는 패턴

 Dropdown,Modal,Tabs,Accordion같은 컴포넌트들이 주로 합성컴포넌트에 어울리는 컴포넌트임.


### 사용하는이유?

1. 부모컴포넌트가 상태와 로직을 담당하고 하위컴포넌트들은 UI를 담당하여 UI와 로직을 분리
    - 하위 컴포넌트들은 주로 부모컴포넌트에서 내려받은 props만으로 UI를 구성
    
2. 사용시 하위컴포넌트들을 조합해서 선언적으로 사용가능
     ```jsx
     //코드 가독성이 좋아짐
     <Tabs defaultTab="home">
  <Tabs.List>
    <Tabs.Tab id="home">홈</Tabs.Tab>
    <Tabs.Tab id="profile">프로필</Tabs.Tab>
    <Tabs.Tab id="settings">설정</Tabs.Tab>
  </Tabs.List>
  
  <Tabs.Panels>
    <Tabs.Panel id="home"><Home /></Tabs.Panel>
    <Tabs.Panel id="profile"><Profile /></Tabs.Panel>
    <Tabs.Panel id="settings"><Settings /></Tabs.Panel>
  </Tabs.Panels>
</Tabs>
     ```


### 장단점

#### 장점
- 구현이후 사용이 직관적임
- 컴포넌트를 사용하는 개발자가 원하는 순서나 조합으로 사용가능
#### 단점
- context를 사용하기때문에 복잡한 컴포넌트를 합성컴포넌트패턴으로 구현하려할수록 
  구현난이도가 증가함




### 예시 DropDown컴포넌트

#### Context정의
```jsx
import { createContext } from 'react';

interface DropDownContextType {
  isOpen: boolean;
  toggleDropDown: () => void;
  closeDropDown: () => void;
}

const DropDownContext = createContext<DropDownContextType | undefined>(
  undefined,
);



export default DropDownContext;

```
필요한 로직이나 상태가 담길 Context를 정의함.
#### useDropDown Hook 정의
``` jsx
import { useContext } from 'react';
import DropDownContext from '../contexts/Context';

export const useDropDown = () => {
  const context = useContext(DropDownContext);
  if (!context) {
    throw new Error('DropDownProvier내부에서만 사용가능!');
  }
  return context;
};

```

DropDownContext를 useDropDown이라는 Hook으로 만들어 꺼내사용할수있도록함.
*DropDownProvider*로 감싸 사용한다면 공유된 값들을 가져올수있지만
바깥에서 사용할경우 undefined가 반환되기때문에
Provider없이 호출할경우 에러를 throw하여 개발자에게 피드백을 알림.

#### DropDown.tsx(부모 컴포넌트)

```jsx
import { useState, useCallback } from 'react';
import DropDownContext from './contexts/Context';

interface DropdownProps {
  children: React.ReactNode;
}

const Dropdown = ({ children }: DropdownProps) => {
  const [isOpen, setIsOpen] = useState(false);

  const toggleDropDown = useCallback(() => {
    setIsOpen((prev) => !prev);
  }, []);

  const closeDropDown = useCallback(() => {
    setIsOpen(false);
  }, []);

  const value = { isOpen, toggleDropDown, closeDropDown };

  return (
    <DropDownContext.Provider value={value}>
      <div onMouseLeave={closeDropDown}>{children}</div>
    </DropDownContext.Provider>
  );
};

export default Dropdown;

```


#### Content.tsx
```jsx
import { useDropDown } from './hooks/useDropDown';

interface ContentProps {
  children: React.ReactNode;
}

const Content = ({ children }: ContentProps) => {
  const { isOpen } = useDropDown();
  return isOpen ? <div>{children}</div> : null;
};

export default Content;

```

#### Item.tsx

```jsx
interface ItemProps {
    children: React.ReactNode;
    onClick?: () => void;
}

const Item = ({ children, onClick }: ItemProps) => {
    return <div onClick={onClick}>{children}</div>;
};

export default Item;
```

#### Trigger.tsx

```jsx
import { useDropDown } from './hooks/useDropDown';

interface TriggerProps {
  children: React.ReactNode;
}

const Trigger = ({ children }: TriggerProps) => {
  const { toggleDropDown } = useDropDown();
  return <div onMouseEnter={toggleDropDown}>{children}</div>;
};

export default Trigger;

```

#### index.ts

```js
import Dropdown from './Dropdown';
import Trigger from './Trigger';
import Content from './Content';
import Item from './Item';

const DropdownComponent = Object.assign(Dropdown, {
    Trigger,
    Content,
    Item,
});

export default DropdownComponent;
```


#### 사용예시

```jsx
 <div>
      <Dropdown>
        <Dropdown.Trigger>
          <button>드롭다운트리거</button>
        </Dropdown.Trigger>
        <Dropdown.Content>
          <Dropdown.Item onClick={() => alert("1")}>1</Dropdown.Item>
          <Dropdown.Item onClick={() => alert("2")}>2</Dropdown.Item>
        </Dropdown.Content>
      </Dropdown>
    </div>
```
