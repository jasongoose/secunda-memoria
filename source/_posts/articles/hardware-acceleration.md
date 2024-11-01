---
title: 카카오웹툰은 하드웨어 가속과 IntersectionObserver를 어떻게 사용했을까?
date: 2024-11-01 22:25:00
categories:
  - Articles
#tags:
---
렌더링 연산을 GPU에 할당하는 하드웨어 가속으로 렌더링 처리소요 시간을 줄일 수 있습니다.

하지만 연산속도를 높일 수는 있어도 연산량을 줄이기 위한 최적화 과정은 interaction에 기반한 렌더링에는 반드시 필요합니다.

렌더링 최적화와 관련해서 css 같은 경우 주로 `opacity`와 `transform`을 사용할 것을 추천합니다.

두 속성은 렌더 프레임 과정에서 Layout, Paint 과정을 생략하기 때문에 연산량을 줄일 수 있습니다.

하드웨어 가속을 사용하려면 GPU에 연산을 할당하는 css property들(`will-change`, `translate3d` 등)을 사용하면 됩니다.

또한 이미지와 같은 asset들의 lazy-load를 구현할 때는 가급적이면 [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)를 권장합니다.

## 참고자료

[카카오웹툰은 하드웨어 가속과 IntersectionObserver를 어떻게 사용했을까?](https://fe-developers.kakaoent.com/2021/211202-gpu-intersection-observer/?fbclid=IwAR0WtlK_ZWCb69vB1oWmo_VqX1_hxjIeG6ra1nNkTdzvBtOgEyp6eBg7bUM)
