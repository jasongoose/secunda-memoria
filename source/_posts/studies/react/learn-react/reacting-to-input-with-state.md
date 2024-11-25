---
title: Reacting to Input with State
date: 2024-11-04 08:15:06
categories:
  - Studies
  - React
  - Learn React
#tags:
---
## 왜 React/Vue가 필요할까?

과거라면 vanillaJS와 DOM API를 사용하여 특정 조건을 만족하거나 event가 발생했을 때, 특정 DOM node에 대해서 임의의 DOM node를 추가/생성/삭제하는 과정을 아래와 길게 썼을겁니다.

```jsx
async function handleFormSubmit(e) {
  e.preventDefault();
  disable(textarea);
  disable(button);
  show(loadingMessage);
  hide(errorMessage);
  try {
    await submitForm(textarea.value);
    show(successMessage);
    hide(form);
  } catch (err) {
    show(errorMessage);
    errorMessage.textContent = err.message;
  } finally {
    hide(loadingMessage);
    enable(textarea);
    enable(button);
  }
}

function handleTextareaChange() {
  if (textarea.value.length === 0) {
    disable(button);
  } else {
    enable(button);
  }
}

function hide(el) {
  el.style.display = "none";
}

function show(el) {
  el.style.display = "";
}

function enable(el) {
  el.disabled = false;
}

function disable(el) {
  el.disabled = true;
}

function submitForm(answer) {
  // Pretend it's hitting the network.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (answer.toLowerCase() == "istanbul") {
        resolve();
      } else {
        reject(new Error("Good guess but a wrong answer. Try again!"));
      }
    }, 1500);
  });
}

let form = document.getElementById("form");
let textarea = document.getElementById("textarea");
let button = document.getElementById("button");
let loadingMessage = document.getElementById("loading");
let errorMessage = document.getElementById("error");
let successMessage = document.getElementById("success");
form.onsubmit = handleFormSubmit;
textarea.oninput = handleTextareaChange;
```

위와 같은 방식은 일단 동작은 제대로 되겠지만 프로젝트 규모가 커지면서 UI를 추가/삭제/수정할 때마다 전체 코드량이 증가하면서 디버깅할 때마다 소스 전체를 traverse해야 하는 번거로움이 있습니다.

React나 Vue 같은 컴포넌트 기반 JS 라이브러리/프레임워크는 개발자로 하여금 특정 상황에서 화면 단에 표시할 UI를 명령형이 아닌 선언형 방식으로 구현하도록 도와줍니다.

즉, 다음에 노출할 UI를 얻기까지 실행할 연산들이 아니라 그냥 “이걸 보여달라”고 통보하는 방식으로 코드를 작성하면 최대한 적은 양의 DOM manipulation으로 현재 UI에서 다음 UI로 변환하는 작업을 대신 해줍니다.

여기서 현재와 다음 UI를 구분짓는 것을 렌더링 대상이 되는 컴포넌트의 상태 즉, state라고 합니다.

그래서 적절한 타이밍에 state를 업데이트하기만 하면 리렌더링이 이루어집니다.

## state 적용하기

1. 컴포넌트가 가질 수 있는 시각적 상태들을 파악합니다.

2. 무엇에 의해서 상태변화가 일어나는지 파악합니다.

    - 사용자 이벤트
    - 네트워크 요청
    - `setTimeOut`, `setInterval`
    - 파일 읽기/쓰기

3. `useState` hook으로 primitive 단위로 상태값들을 정의합니다.

4. 불필요한 상태값들은 제거하거나 하나로 합치거나 구조를 수정합니다.

5. DOM의 event handler로 state setter를 등록합니다.
