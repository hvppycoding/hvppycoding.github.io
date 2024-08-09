---
title: "Ch1. LIB, DB, Verilog file"
excerpt: "Various files in Physical Design"
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
Synthesis Flow
{: .custom-caption}

![pnr flow]({{site.baseurl}}/assets/images/2024-08-09-various-files-in-physical-design-01/pnr-flow.png){: .align-center}
PnR Flow
{: .custom-caption}

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
  - area
  - **For each pins**
    - Pin name
    - Pin direction
    - Power
    - Capacitance
    - Fanout load
    - Rise capacitance
    - Fall capacitance
    - Function(output pin일 경우)
- Characterization of cell
  - Timing and power parameters of cell - 다양한 조건에서 cell simulation을 통해 얻어짐
  - Two techniques to characterize a cell and generate `.lib` file
    - Composite current source(CCS)
    - Non-linear delay model(NLDM)
  - CCS technique는 current source가 사용되고, NLDM은 voltage source가 사용됨
  - CCS가 NLDM보다 더 많은 controlling parameter가 있고, CCS가 더 정확함 - CCS의 Runtime 더 길며, 파일 사이즈가 더 큼
  - Cadence tool Liberate를 통해 custom cell의 `.lib` 파일을 생성할 수 있음

## `.db` file

- LIB file의 compiled version
- 앞에서 설명한 LIB file과 동일한 정보를 가지고 있다.
- LIB 파일을 가지고 있으면 `.db` 파일로 변환할 수 있음(Synopsys Library Compiler)
- `.db` 파일은 Synopsys의 native format으로, synopsys 툴에서 빠르게 처리 가능
- Synopsys tools(Design Compiler, IC Compiler)에서는 `.db` 파일 사용, Cadence tools(Genus, Innovus)에서는 `.lib` 파일 사용
- `.db` 파일은 binary format으로 바로 읽을 수 없음
- Logic synthesis(Design Compiler, Genus)과 PnR(IC Compiler, Innovus) 툴 사용시 필요함

![synthesis-flow-db]({{site.baseurl}}/assets/images/2024-08-09-various-files-in-physical-design-01/synthesis-flow-db.png){: .align-center}
Synthesis flow에서의 `.db` 파일
{: .custom-caption}

![pnr-flow-db]({{site.baseurl}}/assets/images/2024-08-09-various-files-in-physical-design-01/pnr-flow-db.png){: .align-center}
PnR flow에서의 `.db` 파일
{: .custom-caption}
