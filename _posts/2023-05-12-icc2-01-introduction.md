---
title: "IC Compiler II - 01 Introduction"
excerpt: "Block-level Implementation"
date: 2023-05-12 01:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
---

## Introduction

Hello there, welcome to IC Compiler II Block-level Implementation self-paced learning. Here we will discuss the first module of the course i.e. Introduction  
안녕하세요, IC Compiler II 블록 수준 구현 자기 학습에 오신 것을 환영합니다. 여기에서는 해당 과정의 첫 번째 모듈인 소개에 대해 논의할 것입니다.
{: .notice--warning}

## Target Audience

ASIC, back-end, or layout designers who will be using IC Compiler II to perform placement, CTS and routing on block-level designs

The target audience for this training are ASIC back-end, or layout engineers who will be using IC Compiler II to perform placement, CTS, and routing on block-level designs. This is specifically not a hierarchical training, except for the topic of top-level implementation, which we will cover. Hierarchical design planning, on the other hand, is covered by a different training.  
이 교육의 대상은 ASIC 백엔드 또는 레이아웃 엔지니어로, IC Compiler II를 사용하여 블록 수준 설계에 대한 배치, CTS(클록 트리 선형화), 라우팅 작업을 수행할 예정입니다. 이 교육은 특히 계층적인 교육이 아닙니다. 다만, 최상위 구현에 대한 주제는 다룰 예정입니다. 그 외에 계층적인 설계 계획은 별도의 교육에서 다루게 됩니다.
{: .notice--warning}

## Objectives

This training will cover the entire block-level implementation flow

- Design & Timing Setup
- Floorplanning (introduction)
- Placement and Optimization
- Clock Tree Synthesis and Optimization
- Routing and Post-Route Optimization, including InDesign PrimeTime and StarRC
- Top-level Implementation, Signoff DRC, fill and ECO

To give you a little overview, this training will cover the entire block-level implementation flow. Topics such as design & timing set up, floorplanning, placement, clock tree synthesis, routing and optimization, along with InDesign PrimeTime and StarRC. And finally, top-level integration and implementation, signoff DRC, metal fill, and ECO operations.  
간단히 설명드리면, 이 교육은 전체 블록 수준 구현 플로우를 다룰 예정입니다. 디자인 및 타이밍 설정, 플로어플래닝, 배치, 클럭 트리 합성, 라우팅 및 최적화, 그리고 InDesign PrimeTime과 StarRC와 같은 주제들을 다룰 것입니다. 마지막으로, 최상위 통합과 구현, 사인오프 DRC, 금속 채움, 그리고 ECO 작업에 대해서도 다룰 예정입니다.
{: .notice--warning}

## Prerequisites

- Prior knowledge of IC Compiler or IC Compiler II is not needed
- Knowledge of general standard cell-based placement, CTS and routing concepts and terms is helpful
- An understanding of basic digital ASIC design concepts is assumed, including:
  - Combinational and sequential logic functionality
  - Setup and hold timing

There are a few prerequisites for this training. Most importantly, you are not expected to know IC Compiler, or IC Compiler II. If this is the first time you see or use these tools then you're in the right training. Of course, previous knowledge of IC Compiler will make your learning easier. We do assume knowledge of general standard cell-based placement, CTS, and routing concepts, as well as an understanding of basic digital ASIC design.  
이 교육을 위한 몇 가지 선수 조건이 있습니다. 가장 중요한 것은 IC Compiler 또는 IC Compiler II에 대해 알지 못한다는 것입니다. 만약 이 도구들을 처음 접하거나 사용하는 경우에도 이 교육이 적합합니다. 물론, IC Compiler에 대한 이전 지식은 학습을 보다 쉽게 만들 수 있습니다. 우리는 일반적인 스탠다드셀 기반 배치, CTS 및 라우팅 개념에 대한 지식, 그리고 기본적인 디지털 ASIC 설계에 대한 이해를 전제로 합니다.
{: .notice--warning}

## Agenda

- i Introduction and GUI Lab  
- 1 Objects, Blocks and App Options  
- 2 Floorplanning  
- 3 Placement & Optimization  
- 4 NDM Cell Libraries  

This training is composed of 14 modules, not including this introduction. A number of modules are also broken into 2 or 3 parts. After this introduction, it is recommended to run lab 0, which will introduce you to the GUI, or Graphical User Interface. In module 1, we're going to cover objects, blocks, and app options. We then dive into floor-planning, followed by placement and optimization. Next, we'll talk about NDM cell library preparation, these are the standard and macro cell libraries used by your design.  
이 교육은 소개를 제외한 14개의 모듈로 구성되어 있습니다. 일부 모듈은 2개 또는 3개의 파트로 나뉘어집니다. 이 소개 이후에는 GUI(그래픽 사용자 인터페이스)에 대해 소개하는 lab 0을 실행하는 것이 좋습니다. 모듈 1에서는 객체, 블록 및 앱 옵션에 대해 다룰 예정입니다. 그 다음에는 플로어플래닝, 배치 및 최적화에 대해 논의합니다. 그 다음으로는 NDM 셀 라이브러리 준비에 대해 이야기할 것입니다. 이는 디자인에서 사용되는 스탠다드 및 매크로 셀 라이브러리입니다.
{: .notice--warning}

- 5 Design Setup  
- 6 Timing Setup  
- 7 Setting Up CTS  
- 8 Running CTS  

Followed by design setup in unit 5 as well as timing setup, in unit 6. Then we'll continue with the topic of CTS: setting up and running CTS.  
그 뒤에는 유닛 5에서 디자인 설정을 다루고, 유닛 6에서 타이밍 설정을 다룰 것입니다. 그 다음에는 CTS에 대한 주제로 이어집니다: CTS 설정 및 실행 방법을 다룰 예정입니다.
{: .notice--warning}

- 9 Routing  
- 10 Routing DRC  
- 11 Via Ladder  
- 12 Post-Route Optimization  
- 13 Top Level Implementation  
- 14 Signoff  

Module 9 and the following modules cover routing, routing DRC, via ladder & routing optimization. In module 13, we will cover top level implementation. The training will close with a module on signoff.  
모듈 9 및 그 이후의 모듈은 라우팅, 라우팅 DRC, 비아 사다리 및 라우팅 최적화를 다룰 예정입니다. 모듈 13에서는 최상위 구현에 대해 다룰 것입니다. 교육은 사인오프에 대한 모듈로 마감됩니다.
{: .notice--warning}

## IC Compiler II

- To maximize the capabilities and performance of IC Compiler II, we recommend that you start with new ICC II Reference Methodology (RM) scripts

- Many things that you are used to doing in IC Compiler are no longer valid in IC Compiler II

- On the other hand, many concepts / use models are carried over to IC Compiler II
  - Similar commands can have subtle or big differences!
  - Always consult the man page

RM: solvnet.synopsys.com/rmgen

To enhance or maximize the capabilities and the performance of ICC II, the recommendation is to start with a fresh set of RM scripts, which are available on SolvNet plus for download. The scripts will provide you with all the key commands and features that are necessary to get the best performance in ICC II. There are many things that you're used to doing in IC Compiler that are no longer valid in ICC II, that's why we recommend starting afresh from RM scripts. There are many concepts and use models that are the same. The tools still use Tcl to interact; a lot of the Tcl commands are identical but some have slightly different switches and can have subtle or big differences. We recommend to always consult the man page and not just blindly take a script from ICC and use it in ICC II.  
ICC II의 기능과 성능을 향상하거나 극대화하기 위해서는 SolvNet Plus에서 다운로드할 수 있는 새로운 RM 스크립트 세트를 사용하는 것이 권장됩니다. 이 스크립트는 ICC II에서 최상의 성능을 얻기 위해 필요한 모든 주요 명령과 기능을 제공합니다.  
IC Compiler에서 사용하던 일부 방법이나 명령이 ICC II에서는 더 이상 유효하지 않을 수 있습니다. 이러한 이유로 우리는 RM 스크립트를 통해 새롭게 시작하는 것을 권장합니다. 그러나 많은 개념과 사용 모델은 여전히 동일합니다. 도구는 여전히 Tcl을 사용하여 상호 작용하며, 많은 Tcl 명령은 동일하지만 스위치가 약간 다를 수 있으며, 섬세하거나 큰 차이가 있을 수 있습니다.  
따라서, ICC에서 스크립트를 가져와서 ICC II에서 사용하기 전에 항상 매뉴얼을 참고하고 내용을 확인하는 것을 권장합니다. 이러한 접근 방식으로 ICC II의 기능과 성능을 극대화할 수 있습니다.
{: .notice--warning}

## How To Access RM

- RM is built right into ICC II - Use the GUI or the Command Line

![2023-05-12-how-to-access-rm.png]({{site.baseurl}}/assets/images/2023-05-12-how-to-access-rm.png)

Due to NDA protection, the foundry specific settings are included in side files which are not included in the RM scripts. To get the side files, contact your Synopsys support team.The JumpStart version of the RM scripts are expanded scripts that show the various application options and settings in a flat fashion. They are meant as a way to get an overview, but are not meant to be used in production.  Once you are comfortable with the scripts, switch to "Full RM".  
NDA 보호로 인해 파운드리별 특정 설정은 RM 스크립트에 포함되지 않은 사이드 파일에 포함되어 있습니다. 사이드 파일을 얻으려면 Synopsys 지원팀에 문의하십시오. RM 스크립트의 JumpStart 버전은 다양한 애플리케이션 옵션과 설정을 평면적으로 보여주는 확장된 스크립트입니다. 이 스크립트는 개요를 제공하기 위한 것이며 실제 제품 생산에 사용하도록 의도되지 않았습니다. 스크립트에 익숙해지면 "Full RM"으로 전환하여 실제 프로덕션 작업에 사용하시기 바랍니다. 스크립트와 사이드 파일에 대한 자세한 정보 및 사용법은 Synopsys 지원팀과의 상호 작용에 따라 달라질 수 있으며, 고객의 라이선싱 계약에 따라 다를 수 있습니다.
{: .notice--warning}

## IC Compiler II GUI

![2023-05-12-icc-gui.png]({{site.baseurl}}/assets/images/2023-05-12-icc-gui.png)

Here is a screenshot of the ICC II GUI, which will be the subject of your first lab. You can see the layout view, which takes up most of the window. The View Settings palette on the right side allows you to turn object visibility on or off, as well as control layer coloring and visibility, and controls many other aspects of what is displayed in the layout. One great addition is the command search. At the top of the window there is a magnifying glass. By clicking it, a field opens in which you can type in a search string. ICCII searches that string in menu names, task actions, in shell commands, and application options. You can quickly find and access all these functions directly from that panel.
다음은 ICC II GUI의 스크린샷입니다. 이는 첫 번째 랩의 주제가 될 것입니다. 창의 대부분을 차지하는 레이아웃 뷰를 볼 수 있습니다. 오른쪽에는 뷰 설정 팔레트가 있으며, 여기에서 객체의 가시성을 켜거나 끌 수 있으며, 레이어의 색상과 가시성을 제어하고 레이아웃에 표시되는 다른 측면을 제어할 수 있습니다.
하나의 훌륭한 추가 기능은 명령 검색입니다. 창 상단에 돋보기 아이콘이 있습니다. 이를 클릭하면 검색 문자열을 입력할 수 있는 필드가 열립니다. ICC II는 메뉴 이름, 작업 동작, 셸 명령 및 애플리케이션 옵션에서 해당 문자열을 검색합니다. 이 패널에서 직접 이러한 기능을 빠르게 찾고 액세스할 수 있습니다.
{: .notice--warning}

## Task Assistant

![2023-05-12-task-assistant.png]({{site.baseurl}}/assets/images/2023-05-12-task-assistant.png)

In ICCII, a lot of the tool functionality is not accessed through the menus. Instead, you use the task assistant. There, the tasks are sorted in the order they're supposed to be executed. Here's an example for design planning. You see on the left side the steps are all listed in the correct order from floorplan preparation, to setting up multi-threading, to performing block shaping, cell placement, et cetera. By clicking on these buttons you can access the functionality.  
ICCII에서는 많은 도구 기능을 메뉴를 통해 액세스하는 것이 아닌 태스크 어시스턴트를 사용합니다. 여기에서 태스크는 실행되어야 하는 순서대로 정렬됩니다. 디자인 플래닝을 예로 들어보겠습니다. 왼쪽에는 플로어플랜 준비부터 멀티 스레딩 설정, 블록 형성, 셀 배치 등과 같은 모든 단계가 올바른 순서대로 나열되어 있습니다. 이 버튼들을 클릭하여 해당 기능에 액세스할 수 있습니다.
{: .notice--warning}

## Built-In Script Editor

![2023-05-12-script-editor.png]({{site.baseurl}}/assets/images/2023-05-12-script-editor.png)

- Create scripts interactively
- Execute entire script or a selection
- Save script to a file

Another great feature in the ICC II GUI is the built-in script editor. If you look towards the bottom of your window you'll see multiple tabs. In the console tab, you can execute individual commands and see the output directly. Using the script editor on the other hand, you can open a script, perform edits, run specific lines or the entire script. You can also use the GUI dialogs to populate the script with your commands. As you're experimenting with a dialogue, once you find the correct combination of switches you require, you can copy the final command into the script editor by simply pressing a button. This should make building up your script a lot easier.  
ICC II GUI의 또 다른 훌륭한 기능은 내장된 스크립트 편집기입니다. 창의 하단쪽을 살펴보면 여러 개의 탭이 있습니다. 콘솔 탭에서 개별 명령을 실행하고 결과를 직접 확인할 수 있습니다. 반면에 스크립트 편집기를 사용하면 스크립트를 열고 편집하며, 특정 라인 또는 전체 스크립트를 실행할 수 있습니다. 또한 GUI 대화 상자를 사용하여 스크립트에 명령을 채울 수도 있습니다. 대화 상자를 실험하면서 필요한 옵션 조합을 찾으면 최종 명령을 단순히 버튼을 눌러 스크립트 편집기로 복사할 수 있습니다. 이를 통해 스크립트 작성이 훨씬 쉬워집니다.
{: .notice--warning}

## Icons Used

![2023-05-12-icons-used.png]({{site.baseurl}}/assets/images/2023-05-12-icons-used.png)

Here are the various icons used in this training.
{: .notice--warning}

## Summary

- Navigate the IC Compiler II layout view
- Control object and layer visibility
- Select and query layout objects
- Control the view level
- Rearrange panels in the GUI
- Use the Recent menu and favorites
- Use help and man to get help and additional information about commands and options

This concludes the introduction section. Before we go any further it is highly recommended that you perform the first lab. This lab will set the stage for ICC II. In this lab you will navigate the ICC II layout view, control the visibility of objects, select and query objects, and rearrange panels in the GUI to see all the features available to you. You will also use the GUI to perform timing analysis, and finally learn about the help and man commands to get information about commands and application options.This lab can be considered as a foundation; in the following labs, there are no instructions on how to perform GUI actions, which were covered here. Good luck.  
이로써 소개 섹션을 마칩니다. 더 나아가기 전에 첫 번째 랩을 수행하는 것을 강력히 권장합니다. 이 랩은 ICC II에 대한 준비 단계를 설정합니다. 이 랩에서는 ICC II 레이아웃 뷰를 탐색하고 객체의 가시성을 제어하며, 객체를 선택하고 조회하며, GUI에서 패널을 재정렬하여 사용 가능한 모든 기능을 확인할 수 있습니다. 또한 타이밍 분석을 수행하기 위해 GUI를 사용하고, 마지막으로 도움말과 man 명령을 사용하여 명령 및 애플리케이션 옵션에 대한 정보를 얻을 수 있습니다. 
이 랩은 기초로 간주될 수 있으며, 다음 랩에서는 여기서 다룬 GUI 동작을 수행하는 방법에 대한 지침이 없습니다. 행운을 빕니다.
{: .notice--warning}
