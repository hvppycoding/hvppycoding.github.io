---
title: "Disjoint Sets"
excerpt: "Advanced Programming Methodologies 10"
date: 2023-05-15 01:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

## Disjoint Sets Data Structure

- Set를 Union하는 연산을 어떻게 효율적으로 구현할 것인가?
- N개의 element를 그룹으로 가지는 set의 collection을 필요로 할 때가 있다. 주로 set을 찾거나 set을 합치는 연산을 수행한다.
- 예를 들어 k개의 disjoint set $S = \lbrace S_1, S_2, S_k \rbrace$
- 각 set은 각 set에 속한 representative(대표) element로 indentify할 것이다.
- x가 object일 때 다음과 같은 operation들을 지원해야 한다.
  - `MAKE-SET(x)`: x를 포함하는 새로운 set을 생성한다.
    - set들이 서로 disjoint해야 하기 때문에 x는 어떤 set에도 속하지 않아야 한다.
  - `UNION(x, y)`: x와 y를 포함하는 두 set을 합친다.
    - 두 set은 서로 disjoint하다고 가정한다.
    - 결과의 representative는 $S_x \cup S_y$ 중 아무 원소나 가능하다.
  - `FIND-SET(x)`: x를 포함하는 set의 representative element를 반환한다.

## Disjoint-set Operations

- Terms
  - n: `MAKE-SET` operation 수
  - m: `MAKE-SET`, `UNION`, `FIND-SET` operation 총 수
- n번의 `MAKE-SET` operation이 제일 처음에 수행된다고 가정한다.
- set들이 disjoint하기 때문에 `UNION`은 set의 수를 하나 줄인다. 따라서 `UNION` 연산은 최대 n-1번이다.

## An Application of Disjoint-set Data Structure

- undirected graph가 있을 때 connected component를 찾는 문제를 생각할 수 있다.

```text
CONNECTED-COMPONENTS(G)
1.  for each vertex v ∈ V
2.    MAKE-SET(v)
3.  for each edge (u, v) ∈ E
4.    if FIND-SET(u) ≠ FIND-SET(v)
5.      UNION(u, v)
```
