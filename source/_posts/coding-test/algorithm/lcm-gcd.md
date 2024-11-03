---
title: 최소공배수, 최대공약수
date: 2024-11-03 23:45:31
categories:
  - Coding Test
  - Algorithm
#tags:
---
최대공약수(Greatest Common Division, GCD)는 유클리드 호제법에 의해서 다음과 같이 구할 수 있습니다.

> 2개의 자연수(또는 정수) a, b에 대해서 a를 b로 나눈 나머지를 r이라 하면(단, a>b), a와 b의 최대공약수는 b와 r의 최대공약수와 같다.

```js
// 단, y < x
const getGcd = (x, y) => {
  if (x % y === 0) {
    return y;
  }

  return getGcd(y, x % y);
};
```

최소공배수(Least Common Multiple, LCM)은 아래와 같은 성질로 간단하게 구할 수 있습니다.

> 두 수 a, b의 최대공약수를 G, 최소공배수를 L이라고 한다면 a x b = G x L 이다.

```js
// 단, y < x
const getLcm = (x, y) => {
  return (x * y) / getGcd(x, y);
};
```

## 참고자료

[HackerRank](https://www.youtube.com/@HackerrankOfficial/playlists)