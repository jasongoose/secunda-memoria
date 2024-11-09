---
title: Prefix Sum
date: 2024-11-03 23:47:17
categories:
  - Coding Test
  - Algorithm
#tags:
---
길이가 `n`인 배열에서 임의의 index `i`부터 `j`까지의 요소들의 합을 `O(N)`의 시간복잡도로 구하는 알고리즘으로, "구간 합"이라고도 불립니다.

## 동작원리

배열 내의 두 index들 사이 구간의 요소들의 합 `Sum(i, j)`는 다음과 같이 구할 수 있습니다.

```js
S(i, j) = S(0, j) - S(0, i - 1);
```

따라서 기존 배열을 처음부터 끝까지 순회하면서 `Sum(0, x)`의 값을 저장하는 배열 `acc`를 만들면 됩니다.

```js
const arr = [1, 2, 3, 3, 4, 5];
const acc = Array(arr.length).fill(0);
const x = 1;
const y = 4;

arr.forEach((el, i) => {
  acc[i] = i === 0 ? arr[i] : acc[i - 1] + el;
});

console.log(arr[y] - arr[x - 1]); // index x부터 y까지의 구간 합
```

## 응용

임의의 두 index `a`와 `b` 사이에 있는 요소들을 대상으로 더해야할 수 `k`를 명시한 query가 있다고 합시다.

여기서 2개 이상의 query들을 적용한 뒤 특정 index `x`의 값은 누적합으로 구할 수 있습니다.

```js
const n = 10;
const queries = [
  [0, 4, 3],
  [3, 7, 7],
  [5, 8, 1],
];

const acc = Array(n + 1).fill(0);

for (const [a, b, k] of queries) {
  // 실제 k값의 추가는 index a에만 적용하고 나머지는 하지 않습니다.
  acc[a] += k;
  // 대신 index b 이후에는 k값이 더해지지 않으므로 반대로 k값을 제거합니다.
  acc[b + 1] -= k;
}

const x = 4;
// 이제 index 0부터 index x까지 차레대로 더해서 누적합을 구합니다.
console.log(acc.slice(0, x + 1).reduce((acc, curr) => acc + curr));
```
