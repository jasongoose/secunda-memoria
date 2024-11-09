---
title: You Can throw() Anything In JavaScript - And Other async/await Considerations
date: 2024-11-01 22:23:31
categories:
  - Articles
#tags:
---
`await` 구문 뒤에서 에러가 발생하거나 rejected Promise가 return되면 try-catch 구문에서 catch 함수의 paramater로 `Error` 객체 또는 rejected value가 전달된다.

```js
async function makeRequest() {
  try {
      var fetchResponse = await fetch();
      var data = await this.unwrapResponseData(fetchResponse);

      if (!fetchResponse.ok) {
        // return( Promise.reject( this.normalizeError( data ) ) );
        throw(this.normalizeError(data));
      }
  } catch (error) {
    // return( Promise.reject( this.normalizeTransportError( error ) ) );
    throw(this.normalizeTransportError(error));
  }
}
```

## 참고자료

[You Can throw() Anything In JavaScript - And Other async/await Considerations](https://www.bennadel.com/blog/4210-you-can-throw-anything-in-javascript-and-other-async-await-considerations.htm)
