---
title: "OpenROAD Internals"
excerpt: "OpenROAD Internals"
date: 2024-05-22 01:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
---

## 1. Flow

OpenROAD의 Flow는 `flow/Makefile` 파일에 정의되어 있다.

순서는 다음과 같다.

* `do-yosys`
  * `tools/install/yosys/bin/yosys`
  * output: `1_synth.v`, `1_synth.sdc`
* `do-floorplan`
  1. `floorplan.tcl`: STEP 1: Translate verilog to odb, `2_1_floorplan`
  2. `io_place_random.tcl`: STEP 2: IO Placement (random), `2_2_floorplan_io`
  3. `tdms_place.tcl`: STEP 3: Timing Driven Mixed Sized Placement, `2_3_floorplan_tdms`
  4. `macro_place.tcl`: STEP 4: Macro Placement, `2_4_floorplan_macro`
    * `floorplan_debug_macros.tcl`: `2_floorplan_debug_macros`
  5. `tapcell.tcl`: STEP 5: Tapcell and Welltie insertion, `2_5_floorplan_tapcell`
  6. `pdn.tcl`: STEP 6: PDN generation, `2_6_floorplan_pdn`
* `do-place`
  1. `global_place_skip_io.tcl`: STEP 1: Global placement without placed IOs, timing-driven, and routability-driven, `3_1_place_gp_skip_io`
  2. `io_placement.tcl`: STEP 2: IO placement (non-random), `3_2_place_iop`
  3. `global_place.tcl`: STEP 3: Global placement with placed IOs, timing-driven, and routability-driven, `3_3_place_gp`
  4. `resize`: STEP 4: Resizing & Buffering, `3_4_place_resize`
* `do-cts`
  1. `cts.tcl`: Run TritonCTS, `4_1_cts`
* `do-route`
  1. `global_route.tcl`: STEP 1: Run global route, `5_1_grt`
  2. `fillcell.tcl`: STEP 2: Filler cell insertion, `5_2_fillcell`
  3. `detail_route.tcl`: STEP 3: Run detailed route, `5_3_route`
* `do-finish`
  1. `density_fill.tcl`: `6_1_fill`
  2. `final_report.tcl`: `6_report`, `6_final.def`, `6_final.v`, `6_final.spef`
