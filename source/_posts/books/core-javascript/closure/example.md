---
title: 클로져 활용예시
date: 2024-11-01 00:45:22
categories:
  - Books
  - 코어 자바스크립트
#tags:
---
## cb에서 외부 데이터를 사용할 때

```js
var fruits = ["apple", "banana", "peach"];
var $ul = document.createElement("ul");

var alertFruitBuilder = function (fruit) {
  return function () {
    alert(`your choice is ${fruit}`); // 👈
  };
};

fruits.forEach(function (fruit) {
  var $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListener("click", alertFruitBuilder(fruit));
  $ul.appendChild($li);
});

document.body.appendChild($ul);
```

## 정보 은닉

모듈의 내부 로직을 외부로부터 숨겨서 모듈 간의 결합도를 낮추고 유연성을 높이는 현대 프로그래밍 언어의 중요한 개념 중 하나입니다.

클로져를 사용하면 외부로 노출할 public 데이터와 context 내에서만 유효한 private 데이터를 구분하여 정의할 수 있습니다.

```js
var outer = function () {
  var a = 1; // 👈 은닉대상
  var inner = function () {
    return ++a;
  }; // 👈 공개대상
  return inner;
};

var outer2 = outer();
console.log(outer2()); // 2
console.log(outer2()); // 3
outer2 = null;
```

## 부분적용함수

n개의 인자를 받는 함수에 미리 m(< n)개의 인자만 넘겨 기억하다가 나중에 (n - m)개의 인자를 넘기면 비로소 원래의 함수의 실행결과를 얻을 수 있는 함수입니다.

`Function.prototype.bind`의 반환값이기도 합니다!

```js
var partial = function () {
  var originalPartialArgs = arguments; // 은닉된 변수
  var func = originalPartialArgs[0];
  if (typeof func !== "function") {
    throw new Error("first arg is not function");
  }
  return function () {
    var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1);
    var restArgs = Array.prototype.slice.call(arguments);
    return func.apply(this, partialArgs.concat(restArgs));
    // 실행시점의 this를 그대로 넘겨주면 this binding 문제를 예방할 수 있습니다.
  };
};
```

```js
var add = function () {
  var result = 0;
  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i];
  }
  return result;
};

var addPartial = partial(add, 1, 2, 3, 4, 5);
console.log(addPartial(6, 7, 8, 9, 10));
```

```js
var dog = {
  name: "dog",
  greet: partial(function (prefix, suffix) {
    return prefix + this.name + suffix;
  }),
};

dog.greet("hi", "hi");
```

## 커링함수

여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수들로 나눠서 순차적으로 호출될 수 있게 chain 형태로 구성한 형태입니다.

커링은 한번에 하나의 인자만 전달하는 것을 원칙으로 합니다.

중간에 함수를 실행한 결과는 다음 인자를 받기 위해 대기만 할 뿐이고, 마지막 인자가 전달되기 전까지는 원본 함수가 실행되지 않습니다.

부분적용함수는 한번에 여러 개의 인자를 전달할 수 있고 인자를 부분적으로 전달하여 실행결과를 재실행할 때 원본 함수가 무조건 실행된다는 점에서 커링함수와 다릅니다.

커링의 각 단계에서 받은 인자들은 모두 마지막 단계에서 참조하기 때문에 회수되지 않고 메모리에 차곡차곡 쌓였다가, 마지막 호출로 context가 종료된 후에야 비로소 한꺼번에 가비지 컬렉터의 수거대상이 됩니다.

이와 같이 당장 필요한 정보만 받아서 차례대로 전달하면서 최종 함수의 실행을 미루는 방식을 함수형 프로그래밍에서는 지연실행(Lazy Execution)이라고 합니다.

커링함수는 ES6의 arrow function syntax를 이용하면 깔끔하게 구현할 수 있습니다.

```js
const curry3 = (func) => (a) => (b) => func(a, b);
```

```js
const getMaxWith10 = curry3(Math.max)(10);

console.log(getMaxWith10(8)); // 10
console.log(getMaxWith10(25); // 25
```

원하는 시점까지 지연시켰다가 실행하거나 프로젝트 내에서 자주 쓰이는 함수의 매개변수가 항상 비슷하고 일부만 바뀌는 경우에 커링을 사용할 수 있습니다.

```js
const getInformation = (baseUrl) => (path) => (id) =>
  fetch(`${baseUrl}${path}/${id}`);
```

위와 같이 서버에 정보를 요청할 때마다 전체 URL을 전부 기입할 필요없이 공통적인 요소는 먼저 기억시켜두고 특정한 값만 대입하여 요청을 수행하는 함수를 만들어두면 개발 효율성과 가독성을 높일 수 있습니다.

## debounce, throttle

함수호출 횟수를 제한하기 위한 프로그래밍 기법들입니다.

주로 짧은 시간에 동일한 이벤트(scrolling, wheel, mousemove, window resizing 등)가 빈번하게 발생할 때마다 API 요청처리와 같이 시간이 걸리는 작업을 처리해야 하는 상황에 활용됩니다.

### debounce

이벤트의 발생빈도 수와 상관없이 마지막으로 이벤트가 발생한 시점부터 지정된 시간 뒤에 콜백함수를 실행하는 방법입니다.

```js
const debounce = function (func, wait) {
  let timeoutId = null;
  return function (event) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(function () {
      func.call(this, event);
    }, wait);
  };
};
```

```js
const moveHandler = function (e) {
  console.log("move event 처리");
};

const wheelHandler = function (e) {
  console.log("wheel event 처리");
};

document.body.addEventListener("mousemove", debounce(moveHandler, 500));
document.body.addEventListener("mousewheel", debounce(wheelHandler, 500));
```

### throttle

첫 이벤트가 발생한 시점부터 일정한 간격으로 콜백함수를 실행하는 방법입니다.

```js
const throttle = function (func, interval) {
  let timerId = null;
  return function (event) {
    if (timerId) return;
    timerId = setTimeout(function () {
      func.call(this, event);
      clearTimeout(timerId);
      timerId = null;
    }, interval);
  };
};
```

```js
const moveHandler = function (e) {
  console.log("move event 처리");
};

const wheelHandler = function (e) {
  console.log("wheel event 처리");
};

document.body.addEventListener("mousemove", throttle(moveHandler, 500));
document.body.addEventListener("mousewheel", throttle(wheelHandler, 500));
```

위 두 예시에서 event handler 내부의 `this`는 `bind` 메서드에 의해서 `document.body`로 바인딩됩니다.
