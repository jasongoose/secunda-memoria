---
title: Extract parameter types from string literal types
date: 2024-11-01 22:11:38
categories:
  - Articles
#tags:
---
Literal Type은 JS value로부터 해당 값만을 가질 수 타입을 가리키는데 2가지 방법으로 만들 수 있습니다.

- `as const` 사용
- `const` 키워드 사용

또한 generic type parameter의 타입은 굳이 전달하지 않아도 일관성이 있다면 해당 타입의 변수에 할당된 값을 통해서 타입을 추론할 수 있습니다😳

```ts
function firstElement<Item>(arr: Item[]): Item | undefined {
  return arr[0];
}

const obj = firstElement([{ a: 1 }, { a: 3 }, { a: 5 }]);
// { a: number; } | undefined
```

위 두 사실들을 기반으로 String Literal Type으로부터 필요한 parameter들만 모으는 custom type은 아래와 같이 만들 수 있습니다.

> part는 "/", parameter는 "[]"로 구분하는 상황을 전제

> ex) '/purchase/[shopid]/[itemid]/args/[...args]

```ts
type IsParameter<Part> = Part extends `[${infer Anything}]` ? Part : never;

type FilteredParts<Path> = Path extends `${infer PartA}/${infer PartB}`
  ? IsParameter<PartA> | FilteredParts<PartB>
  : IsParameter<Path>;

type ParamValue<Key> = Key extends `...${infer Anything}` ? string[] : number;

type RemovePrefixDots<Key> = Key extends `...${infer Name}` ? Name : Key;

type Params<Path> = {
  [Key in FilteredParts<Path> as RemovePrefixDots<Key>]: ParamValue<Key>;
};

type CallbackFn<Path> = (req: { params: Params<Path> }) => void;
```

```ts
function get<Path extends string>(path: Path, callback: CallbackFn<Path>) {
  // TODO: implement
}
```

String Literal Type + Conditional Type의 조합은 무궁무진합니다.

## 출처

[Extract parameter types from string literal types with TypeScript](https://lihautan.com/extract-parameters-type-from-string-literal-types-with-typescript/)
