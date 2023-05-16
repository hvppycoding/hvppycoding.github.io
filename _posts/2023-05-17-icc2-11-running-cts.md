---
title: "IC Compiler II - 11 Running CTS"
excerpt: "Block-level Implementation"
date: 2023-05-17 07:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - IC Compiler II
---

## Objectives

After completing this module, you should be able to

- Execute the Classic CTS, or the concurrent clock-and-data (CCD) flow
- Analyze the clock tree
- Control post-CTS I/O auto-latency update
- Run post-CTS `global` route-based optimization

This training module does not address Multi-Source CTS

## ICC II PnR Flow

![2023-05-17-icc2-pnr-flow-clock-opt.png]({{site.baseurl}}/assets/images/2023-05-17-icc2-pnr-flow-clock-opt.png){: .align-center}  

## Clock Tree Synthesis Goal and Flows

- Placement should be completed
  - Acceptable congestion, setup timing, and logical DRCs
  - High fanout signal nets (reset, scan enable, etc) are buffered
- The goal of the clock tree synthesis phase is to:
  - Build the clock tree buffer structure
  - Detail-route the clock nets
  - Global-route and optimize data-path logic for setup and hold timing, power and logic DRCs
- Two CTS flows are supported:
  - Classic CTS flow: CTS first, followed by data path optimization
  - Concurrent Clock & Data flow (CCD): CTS and data path optimization performed concurrently
    - CCD is enabled by default during `place_opt` and `clock_opt`
    - CCD is also available during `route_opt` / `hyper_route_opt`

Before you start the CTS step you should have successfully completed physical synthesis. In practical terms this means that all cells in your design should be legally placed and the design has acceptable QoR metrics. Please review that you have acceptable QoR by ensuring the design is not congested and that your setup timing and logical DRCs (max_transition, max_capacitance etc) do not pose any unexpected issues for design closure. It is important to ensure that your high fanout signal nets have already been buffered. The only unbuffered high fanout net structures at this point should be your clock nets. When you run CTS: 1.You expect it to fully buffer and then detail route your clock nets 2.Fully globally route the signal nets and subsequently optimize the data path logic to meet setup and hold timing requirements using a power efficient implementation and without leaving logical DRC violations The two flows I will describe in this module are the classic CTS flow and the CCD flow. As mentioned, I will not be discussing multisource CTS in this module.  
CTS 단계를 시작하기 전에 물리 합성을 성공적으로 완료해야 합니다. 실제로, 이는 설계 내의 모든 셀이 올바르게 배치되어 있고, 설계에 허용 가능한 QoR(품질 지표) 메트릭이 있어야 함을 의미합니다. 설계가 혼잡하지 않고, 설정 타이밍 및 논리적 DRC(max_transition, max_capacitance 등)가 설계 클로저에 예상치 않은 문제를 일으키지 않는지 확인하여 허용 가능한 QoR을 갖고 있는지 확인해야 합니다. 또한, 고파워 신호 라인은 이미 버퍼링되어 있어야 합니다. 이 시점에서 아직 버퍼링되지 않은 고파워 신호 라인 구조는 클록 신호 라인이어야 합니다. CTS를 실행할 때 다음과 같은 작업을 예상합니다: 1. 클록 신호 라인을 완전히 버퍼링하고 상세 라우팅합니다. 2. 신호 라인을 전역적으로 완전히 라우팅하고, 이후 데이터 경로 로직을 최적화하여 설정 및 홀드 타이밍 요구 사항을 충족시키며 전력 효율적인 구현을 수행하고 논리적 DRC 위반을 허용하지 않습니다. 이 모듈에서 설명할 두 가지 플로우는 클래식 CTS 플로우와 CCD 플로우입니다. 언급한 대로, 이 모듈에서는 다중 소스 CTS에 대해 논의하지 않을 것입니다.  
{: .notice--warning}  

## Comparing Classic CTS versus CCD

![2023-05-17-classic-cts-vs-ccd.png]({{site.baseurl}}/assets/images/2023-05-17-classic-cts-vs-ccd.png){: .align-center}  

Let's compare Classic CTS and CCD. Classic CTS represents the traditional approach to CTS whereby all registers in the fanout cone of a clock source are expecting to receive their clock with the same buffering delay. This results in achieving the goal of minimal skew when measuring the arrival time of the clock between any two registers of a specific clock. Data path optimisation is performed as an independent sub-step where it is not allowed to make any changes to the clock tree. The benefit of this approach is its simplicity. Classic CTS enables a full clock cycle for register to register timing paths. However, one downside of this approach is that some complex data paths can remain problematic to achieve setup and hold timing. Another downside is that by encouraging alignment of clock edges the power of the design is higher than it needs to be. Concurrent Clock and Data, CCD for short, is a fundamentally different clock building strategy where the goal is to meet setup and hold timing. It does not focus on minimizing skew at the register clock pins. It can do this because it has full visibility of the data path timing. Visually you can see that the CCD engine adjusts the delay in the clock tree on a per register basis. Advancing and retarding the latency are known as preponing and postponing respectively. Moving the registers up and down the clock tree relative to each other is an elegant method to structurally adjust setup and hold margins and drive towards timing closure without requiring potentially extensive datapath optimisation. In a CCD flow the datapath optimization is performed in conjunction with building the clock tree. It has the flexibility to tune the clock tree if required. This strategy is illustrated in the diagram with the green buffer. While CCD is more computationally intensive than classic CTS it offers significant benefits in terms of overall timing and power QoR.  
클래식 CTS와 CCD를 비교해 보겠습니다. 클래식 CTS는 클록 소스의 팬아웃 콘에 있는 모든 레지스터가 동일한 버퍼링 지연으로 클록을 받을 것으로 예상하는 전통적인 CTS 접근 방식을 나타냅니다. 이는 특정 클록의 두 레지스터 간 클록 도착 시간을 측정할 때 최소 스큐 목표를 달성하는 결과를 가져옵니다. 데이터 경로 최적화는 클록 트리에 대한 변경을 허용하지 않는 별개의 하위 단계로 수행됩니다. 이 접근 방식의 장점은 간단함입니다. 클래식 CTS는 레지스터 간 타이밍 경로에 대한 전체 클록 주기를 가능하게 합니다. 그러나 이 접근 방식의 단점 중 하나는 일부 복잡한 데이터 경로가 설정 및 홀드 타이밍을 달성하기 어려울 수 있다는 점입니다. 또 다른 단점은 클록 엣지의 정렬을 장려하여 설계의 전력이 필요 이상으로 높아진다는 점입니다. 동시 클록 및 데이터(CCD)는 기본적으로 다른 클럭 구축 전략으로, 설정 및 홀드 타이밍을 충족하는 것이 목표입니다. 이는 레지스터 클록 핀의 스큐 최소화에 초점을 두지 않습니다. 데이터 경로 타이밍에 대한 완전한 가시성을 갖기 때문에 가능한 것입니다. 시각적으로 CCD 엔진은 클록 트리에서 레지스터별로 지연을 조정합니다. 지연을 조기화하는 것을 preponing, 연기하는 것을 postponing이라고 합니다. 레지스터를 클록 트리에서 상대적으로 위아래로 이동시킴으로써 설정 및 홀드 여유를 구조적으로 조정하고, 잠재적으로 광범위한 데이터 경로 최적화 없이도 타이밍 클로저에 가까워지는 방법입니다. CCD 플로우에서 데이터 경로 최적화는 클록 트리 구축과 함께 수행됩니다. 필요한 경우 클록 트리를 조정할 수 있는 유연성을 가지고 있습니다. 이 전략은 그림의 녹색 버퍼로 설명되어 있습니다. CCD는 클래식 CTS보다 계산량이 많지만 전체적인 타이밍 및 전력 QoR 측면에서 상당한 이점을 제공합니다.  
{: .notice--warning}  

## Classic CTS and CCD Flow

![2023-05-17-classic-cts-and-ccd-flow.png]({{site.baseurl}}/assets/images/2023-05-17-classic-cts-and-ccd-flow.png){: .align-center}  

This flow chart zooms into the CTS step. We have already explained the underlying differences between Classic CTS and CCD. Note however, that the flow around that choice is equivalent for both flows. The first task is to configure our CTS requirements. Please remember we cover these extensively in their own training module. CTS setup includes, amongst other things, defining specific balancing requirements and exceptions, what cells can be used in the clock tree as well as defining routing rules for the clock network. The choices made in this setup phase will determine what type of clock tree structure you end up with The other choice to make before asking IC Compiler II to build your clock tree is to choose which strategy you desire: Classic or CCD. It is then down to the clock_opt command to perform clock tree synthesis and optimisation. To facilitate intermediate debug and analysis the clock_opt command has three stages which we discuss in detail later. When clock_opt completes, your clock tree will be built, the clock nets will be fully detail routed and the signal nets will be globally routed. At this point we then move beyond CTS to complete the routing of the signal nets and further into the design flow.  
이 플로우 차트는 CTS 단계로 들어갑니다. 이미 클래식 CTS와 CCD 사이의 기본적인 차이점을 설명했습니다. 그러나 이 선택 사항에 대한 플로우는 두 플로우 모두 동일합니다. 첫 번째 작업은 CTS 요구 사항을 구성하는 것입니다. CTS 설정에는 균형 조정 요구 사항 및 예외 사항, 클럭 트리에서 사용할 수 있는 셀 정의, 클록 네트워크에 대한 라우팅 규칙 정의 등이 포함됩니다. 이 설정 단계에서 선택한 내용은 최종적으로 어떤 유형의 클록 트리 구조를 갖게 될지 결정합니다. 클록 트리를 생성하려면 IC Compiler II에게 요청하기 전에 Classic 또는 CCD와 같은 전략을 선택해야 합니다. 그런 다음 clock_opt 명령이 클록 트리 합성 및 최적화를 수행합니다. 중간 디버그와 분석을 용이하게 하기 위해 clock_opt 명령에는 세 가지 단계가 있으며, 이에 대해 자세히 논의합니다. clock_opt가 완료되면 클록 트리가 구축되고 클록 네트가 완전히 상세 라우팅되며, 신호 네트는 전역적으로 라우팅됩니다. 이 시점에서 CTS를 넘어 신호 네트의 라우팅을 완료하고 설계 플로우를 진행합니다.  
{: .notice--warning}  

## Setup for Classic CTS and CCD Flows

The classic CTS and CCD flows require common setup steps prior to executing CTS and optimization  

- Controlling clock tree balancing
- Handling pre-existing clock tree elements
- Specifying non-default routing and via rules
- Specifying clock timing and DRC constraints

→ Setup is covered in the Setting Up CTS module  

## Running CTS

We will now explore more detail on how to check, configure and then run CTS

## Is the Design Ready for CTS?

```tcl
check_design -checks pre_clock_tree_stage
```

- Creates an EMS database. Use the Message Browser to examine
- The clock_trees check runs the atomic command `check_clock_trees`

It is important to minimize unnecessary debug. We can use check_design before clock_opt to help us establish whether any barriers will gate a successful CTS run. The check_design command will invoke the check_clock_trees command enabling us to review and address any potentially problematic issues.  
불필요한 디버그를 최소화하는 것이 중요합니다. clock_opt 이전에 check_design을 사용하여 CTS 실행을 방해할 수 있는 장애물이 있는지 확인하는 것이 도움이 될 수 있습니다. check_design 명령은 check_clock_trees 명령을 호출하여 잠재적으로 문제가 될 수 있는 사항을 검토하고 해결할 수 있도록 도와줍니다.  
{: .notice--warning}  

## Clock Tree Specific Checks

Our clock tree specific checks are quite extensive and will help you avoid many different issues. There is an expectation that all your clocks are properly defined. We will reveal if this is not the case. You need to know, for example, if any generated clocks have been orphaned or whether you have applied don't touch constraints that prevent specific clock tree library cells from being used. It is very important to study the results of the clock tree checks and look to understand and as necessary address the issues it finds.  
저희의 클록 트리 관련 체크는 다양한 문제를 피하는 데 큰 도움이 됩니다. 모든 클록이 올바르게 정의되어 있는 것이 기대됩니다. 이러한 기대에 어긋나는 경우에는 해당 내용을 알려드립니다. 예를 들어, 생성된 클록이 고아되었는지, 특정 클록 트리 라이브러리 셀 사용을 방지하는 "don't touch" 제약 조건이 적용되었는지 등을 확인해야 합니다. 클록 트리 체크의 결과를 꼼꼼히 분석하고 발견된 문제에 대해 이해하고 필요한 조치를 취하는 것이 매우 중요합니다.  
{: .notice--warning}  

## Modes / Corners for CTS

- Clock trees are built for all clocks in all active setup and/or hold scenarios
- CTS will also perform clock net logical DRC fixing on scenarios enabled for `max_transition` and/or `max_capacitance`
- If you do not want a scenario to be used during CTS:
  - `set_scenario_status -active false {s1}`

Clock_opt will build clock trees for all the clocks in all your active setup or hold scenarios. You must set scenarios inactive if you do not want clock_opt to use them during clock tree synthesis. The CTS engine will ensure transition and capacitance issues are addressed for scenarios that are enabled for max_transition and max_capacitance respectively.  
clock_opt는 모든 활성화된 설정 또는 홀드 시나리오의 모든 클록에 대해 클록 트리를 구축합니다. 만약 clock_opt이 클록 트리 합성 중에 해당 시나리오를 사용하지 않도록 하려면 해당 시나리오를 비활성화해야 합니다. CTS 엔진은 최대 전환 및 최대 용량을 위해 각각 max_transition과 max_capacitance에 대해 활성화된 시나리오에 대한 전환 및 용량 문제를 처리합니다.  
{: .notice--warning}  

## Hold Fixing

- Hold fixing occurs in all scenarios that are enabled for hold:
  - `set_scenario_status {s2} -hold true`
- ICC II will automatically pick the best cells for hold fixing that have a library cell purpose of optimization (or hold)
- Control the effort for hold fixing:

```tcl
set_app_options -name clock_opt.hold.effort \
                -value none|low|medium|high
# Default: medium
```

During CTS hold fixing is addressed for the first time in the design flow. Hold fixing forms part of data path optimisation. The tool will perform hold fixing for any scenario that has been enabled for hold. We can control the effort it puts into hold fixing with an application option. We recommend that you always ensure you have enabled at least one scenario for hold fixing. The tool automatically selects the appropriate buffers for hold fixing. It picks library cells whose library cell purpose includes hold or optimization. As a side note, by default, the hold purpose behaves the same way as the optimization purpose. We therefore recommend only using the optimization purpose. The tool will pick the best library cells for fixing hold timing violations.  
CTS 중에는 홀드(fixing) 수정이 디자인 플로우에서 처음으로 처리됩니다. 홀드 수정은 데이터 경로 최적화의 일부입니다. 도구는 홀드에 대해 활성화된 모든 시나리오에 대해 홀드 수정을 수행합니다. 홀드 수정에 투입되는 노력은 응용 프로그램 옵션으로 제어할 수 있습니다. 항상 적어도 하나의 홀드 수정 시나리오를 활성화하는 것이 좋습니다. 도구는 홀드 수정에 적합한 버퍼를 자동으로 선택합니다. 라이브러리 셀의 셀 용도에 홀드나 최적화가 포함된 라이브러리 셀을 선택합니다. 부가적으로, 기본적으로 홀드 용도는 최적화 용도와 동일하게 작동합니다. 따라서 최적화 용도만 사용하는 것을 권장합니다. 도구는 홀드 타이밍 위반을 수정하기 위해 최적의 라이브러리 셀을 선택합니다.  
{: .notice--warning}  

## Minimize Hold Time Violations in Scan Paths

- Enable scan chains reordering to minimize branch crossings
- Can reduce hold time violations in the scan chain
- `set_app_options -name opt.dft.clock_aware_scan_reorder -value true`

![2023-05-17-min-hold-time-violations-in-scan-path.png]({{site.baseurl}}/assets/images/2023-05-17-min-hold-time-violations-in-scan-path.png){: .align-center}  

By reordering the scan chain wiring we can structure their connections to minimize the amount of hold buffering required. In the left image we can see that the scan chain connects from register A to register B, then C through to F. Registers A, C and E are in one branch sharing one insertion delay while registers B, D and F are on a different branch, sharing a different insertion delay. If we take no action, we have branch crossing at each stage of the scan chain. Each crossing holds the possibility of a skew mismatch that could lead to a hold violation that would only be fixed by hold buffer insertion. If we reorder the scan chains to minimize the clock branch crossing as shown in the right-hand image, then we are left with only one crossing point from register E to register B. This technique should lead to a reduction in the amount of hold buffering that would typically be required. The technique needs to be enabled using the application option shown.  
스캔 체인 배선을 재배열함으로써 홀드 버퍼링이 필요한 양을 최소화할 수 있습니다. 왼쪽 이미지에서는 스캔 체인이 레지스터 A에서 레지스터 B, 그리고 C에서 F까지 연결되는 것을 볼 수 있습니다. 레지스터 A, C, E는 하나의 분기에서 공통의 삽입 지연을 공유하고, 레지스터 B, D, F는 다른 분기에서 다른 삽입 지연을 공유합니다. 아무 조치를 취하지 않으면 스캔 체인의 각 단계마다 분기 교차가 발생합니다. 각 교차점은 홀드 위반이 발생할 수 있는 스키 오차의 가능성을 가지고 있으며, 이는 홀드 버퍼 삽입에 의해 수정되어야만 합니다. 오른쪽 이미지와 같이 스캔 체인을 재배열하여 클록 분기 교차를 최소화하면 레지스터 E에서 레지스터 B로의 교차점이 한 개만 남게 됩니다. 이 기법은 일반적으로 필요한 홀드 버퍼링 양을 줄이는 데 도움이 될 것입니다. 이 기법은 표시된 응용 프로그램 옵션을 사용하여 활성화해야 합니다.  
{: .notice--warning}  

## Clock Reconvergence Pessimism

![2023-05-17-clock-reconvergence-pessimism.png]({{site.baseurl}}/assets/images/2023-05-17-clock-reconvergence-pessimism.png){: .align-center}  

- To reduce OCV effects, clock trees try to share as many buffers as possible
- Remove the pessimism from the timing calculation caused by shared clock paths (late/early for launch/capture calculation)
  - Otherwise, false data path timing will lead to pessimistic clock trees (especially for CCD)

```tcl
set_app_options -name time.remove_clock_reconvergence pessimism
                -value true
# Default: false
```

The concept of clock reconvergence pessimism leads to pessimistic path timing. The image demonstrates the problem. Please focus your attention on the four grey clock tree buffers (U1-U4) that are common to the launch and capture timing paths. When we model on-chip variation effects we are applying timing derates into the timing graph. The traditional method is to apply these as early and late derates using the `set_timing_derate` command. The effect of this when we consider setup timing is the late derate  is applied to the launch timing path and early derate to the capture path. This derate modelling means we see a different delay through U1-U4 when we look at launch vs capture paths. In the real world there is only one delay through these four buffers because they are common to both paths. We should remove the difference caused by the modelled derate effects because it leads to pessimistic clock trees as well as sub-optimal data path optimisation. The technique for removing this pessimism is known as clock reconvergence pessimism removal or CRPR for short. Please note that although here we're discussing CRPR in the context of setup timing, this affects hold timing in an equivalent manner  
클록 재수렴 비관론(clock reconvergence pessimism)은 비관적인 패스 타이밍을 유발하는 개념입니다. 이 이미지에서는 런치 타이밍 경로와 캡처 타이밍 경로에 공통으로 사용되는 네 개의 회색 클록 트리 버퍼(U1-U4)에 주목해 주세요. 온칩 변동 효과를 모델링할 때, 타이밍 그래프에 타이밍 디레이트를 적용합니다. 전통적인 방법은 `set_timing_derate` 명령을 사용하여 초기 및 지연 디레이트를 적용하는 것입니다. 설정 타이밍을 고려할 때, 이 디레이트 모델링은 런치 타이밍 경로에 지연 디레이트를, 캡처 경로에 초기 디레이트를 적용하는 것입니다. 이 디레이트 모델링은 런치 경로와 캡처 경로에서 U1-U4를 통과하는 딜레이가 다르게 나타나는 것을 의미합니다. 실제 세계에서는 이 네 개의 버퍼에 대해 런치와 캡처 경로에서 동일한 딜레이가 발생합니다. 따라서 모델링된 디레이트 효과로 인해 발생하는 차이를 제거해야 합니다. 왜냐하면 이는 비관적인 클록 트리 및 최적화되지 않은 데이터 경로 최적화로 이어질 수 있기 때문입니다. 이 비관론 제거 기술을 클록 재수렴 비관론 제거(CRPR, Clock Reconvergence Pessimism Removal)라고 합니다. 이 때문에 CRPR을 설명할 때 설정 타이밍을 예로 들었지만, 이는 동일한 방식으로 홀드 타이밍에도 영향을 미칩니다.  
{: .notice--warning}  

## Clock Reconvergence Pessimism Removal

In this example setup timing report I'm showing how clock reconvergence pessimism, CRP for short, is removed. As you can see, the value 0.08 is added to the data required time to offset the CRP. If it were a hold timing report, we would need to subtract the CRP from the required path to improve the hold timing.  
이 예시의 설정 타이밍 보고서에서는 클록 재수렴 비관론(Clock Reconvergence Pessimism, CRP)이 제거되는 방법을 보여줍니다. 보시다시피, CRP를 상쇄하기 위해 데이터 필요 시간에 0.08이라는 값이 추가되었습니다. 만약 홀드 타이밍 보고서였다면, 홀드 타이밍을 개선하기 위해 필요한 패스에서 CRP를 빼야 할 것입니다.  
{: .notice--warning}  

## Congestion-Aware Initial CTS (Classic CTS + CCD Flow)

- Initial clock tree synthesis (build_clock) is not congestion aware!
  - Based on virtual routing, by default
- On large complex-floorplan designs, this may lead to:
  - Pre- vs. post-route clock skew, latency, and logical DRC degradation
  - Congestion hotspots
  - Route DRCs
- Enable global routing for congestion estimation and congestion-aware clock tree construction during the build_clock stage:

```tcl
set_app_options -name cts.compile.enable_global_route -value true
# Default: false
```

----------

The `build_clock` phase of clock tree synthesis performs two steps: Initial clock tree synthesis or building of the clock trees (CTS), followed by clock tree optimization (CTO). By default, CTS uses virtual routing, but CTO uses global routing. If you enable the application option `cts.compile.enable_global_route`, global routing will also be used during the CTS step.  

**Virtual routing (VR)** uses the shortest Manhattan distance between two points to estimate wire lengths. VR does not consider areas with high congestion with routing blockages.  

**Global routing (GR)**, on the other hand, sees congestion and routing blockages, and either routes around these areas, or uses higher (not congested or blocked) routing layers. GR more accurately predicts what the final detail routing will look like,  

The first stage of clock_opt is called build_clock and it has two jobs. It's first job is to build the initial buffer trees for your clock network. These buffer trees are primarily focused on meeting your max_transition and max_capacitance rules as well as balancing the endpoint loads. We share the acronym CTS to describe this initial clock tree synthesis. By default, these initial trees are built by virtual routing the connections to guide where the buffers should be physically located. Virtual routing uses the shortest Manhattan distance between any two points to estimate the wirelengths. Using virtual routing is fast but the buffering is not implemented in a congestion aware manner. Therefore, using the virtual route based initial clock tree synthesis with complex floorplans can lead several problems arising. You may find that the real routing topology degrades skew, latency and logical DRC's. You may also find that infeasible buffering decisions have been made leading to routing congestion hotspots that then result in routing DRC's. We can however choose to use global routing during this initial building of the clock trees. This is enabled by setting the application option shown on the slide. Using global routing technology enables the tool to avoid congestion and routing blockages by either routing around problem areas or routing in alternative metal layers. The second job of the build_clock stage is to perform clock tree optimisation, often abbreviated to CTO, whereby the buffers are sized. When you are running the classic clock tree synthesis methodology, they are sized to minimize skew. When you are running the CCD methodology, they are sized to intentionally introduce skew into the clock network. Unlike CTS, CTO always uses global routing with the benefits previously described.  
clock_opt의 첫 번째 단계는 build_clock로, 두 가지 작업이 수행됩니다. 첫 번째 작업은 클록 네트워크를 위한 초기 버퍼 트리를 구축하는 것입니다. 이 버퍼 트리는 주로 max_transition 및 max_capacitance 규칙을 충족시키고 엔드포인트 부하를 균형있게 분배하는 데 초점을 맞춥니다. 이 초기 클록 트리 합성을 설명하기 위해 CTS라는 약어를 공유합니다. 기본적으로 이러한 초기 트리는 버퍼가 물리적으로 위치할 위치를 안내하기 위해 연결을 가상 경로로 라우팅하여 구축됩니다. 가상 경로는 두 지점 사이의 최단 맨해튼 거리를 사용하여 와이어 길이를 추정합니다. 가상 경로를 사용하면 빠르지만, 버퍼링은 혼잡도를 고려하지 않는 방식으로 구현되지 않습니다. 따라서 복잡한 플로어플랜과 함께 가상 경로 기반의 초기 클록 트리 합성을 사용하면 여러 문제가 발생할 수 있습니다. 실제 라우팅 토폴로지가 스키, 레이턴시 및 논리 DRC를 저하시킬 수 있습니다. 또한 현실적으로 불가능한 버퍼링 결정이 이루어져 라우팅 혼잡 지점이 발생하고, 이로 인해 라우팅 DRC가 발생할 수도 있습니다. 그러나 이 초기 클록 트리 구축 단계에서 전역 라우팅을 사용할 수 있습니다. 이는 슬라이드에 표시된 응용 프로그램 옵션을 설정하여 활성화할 수 있습니다. 전역 라우팅 기술을 사용하면 도구가 문제가 있는 지역을 우회하거나 대체 금속 레이어에서 라우팅하여 혼잡과 라우팅 블록을 피할 수 있습니다. build_clock 단계의 두 번째 작업은 클록 트리 최적화를 수행하는 것입니다. 이 단계에서 버퍼의 크기를 조정합니다. 클래식 클록 트리 합성 방법론을 실행할 때는 스키를 최소화하기 위해 버퍼의 크기가 조정됩니다. CCD 방법론을 실행할 때는 의도적으로 클록 네트워크에 스키를 도입하기 위해 버퍼의 크기가 조정됩니다. CTS와 달리 CTO는 항상 이전에 설명한 이점을 가진 전역 라우팅을 사용합니다.  
{: .notice--warning}  

## Local Skew Optimization (Classic CTS + CCD Flow)

![2023-05-17-local-skew-opt.png]({{site.baseurl}}/assets/images/2023-05-17-local-skew-opt.png){: .align-center}  

- Performs timing aware clustering (as opposed to location based)
- Optimizes local skew directly for setup and hold timing critical pairs
- Can relax skew target (if slack is positive) to save clock buffer area

By default, the classic clock tree synthesis algorithm performs global skew optimisation. Global skew optimisation attempts to minimize the latency differences between all the flip flops in the clock tree. The global skew is the resulting time difference between the flip flop with the largest latency and the flip flop with the smallest latency, irrespective of whether there is a timing path between them. In the left image the global skew is 200ps. The local skew in this extreme example also happens to be 200ps. This means that you effectively experience a 200ps reduction in clock period affecting setup timing paths or you see the 200ps skew between connected flip flops hurt their hold timing. Local skew optimisation addresses these challenges by performing three useful operations so that timing related registers have minimal skew between them. -First it performs timing aware clustering. This is where flip flops are grouped into clusters when they have a timing relationship. Their physical location is not considered. -Secondly it optimizes local skew directly for setup or hold timing critical pairs. In the example on the right, the net effect could still be that your global skew remains at 200ps. However, the smallest skew is approximately one third of that, 67ps for each flip flop pair. -Finally, it will relax the skew target when positive timing slack exists on a timing path. This can reduce the amount of buffering which consequentially benefits congestion and power.  
기본적으로 클래식 클록 트리 합성 알고리즘은 전역 스키(전체적인 스키) 최적화를 수행합니다. 전역 스키 최적화는 클록 트리의 모든 플립플롭 간의 지연 차이를 최소화하려고 시도합니다. 전역 스키는 가장 큰 지연을 가진 플립플롭과 가장 작은 지연을 가진 플립플롭 사이의 시간 차이를 의미하며, 이들 간에 타이밍 경로가 있는지 여부에 상관없이 계산됩니다. 좌측 이미지에서 전역 스키는 200ps입니다. 이 극단적인 예시에서 지역 스키도 200ps입니다. 이는 클록 주기에 200ps의 감소가 발생하거나 연결된 플립플롭 간에 200ps의 스키가 홀드 타이밍에 영향을 준다는 것을 의미합니다. 지역 스키 최적화는 이러한 문제를 해결하기 위해 세 가지 유용한 작업을 수행합니다. -첫째로, 타이밍을 고려한 클러스터링을 수행합니다. 이는 타이밍 관계가 있는 플립플롭들을 클러스터로 그룹화하는 것입니다. 물리적인 위치는 고려되지 않습니다. -둘째로, 설정 타이밍이나 홀드 타이밍에 중요한 페어에 대해 지역 스키를 직접 최적화합니다. 오른쪽 예시에서도 전역 스키는 여전히 200ps일 수 있습니다. 그러나 가장 작은 스키는 플립플롭 페어 각각에 대해 약 1/3인 약 67ps입니다. -마지막으로, 타이밍 패스에 양의 타이밍 여유가 있는 경우 스키 목표를 완화합니다. 이는 버퍼링 양을 줄일 수 있으며, 이는 혼잡도와 전력에 이점을 줍니다.  
{: .notice--warning}  

## Enable Local Skew CTS and CTO (Classic CTS + CCD Flow)

- To use local skew for classic CTS, you need to enable it specifically:

```tcl
set_app_options -name cts.compile.enable_local_skew -value true
set_app_options -name cts.optimize.enable_local_skew -value true
```

- When using CCD, local skew optimization is always used
  - Not user-configurable

If you have selected the CCD flow, local skew optimisation is always enabled, and no user interaction is necessary. When you are using the classic clock tree synthesis flow you need to enable local skew optimization to take advantage of the benefits it provides. To do this you need to enable two application options, one for the clock tree compilation phase and the other for the clock tree optimization phase. As a reminder, both application options affect the build_clock stage of clock_opt.  
CCD 플로우를 선택한 경우, 지역 스키 최적화는 항상 활성화되며 사용자의 개입이 필요하지 않습니다. 그러나 클래식 클록 트리 합성 플로우를 사용하는 경우, 지역 스키 최적화를 활성화해야 이에 대한 이점을 얻을 수 있습니다. 이를 위해 클록 트리 컴파일레이션 단계와 클록 트리 최적화 단계에 대해 두 개의 애플리케이션 옵션을 활성화해야 합니다. 다시 말해, 두 애플리케이션 옵션 모두 clock_opt의 build_clock 단계에 영향을 줍니다.  
{: .notice--warning}  

## Restricting the Amount of CCD Skewing (CCD Flow)

- By default, CCD skews as much as needed to meet the timing
- You can limit the amount of skewing if needed, for example:

```tcl
set_app_options -name ccd.max_prepone  -value 0.2
set_app_options -name ccd.max_postpone -value 0.4
```

- ccd.max_prepone: max latency reduction (advance) allowed to a sink
- ccd.max_postpone: max latency increase (delay)
- Default value ‘auto’ means there is no limit in clock_opt
- Setting a limit can impact the timing QoR as this restricts CCD

By default, concurrent clock and data optimization can introduce as much skew as it needs to meet timing. If desired, you can independently restrict the amount of preponing and postponing that CCD is allowed to apply. The arguments to the prepone and postpone application options are both absolute time values expressed in the design units. For example, if the max prepone value is 0.2 and the design units are 1ns then CCD can prepone by 0.2ns. I want to point out that it's generally not recommended to change the default settings, as this may restrict the QoR achievable by CCD.  
기본적으로, CCD(Concurrent Clock and Data) 최적화는 타이밍을 충족하기 위해 필요한 만큼의 스키우를 도입할 수 있습니다. 원하는 경우, CCD가 적용할 수 있는 최대 선행(preponing) 및 연기(postponing) 양을 독립적으로 제한할 수 있습니다. prepone 및 postpone 애플리케이션 옵션에 대한 인자는 모두 디자인 단위로 표현된 절대 시간 값입니다. 예를 들어, 최대 선행 값이 0.2이고 디자인 단위가 1ns인 경우, CCD는 0.2ns까지 선행할 수 있습니다. 하지만 기본 설정을 변경하는 것은 일반적으로 권장되지 않습니다. 왜냐하면 이렇게 하면 CCD로 얻을 수 있는 QoR(Quality of Results)이 제한될 수 있기 때문입니다.  
{: .notice--warning}  

## Skipping Path Groups during CCD

