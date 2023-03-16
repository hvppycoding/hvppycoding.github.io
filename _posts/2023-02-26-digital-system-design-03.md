---
title: "Digital System Design 03"
excerpt: "Logic Simplification"
date: 2023-02-26 00:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Digital System Design
mathjax: "true"
---

## 1. K-map based Simplification

- Boolean logic을 K-map으로 표현하는 절차를 습득한다.
- K-map을 사용해서 Boolean logic의 simplified two-level logic을 생성하는 절차를 습득한다.

### Essence of Boolean Simplification

- **Uniting Theorem**
  - F = A'B' + AB' = (A' + A)B' = B'

### Karnaugh Maps(K-maps)

- Truth table을 K-map으로 표현할 수 있다.
- 최대 4개의 Variable까지 표현 가능하다.(2^4=16 cells)
- Wrap-around edges
- 인접한 Cell에서는 하나의 Bit만 변한다.

| A | B | F |
|---|---|---|
| 0 | 0 | 1 |
| 0 | 1 | 0 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

위의  테이블을 K-map화 하면 다음과 같다.

|   | A=0 | A=1 |
|:-:|:---:|:---:|
|B=0|  1  |  1  |
|B=1|  0  |  0  |

### K-map Cell Numbering

![2023-02-26-kmap-examples.png]({{site.baseurl}}/assets/images/2023-02-26-kmap-examples.png)

### BDC increment-by-1 Function

입력 ABCD에 +1을 하여 WXYZ로 보내는 Function을 K-map으로 구해보자.  
단, 결과가 10이면 출력은 0이 된다.  

| A | B | C | D | W | X | Y | Z |
|---|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 |
| 0 | 0 | 0 | 1 | 0 | 0 | 1 | 0 |
| 0 | 0 | 1 | 0 | 0 | 0 | 1 | 1 |
| 0 | 0 | 1 | 1 | 0 | 1 | 0 | 0 |
|...|...|...|...|...|...|...|...|
| 1 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |
| 1 | 0 | 1 | 0 | X | X | X | X |
|...|...|...|...|...|...|...|...|
| 1 | 1 | 1 | 1 | X | X | X | X |

![2023-02-26-increment-kmap.png]({{site.baseurl}}/assets/images/2023-02-26-increment-kmap.png)

### Definition of Terms

1. Implicant: K-map에서 공통으로 묶은 단위
2. Prime implicant: 최대로 확장된 Implicant
3. Essential prime implicant: 다른 Prime implicant로 커버할 수 없는 원소를 포함한 Prime implicant

## 2. Quine-McClusky based Simplification

1. Quine-McClusky 방법을 통해 최적의 SOP를 생성하는 과정을 익힌다.
2. Espresso 방법을 통해 SOP 생성의 휴리스틱한 방법의 예를 알아본다.

### Quine-McClusky Method: Stage 1

- K-map은 손수 구할 때 사용할 수 있는 방법이다. Quine-McCluskey는 컴퓨터의 도움을 받기 위해 체계적으로 구성된다.
- Stage 1에서는 prime implicants를 모두 찾는다.

### Quine-McClusky Method: Stage 2

- Stage 2에서는 minimum cover를 찾는다.
  1. Initial prime implicant chart를 그린다.
  2. Essential prime implicants를 찾아 선택한다.
  3. 남은 set 충 minimum set을 찾는다.

### Espresso Method

앞의 Quine-McClusky는 최적해를 찾으나, 시간이 오래 걸린다.  
Espresso Method는 상대적으로 빠르나 최적을 보장하지는 못한다.  

1. Initial prime implicant를 찾는다.
2. REDUCE 단계: Cover는 유지하면서 최대한 크기를 줄인다.
3. EXPAND 단계: Cover를 다른 방향으로 늘려본다.
4. IRREDUNDANT COVER 단계: 중복되어 삭제할 수 있는 경우가 있으면 삭제한다.

위의 단계를 반복한다(일정 Timeout 시 현재까지의 Best를 사용).  
