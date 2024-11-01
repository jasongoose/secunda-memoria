---
title: 프로토타입
date: 2024-11-01 00:45:44
categories:
  - Books
  - 코어 자바스크립트
#tags:
---
![Prototype Diagram](/images/prototype_diagram.png)

1. Constructor(생성자)의 prototype에는 instance가 사용할 메서드와 속성들이 위치합니다.

2. Constructor를 `new` 키워드와 함께 호출하면 instance가 생성됩니다.

3. instance의 `__proto__`는 Constructor의 prototype을 참조합니다.

4. `__proto__`는 생략이 가능한 속성이기 때문에 instance는 prototype에 정의된 메서드와 속성들을 마치 자신의 것처럼 바로 접근해서 사용할 수 있습니다.

만일 prototype에 접근하고자 한다면 [deprecate된 `__proto__`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto) 대신 다음과 같이 작성해야 합니다.

```js
Object.getPrototypeOf(instance);
Reflect.getPrototypeOf(instance);
```

## `constructor` 속성

Constructor의 prototype 객체의 `constructor` 속성은 생성자 자기 자신을 참조합니다.

```js
Constructor.prototype.constructor === Constructor;
```

prototype의 `constructor`에 다른 객체를 대입해도 그 변화가 반영되지 않으므로, instance의 생성자가 무엇인지에 관한 정보를 얻을 때 prototype의 `constructor` 속성에 의존해서는 안됩니다.

대신 `instanceof` 연산자로 인스턴스 여부를 `boolean` 데이터로 알 수 있습니다.

primitive Wrapper 생성자(`Number`, `String`, `Boolean` 등) prototype의 constructor는 read-only이고, 그 외에는 모두 write가 가능합니다.
