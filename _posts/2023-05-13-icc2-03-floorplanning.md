---
title: "IC Compiler II - 03 Introduction to Block-Level Floorplanning"
excerpt: "Block-level Implementation"
date: 2023-05-13 02:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
---

Welcome to the IC compiler II Block-level Implementation self-paced training. Now we will discuss the block level floorplanning module.  
IC Compiler II 블록 수준 구현 자기주도 트레이닝에 오신 것을 환영합니다. 이제 블록 수준 폴로플래닝 모듈에 대해 이야기하겠습니다.
{: .notice--warning}

## Agenda

This module will introduce you to the basic block level floorplanning operations from creating the initial shape of your design to placing voltage areas, macros and block pins, and finally performing congestion analysis. It will also discuss on how to export the floorplan data for re synthesis. Let's get started.  
이 모듈에서는 디자인의 초기 모양 생성부터 전압 영역, 매크로 및 블록 핀 배치, 그리고 혼잡도 분석까지 기본적인 블록 수준 폴리플래닝 작업을 소개합니다. 또한 폴리플랜 데이터를 재합성을 위해 내보내는 방법에 대해서도 다룰 것입니다. 시작해봅시다.
{: .notice--warning}  

## Objectives

After completing this module, you should be able to describe some key block-level floorplanning steps. The module does not cover complete design planning. If you need to learn about design planning, there is a dedicated training that discusses the entire hierarchical design planning flow. That training is available as eLearning as well.  
이 모듈을 완료한 후에는 몇 가지 중요한 블록 수준 플로어플랜 단계를 설명할 수 있을 것입니다. 그러나 이 모듈은 전체적인 디자인 플래닝을 다루지는 않습니다.  
만약 디자인 플래닝에 대해 학습하고자 한다면, 전체적인 계층적 디자인 플래닝 플로우에 대해 다루는 전용 트레이닝이 있습니다. 해당 트레이닝은 eLearning 형태로도 제공되고 있습니다.  
{: .notice--warning}  

## IC Compiler II Flow

![2023-05-12-icc-flow.png]({{site.baseurl}}/assets/images/2023-05-12-icc-flow.png){: .align-center}  

This flow shows the basic design implementation flow used in IC Compiler II tool. Flow starts with design and timing setup requirements to meet the design constraints. Next stage in the flow is floorpalnning where you can define the shape of the design/block, create the voltage areas, perform pin placements, macro placements. You will define the power mesh for your design/block. Next you will perform the placement optimization, clock tree synthesis and clock tree optimization by using the command `place_opt` and `clock_opt` respectively. After building the clock tree, perform the detailed signal nets routing by using  the command `route_auto` then perform the post route optimization by using `route_opt` command. Finally perform the signoff checks on the design.  
이 플로우는 IC Compiler II 도구에서 사용되는 기본적인 디자인 구현 플로우를 보여줍니다. 플로우는 디자인 제약 조건을 충족하기 위한 디자인 및 타이밍 설정 요구사항으로 시작합니다. 플로우의 다음 단계는 플로어플랜닝입니다. 여기서 디자인/블록의 모양을 정의하고, 전압 영역을 생성하고, 핀 배치와 매크로 배치를 수행할 수 있습니다. 또한 디자인/블록에 대한 전원 메시를 정의할 수 있습니다. 다음으로, `place_opt` 및 `clock_opt` 명령을 사용하여 배치 최적화와 클럭 트리 합성 및 클럭 트리 최적화를 수행합니다. 클럭 트리를 구축한 후, `route_auto` 명령을 사용하여 신호 넷의 상세한 라우팅을 수행하고, `route_opt` 명령을 사용하여 후처리 최적화를 수행합니다. 마지막으로, 디자인에 대한 사이널 오프 체크를 수행합니다.
{: .notice--warning}  

## Floorplanning Overview

![2023-05-12-floorplan-overview.png]({{site.baseurl}}/assets/images/2023-05-12-floorplan-overview.png){: .align-center}  

This unit provides an overview of some of the key steps in floorplanning:  

- Defining
  - Block shape/size
  - Voltage area shapes/locations
  - Macro cell locations
  - I/O pin locations
  - Standard cell placement blockages (to address congestion)
- Overview of PNS
- Exporting the floorplan

Now for a quick overview of what Floorplanning entails. Let's first look at the green box in the flow chart, when you run synthesis without providing any physical information, this is where auto-floorplan comes in, it will help to create the missing physical information. You may ask why you still need to do floorplanning if auto-floorplanning has already created one? Bear in mind, auto-floorplan is designed to have a good enough floorplan to be used as a starting point, but not perfect for implementation. For example, auto-floorplanning does not perform any power mesh creation. Typically, floorplanning happens after you have a netlist. The key floorplanning steps are shown in the blue box. You start by defining the block size and shape, followed by placing voltage areas and hard macros. You then place the IO pins or block ports. Optionally, you can apply placement blockages to help fend off congestion issues. Next, you build the power distribution network and finally export the floorplan for later reuse.  
이제 Floorplanning에 대해 간략히 살펴보겠습니다. 먼저 플로우 차트의 초록색 상자를 살펴보겠습니다. 물리적인 정보를 제공하지 않고 합성을 실행할 때, 이곳에서 자동 플로어플랜(auto-floorplan)이 필요합니다. 이는 누락된 물리적인 정보를 생성하는 데 도움을 줄 것입니다. 이미 자동 플로어플랜이 생성되었는데 왜 여전히 플로어플랜을 수행해야 할까요? 자동 플로어플랜은 시작점으로서 충분히 좋은 플로어플랜을 생성하도록 설계되었지만, 구현에 완벽하지는 않습니다. 예를 들어, 자동 플로어플랜은 전원 메시를 생성하지 않습니다. 일반적으로 플로어플랜은 넷리스트를 보유한 후에 수행됩니다. 주요한 플로어플랜 단계는 파란색 상자에 표시되어 있습니다. 먼저 블록의 크기와 모양을 정의한 후, 전압 영역과 하드 매크로를 배치합니다. 그런 다음 IO 핀이나 블록 포트를 배치합니다. 필요한 경우 혼잡 문제를 해결하기 위해 배치 블록을 적용할 수도 있습니다. 그 다음, 전원 분배 네트워크를 구축하고, 마지막으로 플로어플랜을 내보내어 나중에 재사용할 수 있도록 합니다.  
{: .notice--warning}  

## Create the Initial Floorplan

![2023-05-12-initial-floorplan.png]({{site.baseurl}}/assets/images/2023-05-12-initial-floorplan.png){: .align-center}

- Floorplan Initialization
  - Defines standard cell placement site array within the core area
  - Supports various core shapes:
    - Rectangular
    - L-shape
    - T-shape
    - U-shape
    - Custom

```tcl
initialize_floorplan -shape U -...
```

The very first step is to create the initial floorplan. Floorplan initialization can be performed using a Tcl command or using the GUI. In the screenshot you can see that a U-shaped design is created that's a U that is turned to the left which corresponds to the orientation "West" as you can see in the dialog. The core area is the yellow outline and then you see a purple outline which is the die area, or the block boundary, and that's typically where the pins and ports or IO ports are going to be placed. As you can see in the dialog, you can also create rectangular, L, or T-shaped floorplans. You can also create a custom shaped rectilinear floorplan by selecting "Custom" at the top of the dialog and entering the boundary coordinates. Review the notes for an example. How did we come up with the size of these edges? If you look under the shape and orientation, you'll see "Side size control" – and there two settings: Ratio and Length. If you click on length, then you would be able to enter in microns the length of all the edges. In this example, we used the Ratio method, and there you specify a core utilization, in this case 0.7 or 70%. The tool will add up the areas of all the standard cells and hard macros in your design, and will then calculate a core area in which these standard cells and macros occupy 70% of that total area, so you will have 30% extra room. You don't want to start with 100% utilization because we're going to be adding a bunch of cells for Clock tree synthesis, for hold fixing and various optimizations. So, we need room to grow. Just below the size control, you can apply core side ratios to shape that block level floor plan. The labels in the dialog are reflected in the GUI on the right. Further down, you can specify a spacing to apply between the block boundary or die and the core. Here, it is 40. To get a rough idea of how your floorplan will look like, you can click on the "Preview" button on the right side. Once you're done with the settings, you can press on "Apply" on the bottom to initialize the floorplan or on script to copy the `initialize_floorplan` command that performs this operation to your script editor.  
가장 처음 해야 할 단계는 초기 플로어플랜을 생성하는 것입니다. 플로어플랜 초기화는 Tcl 명령이나 GUI를 사용하여 수행할 수 있습니다. 스크린샷에서는 좌측으로 휘어진 U 모양의 디자인이 생성된 것을 볼 수 있습니다. 이는 대화상자에서 확인할 수 있는 "서쪽(West)" 방향과 해당하는 방향입니다. 노란색 윤곽은 코어 영역이며, 보라색 윤곽은 다이 영역 또는 블록 경계입니다. 일반적으로 핀이나 포트 또는 IO 포트가 배치될 곳입니다. 대화상자에서 확인할 수 있듯이 직사각형, L자형, T자형 플로어플랜도 생성할 수 있습니다. 또한 대화상자 상단에서 "사용자 정의"를 선택하고 경계 좌표를 입력함으로써 사용자 지정 모양의 사각형 플로어플랜을 생성할 수도 있습니다. 예제는 참고 사항을 참조하세요. 이러한 변의 크기는 어떻게 결정되었을까요? 모양과 방향 아래에 보면 "Side size control"이라는 것을 볼 수 있습니다. 이 설정에는 두 가지 옵션이 있습니다: Ratio와 Length입니다. Length를 클릭하면 마이크론 단위로 모든 변의 길이를 입력할 수 있습니다. 이 예제에서는 Ratio 방식을 사용했으며, 여기에서 코어 활용도(이 경우 0.7 또는 70%)를 지정합니다. 도구는 디자인의 모든 스탠다드 셀과 하드 매크로의 면적을 더한 후, 해당 전체 면적의 70%를 차지하는 코어 영역을 계산합니다. 따라서 30%의 여유 공간이 있게 됩니다. 100% 활용도로 시작하지 않는 이유는 클럭 트리 합성, 홀드 수정 및 기타 최적화를 위해 많은 셀을 추가할 예정이기 때문에 확장 공간이 필요하기 때문입니다. 크기 제어 바로 아래에서는 블록 레벨 플로어플랜을 구성하기 위해 코어 측면 비율을 적용할 수 있습니다. 대화상자의 레이블은 오른쪽의 GUI에 반영됩니다. 더 아래쪽에서는 블록 경계 또는 다이와 코어 사이에 적용할 간격을 지정할 수 있습니다. 여기서는 40으로 설정되어 있습니다. 플로어플랜의 대략적인 모습을 확인하려면 오른쪽에 있는 "미리 보기" 버튼을 클릭할 수 있습니다. 설정이 완료되면 아래쪽에 있는 "적용" 버튼을 눌러 플로어플랜을 초기화하거나, 스크립트 에디터로 해당 작업을 수행하는 `initialize_floorplan` 명령을 복사하여 스크립트에 적용할 수 있습니다.  
{: .notice--warning}  

## Floorplan After Initialization

![2023-05-12-floorplan-after-init.png]({{site.baseurl}}/assets/images/2023-05-12-floorplan-after-init.png){: .align-center}  

Initializing the floorplan creates the boundary and also creates the site rows as well as the routing tracks based on your technology file. Here I'm showing what you will see in the GUI after design initialization. You will see the initial design on the right, and all macros on the left. If you zoom in to say the lower left corner of the design, you will see something similar to that shown on the right. You will see the edge of the block or die and the edge of the core. The distance or spacing is what you chose on the dialog from the previous page. You can also see the site array which contains your site rows. Later, the standard cells will be placed in these site rows. You only need to manipulate site arrays. Site rows will automatically be created or adjusted for you inside the site arrays. By default, the size of the site array will correspond to the size of the core. If you need to create multiple site arrays, you can do this using the site array manipulation commands. This is a powerful feature: You can create multiple arrays, overlay them, and control what happens when site arrays overlap. The tool will use the default site definition from your tech file to create the site array. If you want to use a different site definition, you can specify that when initializing the floorplan. On the bottom right I'm showing you the settings to enable the visibility of the site array as well as site rows.  
플로어플랜을 초기화하면 경계를 생성하고, 사이트 로우 및 라우팅 트랙도 기술 파일을 기반으로 생성됩니다. 여기에서는 디자인 초기화 후 GUI에서 볼 수 있는 내용을 보여드리고 있습니다. 오른쪽에는 초기 디자인이 표시되고, 왼쪽에는 모든 매크로가 표시됩니다. 디자인의 왼쪽 하단 모서리로 확대해 보면, 오른쪽에 표시된 것과 유사한 내용을 볼 수 있습니다. 블록이나 다이의 가장자리와 코어의 가장자리를 볼 수 있습니다. 거리 또는 간격은 이전 페이지의 대화상자에서 선택한 값입니다. 또한 사이트 로우를 포함하는 사이트 배열을 볼 수 있습니다. 이후에는 스탠다드 셀이 이 사이트 로우에 배치될 것입니다. 사이트 배열만 조작하면 됩니다. 사이트 로우는 자동으로 사이트 배열 내에서 생성되거나 조정됩니다. 기본적으로 사이트 배열의 크기는 코어의 크기에 해당합니다. 여러 개의 사이트 배열을 생성해야 하는 경우, 사이트 배열 조작 명령을 사용할 수 있습니다. 이는 강력한 기능으로, 여러 배열을 생성하고 겹치는 경우에 어떻게 처리할지를 제어할 수 있습니다. 도구는 기술 파일에서 기본 사이트 정의를 사용하여 사이트 배열을 생성합니다. 다른 사이트 정의를 사용하려면 플로어플랜을 초기화할 때 지정할 수 있습니다. 우측 하단에는 사이트 배열 및 사이트 로우의 가시성을 활성화하기 위한 설정을 보여드리고 있습니다.  
{: .notice--warning}  

## Create Voltage Areas(for MV Designs)

![2023-05-12-create-voltage-area.png]({{site.baseurl}}/assets/images/2023-05-12-create-voltage-area.png){: .align-center}  

So now let's talk about voltage areas. In multi voltage designs you will load something called UPF, that is Unified Power Format. In that file you start off by defining power domains, each power domain can have its own power and ground definitions. These power domains can be always on or they could be shut-down power regions. If they are shut down, then you may or may not need always on logic and retention registers to retain certain values of certain registers etc. So, all of this is defined in UPF. But the UPF power domains don't have any physical definitions, so it's just a logical construct. For the tool to know where to place the cells that belong to the various power domains, it needs a physical representation of the power domain, which we call voltage area. The tool will always create a default voltage area called "Default VA" which corresponds to the top level UPF power domain. In the example you're seeing, the top-level power domain is called PD_Top. For all other power domains in the example, PD1 and PD2, you have to create voltage areas. This can be done by either creating the voltage areas manually using the `create_voltage_area` command or by letting automatic block shaping create and place the voltage areas.  
이제 전압 영역에 대해 이야기해 보겠습니다. 멀티 전압 디자인에서는 통합 전력 형식인 UPF(Unified Power Format)라는 것을 로드하게 됩니다. 이 파일에서는 전력 도메인을 정의하며, 각 전력 도메인은 고유한 전원 및 접지 정의를 가질 수 있습니다. 이 전력 도메인은 항상 켜져 있을 수도 있고, 종료된 전력 영역일 수도 있습니다. 종료된 경우에는 항상 켜진 로직 및 보유 레지스터를 사용하여 특정 레지스터의 값을 보존해야 할 수도 있습니다. 이러한 모든 것들이 UPF에서 정의됩니다. 하지만 UPF 전력 도메인은 물리적인 정의를 가지지 않으므로, 그저 논리적인 구조입니다. 도구가 각 전력 도메인에 속하는 셀을 배치할 위치를 알기 위해서는 전력 도메인의 물리적 표현이 필요한데, 이를 전압 영역이라고 합니다. 도구는 항상 "Default VA"라는 기본 전압 영역을 생성합니다. 이는 최상위 UPF 전력 도메인에 해당합니다. 예시에서는 최상위 전력 도메인이 PD_Top라고 불리고 있습니다. 예시의 다른 전력 도메인인 PD1과 PD2에 대해서는 전압 영역을 직접 생성해야 합니다. 전압 영역은 `create_voltage_area` 명령을 사용하여 수동으로 생성하거나, 자동 블록 형성이 전압 영역을 생성하고 배치하도록 할 수 있습니다.  
{: .notice--warning}  

## Automatic VA Creation: "Shape Blocks"

![2023-05-12-automatic-va-creation.png]({{site.baseurl}}/assets/images/2023-05-12-automatic-va-creation.png){: .align-center}  

Creating the voltage areas is considered a floorplanning task. For automatic voltage area creation and placement, you use the `shape_blocks` command. As you see in the screenshot on the right, `shape_blocks` has created an additional voltage area for the power domain `PD_RISC_CORE`. By default, the name of the voltage area is the same as the Power Domain's name. The voltage area was also sized and placed. To calculate the size, the total area of the standard cells and macros that are part of that voltage area is taken into account, using the design's overall utilization to calculate the final area. Using the design's connectivity, it can also determine the best suited location for the voltage area. There are some other constraints you can set for voltage areas using `set_shaping_options`. You can use the option `-guard_band_size` to create a space around the boundary of the voltage area where cell placement is not allowed. This is generally used to make sure that the cells from the top level don't get too close to the block voltage area to simplify the creation of the PG mesh.  
전압 영역을 생성하는 것은 플로어플랜 작업으로 간주됩니다. 자동 전압 영역 생성과 배치를 위해 `shape_blocks` 명령을 사용합니다. 오른쪽의 스크린샷에서 보시는 것처럼, `shape_blocks`가 `PD_RISC_CORE` 전력 도메인을 위한 추가 전압 영역을 생성했습니다. 전압 영역의 이름은 기본적으로 전력 도메인의 이름과 동일합니다. 전압 영역은 크기와 위치도 결정되었습니다. 크기를 계산하기 위해서는 해당 전압 영역에 포함된 스탠다드 셀과 매크로의 총 면적을 고려하며, 디자인의 전체 활용도를 사용하여 최종 면적을 계산합니다. 디자인의 연결성을 사용하여 전압 영역에 가장 적합한 위치를 결정할 수도 있습니다. `set_shaping_options`를 사용하여 전압 영역에 대한 다른 제약 조건을 설정할 수도 있습니다. `-guard_band_size` 옵션을 사용하여 전압 영역의 경계 주변에 셀 배치가 허용되지 않는 공간을 생성할 수 있습니다. 이는 일반적으로 상위 레벨에서의 셀들이 블록 전압 영역과 너무 가까워지지 않도록 하여 PG 메시의 생성을 간소화하는 데 사용됩니다.  
{: .notice--warning}  

## Place Macros and Standard Cells

- Perform coarse placement of macros and standard cells

```tcl
create_placement -floorplan [-congestion ...]
```

![2023-05-12-place-macros-and-cells.png]({{site.baseurl}}/assets/images/2023-05-12-place-macros-and-cells.png){: .align-center}  

After the voltage areas have been shaped and placed, it's time to place the macros and standard cells. The primary goal of this early placement is to find the best location for the hard macros, that is your memories and other IP blocks. It's not so much about placing standard cells, although the standard cells will somewhat affect the placement of the macros. The placement performed is coarse, meaning they will not be legalized. If you're using the GUI or your task assistant, you can see that the floorplan setting is enabled. In addition, you can also enable congestion if congestion is expected in your design. You can control the efforts for placement in general and for congestion using the pull-down menus.  
전압 영역을 형성하고 배치한 후에는 매크로와 스탠다드 셀을 배치할 차례입니다. 이 초기 배치의 주요 목표는 메모리 및 기타 IP 블록인 하드 매크로에 가장 적합한 위치를 찾는 것입니다. 스탠다드 셀을 배치하는 것은 그다지 중요하지 않지만, 스탠다드 셀은 매크로의 배치에 약간 영향을 미칠 수 있습니다. 배치는 거칠게 수행되며, 즉, 합법화되지 않을 것입니다. GUI나 작업 도우미를 사용하고 있다면, 플로어플랜 설정이 활성화되어 있는 것을 확인할 수 있습니다. 또한, 디자인에서 혼잡이 예상된다면 혼잡도도 활성화할 수 있습니다. 배치 및 혼잡에 대한 노력을 일반적으로 및 혼잡에 대해 드롭다운 메뉴를 사용하여 제어할 수 있습니다.  
{: .notice--warning}  

## Configuring Macro and Standard Cell Placement

![2023-05-12-config-macro-cell-place.png]({{site.baseurl}}/assets/images/2023-05-12-config-macro-cell-place.png){: .align-center}  

There are several application options that affect placement. They are prefixed with `plan.place` and `plan.macro`. Generally, it is recommended to start with the defaults. Let the tool come up with a solution. Then you can analyze that solution and see if it meets your needs. After that, decide on what options to modify. The screenshot is the result of a standard `create placement -floorplan` from the lab design, which you will use later. As you see, Macros will be grouped into arrays if they are at the same hierarchy and have the same reference. This is controlled using the `auto_macro_array` application option shown. Also, the macros will be flipped such that the pins face each other. Most memories will have their pins on a single side, so with this feature the number of channels needed to route to the pins is reduced. As you can see in the highlighted area, the Placer will increase the spacing in the channels that have pins. The spacing is determined automatically based on pin count, but you can apply your own minimum distance - review the notes for more information. Something else you will notice is that `create placement`, adds placement blockages to the design. If you look at the highlighted areas, these are typically areas where you don't want to place standard cells. If standard cells are allowed in these channels, you might run into problems routing to them. Therefore, generally, when you're doing a floorplan manually, you exclude such areas for standard cell placement. `create_placement -floorplan` does this for you automatically, and that is controlled by an application option called `plan.place.auto_generate_blockages`. If you have a macro which needs to be placed at a certain location, maybe a PLL or some other special macro, then you can pre-place this macro manually and fix it. `create_placement -floorplan` will not move a fixed macro.  
배치에 영향을 주는 여러 가지 응용 프로그램 옵션이 있습니다. 이들은 `plan.place`와 `plan.macro`로 접두사가 붙습니다. 일반적으로 기본값으로 시작하는 것이 권장됩니다. 도구에게 솔루션을 제시하도록 하고, 그 솔루션을 분석하여 요구사항을 충족시키는지 확인한 후에 옵션을 수정할지 결정하면 됩니다. 스크린샷은 뒤에서 사용할 랩 디자인의 표준 `create placement -floorplan` 결과입니다. 보시는 것처럼, 동일한 계층에 속하고 같은 참조를 가진 경우 매크로들은 배열로 그룹화됩니다. 이는 `auto_macro_array`라는 응용 프로그램 옵션으로 제어됩니다. 또한, 매크로들은 핀이 서로 마주하도록 뒤집힙니다. 대부분의 메모리는 핀이 한쪽에 있기 때문에, 이 기능으로 핀에 라우팅해야 하는 채널 수를 줄일 수 있습니다. 강조된 영역에서 확인할 수 있듯이, 플레이서는 핀이 있는 채널에서 간격을 늘립니다. 간격은 핀 개수에 따라 자동으로 결정되지만, 최소 거리를 직접 적용할 수도 있습니다. 자세한 내용은 참고 자료를 확인하십시오. 또 다른 사항은 `create placement`가 디자인에 배치 차단을 추가한다는 것입니다. 강조된 영역을 살펴보면, 여기에는 일반적으로 스탠다드 셀을 배치하지 않고자 하는 영역이 있습니다. 이러한 채널에 스탠다드 셀이 허용된다면, 해당 셀에 라우팅하는 데 문제가 발생할 수 있습니다. 따라서 일반적으로 수동으로 플로어플랜을 수행할 때는 이러한 영역을 스탠다드 셀 배치에서 제외합니다. `create_placement -floorplan`은 이를 자동으로 수행하며, 이는 `plan.place.auto_generate_blockages`라는 응용 프로그램 옵션으로 제어됩니다. 특정 위치에 배치해야 하는 매크로(예: PLL 또는 기타 특수한 매크로)가 있는 경우, 해당 매크로를 수동으로 사전에 배치하고 고정할 수 있습니다. `create_placement -floorplan`은 고정된 매크로를 이동시키지 않습니다.
{: .notice--warning}  

## Macro Placement Styles - On-Edge vs. FreeForm

![2023-05-12-macro-place-style.png]({{site.baseurl}}/assets/images/2023-05-12-macro-place-style.png){: .align-center}  

- By default, macros are placed around the core (on-edge)
- You can also use freeform macro placement, which performs concurrent macro
and standard cell placement, generally leading to:
  - Reduction in wirelength
  - Reduction in power
  - Better timing

```tcl
set_app_options -name plan.macro.style \
                -value freeform
create_placement -floorplan
```

By default macros are placed around the core edge leads to uniform area in the core for standard cell placement as well as gives better congestion. In this approach first you will completely finish the macro placement and then perform the standard cell placement. Another way is you can also use the freeform macro placement, where macro placement and standard cell placement are carried out parallelly. This approach is leads to following advantages; Wire length reduction, power reduction and improved timing QoR. To enable this feature in the flow set plan.macro.style app option to freeform then create the floorplan.  
기본적으로 매크로는 코어 가장자리 주변에 배치되어 스탠다드 셀 배치를 위한 균일한 코어 영역을 제공하고 혼잡도를 개선합니다. 이 방식에서는 먼저 매크로 배치를 완전히 완료한 다음 스탠다드 셀 배치를 수행합니다. 또 다른 방법은 프리폼 매크로 배치를 사용하는 것입니다. 이 방식에서는 매크로 배치와 스탠다드 셀 배치가 병렬로 수행됩니다. 이러한 접근 방식은 다음과 같은 이점을 제공합니다. 와이어 길이 감소, 전력 감소 및 시간 QoR 향상. 이 기능을 활성화하려면 플로우에서 plan.macro.style 응용 프로그램 옵션을 freeform으로 설정한 다음 플로어플랜을 생성하면 됩니다.  
{: .notice--warning}  

## Detailed Macro Constraints

- There are many more possibilities for constraining macro placement. Review the documentation for the following commands if required:
  - `set_macro_constraints`
  - `set_macro_relative_location`
  - `create_macro_array`
  - `create_macro_relative_location_placement`

There are more commands and constraints you can set on macros. Here, listing a few of the commands you might want to investigate if you run into problems with your Macros placement. Some examples are `set_macro_constraints` and `set_macro_relative_location`. Also have a look at the relative location placement. With that, you can create relative constraints from an existing placement of macros and you can store these relative placement constraints to a file and reuse them later. For example, if you change the shape of the design, if you enlarge or shrink the block, you can use the same relative constraints to come up with a very similar macro placement.  
매크로에 대해 설정할 수 있는 더 많은 명령과 제약 조건이 있습니다. 다음은 매크로 배치에 문제가 발생한 경우 조사해볼 수 있는 일부 예시입니다. `set_macro_constraints`와 `set_macro_relative_location` 등이 있습니다. 또한, 상대 위치 배치에도 주목해보세요. 이를 통해 기존 매크로 배치에서 상대적인 제약 조건을 생성할 수 있으며, 이러한 상대적인 배치 제약 조건을 파일로 저장하여 나중에 재사용할 수 있습니다. 예를 들어, 디자인의 형태를 변경하거나 블록의 크기를 확장하거나 축소하는 경우, 동일한 상대적인 제약 조건을 사용하여 매우 유사한 매크로 배치를 만들 수 있습니다.  
{: .notice--warning}  

## Analyze Placement of Macros using Data Flow Flylines(DFF)

![2023-05-12-dff.png]({{site.baseurl}}/assets/images/2023-05-12-dff.png){: .align-center}  

A great way to analyze the macro placement quality is to perform dataflow flyline analysis or DFF. The normal connectivity tool will show a connection between two objects, if the net connects directly or through buffers and inverters. DFF, on the other hand, will also show a connection even if the path goes through combinational cells, for example and gates, or gates exor gates and even through registers, if configured. This is quite powerful because you can understand the data flow between your macros and whether you need to place the macros or groups of macros close to each other or not. To use this tool, first the connectivity in the entire design needs to be calculated. Depending on the design size and how many register levels you want to pre-calculate, this can take some time. The reload button on the top launches the dialog to perform this pre-calculation. After that you can begin selecting macros and you will see their fly lines update in real-time. You can change the number of register as well as gate levels to trace through without incurring any extra compute time. This can be useful because in some cases you might want to see connectivity only through combinational logic or only through one or two levels of registers. If the macros connect through more levels of registers, it's probably fine to have them placed further away. Note that the fly lines you see in the GUI will never terminate or start in standard cells. Connectivity is only shown to and from macros, ports, physical blocks, and module boundaries if available.  
매크로 배치 품질을 분석하는 훌륭한 방법은 데이터플로우 플라이라인 분석 또는 DFF를 수행하는 것입니다. 일반적인 연결 도구는 직접 연결되거나 버퍼와 인버터를 통해 연결된 경우 두 개체 간의 연결을 보여줍니다. 반면에 DFF는 조합 셀(예: AND 게이트 또는 XOR 게이트)을 통해 경로가 흐르는 경우에도 연결을 보여줍니다. 또한, 설정에 따라 레지스터를 통해서도 연결을 보여줍니다. 이는 매크로 간의 데이터 플로우를 이해하고, 매크로를 서로 가까이 또는 멀리 배치해야 하는지 여부를 파악하는 데 매우 강력합니다. 이 도구를 사용하려면 먼저 전체 디자인의 연결성을 계산해야 합니다. 디자인의 크기와 사전 계산하려는 레지스터 수준에 따라 시간이 소요될 수 있습니다. 상단의 "Reload" 버튼은 이 사전 계산을 수행하는 대화 상자를 시작합니다. 이후 매크로를 선택하면 실시간으로 플라이라인이 업데이트됩니다. 레지스터 및 게이트 수준을 변경하여 추가 계산 시간 없이 추적할 수 있는 수준을 조정할 수 있습니다. 이는 어떤 경우에는 조합 논리로만 연결을 보고 싶을 수도 있고, 레지스터 수준으로만 연결을 보고 싶을 수도 있습니다. 매크로가 더 많은 레지스터 수준을 통해 연결되는 경우, 더 멀리 배치되어도 괜찮습니다. GUI에서 보이는 플라이라인은 스탠다드 셀에서 시작되거나 종료되지 않습니다. 연결성은 매크로, 포트, 물리 블록 및 모듈 경계와만 연결되어 표시됩니다(있는 경우).  
{: .notice--warning}  

## Register Tracing

![2023-05-12-register-tracing.png]({{site.baseurl}}/assets/images/2023-05-12-register-tracing.png){: .align-center}  

Another useful GUI tool is called register tracing. This will show you the connectivity from an object to one or more levels of registers, and as you see in the screenshot, you can have each level displayed in a different color. This tool to be more useful after standard cell placement not so much after floorplanning because this tells more about the standard cell placement quality then about macros, as is the case with DFF. Nevertheless, it's a good technique to determine whether you may need to move macros apart to create more room for the standard cells.  
다른 유용한 GUI 도구는 레지스터 추적(Register Tracing)입니다. 이 도구를 사용하면 개체에서 하나 이상의 레지스터 수준까지의 연결성을 보여주며, 스크린샷에서 볼 수 있듯이 각 수준을 다른 색상으로 표시할 수 있습니다. 이 도구는 플로어플랜보다는 스탠다드 셀 배치에 더 유용합니다. 왜냐하면 이 도구는 DFF와 마찬가지로 매크로보다는 스탠다드 셀 배치 품질에 대한 정보를 더 많이 제공하기 때문입니다. 그럼에도 불구하고, 이는 매크로를 더욱 넓은 공간을 위해 분리해야 할 필요가 있는지 여부를 결정하는 좋은 기법입니다.  
{: .notice--warning}  

### Place I/O Pins

![2023-05-12-io-pins.png]({{site.baseurl}}/assets/images/2023-05-12-io-pins.png){: .align-center}  

After shaping and placement, it's time to place the block pins. In order to place the pins you need to first constrain, then run pin placement. The first command in the example, `set_block_pin_constraints`, is used to apply constraints for all pins equally. Here lets see how to specify certain layers or how to specify or exclude certain sides of the design. By definition, side number one is the lower leftmost vertical edge as shown in the diagram. The command also uses the option `-self`, in which case the constraints will apply to the current block. In hierarchical designs, the same command without `-self` is used to constrain physical sub blocks. The second command in the example, `create_pin_constraint`, can be used to set constraints on individual or bused pins If, for example, user wanted to have clock pins at a certain XY location, or wanted to change the physical sizes of certain pins, or force these pins on a certain layer, user would use this command. Once you've constrained the pins, use "`place_pins -self`" to perform the actual pin placement. Overall, pin placement from the block level outwards must be viewed critically. After all, the block you're working on is most probably not the only block, and as such must connect to other blocks at the top level. In most cases the pin placement will be performed at the top level and just passed down to the block designer. In that case you would have simply applied Tcl commands or DEF floorplans to place the pins at certain XY locations, and this would have occurred before macro placement. If it is early in the design cycle though, then detailed floorplanning might not have been performed yet. Therefore, it might make sense to place pins at the block level using rough top-level guidelines, for example, databus A should be placed on the bottom.  
블록 핀을 배치하는 것은 모양을 제약한 후 핀 배치를 실행해야 합니다. 예제의 첫 번째 명령인 `set_block_pin_constraints`는 모든 핀에 동일한 제약을 적용하는 데 사용됩니다. 여기서는 특정 레이어를 지정하거나 디자인의 특정 면을 지정하거나 제외하는 방법을 살펴보겠습니다. 여기서 "side number one"은 다이어그램에 표시된 가장 왼쪽 하단의 수직 가장자리를 의미합니다. 이 명령은 또한 `-self` 옵션을 사용하여 제약을 현재 블록에 적용합니다. 계층적 디자인에서는 `-self` 없이 동일한 명령을 사용하여 물리적 하위 블록을 제약합니다. 예제의 두 번째 명령인 `create_pin_constraint`은 개별 또는 버스 형태의 핀에 대한 제약을 설정하는 데 사용됩니다. 예를 들어, 특정 XY 위치에 클럭 핀을 갖도록 하거나 특정 핀의 물리적 크기를 변경하거나 이러한 핀을 특정 레이어에 강제로 배치하려는 경우에 이 명령을 사용합니다. 핀을 제약한 후 "`place_pins -self`"를 사용하여 실제 핀 배치를 수행합니다. 전체적으로, 블록 수준에서 바깥쪽으로 핀 배치를 비판적으로 검토해야 합니다. 작업 중인 블록은 아마도 유일한 블록이 아니므로, 최상위 수준에서 다른 블록과 연결되어야 합니다. 대부분의 경우 핀 배치는 최상위 수준에서 수행되고 블록 디자이너에게 전달됩니다. 이 경우에는 Tcl 명령이나 DEF 플로어플랜을 적용하여 특정 XY 위치에 핀을 배치하며, 이는 매크로 배치 이전에 발생합니다. 그러나 디자인 주기가 초기라면 상세한 플로어플랜이 아직 수행되지 않았을 수도 있습니다. 따라서 블록 수준에서 상위 수준 가이드라인을 사용하여 핀을 배치하는 것이 의미가 있을 수 있습니다. 예를 들어, 데이터 버스 A는 하단에 배치되어야 한다고 가이드라인에 명시할 수 있습니다.  
{: .notice--warning}  

## Power Planning Challenges

![2023-05-12-power-planning-challenges.png]({{site.baseurl}}/assets/images/2023-05-12-power-planning-challenges.png){: .align-center}  

- Mix of digital and analog logic can require complex core rings
- Many designs have multiple voltage areas
  - Each area requires a unique mesh structure
- Special P/G patterns

After you have finalized your macro placement and IO pin placement you need to create the power mesh or network. With the mix of digital and analog logic, you might require complicated core rings, that are very hard to create manually, and if you make any changes, you will have to re-create them. The various voltage areas in your design might require routing of different nets and different meshes. Also, at the lower geometries, you might require quite special power ground net patterns as shown in the example. Designing these manually for a large design can pose quite a challenge. To address the challenges of PG design in modern chips, we offer a technology called pattern based power network synthesis or PPNS.  
매크로 배치와 IO 핀 배치를 완료한 후에는 전원 망 또는 네트워크를 생성해야 합니다. 디지털 및 아날로그 논리의 혼합으로 인해 복잡한 코어 링이 필요할 수 있습니다. 이는 수동으로 생성하기 매우 어려울 뿐만 아니라 변경 사항이 발생할 경우 다시 생성해야 합니다. 디자인의 여러 전압 영역은 다른 넷 및 다른 망의 라우팅을 필요로 할 수 있습니다. 또한, 낮은 기하학적 크기에서는 큰 디자인을 위해 특수한 전원 그라운드 넷 패턴이 필요할 수 있습니다. 이러한 패턴을 대규모 디자인에 수동으로 설계하는 것은 상당한 도전이 될 수 있습니다. 현대 칩에서 전원 그라운드 설계의 도전에 대응하기 위해 우리는 패턴 기반 전원 네트워크 합성 또는 PPNS라고 하는 기술을 제공합니다.  
{: .notice--warning}  

## Pattern-Based Power Network Synthesis

![2023-05-12-pattern-based-power-network-syn.png]({{site.baseurl}}/assets/images/2023-05-12-pattern-based-power-network-syn.png){: .align-center}  

1. Define PG Regions
2. Define **Pattern**: Structure
   - Defines metal layers, spacing, width, ...
3. Define **Strategy**: P/G topology
   - Applies specific **pattern** to specific PG **region**, voltage areas or bounds and specific power/ground nets
   - Flexible to floorplan change (no fixed coordinates)
4. Compile power network

The PPNS approach starts with the definition of a PG region. This can be a voltage area, a group of macros, or any area defined using XY coordinates. Next you define a pattern, which is the definition of the metal layers used for your power mesh, the spacing and the width. Finally, you define a strategy which applies a pattern to a certain PG region. When you define the strategy, you're also supplying the net names, and you can define blockages as well as extension behavior. If you define a region without coordinates and then apply your patterns and strategies, your PG mesh becomes flexible to floorplan changes. If voltage areas are moved around or change size, PPNS will be able to adapt because you haven't specified absolute coordinates. The final step is to compile the power network. Check the notes for a few more details. We're not going to cover more in this module as this is just a short introduction to PPNS. In the lab, you'll be running a PPNS script, so you'll have time to review it and learn more about it. There is also very excellent documentation on SolvNetPlus for PPNS. Finally, there's an extensive module in the hierarchical design planning training offered by Synopsys.  
PPNS(패턴 기반 전원 네트워크 합성) 접근 방식은 PG(파워 그라운드) 영역의 정의로 시작합니다. 이는 전압 영역, 매크로 그룹 또는 XY 좌표를 사용하여 정의된 영역이 될 수 있습니다. 다음으로, 전원 메시에 사용되는 메탈 레이어, 간격 및 폭을 정의하는 패턴을 정의합니다. 마지막으로, 패턴을 특정 PG 영역에 적용하는 전략을 정의합니다. 전략을 정의할 때는 넷 이름을 지정하고, 블록에 대한 제약 사항 및 확장 동작을 정의할 수도 있습니다. 좌표 없이 영역을 정의하고 패턴과 전략을 적용하면, PG 메시가 플로어플랜 변경에 유연하게 적응할 수 있습니다. 전압 영역이 이동하거나 크기가 변경되면, PPNS는 절대 좌표를 지정하지 않았기 때문에 적응할 수 있습니다. 마지막 단계는 전원 네트워크를 컴파일하는 것입니다. 자세한 내용은 참고 자료를 확인하십시오. 이 모듈에서는 PPNS에 대해 더 이상 다루지 않습니다. 랩에서 PPNS 스크립트를 실행하게 되므로 리뷰하고 더 많이 알아볼 시간이 주어집니다. SolvNetPlus에는 PPNS에 대한 훌륭한 문서가 있습니다. 또한 Synopsys에서 제공하는 계층적 설계 계획 교육에도 광범위한 모듈이 있습니다.  
{: .notice--warning}  

## Analyze Global Route Congestion

![2023-05-12-analyze-global-route-congestion.png]({{site.baseurl}}/assets/images/2023-05-12-analyze-global-route-congestion.png){: .align-center}  

```tcl
route_global -floorplan true -congestion_map_only true
```

With the `-floorplan true` option the global router performs fast, lower effort global routing. The global router uses less effort to avoid blockages on blocked pins and makes other simplifying assumptions to complete global routing quickly. Designs can be unlegalized (only coarse placement) in floorplan mode. Floorplan mode also disables timing or crosstalk-driven routing. The resulting congestion map shows the location of congestion hot spots in the floorplan.  
  
If borderline congestion, you should confirm with:  
`route_global -congestion_map_only true -effort_level low|medium|high`  

Once you've performed all these design planning steps, it's time to look at a global route congestion map to determine whether you will have any routing problems with the current macro placement or if you have any large congestion hotspots that might pose challenges down the road. Now we should focus on congestion between or around macros. A congestion map is a predictor of routability. Routing a design can take a long time, generating a congestion map is a very quick operation in comparison. It uses quick global routing to estimate where you might run into trouble later during routing. To generate a congestion map, you select global route congestion from the pull down menu, then click on reload. Global routing routes the design by dividing it up into global routing cells or GRC's. You can see these cells in the layout view I'm showing with labels drawn over each edge of a GRC. Each edge shows the overflow and the available routing resources.  In the zoomed in GRC I'm showing +0 and +1 for the top and bottom edges and +4 and +7 for the left and right edges. The larger the number on the left, the warmer or hotter the color will be, and the larger the overflow is. For example, nets crossing the red edge require 7 more resources than what is available. More on this in a moment. The more of these hot edges you have in one area, the bigger the problem. A congestion hot spot is characterized by a very large number of these hot edges in one area. The command to generate this congestion map is shown below. It's recommended to use the `-floorplan` option because this will speed up the global routing and it also allows for a design that doesn't have legalized standard cell placement, as is the case after floorplanning. The `-congestion_map_only` option means that the global routes are not stored on the design, but are only used for congestion analysis. You should examine the congestion as often and as early as possible, so start right after macro placement, then again when you make macro placement modifications, then again when you have your first rough power mesh etc. That way you can isolate the issues that are causing the congestion.  
위의 설계 계획 단계를 모두 수행한 후, 전역 경로 혼잡도 맵을 확인하여 현재 매크로 배치로 인해 라우팅 문제가 발생하는지 또는 이후에 도전적인 혼잡 핫스팟이 있는지 여부를 확인해야 합니다. 이제는 매크로 간 또는 주위의 혼잡도에 초점을 맞춰야 합니다. 혼잡도 맵은 경로 가능성을 예측하는 도구입니다. 설계 라우팅은 오랜 시간이 걸릴 수 있지만, 혼잡도 맵 생성은 비교적 빠른 작업입니다. 이 작업은 빠른 전역 라우팅을 사용하여 나중에 라우팅 중에 문제가 발생할 수 있는 위치를 추정합니다. 혼잡도 맵을 생성하기 위해 풀다운 메뉴에서 전역 경로 혼잡도를 선택한 다음, 다시 로드 버튼을 클릭합니다. 전역 라우팅은 설계를 전역 라우팅 셀 또는 GRC로 나누어 경로를 지정합니다. 레이아웃 뷰에서 각 GRC의 가장자리 위에 그린 레이블을 보실 수 있습니다. 각 가장자리에는 오버플로우 및 사용 가능한 라우팅 리소스가 표시됩니다. 저는 확대된 GRC에서 위쪽 가장자리에 +0 및 +1, 왼쪽 및 오른쪽 가장자리에 +4 및 +7 을 보여드렸습니다. 왼쪽의 숫자가 클수록 색상이 더 밝거나 더 높게 나타나며 오버플로우도 더 큽니다. 예를 들어, 빨간 가장자리를 거치는 넷은 사용 가능한 리소스보다 7개 더 많이 필요합니다. 이에 대해 잠시 후에 자세히 설명하겠습니다. 한 영역에 더 많은 핫 가장자리가 있다면 문제가 커집니다. 혼잡 핫스팟은 한 영역에서 많은 핫 가장자리를 가지고 있는 경우로 특징짓습니다. 혼잡도 맵을 생성하기 위한 명령어는 아래에 표시되어 있습니다. `-floorplan` 옵션을 사용하는 것이 권장됩니다. 이렇게 하면 전역 라우팅이 빨라지며, 플로어플랜 이후에 표준 셀 배치가 정규화되지 않은 설계에도 적용할 수 있습니다. `-congestion_map_only` 옵션은 전역 경로가 설계에 저장되지 않고 혼잡도 분석에만 사용됨을 의미합니다. 가능한 한 자주, 가능한 한 일찍 혼잡도를 확인해야 합니다. 매크로 배치 후에 바로 확인하고, 매크로 배치 수정 시에도 다시 확인하고, 처음으로 대략적인 전원 메시를 생성한 후에도 다시 확인해야 합니다. 이렇게 하면 혼잡을 일으키는 원인을 격리할 수 있습니다.  
{: .notice--warning}  

## Congestion Segment Labels

- The congestion segment labels have the format:
  - "Signed congestion overflow" / "total supply" (both numbers are in units of routing tracks)

| Layer | Supply | Demand | Overflow |
|:-----:|:-----:|:-----:|:-----:|
| M1 | 8 | 10 | +2 |
| M3 | 7 | 9 | +2 |
| M5 | 5 | 1 | -4 → 0 |

- For example, this segment: +4/20
  - Capacity (supply): 20 tracks
  - Overflow: +4 tracks
  - Underflow (here -4) is counted as 0

To explain how the numbers on the GRC edges are calculated, here's an example. Every edge will be labeled with an overflow number or "signed congestion overflow", followed by the total supply. In my example +4 / 20. In order to calculate the supply, the tool will add the available routing resources or tracks of all layers crossing that edge. For the vertical edge in this example, we consider only horizontal routing layers, M1, M3 and M5. These are the layers that will cross this edge. The available supply of routing tracks for these layers is 8, 7 and 5 respectively. The demand is shown next - this represents the number of nets that need to cross this edge, here 10, 9 and 1. The overflows are calculated layer by layer. For M1, the supply is 8, the demand is 10, therefore the overflow is +2. For M3 the overflow is +2 as well, and for M5 it is -4, the demand is only 1 vs. a supply of 5. Negative overflows are counted as zero. To calculate the final overflow number, we add only positive overflows, which gives us +4 as the final number. So why do we ignore the available routing resources on M5? As mentioned, congestion analysis is a predictor of routability. In this particular case, those 4 signals could be routed, but the router is going to have to work harder. Routing will have to bump the routes all the way up to metal 5, which means that there's going to be a lot of vias. Vias that have to be put in and take up more space which in turn can cause more congestion issues etc. So in general, we have found that this way of calculating congestion correlates quite well to the final routing.  
각 GRC 엣지에는 "혼잡 오버플로"라고 표시된 오버플로 숫자와 총 공급량이 표시됩니다. 예를 들어 +4 / 20입니다. 공급량을 계산하기 위해 도구는 해당 엣지를 가로지르는 모든 레이어의 사용 가능한 경로 리소스 또는 트랙을 추가합니다. 이 예에서는 수직 엣지를 기준으로 M1, M3 및 M5와 같은 수평 경로 레이어만 고려합니다. 이러한 레이어의 사용 가능한 트랙 공급량은 각각 8, 7 및 5입니다. 수요는 다음에 표시됩니다. 이는 해당 엣지를 통과해야 하는 넷의 수를 나타냅니다. 예를 들어, 10, 9 및 1입니다. 오버플로는 레이어별로 계산됩니다. M1의 경우, 공급량은 8이고, 수요는 10이므로 오버플로는 +2입니다. M3의 경우, 오버플로도 +2입니다. M5의 경우, 오버플로는 -4입니다. 수요는 1이고, 공급량은 5입니다. 음의 오버플로는 0으로 처리됩니다. 최종 오버플로 숫자를 계산하기 위해, 양의 오버플로만 더합니다. 이로써 최종 숫자인 +4를 얻을 수 있습니다. 그렇다면 M5의 사용 가능한 경로 리소스를 왜 무시하나요? 이미 언급했듯이, 혼잡도 분석은 경로 생성 가능성을 예측하는 것입니다. 이 특정한 경우에는 해당 신호들을 경로화할 수 있지만, 경로 생성기가 더 많이 작동해야 합니다. 경로화는 더 높은 금속 레이어까지 경로를 이동해야 하므로, 많은 비아가 필요합니다. 비아는 추가 공간을 차지하고, 이는 더 많은 혼잡 문제를 일으킬 수 있습니다. 일반적으로 이러한 혼잡 계산 방식이 최종 라우팅과 상당히 일치한다는 것을 알아냈습니다.  
{: .notice--warning}  

## Congestion Potential Around Macro Cells

![2023-05-12-congestion-potential-around-macro.png]({{site.baseurl}}/assets/images/2023-05-12-congestion-potential-around-macro.png){: .align-center}  

- Routing to standard cells can often be difficult near high pin-count edges or corners of macro cells
- Solution: Apply placement blockages around macros, or use keepout margins
- Note however that automatic placement will have added soft placement blockages already

As mentioned, you should focus on now is the congestion around and between macros. If you look at the situation right here, we do have some congestion between the macros and between the macros and the edge of the design, so you may want to apply some placement blockages. Now this screenshot is actually an old screenshot and the `create_placement -floorplan` command will often find a solution for a lot of these problems by automatically inserting placement blockages in the channels. So the chance that you'll find these types of problems is actually reduced. Nonetheless, you could still run into situations where you might need some extra control.  
언급한대로, 매크로 주변과 매크로 간 혼잡도에 초점을 맞추는 것이 중요합니다. 제공된 스크린샷에서는 매크로 간 및 매크로와 디자인 가장자리 사이에 혼잡이 관찰됩니다. 이러한 혼잡을 해결하기 위해 배치 차단을 적용할 수 있습니다. 그러나 참고하십시오, 언급한 스크린샷은 오래된 버전의 것이며, 현재의 `create_placement -floorplan` 명령은 채널에 자동으로 배치 차단을 삽입하여 이러한 혼잡 문제를 해결하는 기능이 향상되었습니다. 따라서 이러한 문제를 발견할 가능성은 줄어들었습니다. 그러나 여전히 추가적인 제어가 필요한 상황이 발생할 수 있습니다. 혼잡 맵을 분석하고 혼잡이 심한 지역이나 높은 혼잡도를 가진 영역을 관찰하는 것이 권장됩니다. 이를 통해 혼잡을 완화하고 더 나은 라우팅 결과를 보장하기 위해 전략적으로 배치 차단을 적용할 수 있는 구체적인 영역을 식별할 수 있습니다.
{: .notice--warning}  

## Apply "Padding" Around Macro Edges

![2023-05-12-padding-around-macro.png]({{site.baseurl}}/assets/images/2023-05-12-padding-around-macro.png){: .align-center}  

One way of reducing congestion around macro edges is by applying padding, this is called a keep-out margin. In the example it has RAM with pins on the left and the right sides, and if you want to leave some extra space towards the standard cells so the router can easily access the macro pins. Note that by default the tool automatically calculates and applies macro keepout margins based on the macros pin count. If you want to add more padding because you know of a problematic macro, then you can apply this keepout margin yourself. There are two ways to do this. One is using the task assistant and as you see here, you can manipulate every macro in your design using a spreadsheet-like table. You can also run the Tcl command `create_keepout_margin`. Here the type is `hard`. That means no cells to be placed in this area. Using the `-outer` option you specify the left, bottom, right and top margins in microns that you want to set on the macro. For this example, we set a margin of 15 on the left, and 18 on the right. Since there are no pins on the top and bottom of the macro, we set a margin of 0 there.  
매크로 가장자리 주변의 혼잡을 줄이는 한 가지 방법은 패딩을 적용하는 것입니다. 이를 keep-out 마진이라고도 합니다. 예를 들어, 왼쪽과 오른쪽에 핀이 있는 RAM이 있을 때, 라우터가 매크로 핀에 쉽게 접근할 수 있도록 표준 셀 쪽으로 여분의 공간을 남기고 싶을 수 있습니다. 기본적으로 도구는 핀 개수를 기반으로 매크로 keepout 마진을 자동으로 계산하고 적용합니다. 문제가 있는 매크로를 알고 있으므로 추가적인 패딩을 추가하려면 keepout 마진을 직접 적용할 수 있습니다. 이를 위해 두 가지 방법이 있습니다. 하나는 태스크 어시스턴트를 사용하는 것이고, 여기서는 스프레드시트와 유사한 테이블을 사용하여 디자인 내의 각 매크로를 조작할 수 있습니다. 또한 Tcl 명령 `create_keepout_margin`을 실행할 수도 있습니다. 여기에서는 타입을 `hard`로 지정합니다. 이는 이 영역에는 셀이 배치되지 않는다는 의미입니다. `-outer` 옵션을 사용하여 왼쪽, 아래쪽, 오른쪽 및 위쪽 마진을 마이크론 단위로 설정합니다. 이 예시에서는 왼쪽에 15의 마진과 오른쪽에 18의 마진을 설정합니다. 매크로의 위쪽과 아래쪽에 핀이 없으므로 해당 영역에는 0의 마진을 설정합니다.  
{: .notice--warning}  

## Write out the Floorplan

- Fix the placement of all macros before writing out the floorplan
  
```tcl
set_fixed_objects [get_flat_cells -filter "is_hard_macro"]
```
  
- Capture all floorplan information for later re-use in IC Compiler II (Standard cell placement is not included):
  - Output directory `ORCA_TOP.fp` will contain the files:
    - `floorplan.tcl` - Main file, sources the following two files
    - `fp.tcl` - Reads the DEF file and applies Tcl floorplan constraints that cannot be captured as DEF
    - `floorplan.def` - DEF file

```tcl
write_floorplan -output ORCA_TOP.fp \
                -net_types {power ground} -include_physical_status {fixed locked}
```

- To apply the floorplan later in IC Compiler II, use:

```tcl
source ORCA_TOP.fp/floorplan.tcl
```

Once you've completed your floorplan, it's time to save. Before you do that, you should lock down the placement of the macros. This is done using the `set_fixed_objects` command, as shown in the example. Afterwards, use the `write_floorplan` command and specify a target output directory, in this case `ORCA_TOP.fp`.  With the `-net_types power` and `ground` it is specifying that it want to output the power mesh routing. Using the `-include_physical_status fixed` and `locked` we make sure to only include hard macros we have set to fixed earlier. We don't want to capture the standard cell placement, as that is not the intent of a floorplan. Inside the `ORCA_TOP.fp` directory you will find several files. `floorplan.tcl` is the main file.  That file will then source all the other files. Most of the floorplanning data is stored in `floorplan.def`. Whereas Synopsys-specific physical constraints that cannot be expressed in DEF will be stored in `fp.tcl`. And so later on, when you want to reapply or reload this floorplan, you would simply have to source `ORCA_TOP.fp/floorplan.tcl`.  
플로어플랜을 완료한 후에는 저장해야 합니다. 그 전에 매크로의 배치를 고정해야 합니다. 이는 `set_fixed_objects` 명령을 사용하여 수행됩니다. 예시에서와 같이 명령을 사용하여 고정된 매크로를 설정합니다. 그 후에는 `write_floorplan` 명령을 사용하여 대상 출력 디렉토리를 지정합니다. 이 예시에서는 `ORCA_TOP.fp`로 설정합니다. `-net_types power`와 `ground를` 사용하여 전원 메쉬 라우팅을 출력하도록 지정합니다. `-include_physical_status fixed`와 `locked`를 사용하여 이전에 고정한 하드 매크로만 포함하도록 합니다. 플로어플랜의 목적은 표준 셀 배치를 포함하지 않기 때문에 표준 셀 배치를 캡처하고 싶지 않습니다. `ORCA_TOP.fp`디렉토리 내에 여러 파일이 있을 것입니다. `floorplan.tcl`은 주 파일입니다. 이 파일에서 다른 모든 파일을 소스로 포함합니다. 대부분의 플로어플랜 데이터는 `floorplan.def`에 저장됩니다. DEF로 표현할 수 없는 Synopsys 고유의 물리적 제약 조건은 `fp.tcl`에 저장됩니다. 따라서 플로어플랜을 다시 적용하거나 로드하려면 간단히 `ORCA_TOP.fp/floorplan.tcl`을 소스로 포함하면 됩니다.
{: .notice--warning}  

## Write out Floorplan Information for Design Compiler NXT

- Write out the floorplan information that is necessary for Design Compiler NXT
  - Standard cell placement is not included

```tcl
write_floorplan -format icc -output ORCA_TOP.fp.dc \
    -net_types {power ground} \
    -include_physical_status {fixed locked}
```

- To load the floorplan in Design Compiler NXT:

```tcl
create_net -power VDD
create_net -ground VSS
read_floorplan ORCA_TOP.fp.dc/floorplan.tcl
```

Use `write_floorplan` command to write out the floorplan information required for Design compiler tool. Standard cells are not included because a new netlist will be created during "second-pass synthesis", based on this floorplan. The `write_floorplan` command creates a subdirectory named by `-output`. The example above would produce: `ORCA_TOP.fp.dc`. This directory consists files like `floorplan_compare_data.txt`, `floorplan.def`, `floorplan.tcl` and `fp.tcl`. To apply the floorplan in Design Compiler tool, you need to create all power and ground nets first, otherwise Design Compiler tool will not apply the PG net information from the DEF file as shown here. Finally load the floorplan by reading the `floorplan.tcl` file by using the command `read_floorplan`.  
Design Compiler 도구에서 필요한 플로어플랜 정보를 작성하기 위해 `write_floorplan` 명령을 사용합니다. 표준 셀은 포함되지 않으며, 이 플로어플랜을 기반으로 "두 번째 패스 합성" 중에 새로운 넷리스트가 생성될 것입니다. `write_floorplan` 명령은 `-output`에 지정된 이름으로 하위 디렉토리를 생성합니다. 위의 예시에서는 `ORCA_TOP.fp.dc`가 생성될 것입니다. 이 디렉토리에는 `floorplan_compare_data.txt`, `floorplan.def`, `floorplan.tcl` 및 `fp.tcl`과 같은 파일이 포함되어 있습니다. Design Compiler 도구에서 플로어플랜을 적용하기 위해서는 모든 전원 및 그라운드 넷을 먼저 생성해야 합니다. 그렇지 않으면 Design Compiler 도구는 DEF 파일에서 PG 넷 정보를 적용하지 않습니다. 마지막으로 `read_floorplan` 명령을 사용하여 `floorplan.tcl` 파일을 읽어 플로어플랜을 적용합니다.
{: .notice--warning}  

## Synthesis with ICC II Floorplan Before Placement

- Synthesis with the actual floorplan provides *IC Compiler II* with a better optimized netlist for placement  

![2023-05-12-syn-before-placement.png]({{site.baseurl}}/assets/images/2023-05-12-syn-before-placement.png){: .align-center}  

By re-synthesizing the design with Design Compiler NXT using an accurate floorplan you are providing IC Compiler II with a better starting netlist, which may give better results. If QoR is not critical, the re-synthesis step can be skipped and the resulting cell after floorplan definition, with its netlist from the first synthesis execution, is taken directly to placement.  
The above flow assumes that the RTL code is fairly complete when performing the initial synthesis. If the RTL contains a fair number of “black boxes” (i.e. undefined sections) you may need to perform additional iterations of design planning and synthesis, as the RTL code is solidified.  
To pass the design from Design Compiler NXT to ICC II, use Design Compiler's `write_icc2_files`, which will also write the standard cell placement as DEF (among many other things):  

```tcl
# Save the file for IC Compiler II
denxt_shell-topo> write_icc2_files -output ./icc2_files
denxt_shell-topo> quit
# Perform library setup in IC Compiler II (this is a script you create)
icc2_shell> source -echo -verbose icc2_lib_setting.tcl
# Load the design
icc2_shell> source -echo -verbose icc2_files/ORCA.icc2_script.tcl
# Continue with the flow
icc2_shell> commit_upf
icc2_shell> place_opt
```

By re-synthesizing the design with Design Compiler NXT using an accurate floorplan you are providing IC Compiler II with a better starting netlist, which may give better results. If QoR is not critical, the re-synthesis step can be skipped and the resulting cell after floorplan definition, with its netlist from the first synthesis execution, is taken directly to placement. The above flow assumes that the RTL code is fairly complete when performing the initial synthesis. If the RTL contains a fair number of "black boxes" (i.e. undefined sections) you may need to perform additional iterations of design planning and synthesis, as the RTL code is solidified. To pass the design from Design Compiler NXT to ICCII, use Design Compiler's `write_icc2_files`, which will also write the standard cell placement as DEF.  
Design Compiler NXT를 사용하여 정확한 플로어플랜으로 디자인을 재합성하면 IC Compiler II에 더 나은 시작 넷리스트를 제공하여 더 나은 결과를 얻을 수 있습니다. QoR이 중요하지 않은 경우에는 재합성 단계를 건너뛰고 플로어플랜 정의 후의 결과 셀과 처음 합성 실행에서의 넷리스트를 직접 플레이스먼트로 사용할 수 있습니다. 위의 플로우는 RTL 코드가 초기 합성 시에 상당히 완성된 상태를 가정합니다. RTL에 "블랙 박스"라고 하는 정의되지 않은 섹션이 많이 포함되어 있는 경우, RTL 코드가 확정될 때까지 추가적인 디자인 계획과 합성의 반복이 필요할 수 있습니다. Design Compiler의 `write_icc2_files`를 사용하여 Design Compiler NXT에서 ICC II로 디자인을 전달할 수 있으며, 이때 표준 셀 플레이스먼트도 DEF로 작성됩니다.
{: .notice--warning}  

## Summary

- You should now be able to
  - Describe some key block-level floorplanning steps in IC Compiler II

This concludes the floorplanning unit. It has a lab that you can run through now. Even if you will not be performing your own floorplanning as you might have a floorplan that is provided to you, in some cases you might have to move macros or make some changes in your block level floor plan to solve a problem that you might run into. This lab will provide you with the necessary tools. Good luck and thanks for viewing this module.  
이로써 플로어플랜 유닛이 마무리되었습니다. 이제 랩을 진행할 수 있습니다. 제공된 플로어플랜이 있어서 직접 플로어플랜을 수행하지 않을 수 있지만, 경우에 따라서는 마크로를 이동하거나 블록 레벨 플로어플랜을 수정하여 문제를 해결해야 할 수도 있습니다. 이 랩은 필요한 도구를 제공할 것입니다. 행운을 빕니다. 이 모듈을 시청해 주셔서 감사합니다.
{: .notice--warning}  
