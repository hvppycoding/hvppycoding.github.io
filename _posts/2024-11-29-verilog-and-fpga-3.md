---
title: "Part2-2. Verilog 기본 문법 & Modeling"
excerpt: ""
date: 2024-11-29 12:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

## Contents

- Verilog에서 print 하는 법
- Verilog 에서 waveform 만드는 법
- Data type
- Module & Port & Instantiation
- parameter 및 define
- Operator

### How to print

- `$display`: C에서 `printf`와 비슷하다.
- `$monitor`: 특정 변수가 변할 때마다 출력한다.

```verilog
`timescale 1ns/1ns
module hello_world;
  int test = 0;
   
  initial begin
    #10 $display("time %t : Hello, World %d", $time, test++);
    #10
    $display("time %t : Hello, World %d", $time, test++);
    $display("time %t : Hello, World %d", $time, test++);

    $finish;
  end

  initial begin
    $monitor("time %t : test %d", $time, test);
  end
endmodule
```

- `$time`은 현재 시간을 리턴하는 함수이다.
- `initial begin` ~ `end` 블록들은 서로 독립적으로 실행된다.

![print-example-run]({{site.baseurl}}/assets/images/2024-11-29-verilog-and-fpga-3/print-example-run.png){: .align-center}  

- `$monitor`가 마지막에 2로 변하는 것은 나타나지 않은 이유
- `$finish`가 불려서 종료되기 때문이다. `$finish` 호출 전에 delay를 더 주면 해결된다.

### How to debug

- `$dumpfile`: waveform을 저장할 output file을 정의한다(`*.vcd`).
- `$dumpvars`: waveform dump 범위를 정의한다.

```verilog
module z
```