---
title: Broadcast Channel
date: 2024-11-02 00:08:34
categories:
  - Studies
  - Browser
  - Web API
#tags:
---
서로 다른 origin을 가진 [Browsing Context](https://developer.mozilla.org/en-US/docs/Glossary/Browsing_context) 사이의 통신은 제한적이지만, 동일한 origin이라면 Broadcast Channel API를 사용하면 됩니다.

![Broadcast Channel](/images/bc.png)

동일한 origin을 가지는 context들이 공용 채널에 구독하는 방식으로 통신이 이루어지는데, 임의의 context에서 메시지를 채널로 전송하면 해당 채널을 구독하는 나머지 context들에게도 그대로 메시지가 전달됩니다.

context 사이의 통신을 "Cross-context Communication" 이라고 합니다.

채널마다 고유의 origin과 history(지금까지 시간 순으로 렌더링된 페이지들로 구성된 stack)를 가집니다.

동일한 origin을 가진 다른 탭에서 로그인/로그아웃 같은 user action을 감지할 때 유용합니다.

## Interface

처음으로 채널을 생성하거나 이미 생성된 채널을 구독하려면 context에서 동일한 id를 가진 BroadcastChannel 인스턴스를 만들면 됩니다.

```js
// Connection to / Creation of a broadcast channel
const bc = new BroadcastChannel("test_channel");
```

특정 채널로 메시지를 전송할 때는 대응되는 인스턴스의 `postMessage` 메서드를 사용하면 됩니다.

```js
bc.postMessage("This is a test message.");
bc.postMessage({ prop: 1331 });
```

임의의 context가 자신이 구독한 채널로 전송된 다른 메시지를 확인할 때는 인스턴스의 `onmessage`라는 event handler를 사용하면 됩니다.

```js
bc.onmessage = (event) => {
  console.log(event.data);
};
```

임의의 context에서 특정 채널로의 구독을 취소할 때는 대응되는 인스턴스의 `close` 메서드를 사용하면 됩니다.

```js
// Disconnect the channel
bc.close();
```
