---
title: "Innovus PnR"
excerpt: ""
date: 2024-10-01 01:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

## Innovus PnR

<script src="https://gist.github.com/hvppycoding/65a0da027467d33910b13a27dade3869.js"></script>

## Jekyll

{% highlight tcl linenos %}
##############################################################
##############################################################
#                     Innovus routines                       #
##############################################################
##############################################################

# Sets up config for all LEF/LIB/TCH(/CAPTBL) files and commitConfig
proc prc_Cdnc_setupConfig {  } {
  uplevel \#0 {
    # Suppress warnings related to scaled /memory LEF
    suppressMessage ENCLF-200
    suppressMessage ENCLF-201
    suppressMessage LEFPARS-2043
    suppressMessage LEFPARS-2007
    suppressMessage TECHLIB-459
    suppressMessage TECHLIB-436

    set LEFfiles ""
    # TECHLEF setup
    set techLEF "$::env(TECHLEF_DIR)"
    if { $techLEF != "" } {
      foreach fileName [glob -nocomplain -d $techLEF *lef] {
        append LEFfiles " $fileName "
      }
      ## also include compressed files (ex. *lef.gz)
      foreach fileName [glob -nocomplain -d $techLEF *lef.gz] {
        append LEFfiles " $fileName "
      }
    }

    # MACROLEF setup
    set macroLEF "$::env(MACROLEF_DIR)"
    if { $macroLEF != "" } {
      foreach fileName [glob -nocomplain -d $macroLEF *lef] {
        append LEFfiles " $fileName "
      }
      ## also include compressed files (ex. *lef.gz)
      foreach fileName [glob -nocomplain -d $macroLEF *lef.gz] {
        append LEFfiles " $fileName "
      }
    }

    # HARDMACROLEF setup
    if { [ info exists env(HARDMACROLEF_DIR) ] } {
      set hardmacroLEF "$::env(HARDMACROLEF_DIR)"
      if { $hardmacroLEF != "" } {
        foreach fileName [glob -nocomplain -d $hardmacroLEF *lef] {
          append LEFfiles " $fileName "
        }
        ## also include compressed files (ex. *lef.gz)
        foreach fileName [glob -nocomplain -d $hardmacroLEF *lef.gz] {
          append LEFfiles " $fileName "
        }
      }
    }
    if { $LEFfiles == "" } {
      puts "ERR> At least 1 LEF file should exist."
      exit
    }


    set LIBfiles ""
    # MACROLIB setup
    set macroLIB "$::env(MACROLIB_DIR)"
    if { $macroLIB != "" } {
      foreach fileName [glob -nocomplain -d $macroLIB *lib] {
        append LIBfiles " $fileName "
      }
      ## also include compressed files (ex. *lib.gz)
      foreach fileName [glob -nocomplain -d $macroLIB *lib.gz] {
        append LIBfiles " $fileName "
      }
    }

    # HARDMACROLIB setup
    if { [ info exists env(HARDMACROLIB_DIR) ] } {
      set hardmacroLIB "$::env(HARDMACROLIB_DIR)"
      if { $hardmacroLIB != "" } {
        foreach fileName [glob -nocomplain -d $hardmacroLIB *lib] {
          append LIBfiles " $fileName "
        }
        ## also include compressed files (ex. *lib.gz)
        foreach fileName [glob -nocomplain -d $hardmacroLIB *lib.gz] {
          append LIBfiles " $fileName "
        }
      }
    }
    if { $LIBfiles == "" } {
      puts "ERR> At least 1 LIB file should exist."
      exit
    }

    # SDC setup
    set SDCfile $::env(build_name).sdc

    # netlist setup
    set NETlist $::env(build_name).netlist.v

    setLibraryUnit -time $vars(LibUnit,Time) -cap $vars(LibUnit,Cap)
    set rda_Input(ui_settop) 0
    set rda_Input(ui_leffile) "$LEFfiles"
    set rda_Input(ui_timelib) "$LIBfiles"

    global init
    set init_import_mode {-treatUndefinedCellAsBbox 0 -keepEmptyModule 1 }
    set init_verilog $NETlist
    set init_design_netlisttype {Verilog}
    set init_design_settop {0}
    set init_top_cell {}
    set init_io_file ""
    set init_lef_file "$LEFfiles"
    set init_oa_ref_lib {}
    set init_abstract_view {}
    set init_layout_view {}
    set init_pwr_net "VDD"
    set init_gnd_net "VSS"
    if { [ file exists "viewDefinition.tcl" ] } {
      set init_mmmc_file viewDefinition.tcl
    }
  }
}

proc prc_Cdnc_mmmcConfig { } {

  uplevel \#0 {
    # TCH setup
    set TCHfiles ""
    if { [info exists env(TCH_DIR)] } {
      set TCH "$::env(TCH_DIR)"
      if { $TCH != "" } {
        foreach fileName [glob -nocomplain -d $TCH *tch] {
          append TCHfiles "$fileName"
        }
      }
    }

    #CAPTBL
    set CAPfiles ""
    if { [info exists env(CAPTBL_DIR)] } {
      set CAP "$::env(CAPTBL_DIR)"
      if { $CAP != "" } {
        foreach fileName [glob -nocomplain -d $CAP *captbl] {
          append CAPfiles "$fileName"
        }
        ## also include compressed files (ex. *captbl.gz)
        foreach fileName [glob -nocomplain -d $CAP *captbl.gz] {
          append CAPfiles "$fileName"
        }
      }
    }

    if { $TCHfiles == "" && $CAPfiles == "" } {
      puts "ERR> Either TCH or CAPTBL should be defined."
      exit
    }

    # MMMC constriants
    if { [info exists env(CAPTBL_DIR)] && ![info exists env(TCH_DIR)] } {
      create_rc_corner -name curRC -cap_table "$CAPfiles"
    } elseif { ![info exists env(CAPTBL_DIR)] && [info exists env(TCH_DIR)] } {
      create_rc_corner -name curRC -qx_tech_file "$TCHfiles"
    } elseif { [info exists env(CAPTBL_DIR)] && [info exists env(TCH_DIR)] } {
      create_rc_corner -name curRC -cap_table "$CAPfiles"
      update_rc_corner -name curRC -qx_tech_file "$TCHfiles"
    }
    set scaling 1
    set scaling_tmp1 "$scaling $scaling $scaling"
    set scaling_tmp2 "$scaling"
    update_rc_corner -name curRC -postRoute_cap $scaling_tmp1
    update_rc_corner -name curRC -postRoute_clkcap $scaling_tmp1
    update_rc_corner -name curRC -postRoute_clkres $scaling_tmp1
    update_rc_corner -name curRC -postRoute_res $scaling_tmp1
    update_rc_corner -name curRC -postRoute_xcap $scaling_tmp1
    update_rc_corner -name curRC -preRoute_cap $scaling_tmp2
    update_rc_corner -name curRC -preRoute_clkcap {0}
    update_rc_corner -name curRC -preRoute_clkres {0}
    update_rc_corner -name curRC -preRoute_res $scaling_tmp2
    if { [info exists env(techTemp)] } {
      update_rc_corner -name curRC -T "$::env(techTemp)"
    } else {
      update_rc_corner -name curRC -T {25}
    }
    create_library_set -name curLib -timing "$LIBfiles"
    create_delay_corner -name curDel -library_set curLib -rc_corner curRC
    create_constraint_mode -name curCon -sdc_files "$SDCfile"
    create_analysis_view -name curAna -constraint_mode curCon -delay_corner curDel
  }
}

proc prc_Cdnc_setDelayCalMode { } {
  uplevel \#0 {
    setDelayCalMode	\
      -combine_mmmc early_late	\
      -enable_high_fanout false    \
      -engine aae                   \
      -equivalent_waveform_model_propagation false	\
      -equivalent_waveform_model_type none    \
      -honorSlewPropConstraint true  \
      -reportOutBound false   \
      -socv_accuracy_mode low \
      -socv_lvf_mode early_late	\
      -socv_use_lvf_tables all	\
      -SIAware true
  }
}

# Set P&R modes as required. Change certain options if so desired
proc prc_Enc_setPNRmodes {} {
  uplevel \#0 {
    # Set some design specific variables
    set placeMaxDensity $vars(PNR,placeMaxDensity)
    set maxRouteLayer $vars(PNR,maxRouteLayer)
    set minRouteLayer $vars(PNR,minRouteLayer)
    set maxPinRouteLayer $vars(PNR,maxPinRouteLayer)
    set minPinRouteLayer $vars(PNR,minPinRouteLayer)
    set leakageToDynamicRatio $vars(PNR,leakageToDynamicRatio)
    set iteration $vars(PNR,routingIteration)

    # place mode
    setPlaceMode -reset
    setPlaceMode \
      -maxDensity $placeMaxDensity \
      -placeIoPins true \
      -wireLenOptEffort high

    # trialroute mode
    setTrialRouteMode -reset
    setTrialRouteMode \
      -maxRouteLayer $maxRouteLayer \
      -minRouteLayer $minRouteLayer
    if { $version >= 16 } {
      setRouteMode -earlyGlobalMinRouteLayer $minRouteLayer
    }

    # pin assignment mode
    setPinAssignMode -reset
    setPinAssignMode -maxLayer $maxPinRouteLayer -minLayer $minPinRouteLayer
    setOptMode    -preserveAssertions TRUE

    # setting rule for pin assignment
    set layer_list [list ]
    for {set i $minPinRouteLayer} {$i <= $maxPinRouteLayer} {incr i} {
      lappend layer_list $i
    }
    createPinGroup pinG -pin * -optimizeOrder
    createPinGuide -edge 0 -layer $layer_list -pinGroup pinG
    createPinGuide -edge 1 -layer $layer_list -pinGroup pinG
    createPinGuide -edge 2 -layer $layer_list -pinGroup pinG
    createPinGuide -edge 3 -layer $layer_list -pinGroup pinG

    # delay calculation mode
    setDelayCalMode -reset
    setDelayCalMode \
      -reportOutBound false \
      -SIAware true \
      -engine aae


    setAnalysisMode -reset
    setAnalysisMode \
      -analysisType onChipVariation \
      -cppr both

    # nanoroute mode
    setNanoRouteMode -reset
    setNanoRouteMode \
      -drouteEndIteration $iteration \
      -drouteFixAntenna false \
      -routeBottomRoutingLayer $minRouteLayer \
      -routeTopRoutingLayer $maxRouteLayer \
      -routeUnconnectedPorts true \
      -routeWithTimingDriven false \
      -routewithSiDriven false \
      -routeWithViaOnlyForStandardCellPin true
    #-routeWithViaOnlyForStandardCellPin false

    # optimization mode
    setOptMode -reset
    setOptMode -maxDensity $placeMaxDensity \
      -addInst true \
      -addInstancePrefix OPT \
      -allEndPoints true \
      -deleteInst true \
      -downsizeInst true \
      -effort high \
      -fixCap true \
      -fixTran true \
      -fixFanoutLoad true \
      -leakageToDynamicRatio $leakageToDynamicRatio \
      -moveInst true \
      -optimizeFF true \
      -postRouteAllowOverlap false \
      -powerEffort high \
      -simplifyNetlist true \
      -verbose true
  }
}

proc prc_Cdnc_setSwitchingActivity { } {

  uplevel \#0 {

    # use write_tcf command to track down the switching activity of every net
    set_default_switching_activity -reset
    set_default_switching_activity -duty $vars(SwitchingActivity,DutyRatio) -input_activity $vars(SwitchingActivity,IpToggle) \
      -seq_activity $vars(SwitchingActivity,RegToggle)

    set_switching_activity -duty $vars(SwitchingActivity,DutyRatio) -activity $vars(SwitchingActivity,ClkToggle) \
      -input_port [get_ports -filter {is_clock_used_as_clock==true}]

  }
}


if { ![ file exists $::env(build_name).sdc ] } {
  puts "\n\[INFO\] run directory should have $::env(build_name).sdc\n"; exit
}
if { ![ file exists $::env(build_name).netlist.v ] } {
  puts "\n\[INFO\] run directory should have $::env(build_name).netlist.v\n"; exit
}
if { ![ file exists $::env(build_name).variables.tcl ] } {
  puts "\n\[INFO\] run directory should have $::env(build_name).variables.tcl\n"; exit
}

set version [string range [getVersion] 0 [string first "." [getVersion]]-1]

source $::env(build_name).variables.tcl

if { [info exists vars(Design,cpuNo)] } {
  setMultiCpuUsage -localCpu $vars(Design,cpuNo)
}

suppressMessage ENCPTN-1027
suppressMessage ENCPTN-1520

set tcl_precision 6

#make it resume
set outDir "./data/02_pnr_c"

if { [file exists $outDir/00_Floorplan.enc ] } {
  restoreDesign $outDir/00_Floorplan.enc.dat [file rootname [file tail [glob $outDir/00_Floorplan.enc.dat/*.init]]]
} else {
  prc_Cdnc_setupConfig
  prc_Cdnc_mmmcConfig
  init_design -setup curAna -hold curAna
  set_analysis_view -setup curAna -hold curAna

  setDesignMode -reset
  setDesignMode -process $vars(LibUnit,Process) -flowEffort standard -powerEffort high
  report_resource -verbose

  if { [ file exists "floorplan.def" ] } {
    defIn floorplan.def
  } else {
    floorPlan -$vars(FloorPlan,Type)	\
      $vars(FloorPlan,var1) $vars(FloorPlan,var2) \
      $vars(FloorPlan,LeftMargin) \
      $vars(FloorPlan,BottomMargin) \
      $vars(FloorPlan,RightMargin) \
      $vars(FloorPlan,TopMargin)
  }
  prc_Enc_setPNRmodes
  prc_Cdnc_setDelayCalMode

  if { [ info exists vars(ExtractionEngine,postRoute) ] } {
    if { $vars(ExtractionEngine,postRoute) == "IQRC" } {
      setExtractRCMode -engine postRoute -effortLevel high \
        -coupled true -tQuantusForPostRoute false
    } elseif { $vars(ExtractionEngine,postRoute) == "TQRC" } {
      setExtractRCMode -engine postRoute -effortLevel medium \
        -coupled true -tQuantusForPostRoute true -tQuantusModelFile rc_model.bin
    } else {
      setExtractRCMode -engine postRoute -effortLevel high -coupled true
    }
  } else {
    setExtractRCMode -engine postRoute -effortLevel high -coupled true
  }


  if { [ file exists "config.tcl" ] } { source config.tcl }

  if {$version >= 16} {
    saveDesign $outDir/00_Floorplan.enc -compress -verilog
  } else {
    saveDesign $outDir/00_Floorplan.enc -compress
  }
}

if { [file exists $outDir/01_Placement.enc ] } {
  restoreDesign $outDir/01_Placement.enc.dat [file rootname [file tail [glob $outDir/01_Placement.enc.dat/*.init]]]
} else {
  # Set switching activity
  prc_Cdnc_setSwitchingActivity

  # Set pre-CTS clock uncertainty
  set_interactive_constraint_modes [all_constraint_modes -active]
  set_clock_uncertainty $vars(ClockUncertainty,preCTS) [all_clocks]
  report_resource -verbose

  ## If you do not perform pre-CTS opt.
  if {[info exists vars(PNR,place_opt_design)] && $vars(PNR,place_opt_design) == 0} {
    ## If you use sequential placement -> pre-CTS opt.
    placeDesign -prePlaceOpt

    if {[getPlaceMode -placeIoPins]=="false"} {
      assignPtnPin
      assignIoPins -align
    }

    if {$version >= 16} {
      saveDesign $outDir/01_Placement.enc -compress -verilog
    } else {
      saveDesign $outDir/01_Placement.enc -compress
    }
    report_resource -verbose
  } else {
  }
}

if { [file exists $outDir/02_preCTSOpt.enc ] } {
  restoreDesign $outDir/02_preCTSOpt.enc.dat [file rootname [file tail [glob $outDir/02_preCTSOpt.enc.dat/*.init]]]
} else {
  if { [info exists vars(PNR,place_opt_design)] && $vars(PNR,place_opt_design) == 0} {
  } else {
    ## If you use sequential placement -> pre-CTS opt.
    setOptMode -addInstancePrefix placeOPT_
    place_opt_design
    report_resource -verbose

    prc_Cdnc_setSwitchingActivity
    set_interactive_constraint_modes [all_constraint_modes -active]
    set_clock_uncertainty $vars(ClockUncertainty,CTS) [all_clocks]
    set cts_override_minimum_skew_target true
  }

  if {[getPlaceMode -placeIoPins]=="false"} {
    assignPtnPin
    assignIoPins -align
  }

  if {$version >= 16} {
    saveDesign $outDir/02_preCTSOpt.enc -compress -verilog
  } else {
    saveDesign $outDir/02_preCTSOpt.enc -compress
  }
  report_resource -verbose
}

if { [file exists $outDir/03_ccopt.enc ] } {
  restoreDesign $outDir/03_ccopt.enc.dat [file rootname [file tail [glob $outDir/03_ccopt.enc.dat/*.init]]]
} else {
  # Clock-concurrent optimization: CTS + post-CTS opt.
  # Sequential CTS -> post-CTS opt is not recommended...	(clockDesign --> optDesign -postCTS)
  set_interactive_constraint_modes [all_constraint_modes -active]
  set_clock_uncertainty $vars(ClockUncertainty,postCTS) [all_clocks]
  setOptMode -addInstancePrefix ccOPT_
  setAnalysisMode -analysisType onChipVariation \
    -cppr both
  if [dbGet -p head.routeTypes.name top] {
  } else {
    create_route_type	-name							top \
      -preferred_routing_layer_effort	medium \
      -top_preferred_layer			$vars(CCOpt,top_top) \
      -bottom_preferred_layer			$vars(CCOpt,top_bottom)
  }
  if [dbGet -p head.routeTypes.name trunk] {
  } else {
    create_route_type	-name 							trunk \
      -preferred_routing_layer_effort	medium \
      -top_preferred_layer			$vars(CCOpt,trunk_top) \
      -bottom_preferred_layer			$vars(CCOpt,trunk_bottom)
  }
  if [dbGet -p head.routeTypes.name leaf] {
  } else {
    create_route_type	-name							leaf \
      -preferred_routing_layer_effort	medium \
      -top_preferred_layer			$vars(CCOpt,leaf_top) \
      -bottom_preferred_layer			$vars(CCOpt,leaf_bottom)
  }
  set_ccopt_property		route_type		-net_type top	top
  set_ccopt_property		route_type		-net_type trunk	trunk
  set_ccopt_property		route_type		-net_type leaf	leaf
  set_ccopt_property	buffer_cells	$vars(tech,cts_buffer_cells)
  set_ccopt_property	inverter_cells	$vars(tech,cts_inverter_cells)
  set_ccopt_property	target_skew					auto
  set_ccopt_property	target_max_trans -net_type leaf		auto
  set_ccopt_property	target_max_trans -net_type top		auto
  set_ccopt_property	target_max_trans -net_type trunk	auto
  set_ccopt_property	use_inverters		true
  create_ccopt_clock_tree_spec		-immediate
  set_ccopt_effort		-high

  ccopt_design -outDir log/clock_report

  if { [dbIsHeadIlmSpecified] && ![dbIsHeadIlmFlattened] } {
    flattenIlm
  }
  if {$version >= 16} {
    saveDesign $outDir/03_ccopt.enc -compress -verilog
  } else {
    saveDesign $outDir/03_ccopt.enc -compress
  }
  report_resource -verbose
}

if { [ file exists $outDir/04_Route.enc ] } {
  restoreDesign $outDir/04_Route.enc.dat [file rootname [file tail [glob $outDir/04_Route.enc.dat/*.init]]]
} else {
  # Route
  # concurrent route + post-route opt. (route_opt) was not supported until Innovus v18
  routeDesign -globalDetail

  if {$version >= 16} {
    saveDesign $outDir/04_Route.enc -compress -verilog
  } else {
    saveDesign $outDir/04_Route.enc -compress
  }
  report_resource -verbose
}

if { [ file exists $outDir/final.enc ] } {
  restoreDesign $outDir/final.enc.dat [file rootname [file tail [glob $outDir/final.enc.dat/*.init]]]
} else {
  set_interactive_constraint_modes [all_constraint_modes -active]
  set_clock_uncertainty $vars(ClockUncertainty,postRoute) [all_clocks]

  setDelayCalMode -siAware true
  setAnalysisMode -analysisType onChipVariation -cppr both
  setOptMode -addInstancePrefix postrouteOPT_

  if {[info exists vars(PNR,postRouteHold)] && $vars(PNR,postRouteHold)} {
    optDesign -postRoute -setup -hold -outDir $outDir/RPT -prefix PR -expandedViews
  } else {
    optDesign -postRoute -outDir $outDir/RPT -prefix PR -expandedViews
  }

  report_timing -net > $outDir/$::env(build_name).timing.innovus
  report_power > $outDir/$::env(build_name).power.innovus
  report_power -no_wrap -instances * -outfile $outDir/$::env(build_name).cellPower.innovus
  if {$version >= 16} {
    saveDesign -rc $outDir/final.enc -compress -verilog
  } else {
    saveDesign -rc $outDir/final.enc -compress
  }
  report_resource -verbose
}

if { [ file exists "config_pre_wrapup.tcl" ] } { source config_pre_wrapup.tcl }
extractRC
rcOut -spef SPEF/$::env(build_name).spef.gz
saveNetlist $::env(build_name).final.v
set lefDefOutVersion 5.7
defOut -floorplan -netlist -routing $::env(build_name).final.def
summaryReport -outfile $outDir/$::env(build_name).summary.rpt -noHtml
report_analysis_summary > $outDir/$::env(build_name).analysis.summary.rpt
report_resource -verbose
{% endhighlight %}