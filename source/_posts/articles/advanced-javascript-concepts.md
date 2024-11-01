---
title: Advanced JavaScript Concepts that Helped Me Get Better at Coding
date: 2024-11-01 22:09:31
categories:
  - Articles
#tags:
---
## Conditionally add a property to an object

object에 조건을 만족하는 경우에만 특정 속성을 추가하는 로직이 필요할 때가 가끔이 아닌 정말로 자주 있을겁니다.

그럴 때마다 해당 속성을 가진 객체와 그렇지 않은 객체를 따로 생성해서 삼항 연산자로 전달하거나,

```js
return condition ? { ...rest, hello } : { ...rest };
```

아니면 전달하지 않았을 때 default 값을 억지로 할당했습니다.

```js
const { a = null, b = 0 } = what;
```

특히 Axios 요청으로 전달할 query나 body가 없는 경우에 requestConfig 객체를 어떻게 작성할지 엄청 고민했었는데 spread syntax로 이 문제를 간단하게 해결할 수 있습니다.

```js
const moreInfos = { info: "Please go to the desk." };

return {
  address: "20B Rue Lafayette",
  postcode: "75009",
  ...(moreInfos !== undefined && { moreInfos }),
};
```

비슷한 방식으로 array에도 그대로 적용할 수 있습니다.

```js
const cond = false;
const arr = [...(cond ? ["a"] : []), "b"];
// ['b']
```

## Reference for non-primitive value

JS 코어 개념들을 익히면서 가끔씩 헷갈렸던 부분이 있었습니다.

> 함수의 인자로 전달한 객체는 두 context 상에서 공유되는 것인가?

정답은 `true`입니다!

그러므로 함수 body 내에서 객체 인자의 mutation은 함수를 호출한 context상 연산에도 영향을 미치는데 이를 바로 side-effect라고 합니다.

```js
const contactInfos = { address: "20B Rue Lafayette" };

function setAddress(infos) {
  infos.address = "90B Rue laffite";
}

console.warn(contactInfos); // { address: "20B Rue Lafayette" }

setAddress(contactInfos);

console.warn(contactInfos); // { address: "90B Rue laffite" }
```

따라서 가급적이면 함수 body 내에서 객체인자를 mutate하는 로직은 작성하는 것은 피하고, 대신 수정된 버전의 새로운 객체를 생성하여 반환하는 것을 권장합니다.

## 참고자료

[Conditionally adding entries inside Array and object literals](https://2ality.com/2017/04/conditional-literal-entries.html)
