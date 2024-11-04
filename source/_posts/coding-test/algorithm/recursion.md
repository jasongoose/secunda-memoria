---
title: Recursion
date: 2024-11-03 23:47:25
categories:
  - Coding Test
  - Algorithm
#tags:
---
해결하려는 임의의 문제를 동일한 타입의 부분문제로 쪼개고 그 결과물들을 합쳐서 해결하는 방법입니다.

부분문제의 크기가 많이 작아지고 종료할 수 없는 상황이 오지 않으려면 **반드시 탈출조건(stopping point, base case)이 있어야 합니다.**

기본적으로 재귀로 풀리는 방법은 처음부터 차례대로 구하는 방식인 순회로도 구현할 수 있습니다.

언제나 문제를 풀 때는 순회로 풀지, 아니면 재귀로 풀지에 관해서 고민해야 합니다.

```js
// recursion
const fibo = (n) => {
  if (n <= 0) return 0;
  else if (n === 1) return 1;

  return fibo(n - 2) + fibo(n - 1);
};
```

```js
// iteration
const fibo = (n) => {
  const two = [0, 1];

  if (n === 0) {
    return return two[0];
  }
  for (let i = 0; i < n - 1; i++) {
    [two[0], two[1]] = [two[1], two[0] + two[1]];
  }
  return two[1];
};

export default fibo;
```
