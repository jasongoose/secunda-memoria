---
title: Abstraction, Composition
date: 2024-10-28 08:41:01
categories:
- Books
- Composing Software
- Concepts
#tags:
---
본래 의미는 기존의 context로부터 대상을 분리하여 본질을 추출하는 과정이지만 소프트웨어에서는 다음과 같은 의미를 가집니다.

- Decomposition
    - 문제를 해결하는 코드는 다수의 함수나 모듈 등의 독립적인 단위로 분리한다.
- Generalization
    - 반복된 pattern들 사이에서 보이는 유사성들을 은닉하여 중복을 줄인다.
- Specialization
    - 추상화된 단위에서 차이나는 부분만 고정하여 용도별로 사용한다.
- Simplization
    - 코드의 길이를 줄인다.

위와 같은 추상화의 예시들은 다음과 같습니다.

- Algorithm
- 3. Data Structure
- Module
- Class
- Framework

## 하지만...

FP에서 추상화 단위는 오직 **순수함수**입니다.

- 코드를 pure function 단위로 분리 ⇒ decomposition, simplization
- 커리함수와 같은 고차함수의 정의 ⇒ generalization
- 고차함수에 1개 이상의 함수형 인자들을 전달 ⇒ specialization
