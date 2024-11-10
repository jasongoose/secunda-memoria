---
title: 4장. 웹 구성 요소
date: 2024-11-10 20:57:28
categories:
  - Books
  - 프레임워크 없는 프론트엔드
#tags:
---
## API

- 거의 모든 브라우저에서 웹 구성요소(web component)라는 네이티브 API 세트를 사용하여 웹앱을 만들 수 있습니다.
- 웹 구성요소의 3가지 기술을 사용하여 재사용할 수 있는 UI를 만들 수 있습니다.
  - HTML 템플릿
    - `<template></template>`
    - 화면에 렌더링되지 않는 대신 자바스크립트 코드에서 동적으로 컨텐츠를 생성하기 위한 "스템프"로 활용할 수 있습니다.
  - 사용자 정의 요소
    - 개발자가 자신만의 DOM 요소를 만들 수 있습니다.
  - 섀도우 DOM
    - 외부 DOM 요소의 영향을 받지 않는 구성 요소를 활용한 라이브러리나 위젯을 만들 때 활용합니다.
    - [MDN 가이드](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_shadow_DOM)

### 사용할 수 있을까?

- IE를 제외한 모든 브라우저에서 3가지 기능을 모두 지원합니다.
  - [HTML 템플릿](https://caniuse.com/?search=HTML%20templates)
  - [사용자 정의 요소](https://caniuse.com/?search=Custom%20Elements)
  - [Shadow DOM](https://caniuse.com/?search=shadow%20DOM)
- IE를 지원하려면 따로 polyfill을 적용해야 합니다.

### 사용자 정의 요소

- HTML 요소를 확장하는 자바스크립트 클래스
- 사용자 정의 태그는 대시("-")로 구분된 두 단어 이상의 이름만 가질 수 있습니다.
  - 단일 단어 태그는 W3C에서 지정한 표준 태그만 사용할 수 있습니다.

#### 라이프 사이클 메서드

```js
// Chapter04/00/components/HelloWorld.js
export default class HelloWorld extends HTMLElement {
  // 구성요소가 DOM에 연결되었을 때(mount되었을 때?)
  connectedCallback() {
    // 컨텐츠 렌더링
    // 타이머 시작
    // 네트워크로 데이터를 가져오기
    // ...
    window.requestAnimationFrame(() => {
	    this.innerHTML = '<div>Hello World!</div>';
    });
  }

  // 구성요소가 DOM에서 분리되었을 떄(unmount되었을 때?)
  disconnectedCallback() {
    // cleanup 함수
    // ...
  }
}
```

#### 레지스트리에 등록

```js
import HelloWorld from './components/HelloWorld.js';

// 지정한 태그 이름을 사용자 정의 요소 클래스에 연결합니다.
window.customElements.define('hello-world', HelloWorld);
```

#### 속성 관리

- 웹 구성요소가 어떤 프레임워크와도 호환되려면 속성(attribute, ***props***) getter/setter를 정의해야 합니다.
  - 속성 setter
    - HTML 마크업에서 직접 추가
      - `<input type="text" value="Frameworkless" />`
    - setter를 사용하여 속성을 조작
      - `inputEl.value = 'Frameworkless';`
    - `setAttribute` 메서드 사용
      - `inputEl.setAttribute('value', 'Frameworkless');`
  - 속성 getter
      - `getAttribute` 메서드 사용
        - `inputEl.getAttribute('value');`
- 문자열 값을 가지지 않는 속성에 대해서는 필요한 경우에 한해서 getter를 구현하고 setter만 구현해도 충분합니다.

```js
const DEFAULT_COLOR = 'pink';

export default class HelloWorld extends HTMLElement {
  get color() {
    return this.getAttribute('color') || DEFAULT_COLOR;
  }

  set color(value) {
    this.setAttribute('color', value);
  }

  connectedCallback() {
    window.requestAnimationFrame(() => {
      this.div = document.createElement('div');
      this.div.textContent = 'Hello World';
      this.div.style.color = this.color;

      this.appendChild(this.div);
    })
  }
}
```

```html
<hello-world></hello-world>
<hello-world color="red"></hello-world>
<hello-world color="green"></hello-world>
```

#### attributeChangedCallback

```js
export default class HelloWorld extends HTMLElement {
  static get observedAttributes() {
    return ['color'];
  }

  // ...

  // 특정 속성의 값이 변했을 때 호출
  // 단, observedAttributes 배열에 나열된 속성들에 한해서 트리거가 일어난다.
  attributeChangedCallback(name, oldValue, newValue) {
    if (!this.div) return;

    if (name === 'color') {
      this.div.style.color = newValue;
    }
  }
}
```

#### 가상 DOM 통합

```js
import applyDiff from './applyDiff.js';

const createDomElement = (color) => {
  const div = document.createElement('div');
  div.textContent = 'Hello World!';
  div.style.color = color;

  return div;
}

export default class HelloWorld extends HTMLElement {
  // ...

  connectedCallback() {
    window.requestAnimationFrame(() => {
      this.appendChild(createDomElement(this.color));
    });
  }

  attributeChangedCallback(name, oldValue, newValue) {
    // hasChildNodes: Node 메서드
    if (!this.hasChildNodes()) return;

    // firstElementChild: Element 속성
    applyDiff(this, this.firstElementChild, createDomElement(this.color));
  }
}
```

#### 사용자 정의 이벤트

```js
export const EVENTS = {
  AVATAR_LOAD_COMPLETE: 'AVATAR_LOAD_COMPLETE',
  AVATAR_LOAD_ERROR: 'AVATAR_LOAD_ERROR',
}

// 특정 사용자의 아바타 이미지 URL을 반환
const getGitHubAvatarUrl = async (user) => {
  // ...
}

export default class GitHubAvatar extends HTMLElement {
  constructor() {
    super();
    this.url = "/path/to/loading/image";
  }

  get user() {
    return this.getAttribute('user');
  }

  set user(value) {
    this.setAttribute('user', value);
  }

  onLoadAvatarComplete() {
    const payload = {
      detail: { avatar: this.url },
    };
    const event = new CustomEvent(EVENTS.AVATAR_LOAD_COMPLETE, payload);

	this.dispatchEvent(event);
  }

  onLoadAvatarError(reason) {
    const payload = {
      detail: { error },
    };
    const event = new CustomEvent(EVENTS.AVATAR_LOAD_ERROR, payload);

    this.dispatchEvent(event);
  }

  async loadNewAvatar() {
    if (!this.user) return;
    
    try {
      this.url = await getGitHubAvatarUrl(this.user);
      this.onLoadAvatarComplete();
    } catch (reason) {
      this.onLoadAvatarError(reason);
    } finally {
      this.render();
    }
  }

  render() {
    window.requestAnimationFrame(() => {
      this.innerHTML = '';
      const img = document.createElement('img');
      img.src = this.url;
      this.appendChild(img);
    });
  }

  connectedCallback() {
    this.render(); // 최초 렌더링 시, 로딩 이미지 노출
    this.loadNewAvatar();
  }
}
```

```js
import { EVENTS } from './components/GitHubAvatar.js';

const githubAvatarList = document.querySelectorAll('github-avatar');

githubAvatarList.forEach((avatarNode) => {
  // 이벤트 target 단계에서 콜백을 실행
  avatarNode.addEventListener(EVENTS.AVATAR_LOAD_COMPLETE, (payload) => {
    // ...
  });

  avatarNode.addEventListener(EVENTS.AVATAR_LOAD_ERROR, (payload) => {
    // ...
  });
})
```

## TodoMVC에 웹 구성 요소 사용

## 웹 구성 요소와 렌더링 함수

### 코드 스타일

- 웹 구성 요소
  - `HTMLElement` 클래스를 확장해서 사용하므로 객체지향 프로그래밍을 요구합니다.
- 렌더링 함수
  - 함수형 프로그래밍을 요구합니다.
- 간단한 렌더링 함수로 시작한 뒤에 라이브러리 릴리스가 필요합니다면 웹 구성요소로 래핑하면 됩니다.

### 테스트 가능성

- 렌더링 함수를 쉽게 테스트하려면 [Jest](https://jestjs.io/)와 같은 [JSDOM](https://github.com/jsdom/jsdom)과 통합한 테스트 러너를 활용하면 됩니다.
  - [관련 stackoverflow 답변](https://stackoverflow.com/a/64606792)

### 휴대성

- 기존 DOM 요소와 호환성이 보장되어야 다른 종류의 앱에서 동일하게 동작할 수 있습니다.

### 커뮤니티

- 웹 구성요소 클래스는 대부분의 프레임워크에서 DOM으로 UI 요소를 작성하는 표준방법입니다.
- 규모가 크거나 빠르게 성장하는 팀에서 유용한 기능을 활용할 수 있습니다.

## 사라지는 프레임워크

- 웹 구성요소의 등장으로 나타난 "보이지 않는 프레임워크"
- 프레임워크에 맞춰 코드를 작성하고 컴파일을 거치면 표준 웹 구성요소가 반환됩니다.
  - [Sevelte](https://svelte.dev/docs/custom-elements-api)
  - [Stancil.js](https://stenciljs.com/docs/custom-elements)
- 웹 컴포넌트 프레임워크
  - [Lit](https://lit.dev/)