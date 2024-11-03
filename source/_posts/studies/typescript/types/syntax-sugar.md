---
title: Syntax Sugar
date: 2024-11-03 19:31:25
categories:
  - Studies
  - TypeScript
  - Types
#tags
---
## Array Type

Array 내 element-wise 접근은 [Indexed Access Type](../mapped-types#Indexed-Access-Types)을 사용하면 됩니다.

```ts
const a = [1, "a", Promise.resolve(0)] as const;
type Ta = typeof a;
type TaEl = Ta[number]; // 1 | 'a' | Promise<number>

type Tb = [1, "a"];
type TbEl = Tb[number]; // 1 | 'a'

type TbObj = { [K in keyof Tb]: Tb[K] };

type IsArrExtendsObj = Tb extends TbObj ? true : false; // true
type IsObjExtendsArr = TbObj extends Tb ? true : false; // true
```

## Object Type

비어있는 객체(`{}`)의 type은 다음과 같이 확인할 수 있습니다.

```ts
type IsEmpty<T> = T extends Record<string, never> ? true : false;
```

객체의 특정 property type을 제거할 때는 `never`로 지정하면 됩니다.

```ts
type RemoveIndexSignature<T> = {
  [K in keyof T as K extends `${infer P}` ? P : never]: T[K];
};
```

## Tuple Type

TS에서 tuple 또는 array는 한가지 고유의 property 즉, `length`가 있습니다.

길이가 0인 empty tuple은 아래와 같이 확인할 수 있습니다.

```ts
type L<T> = T extends { length: 0 } ? true : false;
type R<T> = T extends Record<number, never> ? true : false;
```

tuple에 대한 conditional type은 다음과 같이 지정하면 됩니다.

```ts
type C<T> = T extends any[] ? true : false;
```

tuple 또는 array에는 요소의 개수(`length`)에 관한 정보가 있기 때문에 길이를 구하거나 개수를 셀 때 활용할 수 있습니다.

```ts
type LengthOfString<
  S extends string,
  T extends any[] = []
> = S extends `${infer H}${infer R}`
  ? LengthOfString<R, [H, ...T]>
  : T["length"];

type N = LengthOfString<"Hello world">; // 11
```

임의의 `string` type으로 `object` type을 만들 때 다음과 같이 작성하면 됩니다.

```ts
type TupleToNestedObject<T extends unknown[], U> = T extends [
  infer H,
  ...infer R
]
  ? { [K in H & string]: TupleToNestedObject<R, U> }
  : U;

type a = TupleToNestedObject<["a"], string>; // {a: string}
type b = TupleToNestedObject<["a", "b"], number>; // {a: {b: number}}
```