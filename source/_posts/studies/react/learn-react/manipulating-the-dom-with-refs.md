---
title: Manipulating the DOM w/ Refs
date: 2024-11-04 08:20:12
categories:
  - Studies
  - React
  - Learn React
#tags:
---
`useRef` hook은 보통 ReactElement에 대응되는 DOM node에 접근할 때 많이 사용합니다.

```jsx
import { useRef } from "react";

export default function Component() {
  const myRef = useRef(null);

  return <div ref={myRef}></div>;
}

// 위와 같이 ref attribute을 전달하면
// myRef.current에는 div에 해당하는 DOM object를 참조합니다.
```

분명 컴포넌트 함수 최상단에서 `ref.current`의 값은 `null`인데 언제 DOM node를 참조하는걸까요?

새로운 snapshot을 렌더링하는 중에는 참조할 DOM node가 생성되지 않았으므로 `ref.current`는 일단 `null` 값을 가집니다.

그러다 [Commit 과정](./render_and_commit.md#committing)을 거쳐서 최근 snapshot이 실제 DOM에 반영되면 해당 위치에 대응되는 DOM node가 `ref.current`로 참조됩니다.

렌더링 중에 `ref.current`를 read/write하는 로직은 crash날 가능성이 높습니다.

React에 의해서 업데이트되는 DOM node 자체 또는 자식 node들을 `useRef`의 `ref.current`로 추가/삭제하는 등의 DOM manipulation 연산을 구현하면 visual crash가 일어날 수 있으므로 주의해야 합니다.

## Accessing another component’s DOM nodes

React에서는 컴포넌트를 `ref`로 참조할 때 내부 컴포넌트들의 DOM node들을 공개하지 못하도록 막습니다.

```jsx
import { useRef } from "react";

function MyInput(props) {
  return <input {...props} />;
}

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
    // <input /> 참조가 되지 않음 => 이 부분에서 에러 발생🚨
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

`ref.current`로 내부 다른 DOM node를 참조하려면 해당 컴포넌트를 `forwardRef`로 wrapping하면 됩니다.

```jsx
const MyInput = fowardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

## Exposing a subset of the API

특정 컴포넌트를 `ref.current`로 참조했을 때 얻는 DOM node에 대한 속성이나 메서드의 일부만 공개할 때는 `useImperativeHandle` hook을 사용합니다.

```jsx
import { forwardRef, useRef, useImperativeHandle } from "react";

const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // Only expose focus and nothing else
    focus() {
      realInputRef.current.focus();
    },
  }));

  return <input {...props} ref={realInputRef} />;
});
```

## How to manage a list of refs

`<ul></ul>` 내부의 다수 `<li></li>` 들을 개별적으로 참조할 때는 여러 개의 `useRef`를 호출하는 대신에 개별 `<li></li>` 마다 [ref callback](https://react.dev/reference/react-dom/components/common#ref-callback)을 전달하여 여러 개의 node들을 `Map` 같은 collection에서 관리하면 됩니다.

```jsx
import { useRef } from "react";

const catList = [];
for (let i = 0; i < 10; i++) {
  catList.push({
    id: i,
    imageUrl: "https://placekitten.com/250/200?image=" + i,
  });
}

export default function CatFriends() {
  const itemsRef = useRef(null);

  function scrollToId(itemId) {
    const map = getMap();
    const node = map.get(itemId);
    node.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      // Initialize the Map on first usage.
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToId(0)}>Tom</button>
        <button onClick={() => scrollToId(5)}>Maru</button>
        <button onClick={() => scrollToId(9)}>Jellylorum</button>
      </nav>
      <div>
        <ul>
          {catList.map((cat) => (
            <li
              key={cat.id}
              ref={(node) => {
                // 최초 mount시, node=(DOM node) 값으로 호출
                // unmount(clear)시, node=null 값으로 호출
                const map = getMap();
                if (node) {
                  map.set(cat.id, node);
                } else {
                  map.delete(cat.id);
                }
              }}
            >
              <img src={cat.imageUrl} alt={"Cat #" + cat.id} />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}
```

리렌더링 commit 전후로 전달되는 함수 객체가 다르므로 ref callback을 참조하는 변수가 전달되지 않는 이상 2번 호출됩니다.

commit 이전에 node=null 값으로, commit 이후에 node=(DOM node) 값으로 호출됩니다.

## Flushing state updates synchronously

state는 비동기적으로 DOM에 반영되기 때문에 state setter 이후의 코드는 기존 state를 기준으로 실행하는게 일반적입니다.

하지만 state setter를 호출하고 나서 DOM을 동기적으로 업데이트한 뒤에 코드를 실행해야 하는 경우에는 `react-dom`의 `flushSync` 함수를 사용하면 됩니다.

```jsx
import { useState, useRef } from "react";
import { flushSync } from "react-dom";

let nextId = 0;
let initialTodos = [];

for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: "Todo #" + (i + 1),
  });
}

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState("");
  const [todos, setTodos] = useState(initialTodos);

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    flushSync(() => {
      setText("");
      setTodos([...todos, newTodo]);
    });
    // 늘어난 todos 기반으로 li가 ul 안에 추가되고 나서 아래 코드를 실행하므로
    // listRef.current.lastChild는 추가된 li node를 참조합니다!
    listRef.current.lastChild.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
    });
  }

  return (
    <>
      <button onClick={handleAdd}>Add</button>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <ul ref={listRef}>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}
```
