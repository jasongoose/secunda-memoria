---
title: 클래스 상속
date: 2024-11-01 00:44:54
categories:
  - Books
  - 코어 자바스크립트
#tags:
---
## `extends` 키워드

JS에서 자식 클래스가 부모 클래스를 상속할 때 `extends` 키워드를 사용합니다.

```js
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
  run(speed) {
    this.speed = speed;
    alert(`${this.name} runs with speed ${this.speed}.`);
  }
  stop() {
    this.speed = 0;
    alert(`${this.name} stands still.`);
  }
}

class Rabbit extends Animal {
  hide() {
    alert(`${this.name} hides!`);
  }
}

let rabbit = new Rabbit("White Rabbit");

rabbit.run(5); // White Rabbit runs with speed 5.
rabbit.hide(); // White Rabbit hides!
```

클래스 Rabbit의 인스턴스인 rabbit이 부모 클래스 Animal의 인스턴스 메서드를 사용하고자 할 때, 프로토타입 체인을 따라서 올라간다는 점을 알 수 있습니다.

![Extends Keyword](/images/extends_keyword.png)

## 메서드 오버라이딩

부모 클래스에서 정의된 메서드와 동일한 이름을 가진 메서드를 자식 클래스에 정의하면 기존의 부모 클래스의 메서드가 오버라이드(override)됩니다.

하지만 오버라이드 없이 부모 클래스의 메서드를 그대로 실행하면서 부분적인 기능들을 추가할 때는 `super` 키워드를 사용하면 됩니다.

![Method Overriding](/images/method_overriding.png)

```js
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }

  run(speed) {
    this.speed = speed;
    alert(`${this.name} runs with speed ${this.speed}.`);
  }

  stop() {
    this.speed = 0;
    alert(`${this.name} stands still.`);
  }
}

class Rabbit extends Animal {
  hide() {
    alert(`${this.name} hides!`);
  }

  stop() {
    super.stop(); // call parent stop
    this.hide(); // and then hide
  }
}

let rabbit = new Rabbit("White Rabbit");

rabbit.run(5); // White Rabbit runs with speed 5.
rabbit.stop(); // White Rabbit stands still. White Rabbit hides!
```

`super` 키워드가 비동기 함수의 콜백함수로 사용될 경우에는 일반함수가 아닌 화살표 함수로 정의해야 합니다.

```js
class Rabbit extends Animal {
  stop() {
    setTimeout(() => super.stop(), 1000); // call parent stop after 1sec

    // Unexpected super
    setTimeout(function () {
      super.stop();
    }, 1000);
  }
}
```

## 생성자 오버라이딩

부모 클래스를 상속한 자식 클래스에 생성자가 없다면 아래와 같이 `super` 키워드와 함께 자동으로 부모 생성자를 사용합니다.

```js
class Rabbit extends Animal {
  // generated for extending classes without own constructors
  constructor(...args) {
    super(...args);
  }
}
```

따라서 자식 클래스의 생성자를 정의할 때는 부모 클래스의 생성자로 속성들을 먼저 초기화 해야합니다!

```js
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }

  // ...
}

class Rabbit extends Animal {
  constructor(name, earLength) {
    super(name);
    this.earLength = earLength;
  }

  // ...
}

// now fine
let rabbit = new Rabbit("White Rabbit", 10);
alert(rabbit.name); // White Rabbit
alert(rabbit.earLength); // 10
```

## 속성, 메서드 초기화 순서

클래스의 속성과 메서드가 초기화되는 순서는 다릅니다.

```js
class Animal {
  name = "animal";

  constructor() {
    alert(this.name); // (*)
  }
}

class Rabbit extends Animal {
  name = "rabbit";
}

new Animal(); // animal
new Rabbit(); // animal
```

자식 클래스 Rabbit의 생성자는 없으므로 `super` 키워드를 이용해 부모 클래스의 생성자를 실행합니다.

하지만 결과를 보면 `new` 키워드로 클래스 Rabbit의 인스턴스를 생성할 때 출력되는 name 필드의 값은 자식 클래스의 name 필드가 아닌 부모 클래스의 name 필드의 값이 출력됩니다.

```js
class Animal {
  showName() {
    // instead of this.name = 'animal'
    alert("animal");
  }

  constructor() {
    this.showName(); // instead of alert(this.name);
  }
}

class Rabbit extends Animal {
  showName() {
    alert("rabbit");
  }
}

new Animal(); // animal
new Rabbit(); // rabbit
```

그런데 부모 클래스에서 필드가 아닌 메서드를 실행하면 자식 클래스에서 정의한 메서드가 실행됩니다🤔

JS에서 클래스 필드의 초기화 순서는 다음과 같습니다.

![Init Order](/images/init_order.png)

즉, 자식 클래스에서 필드 초기화는 부모 클래스의 생성자로 초기화한 직후입니다.

따라서 첫 번째 예시에서 자식 클래스에는 아직 name 필드가 없기 때문에 부모 클래스의 name 필드를 사용했지만 메서드는 자연스럽게도 자식 클래스의 메서드를 사용한겁니다.

이러한 현상은 부모 클래스와 자식 클래스에서 동일한 필드를 사용하고 부모 생성자에서 해당 필드를 사용할 때 일어납니다. 이게 싫다면 자식 클래스에서 겹치는 필드를 메서드나 getter/setter로 바꾸면 됩니다.
