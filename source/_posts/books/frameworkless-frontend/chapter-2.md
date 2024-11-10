---
title: 2장. 렌더링
date: 2024-11-10 20:57:24
categories:
  - Books
  - 프레임워크 없는 프론트엔드
#tags:
---
## 문서 객체 모델

- W3C는 프로그래밍 방식으로 요소를 렌더링하는 방식을 문서 객체 모델(DOM)으로 정의합니다.
- DOM
  - 웹앱을 구성하는 요소를 조작할 수 있는 API
- Node
  - HTML 트리에서 노드(HTML element)를 나타내는 기본 인터페이스

## 렌더링 성능 모니터링

### 크롬 개발자 도구

- 개발자 도구 > cmd + shift + p > FPS

### stats.js

- [stats.js](https://github.com/mrdoob/stats.js)으로 프레임과 할당된 메가바이트의 메모리를 렌더링하는데 필요한 밀리초를 계산할 수 있습니다.

### 사용자 정의 성능 위젯

- `requestAnimationFrame`으로 두 프레임 사이의 시간 간격을 확인해서 1초 내에 콜백이 호출되는 횟수를 계산합니다.

```js
let frameCount = 0;
let panel, start;

const createPanel = () => {
	// DOM 객체 생성
}

const tick = () => {
	frameCount++;

	const now = window.performance.now();

	// 첫 프레임으로부터 1초가 지난 경우, 초기화하고 측정을 재시작
	if (now >= start + 1000) {
		panel.innerText = frameCount;
		frameCount = 0;
		start = now;
	}

	window.requestAmimationFrame(tick);
}

const initPerformanceCheck = (parent = document.body) => {
	panel = createPanel();

	window.requestAnimationFrame(() => {
		start = window.performance.now();
		parent.appendChild(panel);
		tick();
	})
}

export default {
	initPerformanceCheck,
}
```

## 렌더링 함수

- 순수함수로 요소를 DOM에 렌더링한다는 것은 DOM 요소가 앱의 상태에만 의존한다는 것을 의미합니다.
- `view = function(targetElement, state)`

### TodoMVC

- [TodoMVC](https://todomvc.com/)

### 순수함수 렌더링

- 1차 리팩토링 내용
  - 하나의 거대한 함수
    - as-is
      - 여러 DOM 요소를 조작하는 함수가 하나 밖에는 없습니다.
    - to-be
      - 카운터, 필터링, 리스트 영역의 DOM을 조작하는 함수를 개별로 정의합니다.
  - 동일한 작업을 수행하는 여러 방법
    - as-is
      - 문자열로 생성
      - 기존 요소에 텍스트를 추가
      - `classList` 속성으로 관리합니다.
    - to-be
      - 기존 DOM node로부터 clone을 생성하여 새로운 상태값을 반영하고 기존 node를 대체합니다.
- 2차 리팩토링 내용
  - 구성 요소 함수
      - as-is
        - 특정 구성요소에 대해서 적용할 렌더링 함수를 수동으로 호출합니다.
        - 렌더링: 현재 상태를 반영한 DOM 요소를 생성 + 기존 요소를 대체
      - to-be
        - 레지스트리 정의
          - 구성요소들을 구분하기 위한 `data-component` 속성을 지정합니다.
          - `data-component` 속성의 값에 따라 호출할 렌더링 함수를 지정한 레지스트리(=인덱스, 딕셔너리, Map 객체)를 정의합니다.
            - 구성요소 기반(=컴포넌트 기반) 앱에서 여러 구성요소 안에서도 활용할 수 있는 재사용성을 높일 수 있습니다.
            - 컴포넌트: 레지스트리에 등록되어 재사용할 수 있는 렌더링 함수
        - 레지스트리에 컴포넌트 등록
          - 레지스트리를 참조하여 올바른 렌더링 함수를 자동으로 호출하는 고차함수(`renderWrapper`)를 정의합니다.
          - 레지스트리에 `renderWrapper`로 감싼 컴포넌트들을 등록하고 나서 처음으로 app root로부터 렌더링을 수행하면 하위 컴포넌트들에 대해서도 "재귀적으로" 렌더링을 수행합니다.

## 동적 데이터 렌더링

- 이후 예제들에서 상태값이 업데이트 될 때마다 가상 root node를 만들어서 기존 node를 대체하는 방식으로 구현합니다.
  - 사소한 상태 변화에도 전체 mount가 발생하므로 대규모 앱에서는 성능 저하를 불러올 수 있습니다.

### 가상 DOM

- 재조정(Reconciliation)
  - UI를 표현하는 가상 DOM 객체를 메모리에 유지한 상태에서 최소한의 DOM 조작을 수행하여 실제 DOM과 동기화합니다.
    - 근데 예제들이 사용하는 건 Real DOM 객체인데 Sevelte에서도 이렇게 구현하나?
- Diff 알고리즘
  - 가상 DOM 객체가 실제 DOM에 적용되기 위한 가장 빠른 방법을 찾습니다.
- 간단한 가상 DOM 구현
  - Diff 알고리즘
    - 새 node가 정의되지 않은 경우 실제 node를 제거합니다.
    - 실제 node가 정의되지 않았지만 가상 node가 존재하는 경우, 부모 노드에 추가합니다.
    - 두 node가 모두 정의된 경우, 두 node 간에 차이가 있는지 확인합니다.
  - 두 노드의 차이점
    - 속성의 개수가 변한 경우
    - 최소 1개 이상의 속성이 변한 경우
    - 자식 node가 없을 때, textContent가 다른 경우
- 개선된 검사 수행으로 성능을 높일 수 있지만 렌더링 엔진은 최대한 간단하게 유지하는 것이 좋습니다.
  - "시기상조의 최적화는 모든 악의 근원이다."
  - 시간을 들여서 점진적으로 개선해야 합니다.