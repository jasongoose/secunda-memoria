---
title: 명시적 바인딩
date: 2024-11-01 00:38:36
categories:
  - Books
  - 코어 자바스크립트
#tags:
---
## call / apply 메서드

```js
Function.prototype.call(thisArg[, arg1[, arg2...]])
```

```js
Function.prototype.apply(thisArg[, argsArray])
```

### 유사배열객체

실제 `Array`의 인스턴스는 아니지만 0 이상의 정수 index와 `length` 속성을 가지는 객체입니다.

`length` 속성을 수동으로 업데이트한다는 점에서 기존 배열과 차이가 있습니다.

흔히 볼 수 있는 유사배열객체들을 나열하면 다음과 같습니다.

- 함수의 `arguments`
- string
- `NodeList`
- `HTMLCollection`
- `FileList`

유사배열객체는 `Array.prototype` 메서드를 바로 사용할 수 없기 때문에 명시적 바인딩이 필요합니다.

```js
var obj = {
  0: "a",
  1: "b",
  2: "c",
  length: 3,
};

Array.prototype.push.call(obj, "d");
Array.prototype.slice.apply(obj);
```

```js
Array.prototype.forEach.call("a string", (chr) => {
  console.log(chr);
});
```

아니면 유사배열객체를 `Array.from` 메서드나 spread syntax로 배열로 변환해서 구현할 수도 있습니다.

```js
const collection = Array.from(document.getElementsByTagName("li"));

function checkArgs() {
  [...arguments].forEach((elem) => {
    console.log(elem);
  });
}
```

### 생성자 상속

다른 생성자의 `this`를 현재 생성자 내부의 `this`로 전달하여 중복되는 필드들을 간단하게 초기화할 수 있습니다.

```js
function Person(name, gender) {
  this.name = name;
  this.gender = gender;
}

function Student(name, gender, school) {
  Person.call(this, name, gender);
  this.school = school;
}

function Employee(name, gender, company) {
  Person.apply(this, [name, gender]);
  this.company = company;
}

var by = new Student("jason", "male", "du");
var jn = new Employee("shin", "female", "ud");
```

### 여러 인수를 하나의 배열로 전달

`apply` 메서드를 사용하면 여러 개의 인자를 받는 함수에 하나의 배열로 인수들을 전달할 수 있습니다.

```js
var numbers = [11, 17, 73, 97];
var max = Math.max.apply(null, numbers);
var min = Math.min.apply(null, numbers);
```

물론 ES6의 spread syntax를 사용하면 훨씬 더 간단하게 할 수 있습니다.

```js
var numbers = [11, 17, 73, 97];
var max = Math.max(...numbers);
var min = Math.min(...numbers);
```

## bind 메서드

```js
Function.prototype.bind(thisArg[, arg1[, arg2...]])
```

ES5에서 도입된 기능으로, 즉시 호출하지는 않고 넘겨받은 thisArg와 인자들을 바탕으로 새로운 함수를 반환하는데, 이 함수를 호출할 때 넘긴 인자들은 직전 `bind` 메서드를 호출할 때 전달했던 인수 뒤에 이어서 등록됩니다.

`bind` 메서드를 사용하면 `this`를 미리 적용하거나 [부분적용함수](../../closure/example#부분적용함수)를 구현할 수 있습니다.

```js
var func = function (a, b, c, d) {
  console.log(this, a, b, c, d);
};

var bindFunc = func.bind({ x: 1 }, 4, 5);

func(bindFunc(6, 8)); // {x: 1}, 4, 5, 6, 8
```

또한 `bind` 메서드로 반환된 함수의 `name` 속성이 "bound"라는 prefix가 붙는다는 성질을 이용하여 임의의 함수가 `bind`로 생성된 함수인지 여부를 확인할 수 있습니다.

```js
console.log(bindFunc.name); // bound func
```

## callback의 this 지정

`Array`, `Map`, `Set`의 prototype 메서드 다수는 callback 내부에서 `this`로 지정할 객체를 전달할 수 있습니다.

```js
Array.prototype.forEach(callback[, thisArg])
Array.prototype.map(callback[, thisArg])
Array.prototype.filter(callback[, thisArg])
Array.prototype.some(callback[, thisArg])
Array.prototype.find(callback[, thisArg])
Array.prototype.findIndex(callback[, thisArg])
Array.prototype.flatMap(callback[, thisArg])
Array.prototype.from(arrayLink[,callback[, thisArg]])

Set.prototype.forEach(callback[, thisArg])

Map.prototype.forEach(callback[, thisArg])

```

여기서 thisArg를 생략하면 자동으로 전역객체를 가리킵니다.
