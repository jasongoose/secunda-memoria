---
title: type vs. interface
date: 2024-11-03 20:15:24
categories:
  - Studies
  - TypeScript
  - Keywords
#tags
---
두 키워드 모두 타입을 선언할 때 사용하지만 아래와 같은 차이점이 있습니다.

## 확장하는 방식

### interface

`extends` 키워드 또는 `&`(intersection)를 사용합니다.

```ts
interface Animal {
  name: string;
}

interface Bear extends Animal {
  honey: boolean;
}

type Bear = Animal & {
  honey: boolean;
};

const bear = getBear();
bear.name;
bear.honey;
```

### type

`&`(intersection)를 사용합니다.

```ts
type Animal = {
  name: string;
};

type Bear = Animal & {
  honey: boolean;
};

const bear = getBear();
bear.name;
bear.honey;
```

## 새로운 type field를 추가하는 방식

동일한 이름의 interface를 중복으로 정의하면 됩니다.

```ts
interface Window {
  title: string;
}

interface Window {
  ts: TypeScriptAPI;
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
```

type은 한번 정해지면 수정할 수 없습니다.

```ts
type Window = {
  title: string;
};

type Window = {
  ts: TypeScriptAPI;
};

// Error: Duplicate identifier 'Window'.
```
