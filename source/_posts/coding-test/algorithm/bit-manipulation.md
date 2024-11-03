---
title: Bit Manipulation
date: 2024-11-03 23:44:57
categories:
  - Coding Test
  - Algorithm
#tags:
---
## Binary(Positive)

![Bit Binary Positive](/images/bit_binary_positive.png)

## Addition

![Bit Addition](/images/bit_addition.png)

## Binary(Negative)

음수의 이진수는 기존의 양수의 비트들을 모두 반대로 만든 다음에 1을 더한 "2의 보수"입니다.

![Bit Binary Negative](/images/bit_binary_negative.png)

## Shifting

### Logical

![Logical Shift](/images/bit_logical_shift.png)

### Arithmetic

![Arithmetic Shift](/images/bit_arithmetic_shift.png)


두 shifting 방식의 차이점은 바로 sign bit 값을 shift 이후에도 유지하는지 여부에 있습니다.

## Masks

![Bit Masking](/images/bit_masking.png)

```js
const getNthBit = (x, n) => (x & (1 << n) ? 1 : 0);

const setNthBit = (x, n) => x | (1 << n);

const clearNthBit = (x, n) => x & ~(1 << n);
```

## 참고자료

[HackerRank](https://www.youtube.com/@HackerrankOfficial/playlists)