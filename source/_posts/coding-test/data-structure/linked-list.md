---
title: Linked List
date: 2024-11-04 00:09:52
categories:
  - Coding Test
  - Data Structure
#tags:
---
일련의 요소들이 단방향(또는 양방향)으로 연결된 자료구조입니다.

임의의 요소는 다음 요소를 참조하기 때문에 특정 요소를 검색하려면 `O(N)`의 시간 복잡도를 가집니다.

요소를 삽입하거나 제거하는데 `O(1)`의 시간 복잡도를 가지는데, 이는 참조할 다음 요소만 바꾸는 것만으로 요소를 해당 연산을 구현할 수 있기 때문입니다.

## 구현코드

```js
const [DATA, NEXT] = [0, 1];

const pair = (a, b) => [b, a];
const hasData = (node) => Boolean(node.length);

const linkedList = (arr) => {
  let head = arr.reduceRight(pair, []);

  const append = (data) => {
    for (let p = head; hasData(p); p = p[NEXT]) {}
    p.push(data, []);
  };

  const prepend = (data) => {
    head = [data, head];
  };

  const insertAt = (pos, data) => {
    if (pos === 0 || !hasData(head)) {
      prepend(data);
      return;
    }
    for (let p = head, i = 0; hasData(p); p = p[NEXT], i++) {
      if (i + 1 !== pos) continue;
      p[NEXT] = [data, p[NEXT]];
      break;
    }
  };

  const deleteData = (data) => {
    if (head[DATA] === data) {
      head = head[NEXT];
      return;
    }
    for (let p = head; hasData(p); p = p[NEXT]) {
      if (p[NEXT][DATA] !== data) continue;
      p[NEXT] = p[NEXT][NEXT];
      break;
    }
  };

  return {
    append,
    prepend,
    deleteData,
  };
};
```

## 참고자료

[HackerRank](https://www.youtube.com/@HackerrankOfficial/playlists)