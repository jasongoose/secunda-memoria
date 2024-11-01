---
title: 콜백함수
date: 2024-11-01 00:42:50
categories:
  - Books
  - 코어 자바스크립트
#tags:
---
함수의 인자로 전달된 함수 또는 메서드를 의미합니다.

콜백함수(이하 cb)를 언제 호출할지와 매개변수의 순서, 종류는 전적으로 제어권을 가진 함수(이하 제어함수)가 가지고 있고 `call`, `apply` 메서드를 이용하여 cb 내부에서 `this`를 명시적으로 바인딩할 수 있습니다.

cb을 받는 built-in 메서드들은 cb과 thisArg을 선택적으로 받는데, thisArg가 전달되지 않는다면 default로 전역객체를 참조합니다.

물론 정확한 건 제어함수의 정의를 봐야합니다.

어떤 객체의 메서드를 cb으로 전달했을 때 cb 내부 `this`는 부모 객체가 아닌 default로 전역객체를 가리킵니다.

그래도 cb로 전달한 메서드 내부에서 `this`가 부모 객체를 가리키도록 만들고 싶다면 ES5의 `bind` 메서드로 새로운 함수를 만들면 됩니다.

```js
var obj1 = {
  name: "obj1",
  func: function () {
    console.log(this.name);
  },
};

setTimeout(obj1.func.bind(obj1), 1000);

var obj2 = { name: "obj2" };
setTimeout(obj1.func.bind(obj2), 1500);
```

## callback HELL

![callback HELL](/images/callback_hell.png)

cb이 등장한 것은 작업 A를 실행하는 중에 원하는 때에 또 다른 작업 B를 비동기적으로 실행하기 위해서입니다.

하지만 임의의 작업 중에 진행할 작업들을 계속 추가하게 되면 그에 따라 전달해야 하는 cb도 많아질 겁니다.

그럼 함수가 점점 더 깊어지면서 가독성이 떨어지고 디버깅이 어려워지는 지경에 이르는데, 이를 지옥같다고 해서 "callback HELL"이라고 합니다.

## sync, async

동기적인 작업(synchronous)은 CPU에 의해서 즉시 실행이 가능한 연산을 가리킵니다.

반대로 비동기적인 작업(asynchronous)은 **별도의 요청, 실행대기, 보류에 의해서 즉시 실행이 불가한 연산**을 가리킵니다.

일련의 시간이 지나거나 조건을 만족했을 때 실행할 작업은 cb로 전달하면 됩니다.

- 별도의 요청: `XMLHttpRequest`, `fetch`(서버로 요청을 보내고 응답이 와야 cb 실행)
- 실행대기: `addEventListener`(event가 발생할 때까지 대기하다가 감지하면 cb 실행)
- 보류: `setTimeout`(일정 시간이 경과된 뒤에서야 cb 실행)

과거에는 일련의 비동기 작업들을 동기적으로 처리하기 위해서 콜백지옥을 감당해야 했지만 현대 JS에서는 ES6의 Promise, Generator 또는 ES2017의 async, await로 깔끔하게 구현할 수 있습니다🥳
