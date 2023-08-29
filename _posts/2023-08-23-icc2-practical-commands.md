---
title: "IC Compiler II - Practical Commands"
excerpt: ""
date: 2023-08-23 02:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
---

### `foreach_in_collection`

```tcl
foreach_in_collection CELL [get_lib_cells] {
  echo [get_object_name $CELL]
}
```

### attribute 다루기

object별로 attribute를 가지고 있다. object는 종류(class)별로 다른 attribute를 가지고 있는데, 예를 들어 net 클래스는 length 관련 attribute를 가지고 있지만 다른 클래스는 갖고 있지 않다. 해당 클래스에 어떤 attribute가 있는지는 `list_attributes` 명령어로 확인 가능하다.

- attribute 리스트보기: `list_attributes -class <class_name>`

### 클래스 종류 보기

```text
prompt> get_defined_attributes -return_classes

annotation_point annotation_shape block bound bound_shape bounding_box btc_region btp_pattern budget_clock budget_path_type budget_pin budget_pin_constraint budget_pin_data budget_segment bundle bus_buffer_array busplan_bus busplan_element category_node category_tree cell cell_array_pattern cell_bus clock clock_balance_group clock_group clock_group_group clock_path comment constraint_group core_area corner density_rule design design_rule drc_error drc_error_data drc_error_type early_data_check_record eco_bus_buffer_pattern edit_group ems_check ems_database ems_message ems_rule exception exception_group failsafe_fsm_group failsafe_fsm_rule fill_cell geo_mask grid group gui_annotation gui_object input_delay io_guide io_ring keepout_margin layer lib lib_cell lib_pin lib_timing_arc manufacturing_shape matching_type mismatch_object mismatch_type mode module multisource_sink_group net net_bus net_estimation_rule output_delay overlap_blockage parasitic_tech path_group pg_region pin pin_blockage pin_bus pin_constraint pin_guide placement_attraction placement_blockage poly_rect port port_bus power_domain power_state_group power_strategy power_switch_pattern pr_rule retention_element_list routing_blockage routing_corridor routing_corridor_shape routing_guide routing_rule rp_blockage rp_group rtl_statement safety_core_group safety_core_rule safety_register_group safety_register_rule scan_chain scenario shape shape_pattern shaping_blockage shaping_channel shaping_constraint site_array site_def site_row skew_group stub_chain supernet supply_net supply_port supply_set tap tech tech_purpose terminal timing_arc timing_path timing_point topological_constraint topology_edge topology_group topology_node topology_plan topology_repeater track utilization_config variation via via_def via_ladder via_matrix via_region via_rule virtual_connection voltage_area voltage_area_rule voltage_area_shape
```

### `lib_cell`의 attribute

명령어: `list_attribute -application -class lib_cell`

```tcl
icc2_shell> list_attribute -application -class lib_cell
****************************************
Report : List of Attribute Definitions
Version: R-2020.09-SP3
Date   : Wed Aug 23 02:52:20 2023
****************************************

Properties:
    A - Application-defined
    U - User-defined
    I - Importable from design/library (for user-defined)
    S - Settable
    B - Subscripted

Attribute Name            Object     Type       Properties  Constraints
--------------------------------------------------------------------------------
allowable_orientation     lib_cell   orientations
                                                A,S         
always_on                 lib_cell   boolean    A           
antenna_diode_type        lib_cell   string     A           
area                      lib_cell   area       A           
async_force_00            lib_cell   boolean    A,S         
async_force_01            lib_cell   boolean    A,S         
async_force_10            lib_cell   boolean    A,S         
base_name                 lib_cell   string     A           
basic_ff_function_id      lib_cell   string     A,S         
bbox                      lib_cell   rect       A           
block                     lib_cell   collection A           
block_name                lib_cell   string     A           
boundary                  lib_cell   coord_list A,S         
boundary_bbox             lib_cell   rect       A           
boundary_bounding_box     lib_cell   collection A           
bounding_box              lib_cell   collection A           
buf_inv_counts            lib_cell   int        U,S         
cell_footprint            lib_cell   string     A,S         
cell_leakage_power        lib_cell   float      A,S         
clock_gating_integrated_cell
                          lib_cell   string     A           
cmog_class                lib_cell   string     A           
core_area_area            lib_cell   double     A           
core_area_bbox            lib_cell   rect       A           
core_area_boundary        lib_cell   coord_list A           
core_area_bounding_box    lib_cell   collection A           
design                    lib_cell   collection A           
design_name               lib_cell   string     A           
design_type               lib_cell   string     A,S         3dic, abstract, analog, analysis, black_box, bridge, corner, cover, die, diode, end_cap, feedthrough, fill, filler, flip_chip_driver, flip_chip_pad, interposer, lib_cell, macro, module, pad, pad_spacer, rtl, substrate, tsv, vib, well_tap
device_group              lib_cell   string     A,S         
device_group_type         lib_cell   string     A,S         
disable_timing            lib_cell   boolean    A           
dont_touch                lib_cell   boolean    A,S         
dont_use                  lib_cell   boolean    A,S         
ebst_processed            lib_cell   int        A,S         
ebst_valid                lib_cell   boolean    A,S         
excluded_purposes         lib_cell   string     A           
extended_name             lib_cell   string     A           
extended_series_parallel  lib_cell   boolean    A,S         
flip_flop_type            lib_cell   string     A,S         
frame_update              lib_cell   boolean    A,S         
full_name                 lib_cell   string     A           
function_class            lib_cell   string     A           
has_created_deg_copy      lib_cell   boolean    A,S         
has_multi_power_rails     lib_cell   boolean    A           
has_pin_internal_power    lib_cell   boolean    A,S         
has_timing_model          lib_cell   boolean    A           
has_tristate_outputs      lib_cell   boolean    A,S         
height                    lib_cell   distance   A           
ignore_dont_use           lib_cell   string     A,S         
included_purposes         lib_cell   string     A           
inner_keepout_margin_hard lib_cell   margin_list
                                                A,S         
inner_keepout_margin_hard_macro
                          lib_cell   margin_list
                                                A,S         
inner_keepout_margin_route_blockage
                          lib_cell   margin_list
                                                A,S         
inner_keepout_margin_route_blockage_layers
                          lib_cell   collection A           
inner_keepout_margin_soft lib_cell   margin_list
                                                A,S         
input_voltage_range       lib_cell   string     A           
interface_timing          lib_cell   boolean    A,S         
internal_power_derate_factor
                          lib_cell   float      A           
is_a_flip_flop            lib_cell   boolean    A,S         
is_a_flip_flop_bank       lib_cell   boolean    A,S         
is_a_latch                lib_cell   boolean    A,S         
is_a_test_cell            lib_cell   boolean    A,S         
is_ao_cell_without_primary_pg_pin
                          lib_cell   boolean    A           
is_black_box              lib_cell   boolean    A           
is_buffer                 lib_cell   boolean    A           
is_combinational          lib_cell   boolean    A           
is_compressed             lib_cell   boolean    A,S         
is_current                lib_cell   boolean    A           
is_diff_level_shifter     lib_cell   boolean    A           
is_diode_cell             lib_cell   boolean    A           
is_enable_level_shifter   lib_cell   boolean    A           
is_etm_moded_cell         lib_cell   boolean    A           
is_extractable            lib_cell   boolean    A,S         
is_fall_edge_triggered    lib_cell   boolean    A           
is_filler_cell            lib_cell   boolean    A,S         
is_hierarchical           lib_cell   boolean    A           
is_implicit               lib_cell   boolean    A           
is_integrated_clock_gating_cell
                          lib_cell   boolean    A           
is_inverter               lib_cell   boolean    A           
is_isolation              lib_cell   boolean    A           
is_level_shifter          lib_cell   boolean    A           
is_linked                 lib_cell   boolean    A           
is_lssd_cell              lib_cell   boolean    A,S         
is_mask_shiftable         lib_cell   boolean    A,S         
is_memory_cell            lib_cell   boolean    A           
is_modified               lib_cell   boolean    A           
is_multi_seqgen           lib_cell   boolean    A,S         
is_multibit               lib_cell   boolean    A,S         
is_mux                    lib_cell   boolean    A           
is_negative_level_sensitive
                          lib_cell   boolean    A           
is_no_enable              lib_cell   boolean    A,S         
is_open                   lib_cell   boolean    A           
is_port_punching_ok       lib_cell   boolean    A,S         
is_positive_level_sensitive
                          lib_cell   boolean    A           
is_power_switch           lib_cell   boolean    A           
is_remote                 lib_cell   boolean    A           
is_renesas_cell           lib_cell   boolean    A,S         
is_repeater               lib_cell   boolean    A           
is_retention              lib_cell   boolean    A           
is_rise_edge_triggered    lib_cell   boolean    A           
is_sequential             lib_cell   boolean    A           
is_shared_timing_model    lib_cell   boolean    A           
is_soi                    lib_cell   boolean    A           
is_three_state            lib_cell   boolean    A           
keepouts                  lib_cell   collection A           
label_name                lib_cell   string     A           
last_modified             lib_cell   string     A           
last_saved                lib_cell   string     A           
leakage_power_derate_factor
                          lib_cell   float      A           
level_shifter_type        lib_cell   string     A           
lib                       lib_cell   collection A           
lib_name                  lib_cell   string     A           
libgen_usage_class        lib_cell   string     A,S         
macro_density             lib_cell   density_list
                                                A           
mask_shift_layers         lib_cell   collection A           
master_timing_model       lib_cell   collection A           
mb_bussed_ports           lib_cell   string     A           
mb_pin_map                lib_cell   string     A           
mb_scan_chain_ports       lib_cell   string     A           
multibit_width            lib_cell   int        A           
name                      lib_cell   string     A           
new_pg_pin_syntax         lib_cell   boolean    A,S         
number_of_pins            lib_cell   int        A           
object_class              lib_cell   string     A           
object_id                 lib_cell   string     A           
off_state_leakage         lib_cell   double     A,S         
on_state_resistance       lib_cell   double     A,S         
open_count                lib_cell   int        A           
open_mode                 lib_cell   string     A           
outer_keepout_boundary    lib_cell   coord_list A           
outer_keepout_margin_hard lib_cell   margin_list
                                                A,S         
outer_keepout_margin_hard_macro
                          lib_cell   margin_list
                                                A,S         
outer_keepout_margin_route_blockage
                          lib_cell   margin_list
                                                A,S         
outer_keepout_margin_route_blockage_layers
                          lib_cell   collection A           
outer_keepout_margin_soft lib_cell   margin_list
                                                A,S         
output_voltage_range      lib_cell   string     A           
pad_cell                  lib_cell   boolean    A           
pass_rate                 lib_cell   double     A           -1 to 1
pin_unused_in_cell        lib_cell   boolean    A,S         
power_switch_type         lib_cell   string     A           
previous_effective_target_usage
                          lib_cell   double     U,S         
psc_type_id               lib_cell   int        A,S         
read_in_by                lib_cell   string     A,S         
reference_orientation     lib_cell   string     A,S         R0, R90, R180, R270, MX, MXR90, MY, MYR90
restore                   lib_cell   boolean    A,S         
retention_cell_style      lib_cell   string     A           
save                      lib_cell   boolean    A,S         
sb_degen_fid              lib_cell   string     A           
sb_port_cnt               lib_cell   int        A           
scan_element              lib_cell   boolean    A           
shared_timing_models      lib_cell   collection A           
single_bit_degenerate     lib_cell   string     A           
site_def                  lib_cell   collection A           
site_name                 lib_cell   string     A           
specifc_to_formal_seqgen_is_inherited
                          lib_cell   boolean    A,S         
sub_design_type           lib_cell   string     A,S         
subset_name               lib_cell   string     A,S         
switch_list               lib_cell   string     A           
switching_power_derate_factor
                          lib_cell   float      A           
syn_library               lib_cell   boolean    A,S         
target_boundary_area      lib_cell   area       A,S         
target_utilization        lib_cell   float      A,S         
test_normal_func_id       lib_cell   string     A           
threshold_voltage_group   lib_cell   string     A,S         
threshold_voltage_group_type
                          lib_cell   string     A           
top_module                lib_cell   collection A           
top_module_name           lib_cell   string     A,S         
type                      lib_cell   int        A,S         
ufc_created_by_lc         lib_cell   boolean    A,S         
unique_cell_number        lib_cell   int        A,S         
unique_cell_prefix        lib_cell   string     A,S         
unique_net_number         lib_cell   int        A,S         
unique_net_prefix         lib_cell   string     A,S         
use_for_size_only         lib_cell   boolean    A           
use_frame_blockage        lib_cell   boolean    A,S         
user_function_class       lib_cell   string     A,S         
user_power_group          lib_cell   string     A           
valid_purposes            lib_cell   string     A           
via0_alignment            lib_cell   boolean    A,S         
via_ladder_pin_order      lib_cell   collection A,S         
via_regions               lib_cell   collection A           
view_name                 lib_cell   string     A           , abstract, conn, design, frame, layout, outline, rtl, timing
width                     lib_cell   distance   A           
```

### `clock_gating_integrated_cell` 찾기

```tcl
foreach_in_collection CELL [get_lib_cells -filter "defined(clock_gating_integrated_cell)"] {
  echo [get_attribute -name clock_gating_integrated_cell $CELL]
}
```

### Filter Expression 사용하기

- Reference:「Using Tcl With Synopsys Tools」

filter expression은 named attribute(e.g. `area`, `pin_direction`)을 값과 비교한다.  

- `left_side relational_operator right_side`

예를 들면, 아래의 filter expression은 area attribute가 12보다 작은 hierarchical object들을 의미한다.

- `{is_hierarchical == true && area < 12}`

#### Oprators

- `==`, `!=`, `<`, `<=`, `>`, `>=`: 일반적인 프로그램 언어와 유사
- `a =~ b`: (string 연산) a가 b 패턴에 매치되면 1, 아니면 0
- `a !~ b`: (string 연산) a가 b 패턴에 매치되지 않으면 1, 아니면 0
- `defined(a)`: a가 정의되어 있으면 `true`, 아니면 `false`
- `undefined(a)`: a가 정의되어 있지 않으면 `true`, 아니면 `false`
- `sizeof(attribute)`: attribute의 collection 크기(integer), attribute는 collection 타입이어야 함.

```tcl
filter_collection $cells \
  {is_hierarchical == true && dont_touch == true} ; # Static expression

filter_collection $cells \
  "is_hierarchical == true && ref_name == $my_design" ; # Var substitution

set my_filter "is_hierarchical == true && ref_name == $my_design"
filter_collection $cells $my_filter

# right_side에 속성을 주고 싶은 경우 @를 사용

filter_collection $my_pins "actual_rise_transition_max > @constraining_max_transition"
```

### clock gating cell들 가져오기

```tcl
set cg_cells {}

foreach_in_collection LIB_CELL [get_lib_cells -filter "defined(clock_gating_integrated_cell)"] {
  append_to_collection cg_cells [get_cells -of_objects $LIB_CELL]
}

echo [sizeof_collection $cg_cells]
```

### Collection 다루기

```tcl
# first cell
set cg_cell [index_collection $cg_cells 0]
```

```tcl
foreach_in_collection cg_cell $cg_cells {
  echo [get_attribute -name leakage_power $cg_cell]
}
```

```tcl
# fanout
foreach_in_collection cg_cell $cg_cells {
  echo [sizeof_collection [all_fanout -from [get_pins -of_objects $cg_cell -filter "name =~ Q"] -flat]]
}
```

### `get_timing_paths`

Worst Timing Path 얻기

```tcl
get_timing_paths -to [get_pins -of_objects $cg_cells -filter "name =~ EN"] -nworst 5
```

timing path의 attribute list 보기

```tcl
list_attributes -app -class timing_path
```

timing path의 점들 보기

```tcl
get_attribute -name points [get_timing_paths]
```

### Clock Gate Pin 찾기

```tcl
# types: clock, gated_clock, enable, test, observation
get_clock_gate_pins -type gated_clock -of_objects [get_clock_gates]
```

### `get_clock_gates`

`get_clock_gates`로 쉽게 clock gate cell들을 가져올 수 있는 것 같다.

```tcl
foreach_in_collection cg_cell [get_clock_gates] {
  set name [get_object_name $cg_cell]
  set clk_pin [get_clock_gate_pins -type clock -of_objects $cg_cell]
  set gclk_pin [get_clock_gate_pins -type gated_clock -of_objects $cg_cell]
  set en_pin [get_clock_gate_pins -type enable -of_objects $cg_cell]
  set cg_slack [get_attribute -name max_slack $en_pin]
  set gclk_fanout [sizeof_collection [all_fanout -from $gclk_pin -flat]]
  puts "$name, slack: $cg_slack, fanout: $gclk_fanout"
}
```

### GUI에서 select

`change_selection [collection]`

```tcl
# clock gate cell들 선택
change_selection [get_clock_gates]

# 특정 clock gate의 fanout cell들 선택
change_selection [all_fanout -from [get_clock_gate_pins -type gated_clock -of_objects [get_cells uut_2/clk_gate_zero3_reg/latch]] -flat]

# 특정 clock gate의 fanin cell들 선택
change_selection [all_fanin -to [get_clock_gate_pins -type enable -of_objects [get_cells uut_2/clk_gate_zero3_reg/latch]] -flat]
```

### clock gate의 fanout pin 위치 저장

```tcl
set cg_pin [get_clock_gate_pins -type gated_clock -of_objects [get_cells uut_2/clk_gate_zero3_reg/latch]]
set all_cg_fanout [all_fanout -from $cg_pin -flat]
set all_cg_fanout [remove_from_collection $all_cg_fanout $cg_pin]

set fo [open "cg_fanout.csv" "w"]

puts $fo "name,x1,y1,x2,y2"

foreach_in_collection p $all_cg_fanout {
  set pn [get_object_name $p]
  set bb [get_attribute -name bbox $p]
  set x1 [lindex [lindex $bb 0] 0]
  set y1 [lindex [lindex $bb 0] 1]
  set x2 [lindex [lindex $bb 1] 0]
  set y2 [lindex [lindex $bb 1] 1]
  puts $fo "$pn,$x1,$y1,$x2,$y2"
}

close $fo
```

### 살펴볼 Commands

- `place_eco_cells`
- `eco_netlist`

## Split ICG Cell

```tcl
proc save_icg_fanout_pins {icg_name} {
  set icg_cell [get_cells $icg_name]
  set icg_pin [get_clock_gate_pins -type gated_clock -of_objects $icg_cell]
  set all_icg_fanout [all_fanout -from $icg_pin -flat]
  set all_icg_fanout [remove_from_collection $all_icg_fanout $icg_pin]

  set fo [open "icg_fanout_pins.csv" "w"]

  puts $fo "name,x1,y1,x2,y2"

  foreach_in_collection p $all_icg_fanout {
    set pn [get_object_name $p]
    set bb [get_attribute -name bbox $p]
    set x1 [lindex [lindex $bb 0] 0]
    set y1 [lindex [lindex $bb 0] 1]
    set x2 [lindex [lindex $bb 1] 0]
    set y2 [lindex [lindex $bb 1] 1]
    puts $fo "$pn,$x1,$y1,$x2,$y2"
  }

  close $fo
}

proc create_split_icg_cell {icg_name split_idx} {
  set icg_lib_cell [get_cells $icg_name]
  set split_file_name "icg_split_${idx}.txt"
  puts "Reading: $split_file_name"
  create_cell $split_cg_cell_name $cg_cell_lib_name
}

set n_split 2

set cg_cell [get_cells uut_2/clk_gate_zero3_reg/latch]
set cg_cell_name [get_object_name $cg_cell]
set cg_lib_cell_name [get_object_name [get_lib_cell -of_objects $cg_cell]]

set split_cg_cell_name "${cg_cell_name}_split"

for {set idx 1} {$idx <= $n_split} {incr idx} {
  set split_file_name "icg_split_${idx}.txt"
  puts "Reading: $split_file_name"

  # create split cell
  

}


# save fanout pins
save_icg_fanout_pins $cg_cell_name

# get split cell
set split_cg_cell [get_lib_cells -lib $cg_cell_lib_name -name $split_cg_cell_name]

# get pins
set cg_cell_pins [get_pins -of_objects $cg_cell -filter "name =~ Q"]
set split_cg_cell_pins [get_pins -of_objects $split_cg_cell -filter "name =~ Q"]

# connect pins
foreach_in_collection cg_cell_pin $cg_cell_pins {
  set split_cg_cell_pin [index_collection $split_cg_cell_pins [get_attribute -name index $cg_cell_pin]]
  connect_pin $cg_cell_pin $split_cg_cell_pin
}

# remove clock
remove_clock -of_objects $split_cg_cell_pins

# save
save_lib -lib $cg_cell_lib_name
```

### Highlight

```tcl
# Clock Gate 빨간 색으로
gui_change_highlight -color blue -collection [get_cells uut_2/clk_gate_k_reg/latch*]
# The allowed colors: blue, light_blue, yellow, purple, light_purple, orange, light_orange, red, light_red, green, light_green

# Clock Gate의 output net 노란색으로
gui_change_highlight -color yellow -collection [get_nets -of_objects [all_fanout -from [get_clock_gate_pins -type gated_clock -of_objects [get_cells uut_2/clk_gate_k_reg/latch*]] -flat]]

# Clock input path
change_selection [get_attribute -name capture_clock_paths [get_timing_paths -to [get_cells uut_2/clk_gate_k_reg/latch*] -path_type full_clock]]
```

### Clock Path 정보 가져오기

```tcl
# capture_clock_path 가져오기
get_attribute -name capture_clock_paths [get_timing_paths -to [get_cells uut_2/clk_gate_k_reg/latch*] -path_type full_clock]

# clock_path의 속성 리스트 확인
list_attributes -app -class clock_path

proc clock_paths_to {cell_name} {
  set clock_paths {}
  foreach_in_collection cell [get_cells $cell_name] {
    set clock_path [get_attribute -name capture_clock_paths [get_timing_paths -to $cell -path_type full_clock]]
    set clock_paths [add_to_collection $clock_paths $clock_path]
  }
  return $clock_paths
}

change_selection [clock_paths_to uut_2/clk_gate_k_reg/latch*]
gui_change_highlight -color light_blue -collection [clock_paths_to uut_2/clk_gate_k_reg/latch*]
```

### Attribute 내용 확인

```tcl
# 선택한 개체의 attribute 확인
report_attributes -app -compact [get_selection]

# 선택한 개체의 클래스 확인
get_attribute -name object_class [get_selection]
```

## Flow

```tcl
# library setup
set TOP_MODULE "ecg"
set DESIGN_LIB "02_pnr_out"
set TECH_FILE "/home/users/jayoung/02_LIB/SAED14nm/EDK/tech/milkyway/saed14nm_1p9m_mw.tf"
set REF_LIBS "\
/home/users/jayoung/SIMPLE_RUNCAD/00_REF/SAED14_CLIB/saed14hvt_ss0p72v125c.ndm \
/home/users/jayoung/SIMPLE_RUNCAD/00_REF/SAED14_CLIB/saed14io_ss0p72v125c_1p62v.ndm \
/home/users/jayoung/SIMPLE_RUNCAD/00_REF/SAED14_CLIB/saed14lvt_ss0p72v125c.ndm \
/home/users/jayoung/SIMPLE_RUNCAD/00_REF/SAED14_CLIB/saed14pll_ss0p72v125c.ndm \
/home/users/jayoung/SIMPLE_RUNCAD/00_REF/SAED14_CLIB/saed14rvt_ss0p72v125c.ndm \
/home/users/jayoung/SIMPLE_RUNCAD/00_REF/SAED14_CLIB/saed14slvt_ss0p72v125c.ndm"
set TARGET_LIBRARY_FILES "\
/home/users/jayoung/02_LIB/SAED14nm/EDK/lib/stdcell_hvt/db_nldm/saed14hvt_ss0p72v125c.db \
/home/users/jayoung/02_LIB/SAED14nm/EDK/lib/stdcell_rvt/db_nldm/saed14rvt_ss0p72v125c.db \
/home/users/jayoung/02_LIB/SAED14nm/EDK/lib/stdcell_lvt/db_nldm/saed14lvt_ss0p72v125c.db \
/home/users/jayoung/02_LIB/SAED14nm/EDK/lib/stdcell_slvt/db_nldm/saed14slvt_ss0p72v125c.db \
/home/users/jayoung/02_LIB/SAED14nm/EDK/lib/io_std/db_nldm/saed14io_fc_ss0p72v125c_1p62v.db \
/home/users/jayoung/02_LIB/SAED14nm/EDK/lib/io_std/db_nldm/saed14io_wb_ss0p72v125c_1p62v.db \
/home/users/jayoung/02_LIB/SAED14nm/EDK/lib/pll/logic_synth/saed14pll_ss0p72v125c.db"
set TLUPLUS_FILE "/home/users/jayoung/02_LIB/SAED14nm/EDK/tech/star_rc/max/saed14nm_1p9m_Cmax.tluplus"

if { ![file exists ${TOP_MODULE}.sdc] } {
  puts "Error: ${TOP_MODULE}.sdc does not exist"
  exit
}

if { ![file exists ${TOP_MODULE}.netlist.v] } {
  puts "Error: ${TOP_MODULE}.netlist.v does not exist"
  exit
}

# design setup
if { ![file exists ${DESIGN_LIB}] } {
  create_lib $DESIGN_LIB -tech $TECH_FILE -ref_libs $REF_LIBS
} else {
  open_lib $DESIGN_LIB
}

set target_library ${TARGET_LIBRARY_FILES}
set link_library "* $target_library"

read_verilog ${TOP_MODULE}.netlist.v
link_block

# timing setup
set_technology -node s14
create_mode on
create_corner typical
create_scenario -name on_typical -mode on -corner typical

read_parasitic_tech -tlup $TLUPLUS_FILE -name maxTLU
set_parasitic_parameters -early_spec maxTLU -late_spec maxTLU

set_temperature 125
set_process_number 1
set_voltage 0.72 -object_list VDD
set_voltage 0.0 -object_list VSS

source ${TOP_MODULE}.sdc

# app options setup
set_app_options -name place.coarse.auto_density_control -value true
set_app_options -name place.coarse.continue_on_missing_scandef -value true
set_app_options -name place_opt.flow.trial_clock_tree -value true

# floorplan
initialize_floorplan
save_block -as 00_floorplan

# place_coarse
create_placement -floorplan
place_pins -self
save_block -as 01_place_coarse

# place_opt
place_opt -from initial_place -to initial_place
save_block -as 02_place_opt_01_initial_place
report_qor -significant_digits 6 > 02_place_opt_01_initial_place.qor.rpt

place_opt -from initial_drc -to initial_drc
save_block -as 02_place_opt_02_initial_drc
report_qor -significant_digits 6 > 02_place_opt_02_initial_drc.qor.rpt

place_opt -from initial_opto -to initial_opto
save_block -as 02_place_opt_03_initial_opto
report_qor -significant_digits 6 > 02_place_opt_03_initial_opto.qor.rpt

place_opt -from final_place -to final_place
save_block -as 02_place_opt_04_final_place
report_qor -significant_digits 6 > 02_place_opt_04_final_place.qor.rpt

place_opt -from final_opto -to final_opto
save_block -as 02_place_opt_05_final_opto
report_qor -significant_digits 6 > 02_place_opt_05_final_opto.qor.rpt
```

## Toggle Rate

단위 시간(1ns) 당 Switching 횟 수. Clock Period가 600ps인 Clock Net의 경우 1ns 당 3.333번 Switching이 발생하므로 Toggle Rate는 3.33이 된다.

### Redirect

```tcl
set log_file [open "log.txt" a]
proc myeval {cmd} {
 puts "Executing: $cmd"
 uplevel 1 "redirect -tee -channel $::log_file {$cmd}"
}

set world 100
myeval {puts "HELLO, $world"}

set world 22
myeval {puts "HELLO, $world"}

myeval {query_objects [get_nets]}
myeval {report_timing -sig 6}
```

### Get power value

```tcl
set leakage_power [get_attribute -name leakage_power [current_block]]
set switching_power [get_attribute -name switching_power [current_block]]
set internal_power [get_attribute -name internal_power [current_block]]

set total_power [expr $leakage_power + $switching_power + $internal_power]
```

### formatted 현재 시간

`set clock_str [clock format [clock seconds] -format %y%m%d_%H%M%S]`

### fanin power sum

```tcl
proc calculate_icg_fanin_power {icg} {
  set icg_en_pin [get_clock_gate_pins -type enable -of_objects $icg]
  set icg_fanin_pins [all_fanin -to $icg_en_pin -flat]
  set icg_fanin_cells [get_cells -of_objects $icg_fanin_pins]
  set total_power 0.0
  foreach_in_collection cell $icg_fanin_cells {
    set cell_power [get_attribute -name total_power $cell]
    set total_power [expr $total_power + $cell_power]
  }
  return $total_power
}
```

### Sort

```tcl
lsort -real -index 1 -decreasing {"First 24.1" "Second 18.9" "Third 30.5"}
# {Third 30.5} {First 24.1} {Second 18.9}
```

### 존재 체크

```tcl
if {[lsearch -exact $myList 4] >= 0} {
    puts "Found 4 in myList!"
}
```

### Clock gate의 fanout 경로 보기

```tcl
change_selection [get_nets -of_objects [all_fanout -flat -from [get_clock_gate_pins -type gated_clock -of_objects [get_clock_gates]]]]
```

### Define User Attributes

```tcl
define_user_attribute -type double -classes cell -name fanin_power
set_attribute -objects [get_clock_gates] -name fanin_power -value 99
report_collection [get_clock_gates] -columns {full_name fanin_power}
```
