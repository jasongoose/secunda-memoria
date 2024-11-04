---
title: ECMA Spec
date: 2024-11-03 16:42:46
categories:
  - Studies
  - JavaScript
  - ECMA
#tags:
---
## ECMA-262

ECMA에서 제정된 기술 규격들 중 하나로 범용 목적의 스크립트 언어(general purpose scripting language)를 위한 명세서를 정의한 표준문서의 이름입니다.

ECMAScript Language Specification으로도 불립니다.

## ECMA International

1959년에 컴퓨터 보급이 확장되면서 컴퓨터 관련 분야들 사이의 표준 제정이 조금씩 이루어졌지만 각 회사, 나라마다 서로 다른 요구사항을 기반한 다양한 표준들이 만들어지면서 호환성이 제대로 이루어지지 않았습니다.

이에 유럽의 데이터 처리분야에서 선두였던 회사들을 중심으로 1960년에 컴퓨터 제조회사들 사이의 협력을 통해 호환성이 보장되고 일관성 있는 표준을 제정하기 위한 목적으로 비영리 단체 ECMA(European Computer Manufactures Association)이 설립되었습니다.

1994년에 ECMA International으로 이름을 바꿨고 현재까지도 정보통신기술(ICT), 가전(Consumer Electronics) 등 다양한 분야의 표준 제정에 힘쓰고 있습니다.

스위스 제네바에 본부를 두고 있습니다.

## ECMAScript

ECMA-262에 의해서 정의된 범용 목적의 스크립트 언어입니다.

독립된 프로그래밍 언어라기보다는 스크립트 언어가 갖추어야할 뼈대 즉, 문법(syntax)과 표현의 의미(semantics)를 보여주는 명세서라고 볼 수 있습니다.

실제 ECMA International에서 ECMA-262 문서를 보면 데이터 타입의 종류, 함수를 정의하는 방법, 클래스를 정의하는 방법 등이 매우 체계적이고 자세하게 설명되어 있습니다.

1997년에 초판된 이후로 시간이 지나면서 다양한 버전으로 수정되었고 ECMAScript를 준수하는 언어들로는 JavaScript, 마이크로소프트의 JScript, Macromedia의 Actionscript 등이 있습니다.

## JavaScript

ECMAScript 명세를 준수하는 수많은 스크립트 언어들 중에 하나입니다.

ECMAScript에는 없는 입출력 관련 기능이 구현되어 있고 다음과 같은 특징들을 가집니다.

- Prototype-based Object Orientaton(프로토타입 기반 객체지향적)
- First-class Function(고차함수 지원)
- Dynamic Typing(동적 타입 지정 : interpreter가 실행하는 중에 할당된 값에 기반하여 변수의 데이터 타입을 결정하는 것)
- Single Thread(단일 스레드)
- ...

웹 개발 분야 중 HTML, CSS와 함께 웹 클라이언트 쪽에서 무조건 사용되고 Node.js가 등장하면서 서버 사이드 소프트웨어나 임베디드 시스템 분야에서도 활용됩니다.

역사적으로 보면 JavaScript라는 언어가 ECMA International에 제출되었던 해가 1996년이고 그 뒤에 1997년에 첫 번째 ECMA-262 표준문서가 출판되었습니다.

어찌 보면 JavaScript와 ECMAScript는 거의 동일하다고 볼 수도 있는데 이 때부터 용어가 혼용되기 시작한 것으로 보입니다.

## JavaScript Engine

자동차가 움직이기 위해서는 엔진이 필요하듯이 JS로 작성한 코드를 이해하고 파싱하는 프로그램을 JavaScript Engine이라고 합니다.

대표적으로 Chrome의 V8, Firefox의 SpiderMonkey, Edge의 Chakra 등이 있습니다.

브라우저에 따라 성능과 지원하는 ECMAScript 버전이 다르기 때문에, 웹앱을 개발할 때는 브라우저와의 호환성을 고려해야 합니다.

ECMAScript의 새로운 버전이 나왔을 때, 브라우저의 JS 엔진을 업데이트할지 여부는 오로지 JS 엔진을 개발한 회사에서 결정합니다.

## JavaScript Runtime

JS 엔진에 의해서 코드를 실행하여 입출력의 결과를 확인할 수 있는 환경이나 시스템을 가리킵니다.

쉽게 떠올릴 수 있는 런타임으로는 웹 브라우저와 Node.js가 있습니다.

웹 브라우저에서 제공하는 Web API를 이용하여 웹 페이지를 만들거나 Node.js에서 제공하는 파일 시스템, 프로세스, 비동기 요청과 관련된 API를 이용하여 서버 사이드 소프트웨어를 작성할 수 있습니다.

## Scripting Language

기존 응용 소프트웨어의 작업들을 자동화시키는 런타임 시스템을 위한 프로그래밍 언어입니다.

scripting 언어로 작성한 코드를 실행하면 interpreter에 의해서 한 instruction씩 machine code로 해석/변환되고 런타임에 의해서 그 실행결과가 화면에 나타납니다.

이러한 실행과정은 컴파일 언어(Compiled Language)와 차이점이 있는데, 컴파일 언어로 작성된 코드(C 기준)를 빌드(build)하게 되면 compiler에 의해서 한 instruction이 아닌 전체 소스코드(하나의 파일)가 machine code로 변환되고 이러한 파일들이 링커(linker)에 의해 하나의 실행파일(executable file)이 만들어집니다.

그래서 결과를 확인하기 위해서는 특정 주체가 이 파일을 실행해야 합니다.

C언어로 계산기 프로그램을 만들어도 누군가 실행하지 않는 한 사용할 수 없지만 (JavaScript + HTML + CSS)로 만든 계산기는 브라우저에서 컴파일 과정없이 바로 실행할 수 있습니다.

또한 Web API의 `addEventListener` 메서드를 이용하면 사용자에 의해서 특정 이벤트가 일어날 때마다 이벤트 핸들러(event handler)를 자동으로 실행시킬 수 있습니다.

'자동화'라는 단어만 보면 뭔가 스크립트 언어가 컴파일 언어보다 더 낫다고 생각할 수도 있지만 실행속도 면에서 보면 컴파일 언어에 비해서 느리고 실행환경에 종속적이라는 단점이 있습니다.

현재 사용되는 스크립트 언어로는 JavaScript, JSP, PHP, Python, Bash 등이 있습니다.
