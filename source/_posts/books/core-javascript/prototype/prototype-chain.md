---
title: 프로토타입 체인
date: 2024-11-01 00:45:49
categories:
  - Books
  - 코어 자바스크립트
#tags:
---
JS 엔진은 상위 prototype들을 따라서 객체의 속성이나 메서드를 검색하는데 이를 "Prototype Chain"이라고 부릅니다.

![Prototype Chain Diagram](/images/prototype_chain_diagram.png)

prototype 객체는 `Object` 생성자의 instance이고 생성자 자체는 `Function` 생성자의 instance이기 때문에 체인 형태를 가지게 됩니다.

보통 prototype chain 상에서 최상단에는 `Object` 생성자의 prototype이 위치합니다.

## 오버라이딩

scope chain에서 발생하는 [변수 은닉화](../../execution-context/lexical-environment)와 비슷하게 prototype chain에 의해서 instance가 상위 prototype에 있는 동일한 이름의 메서드나 속성에 직접 접근할 수 없는 현상을 가리킵니다.

물론 상위 prototype에 있는 메서드나 속성은 [`call`/`apply`에 의한 적절한 this binding](../../this/this-binding)을 사용하면 접근할 수는 있습니다.

```js
const Person = function (name) {
  this.name = name;
};

Person.prototype.getName = function () {
  return this.name;
};
```

```js
const iu = new Person("지금");

iu.getName = function () {
  return `바로 ${this.name}`;
};

console.log(iu.getName()); // 바로 지금
Object.getPrototypeOf(iu).getName.call(iu); // '지금'
```

## 객체전용 메서드의 예외사항

### object-only

`Object` 생성자의 prototype에는 어떤 종류의 생성자, prototype 객체, primitive Wrapper 등에서 사용할 수 있는 범용 메서드, 속성들이 위치합니다.

하지만 오직 Reference instance들만 사용할 수 있는 객체전용(object-only) 메서드나 속성들은 `Object` 생성자의 정적 메서드와 own-property에 위치합니다.

객체전용 기능들은 명시적 this binding이 아닌 대상 instance를 인자로 전달하는 방식으로 구현할 수 있습니다.

![Object Only](/images/object_only.png)

### 예외사항

누군가 이런 질문을 할 수도 있습니다.

> 프로토타입 체인 상 가장 마지막에는 항상 Object.prototype이 있나요?

자칫하면 맞다고 대답할 수도 있는데 사실은 거짓이고 예외가 존재합니다🙄

`Object.create` 메서드는 인자로 전달된 객체를 prototype으로 가지는 instance를 만들 수 있는데 `null`로 전달하면 prototype이 없는 인스턴스를 만들 수 있습니다.

```js
const _proto = Object.create(null);

_proto.getValue = function (key) {
  return this[key];
};

const obj = Object.create(_proto);
obj.a = 1;

console.log(obj.getValue("a")); // 1
console.dir(obj);
```

![Exception!](/images/exception.png)

출력을 보면 obj의 prototype에 `__proto__`가 없으므로 obj의 prototype이 prototype chain 상에서 최상단에 위치한다고 볼 수 있습니다.

## 다중 프로토타입 체인

생성자의 prototype이 연결하고자 하는 상위 생성자의 인스턴스를 참조하기만 한다면 prototype chain은 원하는 만큼 연결해서 만들 수 있습니다.

아래와 같이 [유사배열객체](../../this/explicit-binding)인 g의 생성자에 `Array` 생성자의 instance인 빈 배열(`[]`)로 할당하면 this binding없이 직접 `Array.proptype`의 메서드들을 직접 사용할 수 있습니다.

```js
const Grade = function () {
  const args = Array.prototype.slice.call(arguments);
  for (var i = 0; i < args; i++) {
    this[i] = args[i];
  }
  this.length = args.length;
};

const g = new Grade(100, 80);
Grade.prototype = [];

g.pop();
g.push(90);
```

![Multi Chaining](/images/multi_chaining.jpeg)
