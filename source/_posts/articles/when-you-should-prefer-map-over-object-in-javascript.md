---
title: When You Should Prefer Map Over Object In JavaScript
date: 2024-11-01 22:22:55
categories:
  - Articles
#tags:
---
## Because of prototype chain...

### Unwanted inheritance

Object Literal로 생성된 객체들은 `Object.prototype`에 위치한 속성들과 메소드들에 접근할 수 있습니다.

이를 반대로 생각해보면 런타임 중에 `Object.prototype`에 custom 속성이나 메소드를 추가하면 모든 객체들에게 상속되어 불필요한 항목들을 iterate해야하는 "Ripple Effect"가 발생할 수 있습니다.

규모가 큰 웹앱에서는 이러한 성질을 이용한 "Prototype Pollution Attack"에 노출될 수 있습니다.

간단한 해결책으로는 `Object.create(null)`로 prototype이 없는 객체를 생성하여 사용하는 방식이 있습니다.

Map은 object와는 다르게 default property(inherited property)들이 처음부터 없기 때문에 불필요한 inheritance가 없습니다.

### Name collisions

사용자가 생성한 객체에 `Object.prototype`에 존재하는 속성이나 메서드와 동일한 이름의 속성이나 메서드가 있다면 이는 prototype chain에 의해서 오버라이딩이 발생합니다.

Map은 object와는 다르게 default property(inherited property)들이 처음부터 없기 때문에 오버라이딩이 발생하지 않습니다.

### Iterate

객체는 iterable이 아니기 때문에 for-of 문을 사용할 수 없고 for-in 문을 사용하면 inherited property까지 접근하여 연산낭비가 발생합니다.

그래서 object 자체가 아닌 `Object.keys`, `Object.values`, `Object.entries`에서 반환된 array로 own-string-enumerable-property들만 접근할 수는 있지만 overhead가 발생합니다.

또한 대부분의 브라우저에서는 객체의 numeric key들은 오름차순으로 정렬되고 string key들보다 우선순위가 높기 때문에 원본 객체에 side-effect가 발생합니다.

Map은 iterable이기 때문에 for-of 문으로 iterate가 가능하고 iterate order는 데이터 타입별 우선순위 없이 insertion order를 따릅니다.

## clear

객체의 모든 own-property를 제거하려면 property별로 일일이 delete 연산을 수행해야 하는데...굉장히 귀찮습니다.

Map은 `Map.prototype.clear` 메서드 하나만으로 모든 요소들을 제거할 수 있습니다.

## size

기존 객체(obj)의 property들의 개수를 파악하는 방법은 key의 데이터 타입별로 달랐습니다.

```js
// string + enumerable + own
Object.keys(obj).length;

// string + unenumerable + own/non-own
Object.getOwnPropertyNames(obj).length;

// symbol + own
Object.getOwnPropertySymbols(obj).length;
```

위 3가지 방법들은 모든 property들을 iterate하기 때문에 O(N)의 시간 복잡도를 가집니다.

Map의 크기는 `Map.prototype.size`로 알 수 있어서 O(1)의 시간 복잡도를 가집니다.

## check property existence

보통 객체에 존재하지 않는 property의 value는 자동으로 `undefined`로 읽힙니다.

그런데 개발하다가 객체의 특정 property의 초기값을 `undefined`로 한다면 해당 property는 없는 것으로 착각할 수 있습니다😇

```js
const obj = { a: undefined };
console.log(obj.a); // undefined?
```

Map에서는 지정된 key에 해당하는 요소가 존재하는지 여부를 `Map.prototype.has` 메서드를 이용하면 되고 반환값은 깔끔하게 `boolean` 타입을 가집니다.

## 성능

Map은 Object와는 다르게 property descriptor(?) 관련 정보를 저장하지 않기 때문에 메모리를 덜 차지합니다.

요소들의 추가/삭제에 의한 업데이트가 자주 일어난다면 Map이 훨씬 더 효율성이 높습니다.

## 참고자료

[When You Should Prefer Map Over Object In JavaScript](https://www.zhenghao.io/posts/object-vs-map)
