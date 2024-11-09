---
title: Array
date: 2024-11-04 00:09:28
categories:
  - Coding Test
  - Data Structure
#tags:
---
일련의 데이터들을 0이상의 index로 **순서대로** 읽거나 쓸 수 있는 데이터 구조입니다.

0이상의 정수를 key로 가지는 hashMap으로도 볼 수 있어서 알파벳과 같이 연속된 값들을 index로 매핑하여 접근할 때 유용합니다.

## 조건별 초기화 방법

1. `null` 로 초기화된 길이가 `l`인 1차원 배열

```js
function initArrNull(l) {
  return Array(l).fill(null);
}
```

2. `(index + 1)`로 초기화된 길이가 `l`인 1차원 배열

```js
function initArrOrd(l) {
  return Array.from(Array(l), (_, i) => i + 1);
}
```

3. `null` 로 초기화된 `(r, c)` 크기의 2차원 배열

```js
function init2dArrNull(r, c) {
  return Array.from(Array(r), () => initArrNull(c));
}
```

4. 순서대로 초기화된 `(r, c)` 크기의 2차원 배열 ⇒ 2차원 배열은 좌표 또는 경로

```js
function init2dArrOrd(r, c) {
  const arr = init2dArrNull(r, c);

  for (let i = 0; i < r; i++) {
    for (let j = 0; j < c; j++) {
      arr[i][j] = c * j + i + 1;
    }
  }

  return arr;
}
```

## 응용

1. 두 문자열이 서로 [anagram](https://en.wikipedia.org/wiki/Anagram)인지 여부를 확인하는 경우

```js
const isAnagram = (s1, s2) => {
  const getCharIdx = (c) => c.charCodeAt(0) - "a".charCodeAt(0);

  const l = getCharIdx("z") + 1; // 전체 알파벳 개수
  const m1 = Array(l).fill(0);
  const m2 = Array(l).fill(0);

  // 각 배열에 알파벳별 빈도수를 저장합니다.
  for (const c of s1) {
    m1[getCharIdx(c)]++;
  }
  for (const c of s2) {
    m2[getCharIdx(c)]++;
  }

  // 알파벳별 빈도수가 모두 같아야만 anagram입니다!
  for (let i = 0; i < l; i++) {
    if (m1[i] !== m2[i]) {
      return false;
    }
  }

  return true;
};
```

임의의 문자열 S의 부분문자열 S'에는 개별 문자도 포함된다는 점을 기억하면 유용합니다!

```js
'abc' => ['a', 'b', 'c', 'ab', 'bc', 'ac', 'abc']
```
