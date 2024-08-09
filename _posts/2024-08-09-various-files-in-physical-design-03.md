---
title: "Ch3. DEF file"
excerpt: "Various files in Physical Design"
date: 2024-08-09 23:00:00 +0900
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
7. **.def file**
8. **.sdc file**
9. **.mw file**
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

## `.def` file

- DEF stands for "Design Exchange Format" file
- DEF file은 IC의 **physical layout**을 나타내기 위해 사용되며, ASCII 형식임
- LEF 파일과 연관되어 있고, physical design을 display하기 위해 두 파일 모두 필요함
- Cadence Design System에서 개발됨
- P&R tool에서 생성되며, esxtraction tool이나 power analysis tool 같은 post analysis tool의 input으로 사용됨
- DEF file은 design-specific information of circuits과 Physical design의 어느 한 시점의 design을 포함함
- DEF는 logical design data와 physical design data를 P&R tool로 보내고 받을 때 사용됨
- Logical design data는 internal connectivity(netlist), group information, phjysical constraints를 포함
- Physical data는 component들의 placement location, orientation과 routing geometry 포함
- 표준 DEF 파일은 다음과 같은 Statements와 Sections로 구성됨

## `.def` 내용

- [ VERSION statement ]
- [ DIVIDERCHAR statement ]: hierarchy 표현을 위한 문자(default: '/')
- [ BUSBITCHARS statement ]: bus bits를 표현하기 위한 문자(default: '[]')
- DESIGN statement: design 이름
- [ TECHNOLOGY statement ]: design의 technology 이름
- [ UNITS statement ]: DISTANCE MICRONS conversion_factor
- [ DIEAREA statement ]: DIEAREA ( llx lly ) ( urx ury ); # rect일 경우이며, polygon일 경우는 여러개의 point로 구성됨
- [ ROWS statement ]
  - [ROW rowName siteName origX origY siteOrient]
  - [DO numX BY numY] numX나 numY 중 하나는 반드시 1이어야 함. numY가 1이면 horizontal
  - [STEP stepX stepY]
  - Orientation - North(N), South(S), Flip-North(FN), Flip-South(FS)
- [ TRACKS statement ]
  - TRACKS X혹은Y start DO numTracks STEP space LAYER layerName ;
- [ GCELLGRID statement ]
- [ VIAS statement ]
  - 모든 via는 3 layer의 모양으로 이루어져 있음
  - 1) A cut layer
  - 2) Two routing layers that connect through that cut layer
- [ NONDEFAULTRULES statement ] (NDR)
  - LEF 파일에 포함되어있지 않는 nondefault rule
- [ COMPONENTS section ]
  - design components, their location, associated attributes
  - DEF 파일에서 큰 섹션이다.
  - 컴포넌트 이름, 좌표 등
- [ PINS section ]
  - External pin 정의
  - 각 external pin에 pin name assign 및 internal net name과 연관
  - pin name과 net name은 같을 수도 있다
- [ BLOCKAGES section ]
  - placement와 routing blockage 정의
  - PUSHDOWN: 해당 blockage가 top level design에서부터 pushed down된 blockage임을 나타냄
- [ FILL section ]
- [ SPECIALNETS section ]
- [ NETS section ]
- [ SCANCHAINS section ]
- [ GROUPS section ]
- [ BEGINEXT section ]
- END DESIGN statement

## 툴

ICC command to write DEF file (General format): `write_def -output <filename>`