---
title: Utility Types
date: 2024-11-03 19:33:39
categories:
  - Studies
  - TypeScript
  - Types
#tags
---
## Index Signature

Mapped type에서 사전에 정의되지 않은 property type을 표시할 때는 index signature를 사용합니다.

사전에 정의되지 않았기 때문에 해당 property는 `string` type을 extend하지 않는 성질이 있습니다.

이 성질을 이용하면 `object` type으로부터 index signature를 제거하는 generic type을 아래와 같이 만들 수 있습니다.

```ts
type RemoveIndexSignature<T> = {
  [K in keyof T as K extends `${infer P}` ? P : never]: T[K];
};

type Foo = {
  [key: string]: any;
  foo(): void;
};

type A = RemoveIndexSignature<Foo>; // expected { foo(): void }
```

## Record Type

property key와 value의 타입이 각각 K, T인 type O는 아래와 같이 생성할 수 있습니다.

```ts
type O = Record<K, T>;
/*
{
  [P in K]: T,
};
*/
```

conditional type에서 `object` type인지 여부를 확인할 때 유용합니다.

```ts
type X<T> = T extends Record<string, unknown> ? true : false;
```

## Recursion Type

뭔가 재귀적인 type 관계가 보였을 때, 재귀함수로 구현하면 됩니다.

```ts
type Awaited<T> = T extends Promise<infer P>
  ? P extends Promise<infer U>
    ? Awaited<U>
    : P
  : T;
```

## Spread Syntax

```ts
type Concat<T extends Array<any>, U extends Array<any>> = [...T, ...U];

type Last<T extends any[]> = T extends [...unknown[], infer L] ? L : never;

type Pop<T extends any[]> = T extends [...infer R, unknown] ? R : never;

type AppendArgument<Fn, A> = Fn extends (...args: infer P) => infer R
  ? (...args: [...P, A]) => R
  : never;
```

## Template Literal Type

JS의 template literal syntax를 이용하면 기존 type의 정보를 기반으로 새로운 `string`형 type을 정의할 수 있습니다.

```ts
type World = "world";

type Greeting = `hello ${World}`;
// type Greeting = "hello world"
```

interpolation(`${...}`) 위치에 union type을 사용하면 union member마다 적용됩니다.

```ts
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
/*
type AllLocaleIDs = 
| "welcome_email_id" 
| "email_heading_id"
| "footer_title_id" 
| "footer_sendoff_id"
*/
```

### String Unions In Types

특정 객체를 전달하면 객체에 `on(eventName: ‘${key}Changed’, cb: (newValue: any) ⇒ void)` 타입의 메서드를 추가하는 함수 `makeWatchedObject`를 만든다고 합시다.

```ts
const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26,
});

person.on("firstNameChanged", (newValue) => {
  console.log(`firstName was changed to ${newValue}!`);
});
```

template literal type을 사용하면 다음과 같이 함수의 타입을 정의할 수 있습니다.

```ts
type PropEventSource<Type> = {
  on(
    eventName: `${string & keyof Type}Changed`,
    cb: (newValue: any) => void
  ): void;
};

declare function makeWatchedObject<Type>(
  obj: Type
): Type & PropEventSource<Type>;
```

위의 예시에서 `cb`의 인자는 그저 `any`로 되어있는데, 아래와 generic을 추가하면 객체의 특정 key에 대한 value의 type을 지정할 수 있습니다.

```ts
type PropEventSource<Type> = {
  on<Key extends string & keyof Type>(
    eventName: `${Key}Changed`,
    callback: (newValue: Type[Key]) => void
  ): void;
};
```

### Intrinsic String Manipulation Types

tsc의 built-in type들 중, `string` type을 수정할 수 있는 generic type들을 정리하면 다음과 같습니다.

#### UpperCase

모든 문자를 대문자로 만듭니다.

```ts
type Greeting = "Hello, world";
type ShoutyGreeting = Uppercase<Greeting>;
// type ShoutyGreeting = "HELLO, WORLD"

type ASCIICacheKey<Str extends string> = `ID-${Uppercase<Str>}`;
type MainID = ASCIICacheKey<"my_app">;
// type MainID = "ID-MY_APP"
```

#### LowerCase

모든 문자를 소문자로 만듭니다.

```ts
type Greeting = "Hello, world";
type QuietGreeting = Lowercase<Greeting>;
// type QuietGreeting = "hello, world"

type ASCIICacheKey<Str extends string> = `id-${Lowercase<Str>}`;
type MainID = ASCIICacheKey<"MY_APP">;
// type MainID = "id-my_app"
```

#### Capitalize

첫번째 문자만 대문자로 만듭니다.

```ts
type LowercaseGreeting = "hello, world";
type Greeting = Capitalize<LowercaseGreeting>;
// type Greeting = "Hello, world"
```

#### Uncapitalize

첫번째 문자만 소문자로 만든다.

```ts
type UppercaseGreeting = "HELLO WORLD";
type UncomfortableGreeting = Uncapitalize<UppercaseGreeting>;
// type UncomfortableGreeting = "hELLO WORLD"
```

### String Type 분해

`infer` 키워드를 사용하여 기존 string literal을 문자가 아닌 `string` 단위로 분해할 수 있습니다.

```ts
type S = "abcdefg";

type X = a extends `${infer F}${infer R}` ? F : a; // type aa = 'a'
type Y = a extends `${infer F}${infer R}` ? R : a; // type bb = 'bcdefg'
```

참고로, empty string type(`’’`)은 template literal로 분해할 수 없습니다.

```ts
type E = "" extends `${infer H}${infer R}` ? true : false; // false
```

## 참고자료

[Template Literal Types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)

[Template Literal Types로 타입 안전하게 코딩하기](https://toss.tech/article/template-literal-types)