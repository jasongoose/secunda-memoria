---
title: declare Keyword
date: 2024-11-03 19:29:07
categories:
  - Studies
  - TypeScript
  - Module
#tags:
---
`.js` 로만 구성된 라이브러리가 제공하는 module, 상수, 함수, 클래스 등을 `.ts` 프로젝트에 사용할 때, tsc가 인식할 수 있는 타입을 선언하는 키워드입니다.

```ts
// typescript/lib/lib.es5.d.ts
declare var JSON: JSON;
declare var Math: Math;
declare var Object: ObjectConstructor;

declare function eval(x: string): any;
declare function isNaN(number: number): boolean;
declare function encodeURI(uri: string): string;
declare function parseInt(string: string, radix?: number): number;
```

`declare` keyword는 이미 정의된 `interface`를 확장할 때도 유용합니다.

```ts
// .d.ts
import { AxiosInstance } from "axios";

declare module "@vue/runtime-core" {
  interface ComponentCustomProperties {
    $axios: AxiosInstance;
  }
}
```

```ts
// src/index.ts
import { createApp } from "vue";
import axios from "axios";
import App from "./App.vue";

const app = createApp(App);
app.config.globalProperties.$axios = axios;
app.mount("#app");
```

이제 임의의 컴포넌트 인스턴스는 `AxiosInstance` 타입을 가지는 $axios를 `import` 없이 사용할 수 있습니다.

```ts
import { getCurrentInstance, ComponentInternalInstance } from "vue";

const { proxy } = getCurrentInstance() as ComponentInternalInstance;

proxy!.$axios
  .get("https://jsonplaceholder.typicode.com/todos/1")
  .then((response) => response.data)
  .then((json) => console.log(json));
```
