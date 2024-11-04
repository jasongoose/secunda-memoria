---
title:  Flexible Arguments
date: 2024-11-03 20:52:51
categories:
  - Studies
  - Vue3
  - Composable
#tags:
---
composable은 primitive 타입 인자 또는 `Ref` 타입 인자를 받을 수 있는데, 타입별로 인자를 구분하는 것보다는 2가지 타입을 모두 받을 수 있도록 구현하는 것을 권장합니다.

## ref(...)

composable의 인자에 반응성을 부여하고 싶다면 아래와 같이 `ref` 함수를 적용하면 됩니다.

```js
// When we need to use a ref in the composable
export default useMyComposable(input: Ref<T> | T) {
  const ref = ref(input);
}
```

위에서 input의 타입이 `Ref`여도 아래와 같은 성질로 기존 반응성은 유지되므로 문제가 없습니다.

```js
ref(ref(ref(ref( ... )))) === ref(...)
```

## unref(...)

composable 인자의 반응성을 제거한 상태로 사용하고 싶다면 아래와 같이 `unref` 함수를 적용하면 됩니다.

```js
// When we need to use a raw value in the composable
export default useMyComposable(input: Ref<T> | T) {
  const rawValue = unref(input);
}
```

위에서 input 타입이 `Ref`가 아니여도 아래와 같은 성질로 문제가 되지 않습니다.

```js
input === unref(input);
```

함수의 인자가 `Ref`인지 여부와 상관없이 동작하기를 원한다면, 아래와 같은 type alias가 유용합니다.

```ts
type MaybeRef<T> = Ref<T> | T;
```
