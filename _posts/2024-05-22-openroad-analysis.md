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

## `do-floorplan`

### `floorplan.tcl`

```tcl
utl::set_metrics_stage "floorplan__{}"
source $::env(SCRIPTS_DIR)/load.tcl
load_design 1_synth.v 1_synth.sdc

proc report_unused_masters {} {
  set db [ord::get_db]
  set libs [$db getLibs]
  set masters ""
  foreach lib $libs {
    foreach master [$lib getMasters] {
      # filter out non-block masters, or you can remove this conditional to detect any unused master
      if {[$master getType] == "BLOCK"} {
        lappend masters $master
      }
    }
  }

  set block [ord::get_db_block]
  set insts [$block getInsts]

  foreach inst $insts {
    set inst_master [$inst getMaster]
    set masters [lsearch -all -not -inline $masters $inst_master]
  }

  foreach master $masters {
    puts "Master [$master getName] is loaded but not used in the design"
  }
}

report_unused_masters

#Run check_setup
puts "\n=========================================================================="
puts "Floorplan check_setup"
puts "--------------------------------------------------------------------------"
check_setup

set num_instances [llength [get_cells -hier *]]
puts "number instances in verilog is $num_instances"

set additional_args ""
if { [info exists ::env(ADDITIONAL_SITES)]} {
  append additional_args " -additional_sites $::env(ADDITIONAL_SITES)"
}

# Initialize floorplan by reading in floorplan DEF
# ---------------------------------------------------------------------------
if {[info exists ::env(FLOORPLAN_DEF)]} {
    puts "Read in Floorplan DEF to initialize floorplan:  $env(FLOORPLAN_DEF)"
    read_def -floorplan_initialize $env(FLOORPLAN_DEF)
# Initialize floorplan using ICeWall FOOTPRINT
# ----------------------------------------------------------------------------
} elseif {[info exists ::env(FOOTPRINT)]} {

  ICeWall load_footprint $env(FOOTPRINT)

  initialize_floorplan \
    -die_area  [ICeWall get_die_area] \
    -core_area [ICeWall get_core_area] \
    -site      $::env(PLACE_SITE)

  ICeWall init_footprint $env(SIG_MAP_FILE)

# Initialize floorplan using CORE_UTILIZATION
# ----------------------------------------------------------------------------
} elseif {[info exists ::env(CORE_UTILIZATION)] && $::env(CORE_UTILIZATION) != "" } {
  set aspect_ratio 1.0
  if {[info exists ::env(CORE_ASPECT_RATIO)] && $::env(CORE_ASPECT_RATIO) != ""} {
    set aspect_ratio $::env(CORE_ASPECT_RATIO)
  }
  set core_margin 1.0
  if {[info exists ::env(CORE_MARGIN)] && $::env(CORE_MARGIN) != ""} {
    set core_margin $::env(CORE_MARGIN)
  }
  initialize_floorplan -utilization $::env(CORE_UTILIZATION) \
                       -aspect_ratio $aspect_ratio \
                       -core_space $core_margin \
                       -site $::env(PLACE_SITE) \
                       {*}$additional_args

# Initialize floorplan using DIE_AREA/CORE_AREA
# ----------------------------------------------------------------------------
} else {
  initialize_floorplan -die_area $::env(DIE_AREA) \
                       -core_area $::env(CORE_AREA) \
                       -site $::env(PLACE_SITE) \
                       {*}$additional_args
}

if { [info exists ::env(MAKE_TRACKS)] } {
  source $::env(MAKE_TRACKS)
} elseif {[file exists $::env(PLATFORM_DIR)/make_tracks.tcl]} {
  source $::env(PLATFORM_DIR)/make_tracks.tcl
} else {
  make_tracks
}

if {[info exists ::env(FOOTPRINT_TCL)]} {
  source $::env(FOOTPRINT_TCL)
}

# remove buffers inserted by yosys/abc
remove_buffers

##### Restructure for timing #########
if { [info exist ::env(RESYNTH_TIMING_RECOVER)] && $::env(RESYNTH_TIMING_RECOVER) == 1 } {
  repair_design
  repair_timing
  # pre restructure area/timing report (ideal clocks)
  puts "Post synth-opt area"
  report_design_area
  report_worst_slack -min -digits 3
  puts "Post synth-opt wns"
  report_worst_slack -max -digits 3
  puts "Post synth-opt tns"
  report_tns -digits 3

  write_verilog $::env(RESULTS_DIR)/2_pre_abc_timing.v

  restructure -target timing -liberty_file $::env(DONT_USE_SC_LIB) \
              -work_dir $::env(RESULTS_DIR)

  write_verilog $::env(RESULTS_DIR)/2_post_abc_timing.v

  # post restructure area/timing report (ideal clocks)
  remove_buffers
  repair_design
  repair_timing

  puts "Post restructure-opt wns"
  report_worst_slack -max -digits 3
  puts "Post restructure-opt tns"
  report_tns -digits 3

  # remove buffers inserted by optimization
  remove_buffers
}


puts "Default units for flow"
report_units
report_units_metric
report_metrics 2 "floorplan final" false false

if { [info exist ::env(RESYNTH_AREA_RECOVER)] && $::env(RESYNTH_AREA_RECOVER) == 1 } {

  utl::push_metrics_stage "floorplan__{}__pre_restruct"
  set num_instances [llength [get_cells -hier *]]
  puts "number instances before restructure is $num_instances"
  puts "Design Area before restructure"
  report_design_area
  report_design_area_metrics
  utl::pop_metrics_stage

  write_verilog $::env(RESULTS_DIR)/2_pre_abc.v

  set tielo_cell_name [lindex $env(TIELO_CELL_AND_PORT) 0]
  set tielo_lib_name [get_name [get_property [lindex [get_lib_cell $tielo_cell_name] 0] library]]
  set tielo_port $tielo_lib_name/$tielo_cell_name/[lindex $env(TIELO_CELL_AND_PORT) 1]

  set tiehi_cell_name [lindex $env(TIEHI_CELL_AND_PORT) 0]
  set tiehi_lib_name [get_name [get_property [lindex [get_lib_cell $tiehi_cell_name] 0] library]]
  set tiehi_port $tiehi_lib_name/$tiehi_cell_name/[lindex $env(TIEHI_CELL_AND_PORT) 1]

  restructure -liberty_file $::env(DONT_USE_SC_LIB) -target "area" \
        -tiehi_port $tiehi_port \
        -tielo_port $tielo_port \
        -work_dir $::env(RESULTS_DIR)

  # remove buffers inserted by abc
  remove_buffers

  write_verilog $::env(RESULTS_DIR)/2_post_abc.v
  utl::push_metrics_stage "floorplan__{}__post_restruct"
  set num_instances [llength [get_cells -hier *]]
  puts "number instances after restructure is $num_instances"
  puts "Design Area after restructure"
  report_design_area
  report_design_area_metrics
  utl::pop_metrics_stage
}

if { [info exists ::env(POST_FLOORPLAN_TCL)] } {
  source $::env(POST_FLOORPLAN_TCL)
}

if {[info exists ::env(GALLERY_REPORT)]  && $::env(GALLERY_REPORT) != 0} {
  write_def $::env(RESULTS_DIR)/2_1_floorplan.def
}
write_db $::env(RESULTS_DIR)/2_1_floorplan.odb
write_sdc -no_timestamp $::env(RESULTS_DIR)/2_floorplan.sdc
```
