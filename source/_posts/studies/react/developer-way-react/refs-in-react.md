---
title: Refs in React, from access to DOM to imperative API
date: 2024-11-04 21:21:09
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
React를 사용하면 기존의 Vanilla JS로 DOM을 제어하는 복잡성을 줄일 수 있지만 실무에서 개발하다보면 직접 DOM에 접근할 필요가 자주 있을 겁니다.

DOM 접근과 관련하여 React에서 제공하는 `useRef`, `useImperativeHandle` hook의 정의와 사용하는 방법에 대해서 알아보겠습니다.

## DOM access in React with useRef

기존 브라우저에서 제공하는 DOM API로 HTML element를 제어할 때는 `getElementById`와 같은 selector를 사용합니다.

하지만 React의 경우 `useRef` hook으로 더 간결하게 구현할 수 있습니다.

`useRef` hook으로부터 반환되는 ref 객체는 mutable object로 컴포넌트 state처럼 컴포넌트의 리렌더링 중에 참조하는 값을 유지하지만 컴포넌트의 리렌더링을 trigger하지 않습니다.

ref 객체의 `current` 속성에 다양한 타입의 값을 저장하여 렌더링 중에 사용할 수 있습니다.

```jsx
const Component = () => {
  const ref = useRef(null);

  useEffect(() => {
    // re-write ref's default value with new object
    ref.current = {
      someFunc: () => {...},
      someValue: stateValue,
    }
  }, [stateValue]);

  return ...
}
```

여기서 DOM을 제어하려면 아래와 같이 실제 HTML element에 대응되는 React 컴포넌트에 ref 객체를 연결하면 됩니다.

```jsx
const Form = () => {
  const [name, setName] = useState("");
  const inputRef = useRef(null);

  const onSubmitClick = () => {
    if (!name) {
      // focus the input field if someone tries to submit empty name
      ref.current.focus();
    } else {
      // submit the data here!
    }
  };

  return (
    <>
      <input onChange={(e) => setName(e.target.value)} ref={ref} />
      <button onClick={onSubmitClick}>Submit the form!</button>
    </>
  );
};
```

## Passing ref from parent to child as a prop

props로 ref 객체를 전달하여 부모 컴포넌트에서 자식 컴포넌트의 DOM element로 접근할 수 있습니다.

```jsx
const Form = () => {
  // create the Ref in Form component
  const inputRef = useRef(null);

  useEffect(() => {
    // the "input" element, that is rendered inside InputField, will be here
    console.log(inputRef.current);
  }, []);

  return (
    <>
      {/* Pass ref as prop to the input field component */}
      <InputField inputRef={inputRef} />
    </>
  );
};
```

```jsx
const InputField = ({ inputRef }) => {
  // the rest of the code is the same

  // pass ref from prop to the internal input component
  return <input ref={inputRef} ... />
}
```

## Passing ref from parent to child with forwardRef

`forwardRef`를 사용하면 위와 같이 ref 객체를 개별 props로 전달하지 않고 그대로 자식 컴포넌트의 ref로 inject할 수 있습니다.

```jsx
// normally, we'd have only props there
// but we wrapped the component's function with forwardRef
// which injects the second argument - ref
// if it's passed to this component by its consumer
const InputField = forwardRef((props, ref) => {
  // the rest of the code is the same

  return <input ref={ref} />;
});
```

위 코드는 `forwardRef`를 적용하기 전과 후로 분리할 수 있습니다.

```jsx
const InputFieldWithRef = (props, ref) => {
  // the rest is the same
};

// this one will be used by the form
export const InputField = forwardRef(InputFieldWithRef);
```

그럼 예시 코드에서 부모인 `Form` 컴포넌트는 아래와 같이 `InputField` 컴포넌트를 사용할 수 있습니다.

```jsx
const Form = () => {
  const inputRef = useRef(null);

  return (
    <>
      {/* Pass ref as prop to the input field component */}
      <InputField ref={inputRef} />
    </>
  );
};
```

## Imperative API with useImperativeHandle

부모 컴포넌트의 ref 객체는 자식 컴포넌트의 DOM element까지 접근할 수 있습니다.

여기서 더 나아가 해당 element로 하여금 특정 동작을 시키려면 자식 컴포넌트에서 `useImperativeHandle` hook으로 해당 동작을 정의하여 public API로 제공하면 됩니다.

Vue3에서는 `ref`로 특정 컴포넌트를 참조하는 경우 [`defineExpose`라는 macro 함수](https://vuejs.org/api/sfc-script-setup.html#defineexpose)를 사용하여 해당 컴포넌트를 통해 접근할 수 있는 속성이나 메서드를 제한할 수 있습니다.

`useImperativeHanle` hook을 사용하려면 public API를 정의한 객체와 연결할 ref 객체가 필요합니다.

```jsx
// pass the Ref that we'll use as our imperative API as a prop
const InputField = ({ apiRef }) => {
  // create another ref - internal to Input component
  const inputRef = useRef(null);

  // remember our state for shaking?
  const [shouldShake, setShouldShake] = useState(false);

  // "merge" our API into the apiRef
  // the returned object will be available for use as apiRef.current
  useImperativeHandle(
    apiRef,
    () => ({
      focus: () => {
        // just trigger focus on internal ref that is attached to the DOM object
        inputRef.current.focus();
      },
      shake: () => {
        setShouldShake(true);
      },
    }),
    []
  );

  return <input ref={inputRef} />;
};
```

위와 같이 정의하면 이제 부모인 `Form` 컴포넌트는 아래와 같이 ref 객체의 `current`를 통해 API를 사용할 수 있습니다.

```jsx
const Form = () => {
  const inputRef = useRef(null);
  const [name, setName] = useState("");

  const onSubmitClick = () => {
    if (!name) {
      // focus the input if the name is empty
      inputRef.current.focus();
      // and shake it off!
      inputRef.current.shake();
    } else {
      // submit the data here!
    }
  };

  return (
    <>
      ...
      <InputField label="name" onChange={setName} apiRef={inputRef} />
      <button onClick={onSubmitClick}>Submit the form!</button>
    </>
  );
};
```

## Imperative API without useImperativeHandle

사실 `useImperativeHandle` hook은 ref 객체의 mutable한 성질을 이용하여 간단하게 재현할 수 있습니다.

물론 실제 구현방식은 더 복잡하겠지만요😉

```jsx
const InputField = ({ apiRef }) => {
  useEffect(() => {
    apiRef.current = {
      focus: () => {},
      shake: () => {},
    };
  }, [apiRef]);
};
```
