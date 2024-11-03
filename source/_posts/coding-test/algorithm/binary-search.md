---
title: Binary Search
date: 2024-11-03 23:44:43
categories:
  - Coding Test
  - Algorithm
#tags:
---
**정렬된 탐색범위**의 크기를 줄여나가면서 구하고자 하는 값으로 접근하는 기법입니다.

![Binary Search](/images/binary_search.png)

탐색범위 내의 중앙값 `mid`을 사용한 임의의 조건에 의해서 왼쪽 또는 오른쪽 범위를 대상으로 재귀적으로 검색합니다.

한번 검색할 때마다 탐색범위가 절반씩 줄어들기 때문에 총 검색하는데 최대 `O(logN)`의 시간복잡도를 가집니다.

관련 문제를 풀 때는 2가지를 정해야 합니다.

- 무엇을 구하는 것인가? => 시간, 인원 수 등
- 탐색범위 즉, 최솟값 `left`와 최댓값 `right`는 무엇인가?
- 어떤 조건으로 탐색범위(왼쪽 혹은 오른쪽)를 정하는가?

## 구현코드

```js
// recusive
const binarySearch = (data, x) => {
  const search = (left = 0, right = data.length - 1) => {
    if (right < left) {
      return false;
    }

    const mid = Math.floor((left + right) / 2);

    if (data[mid] === x) {
      return true;
    }

    x < data[mid] ? search(left, mid) : search(mid + 1, right);
  };

  return search();
};
```

```js
// iterative
const binarySearch = (data, x) => {
  let [left, right] = [0, data.length - 1];

  while (left < right) {
    const mid = Math.floor((left + right) / 2);
    if (data[mid] === x) {
      return true;
    }

    x < data[mid] ? (right = mid) : (left = mid + 1);
  }

  return false;
};
```

다루는 데이터가 10억까지 되는 문제에서 중앙값 mid을 구하는 중 오버플로우가 발생할 수도 있는데, 그 경우 아래와 같이 구하면 됩니다.

```js
const mid = Math.floor(left + (right - left) / 2);
```

## 참고자료

[HackerRank](https://www.youtube.com/@HackerrankOfficial/playlists)