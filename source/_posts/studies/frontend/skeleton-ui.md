---
title: Skeleton UI
date: 2024-11-02 18:21:22
categories:
  - Studies
  - Frontend
#tags:
---
실제 컨텐츠를 로딩하는 동안 사용자에게 앞으로 화면에 보여질 컨텐츠의 전반적인 구조를 미리 보여주는 UI입니다.

사용자에게 로딩 상태를 전달하여 UX를 향상시킬 수 있고 페이지 내의 컨텐츠가 모두 load될 필요없이 점진적으로 노출시킬 수 있는 효과가 있습니다.

페이지에 먼저 placeholder들을 load하고 데이터가 fetch되는대로 lazy-load하는 간단한 원리에 맞춰 구현하는 방법은 2가지가 있습니다.

## Contents Placeholder

페이지의 전반적인 구성을 보여주는 placeholder로, 원본 컨텐츠없이 간단하게 구현할 수 있습니다.

Medium, Slack, Youtube 등 서비스의 메인 페이지에서 해당 기법을 사용합니다.

## Image(Color) Placeholder

웹 페이지에 이미지를 load하기 전에 보여주는 단색 이미지 placeholder로, 실제 컨텐츠가 필요하기 때문에 구현과정이 복잡합니다.

Pinterest, Unsplash 등 서비스에서 해당 기법을 사용합니다.

## 참고자료

[Improve React UX with skeleton UIs](https://blog.logrocket.com/improve-react-ux-skeleton-ui/#:~:text=A%20skeleton%20screen%20is%20a,shape%20similar%20to%20actual%20content.&text=In%20addition%20to%20skeleton%20screens,content%20loaders%2C%20and%20ghost%20elements)
