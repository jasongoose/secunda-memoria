---
title: 6장. 라우팅
date: 2024-11-10 20:57:34
categories:
  - Books
  - 프레임워크 없는 프론트엔드
#tags:
---
## 단일 페이지 애플리케이션

- 하나의 HTML 페이지로 실행되는 웹 애플리케이션
  - 사용자가 다른 뷰로 이동할 때 동적으로 다시 그려서 웹 탐색 효과를 줍니다.

- 기존 다중 페이지 애플리케이션에서 페이지 간의 탐색 시 발생하는 지연을 제거하여 더 나은 UX를 제공합니다.
  - 서버와의 상호작용 시 AJAX를 사용합니다.

- 라우팅 시스템을 통해 경로를 정의할 수 있습니다.
  - 최소 2가지 핵심 요소
    - 애플리케이션의 경로 목록을 수집하는 "레지스트리"
      - URL -> DOM 요소로 매칭합니다.
    - 현재 URL의 리스너
      - URL이 변경되면 라우터는 메인 컨테이너(`<div id="root"></div>`)의 내용을 매칭되는 DOM 요소로 교체합니다.

## 코드 예제

- 프레임워크가 없는 라우팅 시스템
  - 프래그먼트 식별자
  - 히스토리 API
- 프레임워크가 있는 라이팅 시스템
  - Navigo

### 프래그먼트 식별자

- 특정 id를 가진 HTML 요소를 URL 상에서 식별하기 위해서 사용합니다.
- 브라우저는 URL의 프래그먼트로 식별된 요소가 뷰포트의 맨 위로 오도록 페이지를 스크롤합니다.

#### 첫번째 예제

```js
// pages.js
// route별로 container에 교체하는 URL 리스너 작업을 정의
export default container => {  
  const home = () => {  
    container  
      .textContent = 'This is Home page'  
  }  
  
  const list = () => {  
    container  
      .textContent = 'This is List Page'  
  }  
  
  const notFound = () => {  
    container  
      .textContent = 'Page Not Found!'  
  }  

  return {  
    home,  
    list,  
    notFound  
  }  
}
```

```js
// router.js
export default () => {  
  // 라우팅 시스템의 구성요소1 - 레지스트리
  const routes = []
  let notFound = () => {}  
  
  const router = {}  

  const checkRoutes = () => {  
    // 현재 fragment(hash)와 매칭되는 route
    const currentRoute = routes.find(route => {  
      return route.fragment === window.location.hash  
    })  
  
    if (!currentRoute) {  
      notFound()  
      return  
    }  
  
    currentRoute.component()
  }  
  
  router.addRoute = (fragment, component) => {
    const route = { fragment, component };
    
    routes.push(route);  
  
    // 체이닝이 가능하도록 router 객체 반환
    return router  
  }  

  // 레지스트리에 없는 모든 프래그먼트에 대한 generic 구성 요소를 설정
  router.setNotFound = cb => {
    notFound = cb  
    return router  
  }  
  
  router.start = () => {  
    // 라우팅 시스템의 구성요소2 - URL 리스너
    window  
      .addEventListener('hashchange', checkRoutes)  
  
    if (!window.location.hash) {  
      window.location.hash = '#/'  
    }  

    checkRoutes()  
  }  
  
  return router  
}
```

#### 프로그래밍 방식으로 탐색

- 링크가 아닌 코드 상에서 뷰를 이동할 수 있도록 라우터에 `navigate` 메서드 추가합니다.

```js
export default () => {  
  // ...
    
  router.navigate = fragment => {  
    window.location.hash = fragment  
  }  
  
  router.start = () => {  
    window  
      .addEventListener('hashchange', checkRoutes)  
      
    if (!window.location.hash) {  
      window.location.hash = '#/'  
    }  
  
    checkRoutes()  
  }  
  
  return router  
}
```

#### 경로 매개변수

- 라우터에 경로 매개변수(Route Params)를 읽을 수 있는 기능을 추가합니다.

```js
// route params에 매칭되는 문자열을 판별
const ROUTE_PARAMETER_REGEXP = /:(\w+)/g  

//  '/', '\'를 제외한 path를 구성하는 문자열을 판별
const URL_FRAGMENT_REGEXP = '([^\\/]+)'  
  
const extractUrlParams = (route, windowHash) => {  
  const params = {}  
  
  if (route.params.length === 0) {  
    return params  
  }  
  
  const matches = windowHash  
    .match(route.testRegExp)  

  // 원본 pathName이 첫번째 index에 위치하므로 제외
  matches.shift()  

  // paramsName의 순서에 맞게 추출된 값들을 객체로 정리
  // #/list/1234 -> { id: '1234' }
  matches.forEach((paramValue, index) => {  
    const paramName = route.params[index]  
    params[paramName] = paramValue  
  })  
  
  return params  
}  
  
export default () => {  
  const routes = []  
  let notFound = () => {}  
  
  const router = {}  
  
  const checkRoutes = () => {  
  
	const { hash } = window.location  

	// 개별 route params 형식에 맞는 경로인지 확인
    const currentRoute = routes.find(route => {  
      const { testRegExp } = route  
      return testRegExp.test(hash)  
    })  
  
    if (!currentRoute) {  
      notFound()  
      return  
    }  
  
    const urlParams = extractUrlParams(  
      currentRoute,  
      window.location.hash  
    )  
  
    currentRoute.component(urlParams)  
  }  
  
  router.addRoute = (fragment, component) => {  
    const params = []  

	/**
	* /list/:id
	* 
	* 첫번째 replace로...
	* 1. 개별 route params의 이름(=매개변수 이름, 여기서 "id")을 배열에 push
	*    ["id"]
	* 2. params 자리를 정규식으로 교체
	*    /list/:id -> /list/:([^\\/]+)
	* 
	* 두번째 replace로...
	* 기존 경로에 있던 '/'와 정규식에 있던 '/'를 이스케이프 처리
	* /list/:([^\\/]+) -> \/list\/:([^\\\/]+)
	*
	* params를 표시한 path로부터 params를 추출하는 정규식을 만드는 것이 parsedPath의 목적임
	*/
    const parsedFragment = fragment  
      .replace(  
        ROUTE_PARAMETER_REGEXP,  
        (match, paramName) => {  
          params.push(paramName)  
          return URL_FRAGMENT_REGEXP  
        })  
      .replace(/\//g, '\\/')
  
    routes.push({  
      testRegExp: new RegExp(`^${parsedFragment}$`),  
      component,  
      params  
    })  
  
    return router  
  }  
  
  router.setNotFound = cb => {  
    notFound = cb  
    return router  
  }  
  
  router.navigate = fragment => {  
    window.location.hash = fragment  
  }  
  
  router.start = () => {  
    window  
      .addEventListener(  
        'hashchange',  
        checkRoutes  
      )  
  
    if (!window.location.hash) {  
      window.location.hash = '#/'  
    }  
  
    checkRoutes()  
  }  
  
  return router  
}
```

### 히스토리 API

- 사용자 탐색 히스토리를 조작하는데 유용합니다.
- 라우팅에 적용하는 경우 프래그먼트 식별자가 아닌 URL 자체를 활용합니다.

```js
// ... 
  
export default () => {  
  const routes = []  
  let notFound = () => {}  
  let lastPathname  
  
  const router = {}  
  
  const checkRoutes = () => {
    // ...
  }
  
  router.navigate = path => {  
    window.history.pushState(null, null, path)  
  }  
  
  router.start = () => {  
    checkRoutes()

    // URL의 프래그먼트 값의 변화는 "hashchange" 이벤트로 구분할 수 있지만,
    // URL의 변화는 안되기 때문에 현재 URL을 주기적으로 확인하는 방법을 적용
    window.setInterval(checkRoutes, TICKTIME)  
  
    document  
      .body  
      .addEventListener('click', e => {  
        const { target } = e  
        if (target.matches(NAV_A_SELECTOR)) {  
          e.preventDefault()  
          router.navigate(target.href)  
        }  
      })  
  
    return router  
  }  
  
  return router  
}
```

#### 링크 사용

- anchor 태그의 `href`로 router에서 사용하는 경로를 그대로 작성하면 브라우저의 default 동작에 의해 뒤에 "/index.html"이 붙으면서 404 에러가 발생할 수 있습니다.
- `<a href="/list/1234"></a>` -> `localhost:8081/list/1234/index.html`
- 따라서 링크를 통해 이동하는 경우 `preventDefault`를 적용합니다.

```html
<a data-navigation href="/">Go To Index</a>  
<a data-navigation href="/list">Go To List</a>  
<a data-navigation href="/list/1">Go To Detail With Id 1</a>  
<a data-navigation href="/list/2">Go To Detail With Id 2</a>  
<a data-navigation href="/list/1/2">Go To Another Detail</a>  
<a data-navigation href="/dummy">Dummy Page</a>
```

```js
// router.js
const NAV_A_SELECTOR = 'a[data-navigation]'

export default () => {
  // ...
  
  router.start = () => {  
    checkRoutes()  
    window.setInterval(checkRoutes, TICKTIME)  
  
    document  
      .body  
      .addEventListener('click', e => {  
        const { target } = e  
        if (target.matches(NAV_A_SELECTOR)) {
          // 이벤트 위임으로 default 동작 비활성화
          e.preventDefault()
          router.navigate(target.href)  
        }  
      })  
  
    return router  
  }  
  
  return router  
}
```
### Navigo

- [Navigo](https://www.npmjs.com/package/navigo)

## 올바른 라우터를 선택하는 방법

- 프레임워크 없는 구현으로 시작하고 나중에 복잡해지면 Navigo와 같은 프레임워크를 적용합니다.
