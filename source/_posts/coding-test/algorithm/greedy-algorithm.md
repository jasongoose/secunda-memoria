---
title: Greedy Algorithm
date: 2024-11-03 23:45:19
categories:
  - Coding Test
  - Algorithm
#tags:
---
단계마다 선택하는 최선의 선택이 곧 전체 최적해가 되도록 하는 알고리즘입니다.

전체 총합의 최댓값/최솟값을 구하려면 매 순간 최댓값/최솟값을 선택하면 되는 것과 같은 맥락입니다.

## Kruskal's Algorithm

간선별로 가중치가 주어진 트리가 있을 때, `(n - 1)`개의 간선을 사용하여 최소한의 가중치의 합으로 모든 node들이 연결된 부분 트리를 만드는 대표적인 알고리즘입니다.

### Union-Find Algorithm

여기서 사이클을 형성하는지 여부는 선택한 간선이 기존의 선택된 요소들이 이루는 그래프에 속하는지 여부로 알 수 있는데, 요소별로 속하는 집합의 정보를 다루는 "Union-Find Algorithm"으로 구할 수 있습니다.

```js
const unionFind = (n) => {
  // node별로 root를 자기 자신으로 지정합니다.
  const root = Array.from(Array(n), (_, i) => i);

  // 특정 node의 root를 재귀적으로 검색합니다.
  const getRoot = (x) => {
    if (x === root[x]) {
      return x;
    }
    return getRoot(root[x]);
  };

  // 두 node의 root가 동일한지 확인합니다.
  const isSameRoot = (x, y) => {
    return getRoot(x) === getRoot(y);
  };

  // index가 작은 쪽으로 두 node가 속한 집합들을 합칩니다.
  const merge = (x, y) => {
    const [rootX, rootY] = [getRoot(x), getRoot(y)];
    if (rootX < rootY) {
      root[rootY] = rootX;
    } else {
      root[rootX] = rootY;
    }
  };

  return {
    isSameRoot,
    merge,
  };
};
```

### 구현코드

가중치의 오름차순으로 정렬된 간선들을 현재까지 만들어진 트리와 사이클을 형성하지 않도록 차례대로 선택하면 됩니다.

```js
// n = node의 개수
// costs = 트리 내의 간선을 요소로 가지는 배열로 두 node와 가중치의 값을 [v0, v1, cost] 형태로 가집니다.
const kruskal = (n, costs) => {
  const { isSameRoot, merge } = unionFind(n);

  const cl = costs.sort((x, y) => x[2] - y[2]); // 비용의 오름차순으로 먼저 정렬합니다.
  let ans = 0;

  for (const [v0, v1, cost] of cl) {
    if (isSameRoot(v0, v1)) continue;
    merge(v0, v1);
    ans += cost;
  }

  return ans;
};
```

## 참고자료

[HackerRank](https://www.youtube.com/@HackerrankOfficial/playlists)