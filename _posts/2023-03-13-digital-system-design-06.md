---
title: "Digital System Design 06"
excerpt: "Template based Design"
date: 2023-03-13 13:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Digital System Design
mathjax: "true"
---

## Template based Design

1. Template의 의미와 Template 기능을 가지는 logic component(DEMUX, PLA, PAL)들을 알아 본다.
2. Logic을 Template를 사용해서 구현하는 절차를 이해한다.

### Decoder / DEMUX(Demultiplexor)

![2023-03-13-demux.png]({{site.baseurl}}/assets/images/2023-03-13-demux.png)

DECODER의 출력을 OR Gate를 사용하여 출력을 Generation할 수 있다.

![2023-03-13-demux2.png]({{site.baseurl}}/assets/images/2023-03-13-demux2.png)

### PLA(Programmable Logic Array)

- Pre-fabricated building block of many AND/OR gates
- AND: product term 구현, OR: + 구현
- product term은 reuse 가능하다.

![2023-03-13-pla-example.png]({{site.baseurl}}/assets/images/2023-03-13-pla-example.png)

- OR gate의 input 수가 너무 많아지면 delay가 커진다. OR gate의 input 수 조절이 필요하다.

### PAL and PLA

- PLA는 OR gate를 자유롭게 조절할 수 있으나, 연결이 많아질수록 delay가 증가하는 문제가 있었다. PAL은 OR gate 부분이 고정되어 있다.

### Regular logic structures for two-level logic

- ROM - full AND plane, general OR plane
- PAL - programmable AND plane, fixed OR plane
- PLA - programmable AND and OR planes
