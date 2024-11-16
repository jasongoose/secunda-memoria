---
title: Reliably Send an HTTP Request as a User Leaves a Page
date: 2024-11-01 22:20:51
categories:
  - Articles
#tags:
---
현재 페이지가 `terminated` 상태(page life cycle)가 되는 경우는 다음과 같습니다.

- 다른 페이지로 navigate할 때
- 다른 `<form></form>`을 submit할 때
- 브라우저 창이나 탭을 닫을 때

---

아래와 같은 상황이 있다고 합시다.

```html
<a href="/some-other-page" id="link">Go to Page</a>

<script>
  document.getElementById("link").addEventListener("click", (e) => {
    fetch("/log", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        some: "data",
      }),
    });
  });
</script>
```

`<a></a>`를 클릭하면 href 속성에 명시된 URL로 이동하면서 POST 요청을 전송하고 처음에는 페이지로 이동하면서 요청이 알아서 잘 처리되겠지 했지만 실제로는 중간에 cancel됩니다.

페이지가 terminated 상태에서 시작한 fetch와 `XmlHttpRequest`는 **asynchronous, non-blocking(main thread를 붙잡지 않는)** 연산이기 때문에 브라우저 Web API에 의해서 적절한 시점에 Task Queue에 push되고 call stack이 빌 때마다 실행됩니다.

하지만 page가 terminated 상태가 된다면 브라우저에 의해 메모리가 clear되면서 in-process task들은 중간에 중지될 수 있기 때문에 위 상황에서 다른 페이지로 이동했을 때, POST request가 cancel됩니다.

---

그렇다면 현재 페이지 내 `terminated` 상태에서 진행하고 있던 HTTP request를 중지하지 않고 그대로 유지하려면 어떻게 해야 할까요?

다음과 같은 방법들을 고려할 수 있습니다.

1. fetch API의 `keepalive: true` flag 추가하여 중간에 cancel되지 않고 최종 status가 `unknown` 상태로 만들 수 있습니다.

2. `Navigator.sendBeacon` 메서드 사용합니다.

```html
<a href="/some-other-page" id="link">Go to Page</a>

<script>
  document.getElementById('link').addEventListener('click', (e) => {
    const blob = new Blob([JSON.stringify({ some: "data" })], { type: 'application/json; charset=UTF-8' });
    navigator.sendBeacon('/log', blob));
  });
</script>
```

fetch보다는 처리 우선순위가 낮아서 UX 또는 성능에 영향을 주는 request에 영향을 주지 않는 장점이 있다.

3. `<a></a>`에 ping attr 추가합니다.

POST request의 http header에 `ping-from` , `ping-to` 필드가 추가됩니다.

`<a></a>`에서만 사용할 수 있고 Firefox에서는 미지원합니다.

POST request임에도 불구하고 payload data를 전송할 수 없습니다.

## 출처

[Reliably Send an HTTP Request as a User Leaves a Page](https://css-tricks.com/send-an-http-request-on-page-exit/)

[Page Lifecycle API](https://developer.chrome.com/articles/page-lifecycle-api/)
