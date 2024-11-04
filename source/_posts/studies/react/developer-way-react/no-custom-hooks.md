---
title: Why custom react hooks could destroy your app performance?
date: 2024-11-04 21:18:04
categories:
  - Studies
  - React
  - Developer Way React
#tags:
---
hook이란 개발자로 하여금 새로운 컴포넌트를 만들 필요없이 동일한 state, context 관련된 로직을 사용하기 위한 함수입니다.

앱 전반에 걸쳐 동일한 state, context 로직을 재사용할 때 굉장히 유용하게 활용됩니다.

하지만 hook을 컴포넌트 내부에서 실행하면 hook에서 정의한 state, context들이 컴포넌트의 state, context가 되는 lift up 현상이 일어나기 때문에 컴포넌트의 불필요한 렌더링의 원인이 될 수 있습니다.

```jsx
export const useModal = () => {
  const [isOpen, setIsOpen] = useState(false);

  const open = () => setIsOpen(true);
  const close = () => setIsOpen(false);
  const Dialog = () => <ModalBase onClosed={close} isOpen={isOpen} />;

  return { isOpen, Dialog, open, close };
};
```

```jsx
const Page = () => {
  const { Dialog, open } = useModal();

  return (
    <>
      <button onClick={open}>Click me</button>
      <Dialog />
    </>
  );
};
```

![State of State Diagram.png](/images/state_of_state_diagram.png)

특히 페이지 단위 컴포넌트에서는 페이지 일부의 로딩을 지연시킬 수도 있는데, 이 경우 페이지가 아닌 페이지의 하위 컴포넌트에서 호출하여 영향범위를 제한시키면 됩니다.

```jsx
const SettingsButton = () => {
  const { Dialog, open } = useModal();

  return (
    <>
      <button onClick={open}>Open settings</button>
      <Dialog />
    </>
  );
};
```

```jsx
export const Page = () => {
  // same as original page state
  return (
    <ThemeProvider value={{ mode }}>
      // stays the same
      <SettingsButton />
      // stays the same
    </ThemeProvider>
  );
};
```

![State of State Diagram Fixed.png](/images/state_of_state_diagram_fixed.png)

또한 hook은 호출한 hook의 상태에도 영향을 주기 때문에 hook의 상태가 변하면 상위 hook의 상태에도 점진적으로 변화를 주면서 결국에는 컴포넌트의 리렌더링을 일으킬 수 있습니다.

이 경우, hook의 lifted state의 영향범위가 겹치지 않도록 별개의 하위 컴포넌트에서 호출하도록 수정해야 합니다.

custom hook 관련해서 유의할 점들을 정리하면 다음과 같습니다.

1. 특정 hook의 상태변화는 호출한 hook이나 컴포넌트의 상태를 변화시키면서 리렌더링을 일으킬 수 있습니다.
2. hook의 lifted state의 변화가 컴포넌트 전체에 영향이 가지 않도록 더 작은 컴포넌트 단위에서 호출하여 영향범위를 제한합니다.
3. custom hook 내부에 return되지 않는 private state는 디버깅을 위해서 만들지 않습니다.
