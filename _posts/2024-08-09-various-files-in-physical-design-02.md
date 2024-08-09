---
title: "Ch2. LEF file"
excerpt: "Various files in Physical Design"
date: 2024-08-09 19:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

## Various files in Physical Design

1. .v file
2. .vhd file
3. .lib file
4. .db file
5. **.lef file**
6. **.tf file**
7. .mw file
8. .def file
9. .sdc file
10. .tlu file (TLU+ Library)
11. .saif file
12. .spef
13. .sbpf
14. .sdf
15. .map
16. .itf
17. .io file
18. .tcl file
19. .rspf
20. Simv
21. .gds file

## `.lef` file

- Library Exchange Format file
- ASCII
- 2 parts: Technology LEF & Cell LEF
- Technology LEFT는 metal layer, via, design rule 정보를 담고 있고,
- Cell LEF는 abstract view에서의 각 cell의 geometry 관련 정보를 담고 있다.

### Technology LEF

- LEF Version - 5.5, 5.6, 5.7, 5.8, ...
- Units - DATABASE, TIME, RESISTANCE, CAPACITANCE, ...
- Manufacturing grid - 0.005, 0.0025, ...
- Design rules
- details for each BEOL layers
  - Layer name - poly, contact, metal1, via1, etc...
  - Type - Masterslice, Routing, cut, etc...
  - Direction - Horizontal, Vertical
  - Pitch
  - Width
  - Spacing
  - Resistance (per square unit)
  - Layer definition은 bottom에서 top 순서로 정의되어야 함
  - 파운드리로부터 나오는 정보

### Cell LEF

- Cell name - AND2X2, CLKBUIF1, ...
- Class - CORE, PAD
- Origin - 0 0
- Size (width by height of cell, PR boundary) - 0.95 BY 2.47
- Symmetry - X Y, X, Y
- Site - CoreSite, PAD
- Details of each pins
  - Pin name - A, B, Y
  - Direction - INPUT, OUTPUT, INOUT
  - Use - SIGNAL, CLOCK, POWER, GROUND
  - Shape - ABUTMENT in case of pwr and gnd pin
  - Layer - metal1, metal2, ...
  - rectangle coordinates - RECT llx lly urx ury

#### Abstract view

![abstract-layout]({{site.baseurl}}/assets/images/2024-08-09-various-files-in-physical-design-02/abstract-layout.png){: .align-center}
abstract layout
{: .custom-caption}

- P&R을 위한 layout abstract description
- 연결을 위해 자세한 pin 정보 포함
- front-end of the line(FEOL) layer(poly, diffusion, etc...) 정보는 포함하지 않음

![abstract-view]({{site.baseurl}}/assets/images/2024-08-09-various-files-in-physical-design-02/abstract-view.png){: .align-center}
abstract-view
{: .custom-caption}

- Cell에서 Metal1이 사용된 곳을 Obstruction으로 하여 해당 부분에 Metal1이 사용되는 것을 방지

### LEF file in P&R flow

![pnr-flow-lef]({{site.baseurl}}/assets/images/2024-08-09-various-files-in-physical-design-02/pnr-flow-lef.png){: .align-center}

- P&R 툴에서 design import 시 필요한 중요한 파일 중 하나
- LEF는 abstract view에서의 cell layout 정보를 포함하여 physical library라고도 불림
- LEF 파일은 standard cell library에 포함되어 있다. 만약 customized cell로 P&R을 하고 싶다면 LIB file, LEF file, Verilog file, .gds 파일을 생성해야 한다.
- Cadence Virtuoso에는 "Abstract"라고 불리는 LEF 파일 생성툴이 내장되어 있다.
- LEF 파일 생성과 syntax에 대한 자세한 정보는 abstract tool 메뉴얼을 참고할 수 있다.

## `.tf` file

- Technology File
- Technology file은 physical design tool의 가장 중요한 파일 중 하나
- technology LEF에서 다룬 것과 동일한 정보를 가지고 있음
- Cadence P&R tool(Innovus)에서는 technology LEF format을 읽을 수 있고, Synopsys P&R tool(IC Compiler)에서는 technology file(.tf) 파일 형태를 요구함
- Technology file은 모든 metal layer, via, design rule에 대한 세부 정보를 포함함
- Technology file(.tf)는 ASCII 형태
- Technology name, unit of parameters(Length, Time, Voltage, Current, Power, Resistance, Capacitance, Inductance)
- RGB Color - 이 Color들은 layer에 할당되어 visualization에 사용됨, 비슷하게 Pattern도 정의됨
- 각 layer의 다양한 parameter들 정의
- Layer name, Layer Number, Mask Name, appearance(color, pattern)
- Minimum width, Minimum spacing, Pitch, default width, minimum area
- Design rule for via - Minimum enclose, end line enclosure
- Various Capacitance table
- .tf file과 technology LEF file은 동일한 정보를 가지고 있음(사용하는 P&R 툴에 따라 포맷이 다른 것)
