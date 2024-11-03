---
title: Stack
date: 2024-11-04 00:10:05
categories:
  - Coding Test
  - Data Structure
#tags:
---
하나의 index에 하나의 요소만이 존재하는 선형구조를 가집니다.

제일 마지막에 들어간 요소를 제일 먼저 뺄 수 있는 LIFO(Last In First Out) 특성을 가집니다.

보통 인접한 요소간의 관계를 이용하거나 역순으로 나열할 때 유용합니다.

## 구현코드

```js
const stack = () => {
  const arr = [];

  const push = arr.push;

  const pop = arr.pop;

  const getSize = () => arr.length;

  const getTop = () => arr.at(-1);

  return {
    push,
    pop,
    getSize,
    getTop,
  };
};
```

`Array.prototype.pop` 메서드는 마지막 요소를 반환합니다!

## 응용

### 뒤집은 문자열

```js
const reversed = (str) => {
  const st = [];
  for (const c of str) {
    st.push(c);
  }

  let ans = "";
  while (st.length) {
    ans += st.pop();
  }

  return ans;
};
```

### Monotonic Stack

일련의 값에서 바로 다음 큰값이나, 바로 다음 작은 값을 구할 때 사용할 수 있는 풀이법입니다.

1. increasing

- 삽입하려는 값이 stack의 top보다 클때만 push, 삽입하려는 값보다 작은 값은 모두 stack에서 pop합니다.
- 배열에서 다음 작은 값이 무엇인지 알아낼 때 사용합니다.

2. decreasing

- 삽입하려는 값이 stack의 top보다 작을 때만 push, 삽입하려는 값보다 큰값은 모두 stack에서 pop합니다.
- 배열에서 다음 큰 값이 무엇인지 알아낼 때 사용합니다.

[1475. Final Prices With a Special Discount in a Shop](https://leetcode.com/problems/final-prices-with-a-special-discount-in-a-shop/)

## 참고자료

[HackerRank](https://www.youtube.com/@HackerrankOfficial/playlists)