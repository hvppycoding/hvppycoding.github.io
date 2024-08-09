---
title: "LIB, DB, Verilog file"
excerpt: "Vaarious files in Physical Design"
date: 2024-08-09 06:00:00 +0900
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
5. .lef file
6. .tf file
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

## `.v` file

- Verilog file의 확장자
- HDL(Hardware Description Language) 소스 코드
- Synthesis 툴(e.g. Design Compiler, Genus)의 input 및 output 파일
- RTL code synthesis 후에 Gate level netlist(`.v`)가 생성됨
- PnR 툴의 인풋으로 사용됨(source of design, gate level netlist 형태로)
- gate level netlist를 RTL code와 구분하기 위해 `.vg` 확장자를 사용하기도 함

## `.vhd` file

- VHDL 파일의 확장자
- Verilog와 syntax는 다르나, 비슷한 정보를 담고있음

![synthesis flow]({{site.baseurl}}/assets/images/2024-08-09-various-files-in-physical-design-01/synthesis-flow.png){: .align-center}
Synthesis Flow{: .custom-caption}

![pnr flow]({{site.baseurl}}/assets/images/2024-08-09-various-files-in-physical-design-01/pnr-flow.png){: .align-center}
PnR Flow{: .custom-caption}

## `.lib` file

- Liberty Timing file
- 특정 technology node의 cell의 timing, power parameters 정보를 ASCII로 담고 있음
- Timing model; cell delay, cell transition, setup and hold time requirements of a cell
- gate나 macro library의 timing, electrical characteristics
- gate library vendor나 foundry에 의해 제공됨

### `.lib` 파일이 포함하는 정보

- General Information, common for all cells
  - Library name and technololgy
  - Units(of Time, Power, Voltage, Current, Resistance, Capacitance)
  - Value of Process, Voltage and Temperature(Max, Min, Typical - so there are three different files)
- Cell specific Information
  - Cell name
  - pg pin
  - Leakage power