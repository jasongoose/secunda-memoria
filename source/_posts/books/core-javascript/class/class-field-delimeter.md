---
title: Class Field Delimeter
date: 2024-11-01 00:45:49
categories:
  - Books
  - 코어 자바스크립트
#tags:
---
외부에서 클래스의 내부 인터페이스로의 접근을 제한하는 방식을 객체지향언어(OOP)에서는 "Encapsulation"이라고 합니다.

## static

속성이나 메서드를 클래스 **함수의 prototype이 아닌 클래스 자체의 속성, 메서드로 저장합니다.**

`static` 키워드가 붙은 속성, 메서드를 사용할 때는 `<클래스명>.[속성, 메서드 이름]` 과 같이 작성하는데, 인스턴스에서 접근할 수 없어서 클래스 단위의 데이터나 함수를 만들 때 활용됩니다.

static 속성, 메서드는 자식 클래스로 상속할 수 있습니다.

```js
class Animal {
  static planet = "Earth";

  constructor(name, speed) {
    this.speed = speed;
    this.name = name;
  }

  run(speed = 0) {
    this.speed += speed;
    alert(`${this.name} runs with speed ${this.speed}.`);
  }

  static compare(animalA, animalB) {
    return animalA.speed - animalB.speed;
  }
}

// Inherit from Animal
class Rabbit extends Animal {
  hide() {
    alert(`${this.name} hides!`);
  }
}

let rabbits = [new Rabbit("White Rabbit", 10), new Rabbit("Black Rabbit", 5)];

rabbits.sort(Rabbit.compare);

rabbits[0].run(); // Black Rabbit runs with speed 5.

alert(Rabbit.planet); // Earth
```

Rabbit 클래스의 인스턴스인 rabbit이 부모 클래스인 Animal 클래스의 static 메서드인 compare를 사용할 때는 [[Prototype]]을 2번 올라가서 `Animal.prototype.constructor.compare()` 식으로 상속된 메서드를 사용합니다.

![Static Field](/images/static_field.png)

## private

클래스 외부, 인스턴스에서 접근(read, write)할 수 없고 클래스 내부에서만 사용할 수 있는 필드들을 `private` 키워드로 지정합니다.

ES2019부터는 `private` 키워드 대신에 `#` prefix만 쓰도록 바뀌었고 TypeScript에서도 private 필드는 `#`을 붙입니다.

```js
class CoffeeMachine {
  #waterLimit = 200;

  #fixWaterAmount(value) {
    if (value < 0) return 0;
    if (value > this.#waterLimit) return this.#waterLimit;
  }

  setWaterAmount(value) {
    this.#waterLimit = this.#fixWaterAmount(value);
  }
}

let coffeeMachine = new CoffeeMachine();

// can't access privates from outside of the class
coffeeMachine.#fixWaterAmount(123); // Error
coffeeMachine.#waterLimit = 1000; // Error
```

private 필드는 자식 클래스에서도 접근할 수 없습니다!

```js
class MegaCoffeeMachine extends CoffeeMachine {
  method() {
    alert(this.#waterAmount); // Error: can only access from CoffeeMachine
  }
}
```

## (protected)

`private`처럼 클래스 내부에서 접근이 가능할 뿐만 아니라 자식 클래스에서도 접근할 수 있습니다.

단, 외부에서 직접 변경할 수 없도록 getter/setter로만 read와 write가 가능합니다.

자식 클래스로는 저절로 상속됩니다.

실제 JS에서는 구현되지 않아서 `protected` 필드의 이름은 보통 앞에 `_`(underscore) prefix를 붙입니다.

```js
class CoffeeMachine {
  _waterAmount = 0;

  set waterAmount(value) {
    if (value < 0) {
      value = 0;
    }
    this._waterAmount = value;
  }

  get waterAmount() {
    return this._waterAmount;
  }

  constructor(power) {
    this._power = power;
  }
}

// create the coffee machine
let coffeeMachine = new CoffeeMachine(100);

// add water
coffeeMachine.waterAmount = -10; // _waterAmount will become 0, not -10
```

## (public)

class의 외부에서 접근(read, write)할 수 있는 필드나 메서드입니다.

JS에서는 따로 작성할 키워드가 없습니다.
