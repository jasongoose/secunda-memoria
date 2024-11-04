---
title: Queue
date: 2024-11-04 00:10:01
categories:
  - Coding Test
  - Data Structure
#tags:
---
Stack과 마찬가지로 하나의 index에 하나의 요소만이 존재하는 선형구조를 가집니다.

제일 먼저 들어간 요소를 제일 먼저 뺄 수 있는 FIFO(First In First Out) 특성을 가집니다.

## 구현코드

```js
const queue = () => {
  const arr = [];

  const enq = arr.push;

  const deq = arr.shift;

  const getSize = () => arr.length;

  const getHead = () => arr[0];

  const getTail = () => arr.at(-1);

  return {
    enq,
    deq,
    getSize,
    getHead,
    getTail,
  };
};
```

`Array.prototype.shift` 메서드는 첫번째 요소를 반환합니다!

## 응용

### LRU 알고리즘

cache miss가 발생했는데 캐시에 여유공간이 없는 경우, 가장 오랫동안 사용하지 않은 데이터를 캐시에서 제외하는 알고리즘으로 queue로 간단하게 구현할 수 있습니다.

```js
// data : string[]
const lru = (data, cacheSize) => {
  const q = []; // cache 역할

  if (cacheSize === 0) {
    // 크기가 0인 cache에 대한 예외처리
  }

  const isHit = (d) => q.some((el) => el === d);

  for (const x of data) {
    if (isHit(x)) {
      // 기존의 hit된 데이터 우선 제외합니다.
      q.splice(q.indexOf(x), 1);
    }
    // 캐시가 가득 찬 경우, 가장 오래된 요소를 제외합니다.
    if (q.length === cacheSize) {
      q.shift();
    }
    // hit된 데이터의 제외 우선순위를 낮추기 위해서 +
    // miss된 데이터를 캐싱하기 위해서 push합니다.
    q.push(x);
  }
};
```
