---
title: Conditional Types
date: 2024-11-03 19:27:54
categories:
  - Studies
  - TypeScript
  - Types
#tags
---
input type에 따라서 output type을 다르게 지정할 때 사용합니다.

JS의 삼항연산자와 비슷한 구조를 가지는데 이를 통해 input, ouput type간의 관계를 정의할 수 있습니다.

```ts
SomeType extends OtherType ? TrueType : FalseType;
// A extends B => B 타입을 가진 변수에 A 타입 데이터를 할당할 수 있다.
// A는 B의 부분집합
```

간단하게 다음과 같이 사용할 수 있습니다.

```ts
interface Animal {
  live(): void;
}

interface Dog extends Animal {
  woof(): void;
}

type Example1 = Dog extends Animal ? number : string;
//type Example1 = number
```

## With Generic

conditional type은 generic과 함께 사용할 때 굉장히 유용한데, 예를 들어 함수의 parameter와 return type의 관계를 정의할 때 사용할 수 있습니다.

아래와 같이 parameter로 가능한 타입마다 대응되는 return type을 대응하여 중복 정의하는 것보다...

```ts
interface IdLabel {
  id: number; //some fields
}

interface NameLabel {
  name: string; //other fields
}

function createLabel(id: number): IdLabel;
function createLabel(name: string): NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
  throw "unimplemented";
}
```

generic으로 createLabel 함수의 타입을 정의하면 더 간결해집니다.

```ts
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;

function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
  throw "unimplemented";
}

// createLabel<number>(...): IdLabel
// createLabel<string>(...): NameLabel

let c = createLabel(Math.random() ? "hello" : 42); // IdLabel | NameLabel
```

## Conditional Type Constraints

conditional type을 사용하여 특정 type이 아니면 `never` 타입을 부여할 수 있습니다.

```ts
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;

interface Email {
  message: string;
}

interface Dog {
  bark(): void;
}

type EmailMessageContents = MessageOf<Email>; // string;

type DogMessageContents = MessageOf<Dog>; // never
```

## Flatten Type

`Array` 타입의 데이터로부터 요소의 type을 추출할 때 사용할 수 있는 type입니다.

`Array` 타입이 아니라면 그대로 반환합니다.

```ts
type Flatten<T> = T extends any[] ? T[number] : T;

// Extracts out the element type.
type Str = Flatten<string[]>; // string

// Leaves the type alone.
type Num = Flatten<number>; // number
```

## Inferring within Conditional Types

`infer` 키워드를 사용하면 함수의 return type을 구하는 타입을 다르게 구할 수 있습니다.

```ts
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return
  ? Return
  : never;

type Num = GetReturnType<() => number>; // number

type Str = GetReturnType<(x: string) => string>; // string

type Bools = GetReturnType<(a: boolean, b: boolean) => boolean[]>; // boolean[]
```

위의 Flatten type은 사실 다음과 같이 더 간단하게(?) 작성할 수 있습니다.

```ts
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;
```

## Distributive Conditional Types

conditional type에서 generic 인자로 union type을 전달하면 union member마다 적용됩니다.

```ts
type ToArray<Type> = Type extends any ? Type[] : never;

type StrArrOrNumArr = ToArray<string | number>; // string[] | number[]
```

만일 array에 여러 type의 데이터가 포함된 경우(`[1, ‘hello’, null, {a: 1}]`), 다음과 같이 작성하면 됩니다.

```ts
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never;

type StrArrOrNumArr = ToArrayNonDist<string | number>; // (string | number)[]
```
