---
title: WeakMap
date: 2024-11-03 16:21:11
categories:
  - Studies
  - JavaScript
  - Data Structure
#tags:
---
기존의 `Map` 과 같이 key-value pair로 데이터를 저장하는 객체인데, 아래와 같은 두드러진 차이점이 있습니다.

- key가 무조건 객체여야 한다.
- key로 사용되는 객체가 외부에서 garbage collect 대상이 되면 매핑된 value도 동일하게 수거대상이 된다.

`WeakMap` 객체의 key가 언제, 어떻게 수거되어 없어질지 모르기 때문에 반복문을 통한 enumeration이 불가하고, 따로 key들만 조회하는 메서드가 처음부터 없습니다.

그래서 key값이 수거되기 전까지 유용한 정보(metadata)들을 **일시적으로** 저장하거나 자동 메모리 관리가 필요한 임베디드 기기에서 활용할 수 있습니다.

사실 이런 종류의 데이터는 객체 key들을 가진 배열과 index에 맞춰서 위치한 value들을 가진 배열로 충분히 구현할 수 있습니다.

```js
const obj1 = {},
  obj2 = {},
  obj3 = {};

const keys = [obj1, obj2, obj3];
const values = ["hello", Symbol(), 89];

function getValueOf(key: object) {
  const idx = this.keys.indexOf(key);
  return this.values[idx];
}

const weakMap = {
  keys,
  values,
  getValueOf,
};

console.log(weakMap.getValueOf(obj2)); // Symbol()
```

하지만 위와 같이 구현했을 때는 다음과 같은 단점을 가집니다.

- `O(N)` 의 시간복잡도를 가져서 단일 value를 탐색하는데 key의 개수가 늘어날수록 복잡도가 올라갑니다.
- 배열로 저장된 요소들은 garbage collect 대상이 아니기 때문에 key 객체가 다른 영역에서 거의 사용되지 않아도 계속 메모리의 공간을 차지하여 메모리 누수가 발생할 수 있습니다.

## 참고자료

[WeakMap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)