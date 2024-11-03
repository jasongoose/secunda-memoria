---
title: Trie
date: 2024-11-04 00:10:16
categories:
  - Coding Test
  - Data Structure
#tags:
---
문자열들을 저장하고 검색하기 위한 트리기반의 자료구조입니다.

![Trie](/images/trie.png)

제일 긴 문자열의 길이가 L이고 총 문자열의 개수가 M일 때, 생성할 때 시간복잡도는 `O(M * L)`이고 검색할 때 시간복잡도는 `O(L)`입니다.

Trie는 임의의 문단으로부터 단어를 찾거나 자동완성 기능을 구현할 때 활용됩니다.

## 구현코드

```js
const [DATA, ISEND, MAP] = [0, 1, 2];

const getNode = (data = "") => [data, false, new Map()];

const trie = () => {
  let root = getNode();

  const insert = (data) => {
    let p = root;
    for (const c of data) {
      if (!p[MAP].has(c)) {
        p[MAP].set(c, getNode(p[DATA] + c));
      }
      p = p[MAP].get(c);
    }
    p[ISEND] = true;
  };

  const contains = (keyword) => {
    let p = root;
    for (const c of keyword) {
      if (!p[MAP].has(c)) {
        return false;
      }
      p = p[MAP].get(c);
    }
    return true;
  };

  return {
    insert,
    contains,
  };
};
```

## 참고자료

[HackerRank](https://www.youtube.com/@HackerrankOfficial/playlists)