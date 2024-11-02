---
title: Event
date: 2024-11-02 00:08:51
categories:
  - Studies
  - Browser
  - Web API
#tags:
---
## Event 흐름

DOM 트리는 웹 페이지를 구성하는 node들 사이의 포함관계를 나타나는 트리 구조의 데이터로 브라우저에 의해서 생성됩니다.

![Event 흐름](/images/event_flow.png)

사용자 또는 외부 원인에 의해 node로부터 어떤 event가 발생하면 target 요소 뿐만 아니라 상위, 하위 node에도 동일한 event가 전달되는데 event가 흐르는 방향은 다음과 같이 3가지가 있습니다.

이와 같이 임의의 DOM node에서 발생한 이벤트를 상위 node에서도 처리할 수 있는 메커니즘을 "Event Delegation"라고 합니다.

### Capturing

DOM 트리의 최상위 요소(`window`)에서 target의 부모 node까지 event가 전달되는 단계입니다.

여기서 DOM 트리를 타고 내려가면서 거치는 node들의 event listener는 `capture` 옵션을 `true`로 가져야 합니다.

event listener는 보통 `EventTarget.addEventListener` 메서드로 구현하는데 이 메서드의 `capture` 옵션의 default 값은 `false`입니다.

그래서인지 capturing 단계에서 event를 처리하는 코드는 거의 없습니다.

### Targeting

이벤트가 발생한 실제 target node의 event listener를 실행하는 단계입니다.

### Bubbling

target의 부모 node에서 최상위 요소(`window`)로 event가 전달되는 단계입니다.

여기서 DOM 트리를 타고 올라가면서 거치는 node들의 event listener는 `capture` 옵션을 `false`로 가져야 합니다.

## Event 속성

각 단계마다 발생한 event와 관련된 정보는 다음과 같은 속성에서 얻을 수 있습니다.

### Event.target

event가 처음으로 발생한 DOM 객체입니다.

### Event.currentTarget

현재 event listener를 가진 DOM 객체로 listener 내부 `this`가 참조하는 값입니다.

### Event.eventPhase

현재 event listener를 거치는 event flow의 단계를 표현한 `number`형 데이터입니다

- capturing : 1
- target : 2
- bubbling: 3

## Event 메서드

자주 사용하는 Event 메서드들을 정리하면 다음과 같습니다.

### Event.stopPropagation

다음 event bubbling / capturing 단계를 더 이상 진행하지 않을 때 사용합니다.

### Event.preventDefault

브라우저가 특정 event에 의한 default action을 수행하지 않도록 제어하는 메서드입니다.

`<input type="submit" />` 태그를 클릭했을 때 default action은 form을 submit하는 것이고, `<a href="" />` 태그를 클릭했을 때 default action은 화면 새로고침입니다.

여기서 특정 event에 `preventDefault` 메서드를 적용할 수 있는지 여부는 해당 event의 `cancelable` 속성값에 따라 달라집니다.

### Event.stopImmediatePropagation

동일한 node에 동일한 event에 대한 event listener들을 여러 개 추가한다면 등록된 순서대로 호출됩니다.

중간에 특정 listener에서 `stopImmediatePropagation` 메서드를 호출하면 그 뒤에 등록된 listener들은 실행되지 않습니다.

## 참고자료

[버블링과 캡처링](https://ko.javascript.info/bubbling-and-capturing)

[이벤트 버블링, 이벤트 캡처 그리고 이벤트 위임까지](https://joshua1988.github.io/web-development/javascript/event-propagation-delegation/)

[stopPropagation vs stopImmediatePropagation 제대로 이해하기](https://medium.com/%EC%98%A4%EB%8A%98%EC%9D%98-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/stoppropagation-vs-stopimmediatepropagation-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-75edaaed7841)