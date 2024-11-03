---
title: 1. Render Pipeline
date: 2024-11-03 20:56:07
categories:
  - Studies
  - Vue3
  - Rendering Mechanism
#tags:
---
Vue 컴포넌트가 mount되고 patch가 일어나기까지의 과정을 정리하면 다음과 같습니다.

![Render Pipeline](/images/render_pipeline.png)

## Compile

compiler에 의해서 `.vue` 파일에서 작성한 template이 Vtree를 반환하는 render 함수로 변환되는 과정입니다.

## Mount

runtime renderer가 render 함수를 호출하여 반환된 Vtree를 기반으로 Rtree를 생성하는 단계입니다.

## Patch

컴포넌트 내부에서 trigger에 의해 effect들이 재연산되면 그에 맞춰서 Vtree를 생성하고 기존에 메모리에 있던 Vtree와 비교했을 때 달라진 부분을 반영하여 Rtree의 일부를 수정합니다.

## 참고자료

[Rendering Mechanism](https://vuejs.org/guide/extras/rendering-mechanism.html#virtual-dom)