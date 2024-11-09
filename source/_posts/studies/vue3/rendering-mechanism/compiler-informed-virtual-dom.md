---
title: 2. Compiler-Informed Virtual DOM
date: 2024-11-03 20:55:52
categories:
  - Studies
  - Vue3
  - Rendering Mechanism
#tags:
---
VDOM은 vanillaJS, jQuery 등을 사용하여 직접적인 DOM 조작없이 선언적으로, 그리고 동적으로 원하는 UI를 생성할 수 있다는 개발편의성을 장점으로 가집니다.

하지만 runtime 중 사소한 변화에 의해서 리렌더링이 필요할 때마다 변하지 않는 Node들도 포함하여 VDom을 처음부터 재생성하는 “중복에 의한 비효율성”이라는 단점도 가집니다.

Vue3에서는 compile-time optimization으로 위 한계점을 해결합니다.

compiler가 컴포넌트의 template를 파싱하면서 static Vnode를 제외한 실제 patch 대상이 되는 Vnode들의 정보를 renderer에게 전달하여 runtime 중 업데이트에 shortcut을 제공합니다.

## Static Hoisting

예를 들어 아래와 같은 컴포넌트 template이 있다면, foo `<div>` 와 bar `<div>` 는 컴포넌트 data와 binding된 부분이 없어서 변할 경우가 없으므로 리렌더링이 불필요합니다.

```html
<div>
  <div>foo</div>
  <div>bar</div>
  <div>{{ dynamic }}</div>
</div>
```

compiler는 static Vnode들을 생성하는 함수들을 render 함수 밖으로 hoist하여 리렌더링이 발생할 때마다 재사용하고 renderer에게 patch 대상에서 제외하도록 flag(`HOISTED`)를 전달합니다.

```js
const _hoisted_1 = /*#__PURE__*/ _createElementVNode(
  "div",
  null,
  "foo",
  -1 /* HOISTED */
);
const _hoisted_2 = /*#__PURE__*/ _createElementVNode(
  "div",
  null,
  "bar",
  -1 /* HOISTED */
);

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (
    _openBlock(),
    _createElementBlock("div", null, [
      _hoisted_1,
      _hoisted_2,
      _createElementVNode(
        "div",
        null,
        _toDisplayString(_ctx.dynamic),
        1 /* TEXT */
      ),
    ])
  );
}
```

추가적으로 template 상의 static element들이 어느 정도 연속으로 위치한다면, 개별 static Vnode들을 생성하는 대신 단일 Vnode(merged Vnode)로 압축합니다.

해당 Vnode는 element들을 모두 포함하는 HTML string을 내부에 가지는데, `innerHTML` 메서드에 의해서 직접 RDOM에 삽입됩니다.

여기서 첫 mount 단계에서 생성된 merged Vnode는 따로 cache에 저장되고 [cloneNode](https://developer.mozilla.org/en-US/docs/Web/API/Node/cloneNode)로 만든 복사본들은 Vue 앱 전반에 걸쳐 재사용됩니다.

```html
<div>
  <div class="foo">foo</div>
  <div class="foo">foo</div>
  <div class="foo">foo</div>
  <div class="foo">foo</div>
  <div class="foo">foo</div>
  <div>{{ dynamic }}</div>
</div>
```

```js
const _hoisted_1 = /*#__PURE__*/ _createStaticVNode(
  '<div class="foo">foo</div><div class="foo">foo</div><div class="foo">foo</div><div class="foo">foo</div><div class="foo">foo</div>',
  5
);

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (
    _openBlock(),
    _createElementBlock("div", null, [
      _hoisted_1,
      _createElementVNode(
        "div",
        null,
        _toDisplayString(_ctx.dynamic),
        1 /* TEXT */
      ),
    ])
  );
}
```

## Patch Flags

compiler가 dynamic binding(들)을 가진 단일 element에 대응하는 Vnode를 생성할 때 전달하는 인자값으로, 추후 리렌더링에 의해서 진행할 patch의 종류를 나타냅니다.

```html
<!-- class binding only -->
<div :class="{ active }"></div>

<!-- id and value bindings only -->
<input :id="id" :value="value" />

<!-- text children only -->
<div>{{ dynamic }}</div>
```

```js
export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (
    _openBlock(),
    _createElementBlock(
      _Fragment,
      null,
      [
        _createElementVNode(
          "div",
          {
            class: _normalizeClass({ active: _ctx.active }),
          },
          null,
          2 /* CLASS */
        ),
        _createElementVNode(
          "input",
          {
            id: _ctx.id,
            value: _ctx.value,
          },
          null,
          8 /* PROPS */,
          ["id", "value"]
        ),
        _createElementVNode(
          "div",
          null,
          _toDisplayString(_ctx.dynamic),
          1 /* TEXT */
        ),
      ],
      64 /* STABLE_FRAGMENT */
    )
  );
}
```

[원본 소스](https://github.com/vuejs/core/blob/main/packages/shared/src/patchFlags.ts)에서 patch flag는 숫자 1에 `bitwise Left-shift` 연산을 적용한 상수로 구성됩니다.

하나의 element가 여러 개의 dynamic binding을 가진다면 다수의 patch flag들을 가지는데. 여기서 flag들은 `bitwise OR` 연산으로 하나의 숫자로 결합됩니다.

renderer는 결합된 숫자와 개별 patch flag에 `bitwise AND` 연산을 적용하여 필요한 patch의 타입을 구분합니다.

```js
if (vnode.patchFlag & PatchFlags.CLASS /* 2 */) {
  // update the element's class
}
```

patch flag는 개별 element 뿐만 아니라 하위 Vnode들의 타입을 나타내기도 합니다.

여러 개의 root Vnode를 가지는 컴포넌트를 fragment라고 하는데, renderer에게 fragment 내의 하위 Vnode들의 순서는 변할 일이 없다는 사실을 patch flag를 통해서 알려줄 수 있습니다.

```js
export function render() {
  return (
    _openBlock(),
    _createElementBlock(
      _Fragment,
      null,
      [
        /* children */
      ],
      64 /* STABLE_FRAGMENT */
    )
  );
}
```

## Tree Flattening

컴포넌트 template으로부터 생성된 root Vnode는 `createElementBlock` 함수로 생성됩니다.

```js
export function render() {
  return (
    _openBlock(),
    _createElementBlock(
      _Fragment,
      null,
      [
        /* children */
      ],
      64 /* STABLE_FRAGMENT */
    )
  );
}
```

여기서 block이란 내부에 `v-if`, `v-for` directive를 포함한 element가 없는 template 부분을 가리킵니다.

```html
<!-- root block -->
<div>
  <!-- not tracked -->
  <div>...</div>
  <!-- tracked -->
  <div :id="id"></div>
  <!-- not tracked -->
  <div>
    <!-- tracked -->
    <div>{{ bar }}</div>
  </div>
</div>
```

반대로 `v-if`, `v-for` directive가 있는 element는 새로운 block을 생성합니다.

```html
<!-- root block -->
<div>
  <div>
    <!-- if block -->
    <div v-if>
      ...
      <div>
        <!-- for block -->
        <div v-for>
          ...
          <div></div>
        </div>
      </div>
    </div>
  </div>
</div>
```

각 block은 patch flag를 가진(dynamic binding을 가진) 하위 Vnode들만 따로 골라서 flatten array를 생성하는데, 이것들이 모여서 VDom tree가 됩니다.

```js
div (block root)
- div with :id binding
- div with {{ bar }} binding
```

위와 같이 flatten tree는 static Vnode들을 patch 과정에 포함시키지 않기 때문에 불필요한 traverse 연산을 줄일 수 있습니다.
