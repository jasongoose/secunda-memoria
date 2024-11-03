---
title: Generic
date: 2024-11-03 19:28:01
categories:
  - Studies
  - TypeScript
  - Types
#tags
---
## Generic Type

generic type을 사용하면 동적으로 type을 정의할 수 있는데, 특히 함수의 parameter type과 return type 간의 관계를 추측할 때 굉장히 유용합니다.

generic type은 `interface` 형태로 정의할 수 있습니다.

```ts
interface GenericIdentityFn<Type> {
  (arg: Type): Type;
}

function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

또한 default type도 지원합니다.

```ts
type Readonly<T, K extends keyof T = keyof T> = Omit<T, K> & {
  readonly [P in K]: T[P];
};
```

## Generic Constraint

generic 함수의 type parameter로 가능한 type들을 제한하는 것을 의미하는데, 보통 `extends` 키워드를 사용하여 구현합니다.

```ts
interface Lengthwise {
  length: number;
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length); // Now we know it has a .length property, so no more error
  return arg;
}
```

```ts
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a");
getProperty(x, "m");
```

## 참고자료

[TypeScript: 제네릭(Generic)](https://hyunseob.github.io/2017/01/14/typescript-generic/)

[Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)