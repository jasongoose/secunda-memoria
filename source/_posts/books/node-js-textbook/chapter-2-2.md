---
title: 2-2장. 프런트엔드 자바스크립트
date: 2024-10-29 23:53:23
categories:
  - Books
  - Node.js 교과서(개정 3판)
  - Chapter 2
#tags:
---
## AJAX(Asynchronous Javascript And XML)

비동기적 웹 서비스를 개발할 때 사용하는 기법으로, 페이지 이동 없이 서버에 요청을 보내고 응답을 받습니다.

보통 AJAX 요청은 jQuery나 Axios 같은 라이브러리를 사용합니다.

브라우저에서 기본적으로 XMLHttpRequest나 Fetch API를 제공하기 하지만 사용방법이 복잡하고 서버에서는 사용할 수 없습니다(뻥이고 node.js v21부터 API가 안정화되었다는 소식이 있습니다😆).

## FormData

HTML form 태그의 데이터를 동적으로 제어할 수 있는 기능으로, encoding type이 `multipart/form-data`입니다.

```js
(async () => {
  try {
    const formData = new FormData();
    formData.append("name", "jasongoose");
    formData.append("birth", 1997);

    const result = await axios.post("", formData);
  } catch (err) {
    console.error(err);
  }
})();
```

## encodeURIComponent, decodeURIComponent

서버가 한글 주소를 이해하지 못하는 경우가 있는데, 이럴 때 `window` 객체의 메서드인 `encodeURIComponent`와 `decodeURIComponent`를 사용할 수 있습니다.

```js
const encoded = encodeURIComponent("노드");
const decoded = decodeURIComponent(encoded);

console.log("encoded:: ", encoded);
console.log("decoded::", decoded);
```

## data 속성과 dataset

프론트엔드에 민감한 데이터를 내려보내는 것은 실수이므로 조심해야 합니다.

서버에서 클라이언트로 보낸 데이터를 자바스크립트 변수에 저장해도 되지만 HTML5에서 데이터를 저장하는 공식적인 방법을 사용할 수 있습니다.

```html
<ul>
  <li data-id="1" data-user-job="programmer">jason</li>
  <li data-id="2" data-user-job="designer">goose</li>
</ul>
<script>
  console.log(document.querySelector("li").dataset);
  // { id: '1', userJob: 'programmer' }
</script>
```

`data` 속성은 자바스크립트로 쉽게 접근할 수 있다는 장점이 있고 `dataset`에 데이터를 넣어도 HTML 태그에 반영할 수 있는 양방향성을 가집니다.

```js
document.querySelector("li").dataset.monthSalary = 10000;
```
