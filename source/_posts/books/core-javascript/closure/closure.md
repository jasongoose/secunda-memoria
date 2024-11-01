---
title: 클로져
date: 2024-11-01 00:45:14
categories:
  - Books
  - 코어 자바스크립트
#tags:
---
여러 [함수형 프로그래밍](../../../composing-software/introduction/what-is-fp)언어에서 보이는 보편적인 특성으로, JS 고유의 개념이 아닙니다.

책에서 설명하는 정의는 다음과 같습니다.

> 클로저란 어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 B를 외부로 전달할 경우 A의 context가 종료된 이후에도 변수 a가 사라지지 않는 현상을 말합니다.

개인적으로 아래와 같이 다르게 표현할 수 있지 않을까 싶습니다🤔

> 콜 스택 상의 하위 context에서 이미 종료된 상위 context의 lexEnv/EnvRec에 접근할 수 있는 현상

아니면 다음과 같이 간결하게 정의할 수도 있다고 생각합니다.

> 함수와 해당 함수의 lexEnv를 bundling하는 것

클로져가 가능한 이유는 임의의 context/lexEnv는 이를 참조할 예정인 다른 context가 있는 한 실행종료 이후에도 가바지 컬렉터에 의해서 메모리 공간이 회수되지 않기 때문입니다.

클로져가 발생하는 경우는 2가지가 있습니다.

1. 외부함수의 변수를 참조하는 내부함수를 반환하는 경우

```js
var outer = function () {
  var a = 1;
  var inner = function () {
    return ++a;
  };
  return inner;
};

var outer2 = outer();
console.log(outer2()); // 2
console.log(outer2()); // 3
```

2. 즉시실행함수(IIFE)로 비동기 작업(`setInterval`, `setTimeout`, event listener)을 실행하는 경우

```js
(function() {
	var a = 0;
	var intervalId = null;
	var inner = funciton() {
		if (++a >= 10) {
			clearInterval(intervalId);
		}
		console.log(a);
	};

	intervalId = setInterval(inner, 1000);
})()
```

```js
(function () {
  var count = 0;
  var button = document.createElement("button");
  button.innerText = "click";
  button.addEventListener("click", function () {
    console.log(++count, "times clicked");
  });
  document.body.appendChild(button);
})();
```

## 메모리 누수

흔히 클로져의 단점은 "메모리 누수"라고 합니다.

종료된 뒤에 사라져야 할 context에 위치한 변수를 사용하거나 함수를 호출하기 때문에 메모리 공간을 계속 소모하는 점이 문제인데,

적절한 시점에 해당 공간이 가비지 컬렉터에 의해서 자동으로 회수되도록 참조되는 변수로 `null`을 할당하면 됩니다.

```js
var outer = function () {
  var a = 1;
  var inner = function () {
    return ++a;
  };
  return inner;
};

var outer2 = outer();
console.log(outer2()); // 2
console.log(outer2()); // 3
outer2 = null; // [!code ++]
```

```js
(function() {
	var a = 0;
	var intervalId = null;
	var inner = function() {
		if (++a >= 10) {
			clearInterval(intervalId);
			intervalId = null; // [!code ++]
		}
		console.log(a);
	};

	intervalId = setInterval(inner, 1000);
})()
```

```js
(function () {
  var count = 0;
  var button = document.createElement("button");
  button.innerText = "click";

  var clickHandler = function () {
    console.log(++count, "times checked");
    if (count >= 10) {
      button.removeEventListener("click", clickHandler);
      clickHandler = null; // [!code ++]
    }
  };

  button.addEventListener("click", clickHandler);
  document.body.appendChild(button);
})();
```
