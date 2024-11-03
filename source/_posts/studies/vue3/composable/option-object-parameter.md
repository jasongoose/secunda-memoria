---
title: Option Object Parameter
date: 2024-11-03 20:53:08
categories:
  - Studies
  - Vue3
  - Composable
#tags:
---
composable을 정의할 때 전달할 인자들은 다음과 같이 구분할 수 있습니다.

- required 인자들 : 그냥 타입과 함께 전달합니다.
- optional 인자들 : 객체(option object)로 전달합니다.

함수의 인자를 객체로 전달하는 방식의 장점들은 다음과 같습니다.

1. 인자들의 순서를 기억할 필요가 없습니다.
2. 길게 인자들을 작성하는 것보다 가독성이 좋아집니다.
3. 새로운 인자들을 추가하기 쉬워집니다.

여기서 객체로 전달된 optional 인자들의 default 값은 composable 내부에서 지정해야 합니다.

```js
export function useMouse(options) {
  const { asArray = false, throttle = false } = options;
  // ...
}
```

위의 option obejct parameter를 이용하면 원하는 정보만 return하도록 만들 수도 있습니다.

```js
export default useComposable(input, options) {
  // 1. Add in the `controls` option
  const { controls = false } = options;

  // ...
  // 2. Either return a single value or an object
	return controls ? { singleValue, anotherValue, andAnother } : singleValue;
}
```

## 참고자료

[Coding Better Composables: Dynamic Returns (3/5)](https://medium.com/vue-mastery/coding-better-composables-dynamic-returns-3-5-3542702dd618)