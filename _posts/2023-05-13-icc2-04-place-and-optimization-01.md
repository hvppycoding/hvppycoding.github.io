---
title: "IC Compiler II - 04 Placement & Optimization Part 1"
excerpt: "Block-level Implementation"
date: 2023-05-13 04:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - IC Compiler II
---

In this module we introduce you to Placement & Optimization flow used in block level implementation.  
이 모듈에서는 블록 레벨 구현에서 사용되는 배치 및 최적화 플로우에 대해 소개합니다.  
{: .notice--warning}  

## Agenda

The Agenda of this module is placement and optimization, which covers various stages of place_opt, pre placement setup steps and power configuration in detail.  
이 모듈의 주제는 배치와 최적화로, place_opt, 사전 배치 설정 단계 및 전력 구성에 대한 다양한 단계를 자세히 다룹니다.  
{: .notice--warning}  

## Objectives

After completing this module, you should be able to

- List the stages of place opt and describe what they do
- Apply pre-placement setup to control:
  - Design and flow requirements
  - Congestion, timing, area and power QoR
- Configure the design for leakage and total power optimization
- Configure multibit banking
- Describe how placement attractions affect the design

Let's have a quick look at the objectives for this module. We are going to list the stages of place_opt, and describe what they do. We are going to be applying pre-placement setup steps to control design and flow requirements, as well as to address congestion, timing, area, and power QoR. We are going to discuss how to configure the design for leakage and total power optimization. Configure multibit banking, and describe how placement attractions affect the design.  
이 모듈의 목표를 간략히 살펴보겠습니다. place_opt의 단계를 나열하고 각각이 하는 일에 대해 설명합니다. 디자인 및 플로우 요구 사항을 제어하고 혼잡도, 타이밍, 면적 및 전력 QoR을 해결하기 위해 사전 배치 설정 단계를 적용할 것입니다. 또한 누설 전력 및 총 전력 최적화를 위해 디자인을 구성하는 방법에 대해 논의합니다. 멀티비트 뱅킹을 구성하고 배치 어트랙션(Attraction)이 디자인에 어떤 영향을 미치는지 설명합니다.
{: .notice--warning}  

## Key Steps of the Placement Phase

![2023-05-13-placement-phase-key-steps.png]({{site.baseurl}}/assets/images/2023-05-13-placement-phase-key-steps.png){: .align-center}  

The "placement phase" involves several key steps:

- Placement readiness check
- One-time design and flow requirement setup
- Congestion, timing, power and area QoR setup, as needed
- Placement and optimization

Here are the key steps of the placement phase. The previous phase was design setup. That also includes floorplanning or reading in the floorplan. After placement, we are going to continue with clock-tree synthesis. Before you run placement and optimization, you have to perform setup steps. We must distinguish between one-time setup steps, based on the design or the flow. For example, in which scenario does leakage have to be optimized, and QoR, or quality of results setup. For example, how does one increase the effort for congestion removal, or how does one deal with too much negative slack. We are going to start by looking at placement and optimization first, so you know how to run these steps, and then we will go back and look at all the other things that you have to do before placement and optimization.  
배치 단계의 주요 단계는 다음과 같습니다. 이전 단계는 디자인 설정이었습니다. 이에는 플로어플랜 작성이나 플로어플랜 읽기도 포함됩니다. 배치 후에는 클럭 트리 합성을 계속 진행할 것입니다. 배치 및 최적화를 실행하기 전에 설정 단계를 수행해야 합니다. 디자인이나 플로우에 기반한 일회성 설정 단계와 구별해야 합니다. 예를 들어, 어떤 시나리오에서 누설 전력을 최적화해야 하는지, QoR(결과 품질)을 설정해야 하는지에 따라 설정이 달라집니다. 예를 들어, 혼잡 제거를 위한 노력을 어떻게 증가시키는지 또는 너무 많은 음의 슬랙에 대처하는 방법 등입니다. 먼저 배치 및 최적화 단계를 실행하는 방법을 알아보고, 배치 및 최적화 이전에 수행해야 할 모든 작업을 다시 살펴볼 것입니다.
{: .notice--warning}  

## Placement and Optimization Commands

![2023-05-13-placement-and-opt-commands.png]({{site.baseurl}}/assets/images/2023-05-13-placement-and-opt-commands.png){: .align-center}  

- `place_opt`: Performs standard cell placement and optimization
- Placement and optimization is discussed next, pre-placement checking and setup after that

Placement and optimization is performed in ICC II using the command place_opt. place_opt performs placement of all standard cells, and optimization of timing paths to meet your goals whether they be timing, congestion, area, or power.  
ICC II에서 배치와 최적화는 place_opt 명령을 사용하여 수행됩니다. place_opt는 모든 표준 셀의 배치를 수행하고, 타이밍 경로의 최적화를 통해 타이밍, 혼잡도, 면적, 전력과 같은 목표를 달성합니다.  
{: .notice--warning}  

## Placement and Logic Optimization: `place_opt`

- `place_opt`: Performs placement and logic optimization
  - `-list_only`: List the 5 optimization stages
  - `from`: Specify which major stage to start with (default is **initial_place**)
  - `-to`: Specify which major stage to end with (default is **final_opto**)

- By default, place opt runs all five stages
- Use the -from/-to controls for exploration and debugging
  - Allows you to verify quickly whether each stage will be successful
- Various aspects can be controlled via app options

`place_opt` has exactly three command options, and they're shown right here: `-list_only`, `-from`, and `-t`o. By default, the `place_opt` command, when executed, will run through all five optimization stages. Each stage has a different purpose. Using the `-from` and `-to` options, you can run through individual stages, which can be useful for exploration and debugging. Also, some advanced flows may require you to run or skip certain stages. Other aspects of place opt operation are controlled via application options, which we will discuss.  
`place_opt` 명령에는 정확히 세 가지 옵션이 있으며, 여기서 보여지고 있습니다: `-list_only`, `-from`, `-to` 입니다. 기본적으로 `place_opt` 명령은 실행될 때 모든 다섯 가지 최적화 단계를 거칩니다. 각 단계는 다른 목적을 가지고 있습니다. `-from` 및 `-to` 옵션을 사용하여 개별 단계를 실행할 수 있으며, 탐색 및 디버깅에 유용할 수 있습니다. 또한 일부 고급 플로우에서는 특정 단계를 실행하거나 건너뛰어야 할 수도 있습니다. `place_opt` 작업의 다른 측면은 응용 프로그램 옵션을 통해 제어됩니다. 이에 대해서는 이후에 논의하겠습니다.
{: .notice--warning}  

## The Five Stages of `place_opt`

1. Initial Coarse Placement (**`initial_place`**)
   - Performs buffering aware timing-driven placement and scan chain optimization
2. HFN buffering (**`initial_drc`**)
   - Removes buffer trees, performs high fanout synthesis and logic DRC fixing
3. Initial Optimization (**`initial_opto`**)
   - Performs quick timing optimization
4. Final Placement (**`final_place`**)
   - Performs incremental and final timing-driven and global-route based congestion-driven placement, as well as scan optimization
5. Final Optimization (**`final_opto`**)
   - Performs final full-scale optimization and legalizes the design

The `initial_place` stage now performs buffering-aware timing-driven placement, which is controlled by the application option `place_opt.initial_place.buffering_aware`; (it is `true` by default). Details will be covered later. Prior to 2018.06, the initial place stage was purely wire-length driven.​ ​ The second stage is high-fanout-net buffering, or `initial_drc`. ICC II will remove existing buffer trees, and will perform high fan-out net or HFN synthesis. In addition, large logic DRC violations will be fixed. The third stage is the first actual timing optimization phase, `initial_opto`. This is a very quick timing optimization, and takes care of the easy to fix larger violations. This step should already bring down your worst and total negative slack by quite a bit. After stage three, if you were to stop right there, you can perform timing and routability analysis, and see what kind of issues you might be facing. Timewise, getting to stage 3 is rather quick. In stage four, we run `final_placement`. That is an incremental and final timing-driven placement, which works on improving timing and routability. During that phase, global routing is used for congestion estimation and removal. The fifth and final stage is called `final_opto`. During this stage ICC II performs full-scale timing, power, and area optimizations. At the end, the standard cells are legally placed. Typically, this is the longest running stage.  
초기 위치 지정(`initial_place`) 단계는 버퍼 고려 타이밍 주도 위치 지정을 수행하며, 응용 프로그램 옵션 `place_opt.initial_place.buffering_aware`에 의해 제어됩니다(기본적으로 `true`로 설정됨). 자세한 내용은 나중에 다룰 것입니다. 2018.06 이전에는 초기 위치 지정 단계는 순전히 와이어 길이에 의존했습니다. 두 번째 단계는 고팬아웃 넷 버퍼링 또는 `initial_drc`입니다. ICC II는 기존 버퍼 트리를 제거하고 고팬아웃 넷 또는 HFN 합성을 수행합니다. 또한 대형 논리 DRC 위반을 수정합니다. 세 번째 단계는 첫 번째 실제 타이밍 최적화 단계인 `initial_opto`입니다. 이는 매우 빠른 타이밍 최적화로, 쉽게 수정할 수 있는 큰 위반을 해결합니다. 이 단계는 최악 및 총 부정적인 슬랙을 상당히 줄일 것입니다. 세 번째 단계 이후에 중지한다면 타이밍 및 라우터블리티 분석을 수행하여 어떤 문제가 발생할 수 있는지 확인할 수 있습니다. 시간적으로는 3단계까지 진행하는 것이 상당히 빠릅니다. 네 번째 단계에서는 `final_placement`를 실행합니다. 이는 점진적이고 최종 타이밍 주도 위치 지정으로, 타이밍과 라우터블리티 개선에 작용합니다. 이 단계에서는 혼잡도 추정 및 제거를 위해 전역 라우팅이 사용됩니다. 다섯 번째 및 마지막 단계는 `final_opto`로, 이 단계에서 ICC II는 전체적인 타이밍, 전력 및 면적 최적화를 수행합니다. 최종적으로, 표준 셀은 합법적으로 배치됩니다. 일반적으로 이 단계가 가장 오래 걸리는 단계입니다.  
{: .notice--warning}  

## Recommended `place_opt` Exploration Flow

![2023-05-13-recommended-placeopt-flow.png]({{site.baseurl}}/assets/images/2023-05-13-recommended-placeopt-flow.png){: .align-center}  

- Analyze congestion at **`initial_place`** / **`initial_drc`**
  - If not acceptable, return to unplaced design
  - Apply congestion-focused setup steps, re-run `place_opt`
- Analyze timing, power and area (once congestion is acceptable) at **`initial_opto`** / **`final_opto`**
  - If not acceptable, return to initial design
  - Apply timing-, power- and/or area-focused setup steps, and re-run `place_opt`

The following is an example flow, where serious congestion and timing issues can be uncovered and addressed earlier:  

- Run the `initial_place` and `initial_drc` stages of `place_opt`:
  - `place_opt -to initial_drc`
- Analyze congestion, and if serious routability issues exist:
- Return to unplaced design, apply congestion-focused setup steps
- Repeat above `place_opt`

- Once serious congestion is under control, run `initial_opto`:
  - `place_opt -from initial_opto -to initial_opto`
- Analyze timing, and if serious timing violations exist:
- Return to unplaced design, apply timing-focused setup steps
- Repeat above `place_opt`

- Once serious timing violations are addressed, run `final_place` and `final_opto`:
  - `place_opt -from final_place`
- Analyze congestion and timing, and if needed, return to unplaced design and fine-tune
- congestion and timing setup
- Run **complete** `place_opt`.

Based on what we just saw, how should you run place_opt? We recommend that you start by running all five stages of place_opt. You then analyze the design for congestion, and if larger issues are discovered, you return to the unplaced design, apply congestion-focused settings, and re-run place opt. You start by improving the congestion because you cannot proceed with an unroutable design. Next, the focus is timing, power and area. If the results are not satisfactory, you apply timing, power or area focused settings and rerun place opt, again from the unplaced design. See the Notes on this page for an example.  
기본적으로는 place_opt를 모든 다섯 단계를 실행하여 시작하는 것이 좋습니다. 그런 다음 혼잡도를 분석하고, 더 큰 문제가 발견된 경우, 미배치된 디자인으로 돌아가 혼잡도에 초점을 맞춘 설정을 적용하고 place opt를 다시 실행해야 합니다. 혼잡도를 개선하는 것이 불가능한 라우터블하지 않은 디자인으로는 진행할 수 없기 때문에 먼저 혼잡도를 해결해야 합니다. 그 다음은 타이밍, 전력 및 면적에 초점을 맞춥니다. 결과가 만족스럽지 않은 경우, 타이밍, 전력 또는 면적에 초점을 맞춘 설정을 적용하고 다시 미배치된 디자인으로부터 place opt를 재실행합니다. 예제에 대한 자세한 내용은 참고 문서를 확인하십시오.  
{: .notice--warning}  

## Design Status Prior to Placement

- Post-floorplan synthesis is completed
  - Based on the best available floorplan design
- Design Setup is complete
  - New block is generated based on post-floorplan netlist:
  - Standard cells may have been placed(coarse placement from SPG synthesis)

In the example picture, while the power network has been defined, its visibility was turned off, to more clearly show the macro and standard cell placement.​ You have seen in the previous module how to accomplish the following floorplanning steps:​ ·Define block size or shape​ ·Perform pin placement​ ·Define placement blockages​ ·Place macros and lock their location​ ·Build the PG network  
예시 이미지에서는 전원 네트워크가 정의되었지만 가시성이 꺼져 있어서 매크로와 표준 셀 배치를 더 명확하게 보여줍니다. 이전 모듈에서 다음과 같은 floorplanning 단계를 수행하는 방법을 알아보았습니다: ·블록 크기 또는 모양 정의하기 ·핀 배치 수행하기 · 배치 차단 정의하기 ·매크로 배치하고 위치 고정하기 ·PG 네트워크 구축하기  
{: .notice--warning}  

## Pre-Placement Check

- IC Compiler II offers a large number of checks that can be performed at various stages
- Use get_design_checks to list all available checks
  - {`routes` `mib_alignment` `...` **`pre_route_stage`** `legality` `...` **`pre_clock_tree_stage`** **`pre_placement_stage`** `timing` `mv_design` `...`}
  - The bolded checks are mega-checks — they contain several of the smaller checks
- Before placing the design, run the following:
  - `check_design -checks pre_placement_stage`

```text
Running mega-check 'pre_placement_stage'
    Running atomic-check 'design_mismatch'
    Running atomic-check 'scan_chain'
    Running atomic-check 'mv_design'
    Running atomic-check 'rp_constraints'
    Running atomic-check 'timing'
    Running atomic-check 'hier_pre_placement'
```

ICC II has a number of checks that can be performed at each stage. You can get a list of the available checks using the `get_design_checks` command. There are three so-called mega checks, highlighted in bold: `pre_placement_stage`, `pre_clock_tree_stage`, and a `pre_route_stage`. Each of these checks performs other sub-checks. The `pre_placement_stage_check`, for example, will check your scan chains, for timing issues, for design mismatches, and for MV violations. These checks are all run using the command `check_design`. In the example shown `check_design -checks` for issues that might affect placement. ​Slide shows the example of the output generated for the lab design.  
ICC II에는 각 단계에서 수행할 수 있는 여러 가지 체크(check)가 있습니다. `get_design_checks` 명령을 사용하여 사용 가능한 체크 목록을 얻을 수 있습니다. 이 중에서도 `pre_placement_stage`, `pre_clock_tree_stage`, `pre_route_stage`라는 세 가지 "메가 체크"가 있습니다. 이 체크들은 각각 다른 하위 체크들을 수행합니다. 예를 들어, `pre_placement_stage_check`는 스캔 체인, 타이밍 이슈, 디자인 불일치, MV(Multiple Voltage) 위반이 있는지 확인합니다. 이러한 체크는 `check_design` 명령을 사용하여 실행됩니다. 슬라이드에는 랩 디자인에 대한 출력 예시가 표시되어 있습니다.
{: .notice--warning}  

## `check_design` Results

- All checks print a summary to the screen and store most results in an EMS database (Enhanced Messaging System)
- Open the EMS database in the GUI for detailed analysis

![2023-05-13-check-design-results.png]({{site.baseurl}}/assets/images/2023-05-13-check-design-results.png){: .align-center}  

When a check is performed, it will save the results in an EMS database. EMS stands for Enhanced Messaging System. All the checking results, the warnings, and errors, are written to that file. You can then open the file in the message browser, by selecting "Message Browser Window" from the Window menu, as shown. There you can easily browse the categorized messages. You can retrieve additional information if you select the actual violation on the right side. The EMS file can be opened in another ICC II session, so this makes it easy to pass this information on to other users for further analysis.  
체크가 수행되면 결과는 EMS(Enhanced Messaging System) 데이터베이스에 저장됩니다. EMS는 향상된 메시징 시스템을 나타냅니다. 모든 체크 결과, 경고 및 오류가 해당 파일에 기록됩니다. 그런 다음 "Message Browser Window"를 Window 메뉴에서 선택하여 메시지 브라우저에서 해당 파일을 열 수 있습니다. 거기에서는 카테고리별로 메시지를 쉽게 찾아볼 수 있습니다. 오른쪽에서 실제 위반이 선택되면 추가 정보를 얻을 수 있습니다. EMS 파일은 다른 ICC II 세션에서 열 수 있으므로 이를 통해 다른 사용자에게 해당 정보를 전달하여 추가 분석을 수행하기가 용이합니다.  
{: .notice--warning}  

## Also Recommended: Physical Constraint Checks

```tcl
check_design -checks physical_constraints
```

- Performs various checks on floorplan objects such as layers, bounds, placement blockages, cell instances, macros, site rows, etc., to ensure correctness
- Example: 
  - `Information: The layer 'M9' does not contain any PG shapes. (DCHK-104)`
  - `Warning: Keepout margin of cell I_PCI_TOP/PCI_FIFO_RAM 8 lies partially or completely outside the bounds of the core. (DCHK-085)`

Another check that can be very useful before running `place_opt`, which is not included in the `pre-placement` mega-check, is the physical constraints check. This performs various checks on floorplan objects. For example, it will make sure that your boundary is specified correctly. It will make sure that your site rows are specified, that they're not overlapping, and will check placement blockages and make sure that they are not overlapping with any other objects or blockages. This is a very good check if your floorplan comes from some other tool, and you want to make sure there are no issues that will cause place opt to give you bad results. The notes include a more complete list of the various checks performed.  
`place_opt`를 실행하기 전에 매우 유용한 다른 체크는 `pre-placement` 메가 체크에 포함되지 않은 물리적 제약 조건 체크입니다. 이 체크는 floorplan 객체에 대해 다양한 체크를 수행합니다. 예를 들어, 경계가 올바르게 지정되었는지 확인합니다. 사이트 행이 올바르게 지정되었는지 확인하고 서로 겹치지 않는지 확인하며, 배치 제약 조건을 확인하여 다른 객체나 제약 조건과 겹치지 않는지 확인합니다. 이 체크는 floorplan이 다른 도구에서 가져온 경우에 유용하며, `place_opt`에서 잘못된 결과를 얻지 않도록 문제가 없는지 확인하는 데 도움이 됩니다. 자세한 체크 목록은 노트에 포함되어 있습니다.
{: .notice--warning}  

## Design and Flow Requirements Setup Steps

Design and flow requirement setup consists of various one-time steps, performed as applicable:  

```text
Advanced technology setup
  Scenario configuration
    ...
    Library cell restrictions
      Power optimization
        Multibit banking
```

Design and flow requirement setup consists of various one-time steps which are explained in the following slides.  
디자인 및 플로우 요구 사항 설정은 다음 슬라이드에서 설명하는 여러 번 실행되는 단계로 구성됩니다.  
{: .notice--warning}  

## Setup for Advanced Nodes

- Advanced technology nodes require complex app option setup to control RC extraction, placement, legalization, routing and via ladders
- Most settings are not optional, and can be different for each node
- A single command performs node-specific checks and setup
  - Example: `icc2_shell> set_technology -node 7`
  - Many nodes are already supported by set_technology
  - Always use set_technology at the **beginning** of the physical design flow!

"`set_technology`" command allows you to set all the process node specific settings at the beginning of the physical design flow. These settings are persistent in the NDM, and independent of the flow and design. You can see the following example to set IC Compiler II to 7nm mode; by using mega switch '`set_technology -node 7`'. Always use `set_technology` command at the beginning of your place and route flow.  
"`set_technology`" 명령은 물리 설계 플로우의 시작 부분에서 프로세스 노드별 설정을 지정할 수 있게 해줍니다. 이러한 설정은 NDM에서 영속적이며, 플로우와 디자인에 독립적입니다. 다음 예제에서는 IC Compiler II를 7nm 모드로 설정하는 방법을 보여줍니다. '`set_technology -node 7`'과 같은 메가 스위치를 사용합니다. 항상 place 및 route 플로우의 시작 부분에서 `set_technology` 명령을 사용해야 합니다.
{: .notice--warning}  

## Advanced Legalizer

```text
place.legalize.enable_advanced_legalizer
place.legalize.enable_advanced_prerouted_net_check
```

- `set_technology` also enables the advanced legalizer for 12 and below
  - Designed for advanced nodes
  - Natively supports 2D rule legalization in a single pass resulting in higher density
    - Multi-row, single-row, and Half-Row-Offsite cells
  - Enables automatic 2D rule detection

Advanced legalizer targets for advance node usage, and is available since 12nm. Its settings can be enabled by single command set_technology.​ Generally, you need both place.legalize.enable_advanced_legalizer, and place.legalize.enable_advanced_prerouted_net_check, set to true for advanced legalizer enablement. ​ Advanced Legalizer supports 2D rule legalization natively in one single pass for various cell style, for example, multi-row, single-row, and Half-Row-Offsite cells.​  
고급 Legalizer는 고급 노드에서 사용되는 타겟을 지원하며, 12nm 이상에서 사용할 수 있습니다. 이 설정은 단일 명령어인 set_technology를 통해 활성화할 수 있습니다. 일반적으로 고급 Legalizer를 활성화하기 위해서는 place.legalize.enable_advanced_legalizer와 place.legalize.enable_advanced_prerouted_net_check 두 가지 설정을 모두 true로 설정해야 합니다. 고급 Legalizer는 다양한 셀 스타일에 대해 2D 규칙 Legalization을 단일 패스로 네이티브로 지원합니다. 예를 들어, 멀티 로우, 싱글 로우, 그리고 Half-Row-Offsite 셀과 같은 다양한 셀 스타일을 지원합니다. 이를 통해 고급 Legalizer는 높은 효율성과 정확성을 제공하며, 설계의 배치 결과를 개선할 수 있습니다. 고급 Legalizer는 12nm 이상의 노드에서 사용할 수 있으며, 고급 노드를 대상으로 하는 설계에서 고급 Legalizer를 활성화하여 최적의 결과를 얻을 수 있습니다.
{: .notice--warning}  

## Migrating to the Advanced Legalizer

The advanced legalizer is supported in production for advance nodes, and established nodes. For additional nodes and foundry processes supported by advanced legalizer, please contact your Synopsys AE on current up-to-date support! ​ In general, if your design has a large number of multi-row, large or wide cells, timing QoR, full flow runtime, and router DRC benefits can be seen by using Advanced Legalizer, even on older process nodes​.  
고급 Legalizer는 고급 노드와 확립된 노드에서 제품으로 지원됩니다. 고급 Legalizer가 지원하는 추가 노드 및 파운드리 프로세스에 대해서는 최신 지원 정보를 위해 Synopsys AE에 문의하십시오! 일반적으로, 디자인에 멀티 로우, 큰 또는 넓은 셀이 많이 포함되어 있는 경우 고급 Legalizer를 사용하면 타이밍 QoR, 전체 플로우 실행 시간 및 라우터 DRC의 이점을 얻을 수 있습니다. 심지어 오래된 공정 노드에서도 이점을 얻을 수 있습니다.  
{: .notice--warning}  

## Choose Your QoR Strategy

```text
set_qor_strategy -stage pnr \
  -mode [balanced | early_design | extreme_power] \
  -metric [timing | leakage_power | total_power]
```

- For ease of use, configure 10011 to meet your QoR criteria:
  - Selecting `leakage_power` or `total_power` configures power optimization in addition to timing (timing remains higher priority)
  - Selecting `timing` will not enable any power optimization
- The `pnr` stage configures placement, CTS, routing and optimization application options to meet the chosen metric(s)
- The application options that are configured will be printed to the screen and can also be written to a file with `-output <filename>`

To apply tool settings required for improving specific QoR metrics, use the `set_qor_strategy -stage pnr` command. To specify one or more QoR metric to improve, use `-metric_option`. The valid values are area, congestion, leakage power, timing, and total power. When you use this command, the tool applies application option settings that affect placement, legalization, optimization, CTS, routing, and timing analysis.​  
특정 QoR 메트릭을 향상시키기 위해 필요한 도구 설정을 적용하려면 `set_qor_strategy -stage pnr` 명령을 사용하세요. `-metric_option`을 사용하여 개선하려는 하나 이상의 QoR 메트릭을 지정할 수 있습니다. 유효한 값은 area, congestion, leakage power, timing 및 total power입니다. 이 명령을 사용하면 도구가 배치, 합법화, 최적화, CTS, 라우팅 및 타이밍 분석에 영향을 주는 응용 프로그램 옵션 설정을 적용합니다.
{: .notice--warning}  

## Configure Scenarios for MCMM Optimization

- Concurrent MCMM optimization requires creation of scenarios
- Optimization occurs in **active** scenarios, for enabled **analysis types**:
  - `setup`, `hold`
  - `leakage_power`, `dynamic_power`
  - `max_transition`, `max_capacitance`, `min_capacitance`
- **Hold** analysis type is ignored during `place_opt`, even if enabled
- Configure the scenarios appropriately prior to executing `place_opt`

```tcl
# Example configuration for placement:
set_scenario_status * -active false
set_scenario_status <list_of_placement_scenarios> -active true
set_scenario_status *corner_FAST -setup false
set_scenario_status mode_TEST* -leakage_power false
```

By default, `place_opt` will run on all active scenarios that are enabled for the analysis types shown here. setup, leakage and dynamic_power, max_transition, capacitance, and min_capacitance. If you have a scenario that is only enabled for hold analysis, nothing will happen, because `place_opt` ignores hold altogether. If you want to enable fewer scenarios for place opt, then you can do that with the, `set_scenario_status` command. In the example below, I first disabled all scenarios using, `set_scenario_status * -active false`. I then activate the scenarios that I want to use for placement and optimization. In the next two lines, I turn off setup analysis in my fast corners, as well as leakage power optimization in the test modes.  
기본적으로, `place_opt`는 분석 유형에 대해 활성화된 모든 활성 시나리오에서 실행됩니다. setup, leakage and dynamic_power, max_transition, capacitance, min_capacitance과 같은 시나리오입니다. Hold 분석에만 활성화된 시나리오가 있다면, `place_opt`는 hold를 완전히 무시하므로 아무 일도 발생하지 않습니다. place opt에 대해 더 적은 시나리오를 활성화하려면 `set_scenario_status` 명령을 사용할 수 있습니다. 아래의 예에서는 먼저 `set_scenario_status * -active false`를 사용하여 모든 시나리오를 비활성화합니다. 그런 다음 배치 및 최적화에 사용할 시나리오를 활성화합니다. 다음 두 줄에서는 빠른 코너에서의 setup 분석을 비활성화하고, 테스트 모드에서의 leakage power 최적화를 비활성화합니다.  
{: .notice--warning}  

## Configuring `place_opt` for SPG

- If your input netlist was generated by SPG synthesis with Design Compiler NXT, enable the SPG placement flow:
  - `set_app_options -list {place_opt.flow.do_spg true}`
- The `initial_place` and `initial_drc` phases are skipped
  - Coarse placement and HFN/DRC buffering from synthesis are used as the start-point for initial optimization, followed by final placement and optimization
  - Any options/commands shown later that affect these two stages will not have an effect when using the SPG mode
- Standard cell placement must have been read in through DEF
  - Created using `write_icc2_files` in Design Compiler NXT

If you have a design that comes from Design Compiler-NXT, using SPG – that's Synopsys Physical Guidance, then you should also enable the SPG flow in ICC II, using the application option shown. Once you enable SPG, the first two stages of place_opt are skipped, that is, `initial_place`, and `initial_drc`, because Design Compiler NXT has performed these. Standard cell placement must have been imported. The most convenient way is to use the setup script created by DC's `write_icc2_files`.  
만약 Design Compiler-NXT에서 생성된 SPG(Physical Guidance)를 사용하는 설계를 가지고 있다면, ICC II에서도 SPG 플로우를 활성화해야 합니다. 위에 표시된 application 옵션을 사용하여 SPG를 활성화합니다. SPG를 활성화하면 place_opt의 첫 번째 단계인 `initial_place`와 `initial_drc`가 생략됩니다. 왜냐하면 Design Compiler-NXT에서 이미 수행되었기 때문입니다. 표준 셀 배치가 가져와져야 합니다. 가장 편리한 방법은 DC의 `write_icc2_files` 의해 생성된 설정 스크립트를 사용하는 것입니다.  
{: .notice--warning}  

## High Fanout Synthesis Port Punching

- High fanout synthesis / buffering occurs during the initial_drc stage
- By default, port punching may add or remove ports at hierarchy boundaries for HF synthesis
- To disable port punching on specified cells:
  - `set_freeze_ports -data|-clock|-all [get_cells cellA]`

When ICC II performs high fanout synthesis during the `initial_drc` stage, a large number of buffers is added to the design. In order to do a good job and achieve a good balance, HFN synthesis needs to be able to port punch through your hierarchy, meaning it creates additional ports throughout the logical hierarchy. If you have a block where you absolutely need to preserve the interface, then you can use the `set_freeze_ports` command. Of course, limiting the port punching can lead to lower quality buffer trees for your high fan-out nets. The `set_freeze_ports` command is applied on hierarchical cells. You can control whether you want to disable port punching only on data ports, clock ports, or all ports.  
ICC II에서 `initial_drc` 단계에서 고팬아웃 합성을 수행할 때, 대량의 버퍼가 설계에 추가됩니다. 좋은 결과를 얻기 위해서는 HFN 합성이 계층 구조를 통과하여 추가적인 포트를 생성할 수 있어야 합니다. 만약 인터페이스를 보존해야 하는 블록이 있는 경우, `set_freeze_ports` 명령을 사용할 수 있습니다. 물론 포트 펀칭을 제한하는 것은 고팬아웃 네트에 대한 품질이 낮아질 수 있습니다. `set_freeze_ports` 명령은 계층 셀에 적용됩니다. 데이터 포트, 클록 포트 또는 모든 포트에서 포트 펀칭을 비활성화할지 여부를 제어할 수 있습니다.  
{: .notice--warning}  

## Even Distribution of Spare Cells - Automatic

- Spare cells can be included in the synthesis netlist
  - Included in the top module and/or sub-design modules
  - Auto-identified by ICC II
  - For manual marking: `set_attribute [get_flat_cells *SPARE_REG*] spare_cell_mode true`
- Spare cells will be placed along with regular standard cells, and automatically spread out evenly
  - Controlled by `place.coarse.enable_spare_cell_placement` (Default: `true`)
  - When set to `false`, spare cells are still placed, but not spread out evenly
- Placement is hierarchy aware, so spare cells will be placed along with their intended hierarchy

In some designs, spare cells are included in the synthesis netlist, and are used for ECOs later in the flow. Once these spare cells have been identified by ICC II, they will be placed along with regular standard cells, and automatically spread out evenly throughout your design. That is controlled by an application option called, `place.coarse.enable_spare_cell_placement`, which is `true` by default. It's also important to understand that placement is hierarchy aware. Let's say your logical hierarchy A gets placed roughly in the top-left corner, then the spare cells instantiated in cell A will also be in that top left area.  
일부 설계에서는 예비 셀이 합성 넷리스트에 포함되어 있으며, 플로우의 나중 단계에서 ECO(Engineering Change Order)에 사용됩니다. 이러한 예비 셀이 ICC II에 의해 식별된 후, 일반적인 표준 셀과 함께 배치되며, 자동으로 설계 전체에 고르게 분포됩니다. 이는 `place.coarse.enable_spare_cell_placement`라는 응용 옵션으로 제어됩니다. 기본적으로 `true`로 설정되어 있습니다. 또한 배치는 계층 구조를 인식합니다. 예를 들어 논리적인 계층 A가 대략적으로 왼쪽 상단 모서리에 배치된다면, 셀 A에서 인스턴스화된 예비 셀도 해당 상단 왼쪽 영역에 배치될 것입니다.  
{: .notice--warning}  

## Add Spare Cells Not Already in the Netlist

Cells added by `add_spare_cells` are coarse-placed, therefore you need to legalize the cell placement afterwards.​ Spare cells are not "fixed" in place, by default, CTS and routing optimizations can move them.​ Setting the cells to "fixed" may impede the above optimizations, if there are many fixed cells.​ Instead, set the placement status to "legalize only", once the spare cells are distributed​. It prevents placement and optimization from moving spare cells​. Cells can only be moved by the legalizer after CTS and routing optimizations, if needed​. ​Any spare cells which are included in the netlist, and tagged with the attribute is spare cell equals to true, will be automatically placed, and evenly spread during `place_opt`.  
`add_spare_cells`로 추가된 셀은 coarse-placed되므로, 이후에 셀 배치를 legalizing해야 합니다. 예비 셀은 기본적으로 "고정"되지 않으며, CTS 및 라우팅 최적화에 의해 이동될 수 있습니다. 많은 고정 셀이 있는 경우, 셀을 "고정"으로 설정하는 것은 위의 최적화를 방해할 수 있습니다. 대신, 예비 셀의 배치 상태를 "legalize only"로 설정하면, 배치 및 최적화가 예비 셀을 이동시키지 않도록 할 수 있습니다. CTS 및 라우팅 최적화 이후에 필요한 경우에만 legalizer에 의해 셀을 이동시킬 수 있습니다. 넷리스트에 포함되고 is_spare_cell 속성이 true로 태그된 모든 예비 셀은 자동으로 배치되고 `place_opt` 중에 균등하게 분포됩니다.
{: .notice--warning}  

## Remove Unwanted Ideal Networks

- Ideal network constraints on high fanout nets (like set/reset, enable, select) prevent buffering during placement — probably not desired!
  - Can be found using `report_ideal_network`
- Remove all ideal network constraints to allow HFN buffering during placement:
  - `remove_ideal_network -all`

![2023-05-13-remove-ideal-networks.png]({{site.baseurl}}/assets/images/2023-05-13-remove-ideal-networks.png){: .align-center}  

Now to ideal networks: In the past, one usually prevented Design Compiler from synthesizing high fanout nets. Because it just doesn't make sense if you don't have full visibility into the placement and the floorplan. Remember when we talked about that first-pass netlist earlier, in that case, a high fanout net is probably of low quality. Because you don't have an idea of where the macros are sitting. Therefore, often, these high fanout nets were marked as ideal networks. Nowadays, if you're running Design Compiler-NXT, you're performing physical synthesis including physical high fanout net synthesis. So these nets are no longer ideal networks. If they are ideal, you need to make sure that you remove these attributes. Otherwise, ICC II will not touch these networks either. You can find out whether the design has any ideal networks, by using the `report_ideal_network` command. The recommendation is to simply run the command, `remove_ideal_network -all`. This way ICC II can optimize all nets.  
과거에는 Design Compiler에서 고반출 넷을 합성하는 것을 피했습니다. 왜냐하면 배치와 플로어플랜에 대한 완전한 가시성이 없는 경우, 고반출 넷은 품질이 낮을 수 있기 때문입니다. 앞서 언급한 첫 번째 패스 넷리스트에서는 매크로가 어디에 위치하는지 알 수 없습니다. 따라서 이러한 고반출 넷은 종종 이상적인 네트워크로 표시되었습니다. 하지만 현재 Design Compiler-NXT를 실행하고 있는 경우, 물리적 합성(Physical Synthesis)을 수행하면서 물리적인 고반출 넷 합성도 진행됩니다. 따라서 이제 이러한 넷은 이상적인 네트워크가 아닙니다. 이상적인 네트워크인 경우 해당 속성을 제거해야 합니다. 그렇지 않으면 ICC II도 이 네트워크를 건드리지 않습니다. design에 이상적인 네트워크가 있는지 확인하려면 `report_ideal_network` 명령을 사용할 수 있습니다. 권장하는 방법은 단순히 `remove_ideal_network -all` 명령을 실행하는 것입니다. 이렇게 하면 ICC II가 모든 넷을 최적화할 수 있습니다.
{: .notice--warning}  

## Library Cell Purpose

- Every library cell can be restricted to certain optimization tasks using a library cell purpose (attribute: `valid_purposes`)
- The following lib cell purposes are recommended:
  - `optimization`: for setup/hold, power and logical design rule optimization
  - `cts`: for clock tree synthesis
    - `hold` purpose behaves like optimization (best practice: use only `optimization`)
    - `power` purpose is configurable but is not used by the tool
- If the original cells in the **liberty** library (used to create the CLIB) have `dont_use=true`, then **no purposes** are set

To give you a quick summary of what library cell purpose is, and what it does, it's really the evolution of the don't use attribute from Design Compiler and IC Compiler. In IC Compiler, if you wanted the tool not to use a certain cell, whether it is for CTS, for optimization, or for power upsizing downsizing, you would mark that cell as a don't use. In IC Compiler II, there is a concept called the library cell purpose, where you can specify a purpose, or a number of purposes for every standard cell. There are four purposes available for each cell:  power, hold, CTS, and optimization. For example, a cell with a hold purpose will be used for hold optimization, and in order for CTS to use a cell, it has to have the CTS purpose. When you first create a design by reading the Verilog netlist and linking to your libraries, ICC II will set the library cell purpose, based on the don't_use attribute's setting in the liberty dbs. If the don't use attribute is not set, or set to false, then all four lib cell purposes are set. If don't use is true, then no lib cell purposes are set.  
라이브러리 셀용도(library cell purpose)는 Design Compiler와 IC Compiler의 don't use 속성의 진화입니다. IC Compiler에서는 CTS, 최적화, 전력 업사이징/다운사이징을 위해 특정 셀을 사용하지 않기를 원할 경우 해당 셀을 don't use로 표시했습니다. IC Compiler II에서는 라이브러리 셀용도라는 개념이 도입되어 각 표준 셀마다 목적 또는 여러 목적을 지정할 수 있습니다. 각 셀에는 전력(power), 홀드(hold), CTS, 최적화(optimization)와 같은 네 가지 목적이 제공됩니다. 예를 들어, 홀드 목적을 가진 셀은 홀드 최적화에 사용되며, CTS에서 셀을 사용하려면 CTS 목적을 가져야 합니다. Verilog 넷리스트를 읽고 라이브러리에 연결하여 디자인을 처음 생성할 때, ICC II는 liberty 데이터베이스의 don't_use 속성 설정을 기반으로 라이브러리 셀용도를 설정합니다. don't use 속성이 설정되지 않거나 false로 설정된 경우, 네 가지 라이브러리 셀용도가 설정됩니다. don't use가 true로 설정된 경우, 라이브러리 셀용도가 설정되지 않습니다.  
{: .notice--warning}  

## Restrict Library Cell Usage

- You may want to restrict the use of specific cells during CTS, or other logic optimizations, or for all purposes, for example:
  - No big drivers (EM or crosstalk issues)
  - No high-leakage registers from an ultra-low Vt library
  - Only certain buffers for CTS

```tcl
set_lib_cell_purpose -include none \
    [get_lib cells "*/*BUF_X64* */*REG ulvt*"]
set_lib_cell_purpose -exclude cts [get_lib_cells]
set_lib_cell_purpose -include cts [get_lib_cells * /NBUF*]
```

- To determine the current lib cell purpose setting:

```tcl
report_lib_cells -columns {name valid_purposes} \
    -objects [get_lib_cells "*/*BUFX2*"]

IBUFX2_HVT    optimization
NBUFX2_HVT    cts optimization
TNBUFX2_HVT   optimization
```

Let me show you a couple of examples of how to use library cell purpose. There can be a number of reasons why you would want to restrict the use of certain cells. For example, you might want to restrict usage of the largest drivers a very strong buffer could lead to electromigration issues; or you might want to prevent the use of high-leakage registers in ultra-low VT libraries. For the command set_lib_cell purpose, you have to specify the purpose, or purposes, and the library cells to apply the purpose to. Using the option "`-include none`", I specify that I don't want any purposes set for the library cells specified. If you want to find out about the current library cell purpose settings, you would use the command as shown on the bottom. There is no dedicated command for this. You have to use `report_lib_cells`, and in my example, I configure the output to show me only the name of the cell and its configured purposes.  
라이브러리 셀용도(library cell purpose)를 사용하는 몇 가지 예를 보여드리겠습니다. 특정 셀의 사용을 제한하고 싶은 이유는 여러 가지가 있을 수 있습니다. 예를 들어, 가장 큰 드라이버의 사용을 제한하고 싶을 수 있습니다. 아주 강력한 버퍼는 전자이동성 문제를 일으킬 수 있습니다. 또는 초저 전압(VT) 라이브러리에서 고누설 레지스터의 사용을 방지하고 싶을 수도 있습니다. `set_lib_cell_purpose` 명령을 사용할 때, 목적(purpose) 또는 목적들과 해당 목적을 적용할 라이브러리 셀을 지정해야 합니다. "`-include none`" 옵션을 사용하여 지정한 라이브러리 셀에 대해 어떤 목적도 설정하지 않도록 지정합니다. 현재 라이브러리 셀용도 설정에 대한 정보를 확인하려면 아래에 표시된 명령을 사용합니다. 이를 위한 별도의 명령은 없습니다. `report_lib_cells`를 사용해야 하며, 예시에서는 셀의 이름과 설정된 목적만 표시되도록 출력을 구성했습니다.  
{: .notice--warning}  

## Preroute Power Optimization

- By default, ICC II does perform power optimization
- The following power optimization metric can be selected:
  - leakage_power
  - total_power (leakage + dynamic)

```tcl
set_qor_strategy -stage pnr | synthesis \
    -metric total_power | leakage power
```

- Power optimization is only performed in active scenarios enabled for power, for example to enable total power optimization:

```tcl
set_scenario status {func.tt_60c} \
    -leakage_power true -dynamic_power true
```

By default, ICC II does perform power optimization. For using power optimization, the recommendation is to enable `set_qor_strategy` to `leakage_power` or `total_power`, instead of any individual app option​.`set_qor_strategy` will enable all the power related options under the switch.​The prerequisite to enable power optimization is to enable the scenario for power using `set_scenario_status` to enable `leakage_power` and `dynamic_power`.  
기본적으로 ICC II는 전력 최적화를 수행합니다. 전력 최적화를 사용하기 위해서는 개별 애플리케이션 옵션보다 `leakage_power` 또는 `total_power`로 `set_qor_strategy`를 활성화하는 것이 권장됩니다. `set_qor_strategy`는 스위치 아래의 모든 전력 관련 옵션을 활성화합니다. 전력 최적화를 사용하려면 `leakage_power` 및 `dynamic_power`를 활성화하기 위해 `set_scenario_status`를 사용하여 power 시나리오를 활성화해야 합니다.
{: .notice--warning}  

## Leakage Power Optimization

![2023-05-13-leakage-power-opt.png]({{site.baseurl}}/assets/images/2023-05-13-leakage-power-opt.png){: .align-center}  

- Tradeoff between
  - Faster, higher leakage low-Vth, and
  - Slower, lower leakage high-Vth cells
- Leakage power becomes a component of the overall optimization cost
- Multi-Vth/L libraries should be made available
- Occurs if power optimization mode is `leakage` or `total`
  - `set_qor_strategy -stage pnr -metric total_power | leakage power`
  - Leakage power calculation depends on the state of the inputs:
    - `power.leakage_mode`
    - Default: `state`
    - Allowed values: `state`, `average`, `unconditional`

Leakage Power Optimization is always trade-off with cell delay. If you use LVT cells, they provide fastest transition, but more leaky in nature. But in case of HVT cells, they provide less  leakage but they are slower. Leakage power becomes a component of the overall optimization cost. ICC II uses the value of the cell leakage power attribute of the library cells in the block to calculate the leakage power. If a cell does not have a cell leakage power attribute, the tool uses the value of the library-level default cell leakage power attribute for that cell, otherwise 0.​ The following methods are available for calculating the leakage:​ Average: When using this method, the calculation is based on equal weighted probabilities for all states.​ Unconditional: When using this method, the calculation is based purely on the attribute values.​ State: When using this method, the calculation is based on the state of each instance, derived from the switching activity.  
Leakage Power Optimization은 항상 셀 지연과의 균형을 유지해야 합니다. LVT 셀을 사용하면 가장 빠른 전환을 제공하지만 누수 전류가 많이 발생합니다. 반면 HVT 셀은 누수 전류가 적지만 더 느립니다. 누수 전력은 전체 최적화 비용의 구성 요소가 됩니다. ICC II는 블록 내 라이브러리 셀의 누수 전력 속성 값을 사용하여 누수 전력을 계산합니다. 셀에 누수 전력 속성이 없는 경우 동일한 셀에 대해 라이브러리 수준의 기본 셀 누수 전력 속성 값을 사용하고, 그렇지 않은 경우에는 0을 사용합니다. 다음과 같은 방법을 사용하여 누수 전력을 계산할 수 있습니다. 평균: 이 방법을 사용하면 모든 상태에 대해 동일한 가중치가 적용된 확률에 기반하여 계산됩니다. 무조건: 이 방법을 사용하면 계산은 속성 값에만 기반합니다. 상태: 이 방법을 사용하면 각 인스턴스의 상태에 기반하여 전환 활동에서 파생된 계산이 수행됩니다.
{: .notice--warning}  

## Total Power Optimization

![2023-05-13-total-power-opt.png]({{site.baseurl}}/assets/images/2023-05-13-total-power-opt.png){: .align-center}  

- In advanced nodes, leakage and dynamic power may trend differently within a Vt class (e.g. different channel lengths)
  - Leakage: XLVT-S > XLVT-L 
    - *L=Long S=Short*
  - Dynamic: XLVT-S < XLVT-L
- Total power optimization creates a composite cost of dynamic and leakage power numbers and includes this composite cost in the overall optimization costing function
  - `set_qor_strategy -stage pnr -metric total_power`

Total power optimization is not just a shortcut for enabling leakage and dynamic power, instead, total power optimization creates a composite cost of dynamic and leakage power numbers and then this composite cost is added to the overall optimization cost function. The reason for this is that within a VT class, leakage and dynamic power trend differently. If you reduce leakage, that might actually increase your dynamic power and vice versa. This trend was not seen in classic nodes, where, in general, if you reduced leakage power, you also reduced dynamic power. Again, total power optimization also requires switching activity because we are performing dynamic optimization as part of this optimization.  
전체 전력 최적화는 누설 전력과 동적 전력을 활성화하는 단순한 단축키가 아닙니다. 대신, 전체 전력 최적화는 동적 및 누설 전력 수치의 복합 비용을 생성하고 이 복합 비용을 전체 최적화 비용 함수에 추가합니다. 이는 VT 클래스 내에서 누설 및 동적 전력이 다르게 변하는 경우에 해당합니다. 누설 전력을 감소시키면 동적 전력이 증가할 수도 있고 그 반대일 수도 있습니다. 이러한 경향은 과거 노드에서는 일반적으로 누설 전력을 감소시키면 동적 전력도 감소하는 것을 보지 못했습니다. 다시 말해, 전체 전력 최적화는 동적 최적화도 수행하기 때문에 전환 활동이 필요합니다.  
{: .notice--warning}  

## Power Calcultation Requires Switching Activity

**Power calculation requires accurate switching activity:**

- Applied by reading SAIF files from simulation (recommended)

```tcl
set_scenario_status -dynamic_power true -leakage_power true func.tt_60c
read_saif design_sim.saif -scenario func.tt_60c
```

- Alternatively, by "manually" applying toggle information to primary inputs and black box outputs

![2023-05-13-swtiching-activity.png]({{site.baseurl}}/assets/images/2023-05-13-swtiching-activity.png){: .align-center}  

```tcl
current_scenario func.tt_60c
set_switching_activity [get_ports "rst scan_en"] \
    -toggle_rate 0.0 -static_probability 0.0
set_switching_activity [get_ports a] \
    -toggle_rate 0.02 -static_probability 0.7
set_switching_activity [get_ports b] \
    -toggle_rate 0.06 -static_probability 0.3
```

- Switching activity is automatically propagated throughout the design

In order to use dynamic or total power optimization, ICC II requires switching activity. You will get the most accurate results by loading a SAIF file, which comes from simulation. In the example, I enable dynamic power in my scenario, and then use read SAIF to apply the SAIF to my scenario. If SAIF files are not available, then you should at least apply toggle activity to known nodes in your design, and ICC II can propagate the toggle information through your design. This is much better than not applying any toggle information and relying on default values, which are shown in the Notes of this page. The toggle rates and the static probabilities should be applied on primary inputs as well as black box outputs. Switching activity is automatically propagated throughout the design  
동적 또는 전체 전력 최적화를 사용하기 위해서는 ICC II에서 전환 활동을 필요로 합니다. 시뮬레이션으로부터 얻은 SAIF 파일을 로드하여 가장 정확한 결과를 얻을 수 있습니다. 예를 들어, 시나리오에서 동적 전력을 활성화하고, 그런 다음 SAIF를 시나리오에 적용하기 위해 read SAIF를 사용합니다. SAIF 파일을 사용할 수 없는 경우에는 최소한 알려진 노드에 토글 활동을 적용하고, ICC II가 토글 정보를 설계 전체로 전파할 수 있도록 합니다. 이는 토글 정보를 적용하지 않고 기본값에 의존하는 것보다 훨씬 좋은 결과를 얻을 수 있습니다. 토글 속도와 정적 확률은 주요 입력뿐만 아니라 블랙 박스 출력에도 적용되어야 합니다. 전환 활동은 자동으로 설계 전체에 전파됩니다.  
{: .notice--warning}  

## Switching Activity Coverage

- Use `report_activity -driver` to ensure near 100% switching activity coverage on essential points, broken down by activity type:
- Use the `-verbose` option for a detailed breakdown of each activity type’s source

Use `report_activity` command to report the switching activity of a block. Ensure near 100% coverage on essential points.  
"`report_activity`" 명령을 사용하여 블록의 스위칭 액티비티를 보고합니다. 중요한 지점에서 거의 100% 커버리지를 확보하세요.
{: .notice--warning}  

## SAIF-less Flow

- Primary control inputs that are not annotated are set with default switching activity values (A clock port with SDC constraints is set with switching activity derived from SDC constraints)
- Propagated switching activity of sequential output pins tends to approach a zero toggle rate
  - Dynamic power calculations of the downstream logic can be optimistically low
- `​infers_switching_activity` will apply appropriate static values to control signals, as well as better toggle rates for sequential output pins

`set_switching_activity` does not annotate primary control inputs such as reset, enable, causing them to inherit default switching activity values. This causes pessimistically high dynamic power. Propagated switching activity of sequential output pin tends to approach zero toggle rate, and causes optimistically low dynamic power.​ `​infers_switching_activity` command infers the switching activity on the drivers of the Special Control Input pins (SCIs) in the design.  
`set_switching_activity` 명령은 리셋, 이네이블 등과 같은 주요 제어 입력에 주석을 달지 않으면 기본 스위칭 액티비티 값을 상속하게 됩니다. 이로 인해 동적 전력이 비관적으로 높아집니다. 순차 출력 핀의 전파된 스위칭 액티비티는 토글 비율이 거의 0에 가까워져 동적 전력이 낙관적으로 낮아질 수 있습니다. `​infers_switching_activity` 명령은 설계의 특수 제어 입력 핀(SCI)의 드라이버에 대한 스위칭 액티비티를 추론합니다.
{: .notice--warning}  

## `infer_switching_activity`

![2023-05-13-infer-swtiching-activity.png]({{site.baseurl}}/assets/images/2023-05-13-infer-swtiching-activity.png){: .align-center}  

Identifies essential points (EPs) and infers an activity  

- EPs that drive **special control inputs(SCIs)** such as resets and enables are made **static high/low**
- EPs that do not drive SCIs receive **inferred switching activity**
  - The inferred switching activity is calculated using the settings of the `power.default_*` app options
  - Inferred switching activity overrides the propagated activity

Infer switching activity identifies the essential points and infers an activity. Primary Input; Primary in out – the 'in' part of the port; Sequential outputs – not including sequential ICG cells; Tri-state output; No-func – an output pin without a function describing its behaviour, that is, black box; No-driver – a net without a driver form Essential points. Reset; Set; Scan Enable; Isolation control; Power switch control; ICG enable, and retention control that are not SCIs form Special control inputs. An appropriate static high or low setting is applied to these EPs to ensure that no resetting, setting, clamping, or sleeping occurs. Based on the majority endpoint behaviour: If, for example, seven of the eight SCIs that are driven by an EP are active low reset pins, and one of them is active high, then infer switching activity will apply a toggle rate of 0, and a static probability of 1, so that the majority if the SCIs are not in a reset state.​ ​  
추론 스위칭 액티비티는 핵심 포인트를 식별하고 액티비티를 추론합니다. 주요 입력; 주요 입력 출력 - 포트의 'in' 부분; 순차 출력 - 순차 ICG 셀은 포함하지 않음; 삼상 출력; No-func - 동작을 나타내는 기능이 없는 출력 핀, 즉 블랙 박스; No-driver - 드라이버가 없는 넷인 필수 포인트입니다. 리셋; 설정; 스캔 이네이블; 고립 제어; 전원 스위치 제어; ICG 이네이블 및 보유 제어는 특수 제어 입력이 아닌 경우입니다. 이러한 EP(핵심 포인트)에는 초기화, 설정, 클램핑 또는 슬립이 발생하지 않도록 적절한 정적 고 또는 저 설정이 적용됩니다. 대부분의 엔드포인트 동작을 기준으로, 예를 들어 8개의 SCIs 중 7개가 활성화된 로우 리셋 핀이고 1개가 활성화된 하이 리셋 핀인 경우, 추론 스위칭 액티비티는 토글 비율을 0으로 적용하고 정적 확률을 1로 적용하여 대다수의 SCIs가 리셋 상태가 아님을 보장합니다.  
{: .notice--warning}  

## Recommended Setup for `infer_switching_activity`

- Use create clock to define clock activity
- Use set switching activity for any ports with known required activity

```tcl
current_scenario func.tt_60c
set_switching_activity -toggle_rate 0.02 -static_probability 0.7 [get_ports a]
set_switching_activity -toggle_rate 0.06 -static_probability 0.3 [get_ports b]
```

- Use `infer_switching_activity` for all remaining unannotated EPs

```tcl
set_scenario_status -dynamic_power true func.tt_60c
infer_switching_activity -scenarios func.tt_60c \
                         -sci_based all -apply -output inferred_activity.tcl
```

The infer switching activity command will not apply the inferred activity automatically without the specification of the -apply option. The user can execute the command without -apply to review inferred activity only, and they can output the inferred set switching activity commands to a file for sourcing separately. However, if the latter is done, the activity will be applied as 'annotated' activity rather than 'inferred' activity, and activity of type 'annotated' will have a higher precedence than 'inferred'. Consequently, any subsequent executions of infer switching activity will not change these values.​ ​ In addition to infer switching activity there is also report essential points, which can be used to determine what Essential Point is responsible for a particular control point in the design. Using this information can help the user determine what activity should be annotated onto the design.​ For the example, please refer the notes.  
추론 스위칭 액티비티 명령은 -apply 옵션의 지정 없이는 추론된 액티비티를 자동으로 적용하지 않습니다. 사용자는 -apply 없이 명령을 실행하여 추론된 액티비티를 검토할 수 있으며, 추론된 설정 스위칭 액티비티 명령을 별도로 소스화할 수 있는 파일로 출력할 수도 있습니다. 그러나 후자를 수행하는 경우, 액티비티는 '추론된' 액티비티가 아닌 '주석화된' 액티비티로 적용되며, '주석화된' 유형의 액티비티가 '추론된' 액티비티보다 우선 순위가 높습니다. 결과적으로, 이후에 수행되는 추론 스위칭 액티비티 명령은 이러한 값들을 변경하지 않습니다. 추론 스위칭 액티비티 외에도 특정 제어 포인트에 대해 어떤 핵심 포인트가 담당하는지 확인할 수 있는 report essential points도 있습니다. 이 정보를 사용하면 사용자는 설계에 어떤 액티비티를 주석 처리해야 할지 결정하는 데 도움을 받을 수 있습니다. 예시에 대해서는 참고 자료를 확인해주세요.
{: .notice--warning}  

## Multibit Banking

Merging (banking) single bit registers to Multibit registers leads to  

- Smaller area/power, due to shared transistors and optimized transistor- level layout

![2023-05-13-multibit-banking-1.png]({{site.baseurl}}/assets/images/2023-05-13-multibit-banking-1.png){: .align-center}  

- Reduction in total clock tree net length, clock tree power consumption, as well as the number of CT buffers

![2023-05-13-multibit-banking-2.png]({{site.baseurl}}/assets/images/2023-05-13-multibit-banking-2.png){: .align-center}  

Merging single bit registers to Multibit registers leads to smaller area and power. Reduction in total clock tree net length, clock tree power consumption. The picture on the top-left shows 1-bit master or slave D-FFs.​ Replacing single-bit cells with multibit cells offers the following benefits: •Reduction in area due to shared transistors and optimized transistor-level layout​. •Reduction in the total length of the clock tree net​. •Reduction in clock tree buffers and clock tree power.  
단일 비트 레지스터를 다중 비트 레지스터로 병합하면 면적과 전력이 줄어듭니다. 총 클럭 트리 넷 길이와 클럭 트리 전력 소비가 감소합니다. 왼쪽 상단의 그림은 1비트 마스터 또는 슬레이브 D 플립플롭을 보여줍니다. 다음은 단일 비트 셀을 다중 비트 셀로 대체하는 것이 가져다주는 이점입니다: •공유 트랜지스터 및 최적화된 트랜지스터 수준 레이아웃으로 인한 면적 감소 •클럭 트리 넷의 총 길이 감소 •클럭 트리 버퍼와 클럭 트리 전력의 감소  
{: .notice--warning}  

## Integrated Multibit Banking

Multibit (de)banking are integrated into the `place_opt` command

- Banking is performed during the `initial_opto` stage
  - Automatically identifies groups of single-bit cells based on physical proximity and replaces them with a multi-bit cell to reduce dynamic power without impacting timing
- Debanking is performed during the `final_opto` stage on critical sequential multibit cells to recover timing

```tcl
# default: false
place_opt.flow.enable multibit_banking
# default: false
place_opt.flow.enable multibit_debanking
```

Multibit banking and debanking is integrated in `place_opt` command, and can be enabled using the following app options. The banking step enabled using `place_opt.flow.enable_multibit_banking true` banks compatible single bit registers that are placed close to each other to multibit registers during initial_opto stage. The debanking step enabled using `place_opt.flow.enable_multibit_debanking true` debanks multibit registers in timing critical path to smaller multibit, or single bit registers to improve timing QoR during `final_opto` stage. These two app options should help provide out of the box QoR and banking ratio for your designs  
Multibit banking과 debanking은 `place_opt` 명령어에 통합되어 있으며, 다음과 같은 애플리케이션 옵션을 사용하여 활성화할 수 있습니다. `place_opt.flow.enable_multibit_banking true`를 사용하여 banking 단계를 활성화하면, initial_opto 단계에서 서로 가까이 배치된 호환 가능한 단일 비트 레지스터를 다중 비트 레지스터로 변환합니다. `place_opt.flow.enable_multibit_debanking true`를 사용하여 debanking 단계를 활성화하면, `final_opto` 단계에서 타이밍이 중요한 경로에서 다중 비트 레지스터를 더 작은 다중 비트 레지스터 또는 단일 비트 레지스터로 분해하여 타이밍 QoR을 개선합니다. 이 두 가지 애플리케이션 옵션은 설계에 대한 기본 QoR 및 banking 비율을 제공하는 데 도움이 될 것입니다.  
{: .notice--warning}  

## Configuring Multibit Banking

Multibit banking can be configured by using the command `set_multibit_option`. This command can have 3 options namely, `-exclude`, `-exclude_library_cell`, and `-slack_threshold`. Naming of multibit_banking can be controlled by using `multibit.naming.*` app option. Use `report_multibit` to report statistics about multi bit cells, and reasons for not banking or debanking. To know more about `multibit.naming.*` app option, refer the notes section.  
Multibit banking은 `set_multibit_option` 명령어를 사용하여 구성할 수 있습니다. 이 명령어에는 `-exclude`, `-exclude_library_cell` 및 `-slack_threshold`와 같은 3가지 옵션이 있습니다. multibit_banking의 명명은 `multibit.naming.*` 애플리케이션 옵션을 사용하여 제어할 수 있습니다. `report_multibit`을 사용하여 다중 비트 셀에 대한 통계 및 banking 또는 debanking이 발생하지 않는 이유를 보고할 수 있습니다. `multibit.naming.*` 애플리케이션 옵션에 대한 자세한 내용은 노트 섹션을 참조하십시오.  
{: .notice--warning}  

## Multibit library Checking Command

You can use `check_multibit_library` command to check if the multibit library cells have compatible single bit equivalents or not. This command in addition with `report_multibit` will help you debug banking ratio issues in your design.  
`check_multibit_library` 명령을 사용하여 다중 비트 라이브러리 셀이 호환되는 단일 비트 동등 셀을 가지고 있는지 확인할 수 있습니다. 이 명령은 `report_multibit`과 함께 사용하여 설계의 banking 비율 문제를 디버깅하는 데 도움이 됩니다.  
{: .notice--warning}  

## Tracking Design Transformations

- Design transformations can cause problems for logic equivalence checking(Formality) and are not automatically captured
  - Multibit transformations
  - Name changes (`change_names`, ...)
- Enable SVF tracking at the beginning of your flow:
  - `set_svf MY SVF.svf`
  - Creates the specified SVF file and captures design transformations from this point forward
  - The SVF file is read by Formality to help LEQ

Design transformations due to multibit transformation and name changes, can cause problems for logic equivalence checking, and are not automatically captured. Enable SVF tracking at the beginning of the flow to capture the design transformations. SVF file captures the design transformation due to multibit transformations and name changes. This file is read by Formality to help LEQ.  
다중 비트 변환 및 이름 변경으로 인해 발생하는 설계 변환은 논리 동등성 검사에 문제를 일으킬 수 있으며, 자동으로 포착되지 않습니다. 플로우 시작 시 SVF 추적을 활성화하여 설계 변환을 포착할 수 있습니다. SVF 파일은 다중 비트 변환 및 이름 변경으로 인한 설계 변환을 포착하는 데 사용됩니다. 이 파일은 동등성 검사를 위해 Formality에서 읽힙니다.  
{: .notice--warning}  

## Placement Attractions

- Placement attraction constraints allow more control over the general placement location of a large group of cells, e.g. modules
  - Can prevent fragmentation of modules
  - 
![2023-05-13-placement-attractions.png]({{site.baseurl}}/assets/images/2023-05-13-placement-attractions.png){: .align-center}  

Placement attractions should not be expected to always result in such excellent cohesion as shown above. There may be more spreading or fragmentation if that is beneficial for the design: Cells within placement attractions are allowed to be placed outside of their main collection if it is important for resolving timing or congestion.  
위에 표시된 것과 같은 탁월한 응집성이 항상 발생하는 것으로 기대해서는 안 됩니다. 디자인에 유리하다면 더 많은 확산 또는 조각화가 발생할 수 있습니다. 배치 의미 영역 내의 셀들은 타이밍이나 혼잡을 해결하는 데 중요하다면 주요 컬렉션 외부에 배치될 수 있습니다.  
{: .notice--warning}  

## When to Use Placement Attractions

- Only add attraction constraints where they are needed to achieve the desired result, for example to improve timing or congestion
- Attractions on too many modules can have a negative effect on the placement of other modules
- Attractions only work well during initial placement (`initial_place`), not incremental placement
  - If you add a new attraction, start placement again from scratch

We recommend to have only one placement attraction for better QoR. When you have multiple placement attractions for many modules, it may degrade QoR,because such too many constraints may conflict with original placement constraints. Only initial placement honour placement attraction, not incremental placement.  
우리는 더 좋은 QoR을 위해 하나의 배치 의미 영역을 가지는 것을 권장합니다. 여러 모듈에 대해 여러 개의 배치 의미 영역이 있는 경우 원래의 배치 제약과 충돌할 수 있으므로 QoR이 저하될 수 있습니다. 초기 배치 단계에서만 배치 의미 영역이 존중되며, 증분적인 배치 단계에서는 적용되지 않습니다.  
{: .notice--warning}  

## Usage and Recommendations

- Placement attractions are well suited for a large number of cells (e.g. modules), use bounds for a small group of cells

```tcl
create_placement_attraction -name risc module \
    -effort medium [get_cells I_RISC_CORE]
place_opt
```

- Cells in a placement group are attracted towards each other, but the group can be placed anywhere in the core, i.e. floating (by default)
  - Use the `-region` option to specify one or two coordinates to define a point, an edge or a rectangular region towards which the cells are attracted, as needed
  - `[ -region {{ 0 0} {0 1500}} ]`
  - See man page for additional options and command details

The `create_placement_attraction` command is a much "softer" placement constraint than the move bound, or group bound constraints created by the `create_bound` command. The placement attraction is used to tell the placer about the module relationships that are important, without the need to strongly constrain every cell in each module. In general, the placement attractions will steer the placer in the right direction, but it will also trade off with other considerations like wire length and timing. The region may be exactly one of the following: unset, point, line, rectangle. The argument takes a coordinate list and parses it to determine the desired region. If the user does not specify a value the region is unset: Cells in the cell list are attracted towards each other, but the group can be placed anywhere in the core area. A coordinate list with one point corresponds to a point region, and cells are attracted towards the point. A coordinate list with two colinear points (vertical or horizontal line) corresponds to an edge region, and cells are attracted towards the line. A coordinate list with two non-colinear points corresponds to a rectangular region. Cells will be attracted towards this region (but are not forced to be inside the region, like create bound).  
`create_placement_attraction` 명령은 `create_bound` 명령으로 생성된 이동 경계 또는 그룹 경계 제약 조건보다 훨씬 "유연한" 배치 제약 조건입니다. 배치 의미 영역은 각 모듈의 모든 셀을 강력하게 제약하는 것이 아니라 중요한 모듈 간 관계에 대해 플레이서에게 알려주는 데 사용됩니다. 일반적으로 배치 의미 영역은 플레이서를 올바른 방향으로 유도하지만, 전선 길이와 타이밍과 같은 다른 고려 사항과 균형을 맞추게 됩니다. 영역은 다음 중 하나일 수 있습니다: unset, point, line, rectangle. 인수는 좌표 목록을 받아 해당하는 영역을 결정하기 위해 구문 분석합니다. 사용자가 값을 지정하지 않은 경우 영역은 unset입니다: 셀 목록의 셀은 서로에게서 끌어당기지만, 그룹은 코어 영역 어디에나 배치될 수 있습니다. 한 점으로 이루어진 좌표 목록은 점 영역에 해당하며, 셀은 해당 점에 끌어당겨집니다. 두 개의 일직선 상의 점으로 이루어진 좌표 목록(수직 또는 수평선)은 가장자리 영역에 해당하며, 셀은 해당 선에 끌어당겨집니다. 두 개의 일직선 상에 있지 않은 점으로 이루어진 좌표 목록은 직사각형 영역에 해당합니다. 셀은 이 영역으로 끌어당겨질 수 있지만(하지만 `create_bound`와 달리 영역 내에 강제로 위치할 필요는 없습니다).
{: .notice--warning}  
