---
title: "Chapter 7. Specialized Routing"
excerpt: "VLSI Physical Design"
date: 2024-06-24 00:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

- Area routing: global, detailed routing 단계를 나누지 않고 바로 metal route를 만드는 방법, design size가 크지 않을 때 사용할 수 있다.
- Non-Manhattan routing
- Clock signals

### Introduction to Area Routing

- The goal of area routing is to route all nets in the design
  - without global routing
  - within the given layout space
  - while meeting all geometric and electrical design rules
- Area routing performs the following optimizations
  - minimizing the total routed length and number of vias of all nets
  - minimizing the total area of wiring and the number of routing layers
  - minimizing the circuit delay and ensuring an even(골고루 배치) wire density
  - avoiding harmful capacitive coupling between neighboring routes
- Subject to
  - technology constraints (number of routing layers, minimal wire width, etc.)
  - electrical constraints (signal integrity, coupling, etc.)
  - geometry constraints (preferred routing directions(레이어별 주 사용 방향이 있으나 약간은 위반할 수 있다), wire pitch, etc.)

### Net Ordering in Area Routing



### Non-Manhattan Routing

#### Octilinear Steiner Trees

#### Octilinear Maze Search

### Basic Concepts in Clock Networks

#### Terminology

#### Problem Formulations for Clock-Tree Routing

### Modern Clock Tree Synthesis

#### Constructing Trees with Zero Global Skew

#### Clock Tree Buffering in the Presence of Variation

