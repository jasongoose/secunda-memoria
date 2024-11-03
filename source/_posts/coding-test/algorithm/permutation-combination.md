---
title: 순열, 조합
date: 2024-11-03 23:48:04
categories:
  - Coding Test
  - Algorithm
#tags:
---
## 순열

길이가 n인 배열에서 r개의 요소를 차례대로 뽑아 새로운 배열을 만드는 모든 경우의 수들을 나열합니다.

### 구현코드

배열의 요소를 차례대로 하나씩 선택한 상태에서 선택하지 않은 요소들을 재귀적으로 순회하면 됩니다.

여기서 한번 재귀호출을 할 때마다 선택한 요소들을 누적해야 한다는 점에서 [DFS](../dfs-bfs#DFS)와 동일합니다!

```js
const permutation = (arr, r) => {
  const visited = new Set();
  const ans = [];

  const calc = () => {
    if (visited.size === r) {
      ans.push([...visited]);
      return;
    }
    for (const el of arr) {
      if (visited.has(el)) {
        continue;
      }
      visited.add(el);
      calc();
      visited.delete(el);
    }
  };

  calc();

  return ans;
};
```

### 응용

1. 일련의 0 ~ 9까지의 숫자들(중복될 수도 있음)이 배열이나 문자열로 주어졌을 때, 최소 1개라도 골라서 만들 수 있는 수들은 다음과 같이 구할 수 있습니다.

```js
const numbers = "711";

// 만들 수 있는 모든 수들(string)
const cases = new Set();
{
  // 숫자는 중복될 수도 있으니 숫자에 대응되는 index들을 기준으로 선택여부를 판단합니다.
  const visited = new Set();

  const calc = (currStr = "") => {
    // 생성한 문자열의 길이가 numbers의 길이를 넘지 않도록 제한합니다.
    if (numbers.length < currStr.length) {
      return;
    }

    // 생성한 문자열을 cases에 추가합니다.
    cases.add(currStr);

    for (let i = 0; i < numbers.length; i++) {
      // 선택한 숫자들은 생략하여 무한 loop를 방지합니다.
      if (visited.has(i)) continue;
      // 다음에 올 숫자의 index를 선택하여 재귀적으로 처리합니다.
      visited.add(i);
      calc(currStr + numbers[i]);
      visited.delete(i);
    }
  };

  calc();
}

console.log(...cases); // 7 71 711 1 17 171 11 117
```

2. 주어진 문자들로 만들 수 있는 모든 길이의 단어들은 다음과 같이 구할 수 있습니다.

```js
const CHARS = "AEIOU";
const dict = [];
{
  const calc = (currStr = "") => {
    if (CHARS.length < currStr.length) {
      return;
    }
    dict.push(currStr);

    for (let i = 0; i < CHARS.length; i++) {
      calc(currStr + CHARS[i]);
    }
  };
  calc();
}
```

현재까지 선택한 요소들은 `visited`라는 Set에 저장할 수도 있지만 재귀함수의 인자(`currStr`)로도 전달하는 방식도 요긴하게 이용할 수 있습니다!

## 조합

길이가 n인 배열에서 r개의 요소를 뽑을 수 있는 경우의 수들을 나열한 것으로 순열과 다르게 요소들간의 순서를 고려하지 않습니다.

### 구현코드

순열과 동일하게 구현하되, 한번 재귀호출을 할 때마다 순회하는 범위를 다음 요소부터 시작하도록 좁혀야 합니다.

```js
const combination = (arr, r) => {
  const visited = new Set();
  const ans = [];

  const calc = (lastIndex = 0) => {
    if (visited.size === r) {
      ans.push([...visited]);
      return;
    }
    for (let i = lastIndex; i < arr.length; i++) {
      visited.add(arr[i]);
      calc(i + 1);
      visited.delete(arr[i]);
    }
  };

  calc();

  return ans;
};
```

길이가 n인 배열에서 유한개의 요소들을 선택하는 것은 `[0, 2^n)`를 순회하면서 비트셋을 해석하는 방식으로도 해결할 수 있습니다.

비트연산 자체는 속도는 빠르지만 시간복잡도는 `O(2^N)`를 가진다는 단점은 있습니다.

## 참고자료

[HackerRank](https://www.youtube.com/@HackerrankOfficial/playlists)

[순열(Permutation)과 조합(Combination) 알고리즘](https://aerocode.net/376)