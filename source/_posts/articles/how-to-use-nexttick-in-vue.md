---
title: How to Use $nextTick() in Vue
date: 2024-11-01 22:13:32
categories:
  - Articles
#tags:
---
`$nextTick` 메서드는 component data의 변화가 DOM 트리(현재 페이지 또는 컴포넌트 template)에 반영되었을 시점에 실행할 task들을 지정할 수 있습니다.

`$nextTick` 메서드를 사용하면 컴포넌트 data와 동기화된 최신(?) DOM 트리를 참조할 수 있습니다.

## Vue.$nextTick(() ⇒ {...})

현재 페이지 전체 기준으로 DOM 트리 update가 발생했을 때 수행할 task를 callback으로 전달할 수 있습니다.

현재 컴포넌트 뿐만 아니라 하위 컴포넌트들의 template update가 모두 수행한 뒤에 callback을 수행합니다.

## this.$nextTick(() ⇒ {...})

현재 컴포넌트 인스턴스의 template update가 일어났을 경우에만 수행한 task를 callback으로 전달할 수 있습니다.

## await this.$nextTick()

`$nextTick` 메서드에 callback을 전달하지 않는다면 DOM 트리 update가 발생할 때까지 await할 수 있는 효과가 있습니다.

`await` 다음에는 component data와 동기화된 template에 접근하여 수행할 task를 작성하면 됩니다.

```html
<template>
  <div>
    <button @click="handleClick">Insert/Remove</button>
    <div v-if="show" ref="content">I am an element</div>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        show: true,
      };
    },
    methods: {
      async handleClick() {
        this.show = !this.show;
        //this.$nextTick(() => {
        //  console.log(this.show, this.$refs.content);
        //});
        await this.$nextTick();
        console.log(this.show, this.$refs.content);
      },
    },
  };
</script>
```

## 출처

[How to Use nextTick() in Vue](https://dmitripavlutin.com/vue-next-tick/)
