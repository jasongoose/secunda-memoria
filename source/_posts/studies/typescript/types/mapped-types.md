---
title: Mapped Types
date: 2024-11-03 19:28:29
categories:
  - Studies
  - TypeScript
  - Types
#tags
---
객체의 type을 정의할 때 속성의 key type은 "index signature"로 지정합니다.

```ts
type OnlyBoolsAndHorses = {
  [key: string]: boolean | Horse;
};

const conforms: OnlyBoolsAndHorses = {
  del: true,
  rodney: false,
};
```

```ts
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};

type FeatureFlags = {
  darkMode: () => void;
  newUserProfile: () => void;
};

type FeatureOptions = OptionsFlags<FeatureFlags>;
/*
{
    darkMode: boolean; 
	newUserProfile: boolean;
}
*/
```

## Mapping Modifiers

매핑하는 속성의 key type에 있는 `readonly` 또는 `?` 같은 modifier를 추가하거나 제거할 때 `+`, `-`를 prefix로 붙이면 됩니다.

```ts
// Removes 'readonly' attributes from a type's properties
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

type LockedAccount = {
  readonly id: string;
  readonly name: string;
};

type UnlockedAccount = CreateMutable<LockedAccount>;
/*
{
	id: string;
	name: string;
}
*/
```

```ts
// Removes 'optional' attributes from a type's properties
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};

type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};

type User = Concrete<MaybeUser>;
/*
{
	id: string;
	name: string;
	age: number;
}
*/
```

## Key Remapping via `as`

mapping된 속성의 key type은 `as` 키워드를 사용해서 다른 type으로 remapping할 수 있습니다.

```ts
type MappedTypeWithNewProperties<Type> = {
  [Property in keyof Type as NewKeyType]: Type[Properties];
};
```

### 예시1

template literal type을 사용하면 새로운 속성의 key type을 정의할 수 있습니다.

```ts
type Getters<Type> = {
  [Property in keyof Type as `get${Capitalize<
    string & Property
  >}`]: () => Type[Property];
};

interface Person {
  name: string;
  age: number;
  location: string;
}

type LazyPerson = Getters<Person>;
/*
type LazyPerson = {
    getName: () => string;
    getAge: () => number;
    getLocation: () => string;
}
*/
```

### 예시2

`as` 키워드 뒤의 conditional type으로 원하는 속성의 key type만 남길 수 있습니다.

```ts
// Remove the 'kind' property
type RemoveKindField<Type> = {
  [Property in keyof Type as Exclude<Property, "kind">]: Type[Property];
};

interface Circle {
  kind: "circle";
  radius: number;
}

type KindlessCircle = RemoveKindField<Circle>;
/*
type KindlessCircle = {
    radius: number;
}
*/
```

### 예시3

union type을 mapped type의 입력으로 전달할 수 있습니다.

```ts
type EventConfig<Events extends { kind: string }> = {
  [E in Events as E["kind"]]: (event: E) => void;
};

type SquareEvent = { kind: "square"; x: number; y: number };
type CircleEvent = { kind: "circle"; radius: number };

type Config = EventConfig<SquareEvent | CircleEvent>;
/*
type Config = {
    square: (event: SquareEvent) => void;
    circle: (event: CircleEvent) => void;
}
*/
```

## Indexed Access Types

임의의 `type` 또는 `interface`에서 특정 속성의 value type을 참조할 때 사용하는데, `keyof` 연산자와 함께 사용하면 굉장히 유용합니다.

```ts
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"]; // number

type I1 = Person["age" | "name"]; // number | string

type I2 = Person[keyof Person]; // number | string | boolean

type AliveOrName = "alive" | "name";
type I3 = Person[AliveOrName]; // boolean | string
```

`number` type을 indexing type(`[...]` 안에 들어가는 type)으로 사용하면 array 요소의 type을 한번에 알 수 있습니다.

```ts
const MyArray = [
  { name: "Alice", age: 15 },
  { name: "Bob", age: 23 },
  { name: "Eve", age: 38 },
];

type Person = typeof MyArray[number]; // { name: string; age: number }
type Age2 = Person["age"]; // number
```

type alias를 사용하면 약간의 리팩토링도 가능합니다.

```ts
type key = "age";
type Age = Person[key];
```
