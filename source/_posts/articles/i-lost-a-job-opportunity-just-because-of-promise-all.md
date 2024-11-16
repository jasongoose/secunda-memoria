---
title: I Lost a Job Opportunity Just Because of Promise.all
date: 2024-11-01 22:14:09
categories:
  - Articles
#tags:
---
## Promise.resolve

```ts
const $resolve = (t: unknown) => {
  return new Promise((rs, _) => {
    rs(t);
  });
};

$resolve(1).then(console.log); // 1
$resolve(Promise.resolve(Promise.resolve(1))).then(console.log); // 1
```

## Promise.reject

```ts
const $reject = (t: unknown) => {
	return new Promise((_, rj) => { rj(t); })
}

$reject(new Error('fail'))
  .then(() => console.log('Resolved')
  .catch((err) => console.log('Rejected', err)) // "Rejected",  fail
```

## Promise.all

서로 관련있는 Promise들을 차례대로 실행하여 모두 resolve되어야 하는 상황에서 많이 활용되고, resolved value 배열을 resolve하는 promise를 반환합니다.

처음에는 아래처럼 `values` 배열 요소에 대해서 한번씩 Promise를 resolve하는 방식으로 구현했었는데,

```js
const $all = async (values) => {
	return new Promise((rs, rj)=> {
		const result = [];
		values.forEach((v) => {
			v.then(result.push).catch(rj)
		});
		rs(result);
	})
```

하지만 막상 실행하면 길이가 1인 배열만 반환되는데 왜 그런걸까요?

Promise는 비동기 작업을 수행하기 때문에 A→B→C→D→… 순서대로 resolve를 수행하려면 동일한 Promise chain 상에 있어야 합니다.

동기 작업들을 연속적으로 나열하면 차례대로 실행하지만 비동기 작업들을 연속적으로 나열하면 각 작업은 순서와 상관없이 독립적으로(병렬로) 수행됩니다.

배열에 있는 Promise들을 차례대로 resolve하려면 원래 아래와 같이 단일 chaining을 적용해야 합니다.

```js
values[0]
	.then(() => values[1])
	.then(() => values[2])
	.then(() => values[3])
	.then(() => values[4])
	.then( ... )	 // chain_0
```

하지만 forEach에서 요소별 resolve만 수행한다면 서로 다른 chain상에 위치하기 때문에 아래와 같이 동기적으로 나열한 것과 동일합니다.

```js
values[0]
values[1]
values[2]
values[3]
values[4]
...
```

그렇다면 `Promise.all`처럼 차례대로 resolve하려면 어떻게 해야할까요?

### for-loop

```js
for (let i = 0, p = Promise.resolve(); i < 10; i++) {
  p = p.then(() => console.log(i));
}
```

처음 변수 `p`에 즉시 resolve되는 Promise를 전달하고 매번 for-loop를 돌 때마다 동일한 Promise chain을 유지할 수 있도록 then callback에서 반환된 Promise를 다시 `p`에 대입하는 방식입니다.

```js
const $all = (promises) =>
  new Promise((rs, rj) => {
    const result = [];
    for (let i = 0, p = Promise.resolve(); i < promises.length; i++) {
      p = p
        .then(() => promises[i])
        .then((v) => {
          result.push(v);
          i + 1 === promises.length && rs(result);
        })
        .catch(rj);
    }
  });
```

### reduce

```js
Array(10).reduce((p, _, i) => {
  return p.then(() => console.log(i));
}, Promise.resolve());
```

for-loop 방식을 1줄(?)로 표현하는 방식입니다.

```js
const $all = (promises) =>
  new Promise((rs, rj) => {
    const result = [];
    promises.reduce((acc, p, i) => {
      return acc
        .then(() => p)
        .then((v) => {
          result.push(v);
          i + 1 === promises.length && rs(result);
        })
        .catch(rj);
    }, Promise.resolve());
  });
```

### for await-of

```js
async function* genPromise(count) {
  for (let i = 0; i < count; i++) {
    yield Promise.resolve(i);
  }
}

(async function () {
  for await (let i of genPromise(10)) {
    console.log(i);
  }
})();
```

generator 함수에서 반복적으로 resolved Promise를 yield하면 ES2020에서 새로 추가된 `for await-of` 문에서 resolved value를 처리하는 방식입니다.

```js
const $all = async (promises) => {
  const result = [];
  try {
    for await (const v of promises) {
      result.push(v);
    }
  } catch (err) {
    return Promise.reject(err);
  }
  return Promise.resolve(result);
};
```

## Promise.allSettled

서로 관련있는 Promise들을 차례대로 실행하여 모두 settle(resolve | reject)되어야 하는 상황에서 많이 활용되고, settled value 배열을 resolve하는 promise를 반환합니다.

기존 `Promise.all` 함수의 `.catch` 부분에서 result 배열에 push하고 result 배열을 `.finally` 부분에서 resolve하면 됩니다.

```js
// for-loop
const $allSettled = (promises) =>
  new Promise((rs, rj) => {
    const result = [];
    for (let i = 0, p = Promise.resolve(); i < promises.length; i++) {
      p = p
        .then(() => promises[i])
        .then((value) => {
          result.push({ status: "fulfilled", value });
        })
        .catch((value) => {
          result.push({ status: "rejected", value });
        })
        .finally(() => {
          i + 1 === promises.length && rs(result);
        });
    }
  });
```

```js
// reduce
const $allSettled = (promises) =>
  new Promise((rs, rj) => {
    const result = [];
    promises.reduce((acc, p, i) => {
      return acc
        .then(() => p)
        .then((value) => {
          result.push({ status: "fulfilled", value });
        })
        .catch((value) => {
          result.push({ status: "reject", value });
        })
        .finally(() => {
          i + 1 === promises.length && rs(result);
        });
    }, Promise.resolve());
  });
```

## 출처

[JavaScript ES6 promise for loop](https://stackoverflow.com/questions/40328932/javascript-es6-promise-for-loop)

[I Lost a Job Opportunity Just Because of Promise.all](https://javascript.plainenglish.io/i-lost-a-job-opportunity-just-because-of-promise-all-be396f6efe87)
