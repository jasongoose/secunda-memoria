---
title:  Share Type Declarations Between Modules
date: 2024-11-03 19:30:58
categories:
  - Studies
  - TypeScript
  - Module
#tags:
---
TS 모듈에서 export한 type declaration(`type`, `interface`)은 다른 모듈에서 import하여 사용할 수 있습니다.

tsc에 의해서 원본 `.ts` 파일을 컴파일하여 만들어진 `.js` 파일에는 기존의 type declaration의 import/export한 부분은 모두 없어집니다.

```ts
// b.ts
export interface PersonInterface {
  fname: string;
  lname: string;
  fullName(): string;
}

export class Person implements PersonInterface {
  constructor(public fname: string, public lname: string) {}
  fullName(): string {
    return `${this.fname} ${this.lname}`;
  }
}

// a.ts
import { Person, PersonInterface } from "./b";
const p: PersonInterface = new Person("Ross", "Geller");
```

```ts
// b.js
export class Person {
  constructor(fname, lname) {
    this.fname = fname;
    this.lname = lname;
  }
  fullName() {
    return `${this.fname} ${this.lname}`;
  }
}

// a.js
import { Person } from "./b";
const p = new Person("Ross", "Geller");
```

tsconfig.json의 필드 지정으로 `import`문 없이 외부에서 export된 type을 사용할 수도 있습니다.

## 참고자료

[A quick introduction to “Type Declaration” files and adding type support to your JavaScript packages](https://medium.com/jspoint/typescript-type-declaration-files-4b29077c43)

