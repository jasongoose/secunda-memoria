---
title: 소수
date: 2024-11-03 23:47:44
categories:
  - Coding Test
  - Algorithm
#tags:
---
1과 자기 자신만을 약수로 가지는 자연수를 소수라고 합니다.

소수를 찾는 대표적인 방법으로 "에라토스테네스의 체(Sieve of Eratosthenes)"가 있습니다.

```js
const era = (n) => {
  // 2부터 n까지 표시하므로 배열의 길이를 (n + 1)로 잡습니다.
  const arr = Array(n + 1).fill(true);
  arr[0] = arr[1] = false; // 0, 1은 미리 false로 고정합니다.

  for (let i = 2; i < arr.length; i++) {
    for (let j = i; j < arr.length; j += i) {
      if (i !== j && era[j]) {
        era[j] = false;
      }
    }
  }

  return arr;
};
```

정말 가끔식은 `n`보다 작은 수들로 일일이 나누면서 소수인지 여부를 확인하는 로직이 필요합니다.
