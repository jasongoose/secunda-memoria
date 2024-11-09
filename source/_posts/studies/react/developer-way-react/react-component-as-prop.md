---
title: React component as prop, the right way
date: 2024-11-04 21:19:02
categories:
- Studies
- React
- Developer Way React
#tags:
---
props로 컴포넌트를 전달하는 방식은 총 3가지가 있습니다.

- passing as an Element ⇒ 정해진 위치에 그저 렌더링하는 경우
  ```jsx
  <Button icon={<Icon />} />
  ```
- passing as a Component ⇒ 받는 쪽에서 이 컴포넌트로 전달할 props를 override해야 하는 경우
  ```jsx
  <Button Icon={Icon} />
  ```
- passing as a Function(render Fn) ⇒ 받는 쪽이 특정 조건에 의해서 동적으로 렌더링해야 하는 경우
  ```jsx
  <Button renderIcon={() => <Icon />} />
  ```

컴포넌트 ⇒ ReactElement(Vnode) 객체를 생성하는 함수