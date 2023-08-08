---
title: "IC Compiler II - 11 Running CTS"
excerpt: "Block-level Implementation"
date: 2023-05-17 07:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
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

## i.2 Clock Tree Synthesis Goal and Flows

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

## i.3 Comparing Classic CTS versus CCD

![2023-05-17-classic-cts-vs-ccd.png]({{site.baseurl}}/assets/images/2023-05-17-classic-cts-vs-ccd.png){: .align-center}  

Let's compare Classic CTS and CCD. Classic CTS represents the traditional approach to CTS whereby all registers in the fanout cone of a clock source are expecting to receive their clock with the same buffering delay. This results in achieving the goal of minimal skew when measuring the arrival time of the clock between any two registers of a specific clock. Data path optimisation is performed as an independent sub-step where it is not allowed to make any changes to the clock tree. The benefit of this approach is its simplicity. Classic CTS enables a full clock cycle for register to register timing paths. However, one downside of this approach is that some complex data paths can remain problematic to achieve setup and hold timing. Another downside is that by encouraging alignment of clock edges the power of the design is higher than it needs to be. Concurrent Clock and Data, CCD for short, is a fundamentally different clock building strategy where the goal is to meet setup and hold timing. It does not focus on minimizing skew at the register clock pins. It can do this because it has full visibility of the data path timing. Visually you can see that the CCD engine adjusts the delay in the clock tree on a per register basis. Advancing and retarding the latency are known as preponing and postponing respectively. Moving the registers up and down the clock tree relative to each other is an elegant method to structurally adjust setup and hold margins and drive towards timing closure without requiring potentially extensive datapath optimisation. In a CCD flow the datapath optimization is performed in conjunction with building the clock tree. It has the flexibility to tune the clock tree if required. This strategy is illustrated in the diagram with the green buffer. While CCD is more computationally intensive than classic CTS it offers significant benefits in terms of overall timing and power QoR.  
클래식 CTS와 CCD를 비교해 보겠습니다. 클래식 CTS는 클록 소스의 팬아웃 콘에 있는 모든 레지스터가 동일한 버퍼링 지연으로 클록을 받을 것으로 예상하는 전통적인 CTS 접근 방식을 나타냅니다. 이는 특정 클록의 두 레지스터 간 클록 도착 시간을 측정할 때 최소 스큐 목표를 달성하는 결과를 가져옵니다. 데이터 경로 최적화는 클록 트리에 대한 변경을 허용하지 않는 별개의 하위 단계로 수행됩니다. 이 접근 방식의 장점은 간단함입니다. 클래식 CTS는 레지스터 간 타이밍 경로에 대한 전체 클록 주기를 가능하게 합니다. 그러나 이 접근 방식의 단점 중 하나는 일부 복잡한 데이터 경로가 설정 및 홀드 타이밍을 달성하기 어려울 수 있다는 점입니다. 또 다른 단점은 클록 엣지의 정렬을 장려하여 설계의 전력이 필요 이상으로 높아진다는 점입니다. 동시 클록 및 데이터(CCD)는 기본적으로 다른 클럭 구축 전략으로, 설정 및 홀드 타이밍을 충족하는 것이 목표입니다. 이는 레지스터 클록 핀의 스큐 최소화에 초점을 두지 않습니다. 데이터 경로 타이밍에 대한 완전한 가시성을 갖기 때문에 가능한 것입니다. 시각적으로 CCD 엔진은 클록 트리에서 레지스터별로 지연을 조정합니다. 지연을 조기화하는 것을 preponing, 연기하는 것을 postponing이라고 합니다. 레지스터를 클록 트리에서 상대적으로 위아래로 이동시킴으로써 설정 및 홀드 여유를 구조적으로 조정하고, 잠재적으로 광범위한 데이터 경로 최적화 없이도 타이밍 클로저에 가까워지는 방법입니다. CCD 플로우에서 데이터 경로 최적화는 클록 트리 구축과 함께 수행됩니다. 필요한 경우 클록 트리를 조정할 수 있는 유연성을 가지고 있습니다. 이 전략은 그림의 녹색 버퍼로 설명되어 있습니다. CCD는 클래식 CTS보다 계산량이 많지만 전체적인 타이밍 및 전력 QoR 측면에서 상당한 이점을 제공합니다.  
{: .notice--warning}  

## i.4 Classic CTS and CCD Flow

![2023-05-17-classic-cts-and-ccd-flow.png]({{site.baseurl}}/assets/images/2023-05-17-classic-cts-and-ccd-flow.png){: .align-center}  

This flow chart zooms into the CTS step. We have already explained the underlying differences between Classic CTS and CCD. Note however, that the flow around that choice is equivalent for both flows. The first task is to configure our CTS requirements. Please remember we cover these extensively in their own training module. CTS setup includes, amongst other things, defining specific balancing requirements and exceptions, what cells can be used in the clock tree as well as defining routing rules for the clock network. The choices made in this setup phase will determine what type of clock tree structure you end up with The other choice to make before asking IC Compiler II to build your clock tree is to choose which strategy you desire: Classic or CCD. It is then down to the clock_opt command to perform clock tree synthesis and optimisation. To facilitate intermediate debug and analysis the clock_opt command has three stages which we discuss in detail later. When clock_opt completes, your clock tree will be built, the clock nets will be fully detail routed and the signal nets will be globally routed. At this point we then move beyond CTS to complete the routing of the signal nets and further into the design flow.  
이 플로우 차트는 CTS 단계로 들어갑니다. 이미 클래식 CTS와 CCD 사이의 기본적인 차이점을 설명했습니다. 그러나 이 선택 사항에 대한 플로우는 두 플로우 모두 동일합니다. 첫 번째 작업은 CTS 요구 사항을 구성하는 것입니다. CTS 설정에는 균형 조정 요구 사항 및 예외 사항, 클럭 트리에서 사용할 수 있는 셀 정의, 클록 네트워크에 대한 라우팅 규칙 정의 등이 포함됩니다. 이 설정 단계에서 선택한 내용은 최종적으로 어떤 유형의 클록 트리 구조를 갖게 될지 결정합니다. 클록 트리를 생성하려면 IC Compiler II에게 요청하기 전에 Classic 또는 CCD와 같은 전략을 선택해야 합니다. 그런 다음 clock_opt 명령이 클록 트리 합성 및 최적화를 수행합니다. 중간 디버그와 분석을 용이하게 하기 위해 clock_opt 명령에는 세 가지 단계가 있으며, 이에 대해 자세히 논의합니다. clock_opt가 완료되면 클록 트리가 구축되고 클록 네트가 완전히 상세 라우팅되며, 신호 네트는 전역적으로 라우팅됩니다. 이 시점에서 CTS를 넘어 신호 네트의 라우팅을 완료하고 설계 플로우를 진행합니다.  
{: .notice--warning}  

## i.5 Setup for Classic CTS and CCD Flows

The classic CTS and CCD flows require common setup steps prior to executing CTS and optimization  

- Controlling clock tree balancing
- Handling pre-existing clock tree elements
- Specifying non-default routing and via rules
- Specifying clock timing and DRC constraints

→ Setup is covered in the Setting Up CTS module  

## 1.1 Running CTS

We will now explore more detail on how to check, configure and then run CTS

## 1.2 Is the Design Ready for CTS?

```tcl
check_design -checks pre_clock_tree_stage
```

- Creates an EMS database. Use the Message Browser to examine
- The clock_trees check runs the atomic command `check_clock_trees`

It is important to minimize unnecessary debug. We can use check_design before clock_opt to help us establish whether any barriers will gate a successful CTS run. The check_design command will invoke the check_clock_trees command enabling us to review and address any potentially problematic issues.  
불필요한 디버그를 최소화하는 것이 중요합니다. clock_opt 이전에 check_design을 사용하여 CTS 실행을 방해할 수 있는 장애물이 있는지 확인하는 것이 도움이 될 수 있습니다. check_design 명령은 check_clock_trees 명령을 호출하여 잠재적으로 문제가 될 수 있는 사항을 검토하고 해결할 수 있도록 도와줍니다.  
{: .notice--warning}  

## 1.3 Clock Tree Specific Checks

Our clock tree specific checks are quite extensive and will help you avoid many different issues. There is an expectation that all your clocks are properly defined. We will reveal if this is not the case. You need to know, for example, if any generated clocks have been orphaned or whether you have applied don't touch constraints that prevent specific clock tree library cells from being used. It is very important to study the results of the clock tree checks and look to understand and as necessary address the issues it finds.  
저희의 클록 트리 관련 체크는 다양한 문제를 피하는 데 큰 도움이 됩니다. 모든 클록이 올바르게 정의되어 있는 것이 기대됩니다. 이러한 기대에 어긋나는 경우에는 해당 내용을 알려드립니다. 예를 들어, 생성된 클록이 고아되었는지, 특정 클록 트리 라이브러리 셀 사용을 방지하는 "don't touch" 제약 조건이 적용되었는지 등을 확인해야 합니다. 클록 트리 체크의 결과를 꼼꼼히 분석하고 발견된 문제에 대해 이해하고 필요한 조치를 취하는 것이 매우 중요합니다.  
{: .notice--warning}  

## 1.4 Modes / Corners for CTS

- Clock trees are built for all clocks in all active setup and/or hold scenarios
- CTS will also perform clock net logical DRC fixing on scenarios enabled for `max_transition` and/or `max_capacitance`
- If you do not want a scenario to be used during CTS:
  - `set_scenario_status -active false {s1}`

Clock_opt will build clock trees for all the clocks in all your active setup or hold scenarios. You must set scenarios inactive if you do not want clock_opt to use them during clock tree synthesis. The CTS engine will ensure transition and capacitance issues are addressed for scenarios that are enabled for max_transition and max_capacitance respectively.  
clock_opt는 모든 활성화된 설정 또는 홀드 시나리오의 모든 클록에 대해 클록 트리를 구축합니다. 만약 clock_opt이 클록 트리 합성 중에 해당 시나리오를 사용하지 않도록 하려면 해당 시나리오를 비활성화해야 합니다. CTS 엔진은 최대 전환 및 최대 용량을 위해 각각 max_transition과 max_capacitance에 대해 활성화된 시나리오에 대한 전환 및 용량 문제를 처리합니다.  
{: .notice--warning}  

## 1.5 Hold Fixing

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

## 1.6 Minimize Hold Time Violations in Scan Paths

- Enable scan chains reordering to minimize branch crossings
- Can reduce hold time violations in the scan chain
- `set_app_options -name opt.dft.clock_aware_scan_reorder -value true`

![2023-05-17-min-hold-time-violations-in-scan-path.png]({{site.baseurl}}/assets/images/2023-05-17-min-hold-time-violations-in-scan-path.png){: .align-center}  

By reordering the scan chain wiring we can structure their connections to minimize the amount of hold buffering required. In the left image we can see that the scan chain connects from register A to register B, then C through to F. Registers A, C and E are in one branch sharing one insertion delay while registers B, D and F are on a different branch, sharing a different insertion delay. If we take no action, we have branch crossing at each stage of the scan chain. Each crossing holds the possibility of a skew mismatch that could lead to a hold violation that would only be fixed by hold buffer insertion. If we reorder the scan chains to minimize the clock branch crossing as shown in the right-hand image, then we are left with only one crossing point from register E to register B. This technique should lead to a reduction in the amount of hold buffering that would typically be required. The technique needs to be enabled using the application option shown.  
스캔 체인 배선을 재배열함으로써 홀드 버퍼링이 필요한 양을 최소화할 수 있습니다. 왼쪽 이미지에서는 스캔 체인이 레지스터 A에서 레지스터 B, 그리고 C에서 F까지 연결되는 것을 볼 수 있습니다. 레지스터 A, C, E는 하나의 분기에서 공통의 삽입 지연을 공유하고, 레지스터 B, D, F는 다른 분기에서 다른 삽입 지연을 공유합니다. 아무 조치를 취하지 않으면 스캔 체인의 각 단계마다 분기 교차가 발생합니다. 각 교차점은 홀드 위반이 발생할 수 있는 스키 오차의 가능성을 가지고 있으며, 이는 홀드 버퍼 삽입에 의해 수정되어야만 합니다. 오른쪽 이미지와 같이 스캔 체인을 재배열하여 클록 분기 교차를 최소화하면 레지스터 E에서 레지스터 B로의 교차점이 한 개만 남게 됩니다. 이 기법은 일반적으로 필요한 홀드 버퍼링 양을 줄이는 데 도움이 될 것입니다. 이 기법은 표시된 응용 프로그램 옵션을 사용하여 활성화해야 합니다.  
{: .notice--warning}  

## 1.7 Clock Reconvergence Pessimism

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

## 1.8 Clock Reconvergence Pessimism Removal

In this example setup timing report I'm showing how clock reconvergence pessimism, CRP for short, is removed. As you can see, the value 0.08 is added to the data required time to offset the CRP. If it were a hold timing report, we would need to subtract the CRP from the required path to improve the hold timing.  
이 예시의 설정 타이밍 보고서에서는 클록 재수렴 비관론(Clock Reconvergence Pessimism, CRP)이 제거되는 방법을 보여줍니다. 보시다시피, CRP를 상쇄하기 위해 데이터 필요 시간에 0.08이라는 값이 추가되었습니다. 만약 홀드 타이밍 보고서였다면, 홀드 타이밍을 개선하기 위해 필요한 패스에서 CRP를 빼야 할 것입니다.  
{: .notice--warning}  

## 1.9 Congestion-Aware Initial CTS (Classic CTS + CCD Flow)

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

## 1.10 Local Skew Optimization (Classic CTS + CCD Flow)

![2023-05-17-local-skew-opt.png]({{site.baseurl}}/assets/images/2023-05-17-local-skew-opt.png){: .align-center}  

- Performs timing aware clustering (as opposed to location based)
- Optimizes local skew directly for setup and hold timing critical pairs
- Can relax skew target (if slack is positive) to save clock buffer area

By default, the classic clock tree synthesis algorithm performs global skew optimisation. Global skew optimisation attempts to minimize the latency differences between all the flip flops in the clock tree. The global skew is the resulting time difference between the flip flop with the largest latency and the flip flop with the smallest latency, irrespective of whether there is a timing path between them. In the left image the global skew is 200ps. The local skew in this extreme example also happens to be 200ps. This means that you effectively experience a 200ps reduction in clock period affecting setup timing paths or you see the 200ps skew between connected flip flops hurt their hold timing. Local skew optimisation addresses these challenges by performing three useful operations so that timing related registers have minimal skew between them. -First it performs timing aware clustering. This is where flip flops are grouped into clusters when they have a timing relationship. Their physical location is not considered. -Secondly it optimizes local skew directly for setup or hold timing critical pairs. In the example on the right, the net effect could still be that your global skew remains at 200ps. However, the smallest skew is approximately one third of that, 67ps for each flip flop pair. -Finally, it will relax the skew target when positive timing slack exists on a timing path. This can reduce the amount of buffering which consequentially benefits congestion and power.  
기본적으로 클래식 클록 트리 합성 알고리즘은 전역 스키(전체적인 스키) 최적화를 수행합니다. 전역 스키 최적화는 클록 트리의 모든 플립플롭 간의 지연 차이를 최소화하려고 시도합니다. 전역 스키는 가장 큰 지연을 가진 플립플롭과 가장 작은 지연을 가진 플립플롭 사이의 시간 차이를 의미하며, 이들 간에 타이밍 경로가 있는지 여부에 상관없이 계산됩니다. 좌측 이미지에서 전역 스키는 200ps입니다. 이 극단적인 예시에서 지역 스키도 200ps입니다. 이는 클록 주기에 200ps의 감소가 발생하거나 연결된 플립플롭 간에 200ps의 스키가 홀드 타이밍에 영향을 준다는 것을 의미합니다. 지역 스키 최적화는 이러한 문제를 해결하기 위해 세 가지 유용한 작업을 수행합니다. -첫째로, 타이밍을 고려한 클러스터링을 수행합니다. 이는 타이밍 관계가 있는 플립플롭들을 클러스터로 그룹화하는 것입니다. 물리적인 위치는 고려되지 않습니다. -둘째로, 설정 타이밍이나 홀드 타이밍에 중요한 페어에 대해 지역 스키를 직접 최적화합니다. 오른쪽 예시에서도 전역 스키는 여전히 200ps일 수 있습니다. 그러나 가장 작은 스키는 플립플롭 페어 각각에 대해 약 1/3인 약 67ps입니다. -마지막으로, 타이밍 패스에 양의 타이밍 여유가 있는 경우 스키 목표를 완화합니다. 이는 버퍼링 양을 줄일 수 있으며, 이는 혼잡도와 전력에 이점을 줍니다.  
{: .notice--warning}  

## 1.11 Enable Local Skew CTS and CTO (Classic CTS + CCD Flow)

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

## 1.12 Restricting the Amount of CCD Skewing (CCD Flow)

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

## 1.13 Skipping Path Groups during CCD (CCD Flow)

- You can configure CCD to skip the clock pins that belong to particular path groups during clock latency adjustment

- Datapath optimization will still be performed on these path groups
  - If you specify a path group name without a scenario name, all path groups with the name specified from all scenarios are skipped
  - If you specify a path group name with a scenario name, only the path group in the specified scenario is skipped

```tcl
set_app_options -name ccd.skip_path_groups
                -value { pathgroup1 {scenarioA pathgroup2} }
```

You can configure CCD to skip clock pins that belong to specified path groups during clock latency adjustment. However, datapath optimization will still be performed on these path groups. You control this feature using the application option shown on the slide. The application option allows you to limit the choice of path groups to a specific scenario if you prefer. Alternatively, you can skip all path groups with the specified name.  
CCD에서는 클럭 지연 조정 중 특정 경로 그룹에 속하는 클럭 핀을 건너뛰도록 설정할 수 있습니다. 그러나 이러한 경로 그룹에 대해서는 데이터패스 최적화는 여전히 수행됩니다. 이 기능은 슬라이드에 표시된 애플리케이션 옵션을 사용하여 제어할 수 있습니다. 애플리케이션 옵션을 사용하면 특정 시나리오에 대한 경로 그룹 선택을 제한할 수 있습니다. 또는 지정된 이름의 모든 경로 그룹을 건너뛸 수도 있습니다.  
{: .notice--warning}  

## 1.14 Ignore I/O Timing for Boundary Register Skewing (CCD Flow)

![2023-05-17-ignore-io-timing-for-skew.png]({{site.baseurl}}/assets/images/2023-05-17-ignore-io-timing-for-skew.png){: .align-center}  

Situations arise where it becomes desirable to prevent CCD skewing the clocks based on IO timing budgets. In the example shown, FF1 fails to meet its input setup timing budget. We also have a small setup timing violation between FF1 and FF2. The FF1 timing violation may simply exist because the input timing budgets are currently pessimistic. If we were to allow CCD to address the violation associated with In1 it would add latency to FF1 with consequences for Path A. If we know that this input violation isn't real, perhaps we are in the early stages of our project, we can take action to prevent CCD from attempting to fix it. The methodology is to instruct CCD to skip specific path groups. For the example shown, we ensure that In1 is placed in its own path group and then pass that path group, MY_IN, into the application option we just discussed. As illustrated in the lower image, CCD will no longer address the input timing path but is able to reduce the latency on FF1 to close the timing in PathA.  
특정 IO 타이밍 예산에 따라 CCD가 클럭을 변형하는 것을 방지하는 것이 바람직한 경우가 발생할 수 있습니다. 예시에서는 FF1이 입력 설정 타이밍 예산을 충족하지 못하고 있습니다. 또한 FF1과 FF2 사이에 작은 설정 타이밍 위반이 있습니다. FF1의 타이밍 위반은 현재 비관적인 입력 타이밍 예산 때문에 발생할 수 있습니다. In1과 관련된 위반이 CCD에 해결되도록 허용한다면 FF1에 지연이 추가되어 Path A에 영향을 줄 수 있습니다. 이러한 입력 위반이 실제로 존재하지 않는 것을 알고 있다면(프로젝트 초기 단계 등), CCD가 이를 수정하려는 시도를 방지할 수 있도록 조치를 취할 수 있습니다. 이 방법론은 CCD에게 특정 경로 그룹을 건너뛰도록 지시하는 것입니다. 예시에서는 In1이 자체 경로 그룹에 배치되도록 하고, 그 경로 그룹(MY_IN)을 방금 논의한 애플리케이션 옵션에 전달합니다. 아래 이미지에서 설명한대로, CCD는 더 이상 입력 타이밍 경로에 대해 처리하지 않지만 FF1의 지연을 줄여 PathA의 타이밍을 닫을 수 있습니다.  
{: .notice--warning}  

## 1.15 CCD Optimization and Boundary Registers (CCD Flow)

![2023-05-17-ccd-opt-bnd-regs.png]({{site.baseurl}}/assets/images/2023-05-17-ccd-opt-bnd-regs.png){: .align-center}  

By default, CCD will skew all registers, including boundary registers. Examples of boundary register are shown in red in the picture. They are registers that are connected to input and output ports of a design. We can tell the tool that it is not allowed to make any latency adjustments on boundary registers by setting the application option to false. However, while possible, we don't recommend that you exclude all boundary registers because that can severely impact what CCD can do, especially to adjust the internal register paths.  
기본적으로 CCD는 경계 레지스터를 포함하여 모든 레지스터를 조정합니다. 예시에서는 경계 레지스터가 사진에서 빨간색으로 표시되었습니다. 이들은 설계의 입력 및 출력 포트에 연결된 레지스터입니다. 애플리케이션 옵션을 false로 설정하여 도구에게 경계 레지스터에 대한 지연 조정을 허용하지 않도록 알릴 수 있습니다. 그러나 가능한 경우에도 내부 레지스터 경로를 조정하는 데 CCD가 할 수 있는 작업에 심각한 영향을 미칠 수 있으므로 모든 경계 레지스터를 제외하는 것은 권장되지 않습니다.  
{: .notice--warning}  

![2023-05-17-ccd-opt-bnd-regs2.png]({{site.baseurl}}/assets/images/2023-05-17-ccd-opt-bnd-regs2.png){: .align-center}  

By default, any register that connects to a boundary port, like D-in or D-out in the picture, are automatically considered as boundary registers. In the pictures please notice that the reset boundary port connects to many of the registers. Unless we take further action, none of the registers connected to reset would be optimized by CCD. They would all be ignored because the tool understands that their reset pins are connected to a boundary port. With the application option shown, we can tell the tool to automatically ignore scan and reset ports when performing boundary register identification. If necessary, we can provide a list of specific ports to ignore in addition to scan and reset ports.  
기본적으로, 그림에서 보듯이 D-in 또는 D-out과 같은 경계 포트에 연결되는 모든 레지스터는 자동으로 경계 레지스터로 간주됩니다. 사진에서 확인해 주세요. 리셋 경계 포트는 많은 레지스터에 연결되어 있습니다. 추가 조치를 취하지 않는 한, 리셋에 연결된 모든 레지스터는 CCD에 의해 최적화되지 않습니다. 이 도구는 리셋 핀이 경계 포트에 연결되어 있다고 인식하기 때문에 이들은 모두 무시됩니다. 표시된 애플리케이션 옵션을 사용하여 도구에게 경계 레지스터 식별 시 스캔 및 리셋 포트를 자동으로 무시하도록 알릴 수 있습니다. 필요한 경우, 스캔 및 리셋 포트 외에도 무시해야 할 특정 포트 목록을 제공할 수도 있습니다.  
{: .notice--warning}  

## 1.17 Hold Criticality during CCD (CCD Flow)

- Although CCD setup optimizations are hold-aware, hold violations can be introduced to improve setup timing
  - Usually, data path optimizations are able to address these hold violations
- You may want to prioritize hold during CCD if
  - Fixing hold in your design is difficult / not possible
  - Your design is area-critical, and the hold-buffer area increase is to be avoided
- To specify hold criticality during CCD, use:
  - `ccd.hold_control_effort` (Default: low)
  - Set to `medium`, `high` or `ultra` to reduce hold degradation
  - Use only if hold timing is critical, since setup optimizations may degrade
  - Only affects `final_opto`, as well as CCD optimization during `route_opt`

CCD setup optimizations are aware of their impact on hold timing. However, while optimizing setup timing, hold violations can still be introduced. This is allowed because subsequent data path optimizations are able to easily fix the remaining hold violations. Situations arise where you might want to prioritize hold during CCD. For example, fixing hold in your design may be very difficult or it's going to be impossible through regular means. You may have a design that is area critical and you need to minimize the penalty from regular hold buffer insertion. The application option shown is used to change the priority of hold timing during CCD latency adjustments or skewing. Be default, the low setting ensures that setup is given the highest priority, by changing this option to medium, high or ultra you indicate the increasing importance of hold. A side effect of increasing the effort level for hold may be a reduction in setup QoR. Just to emphasize, this option does not control the hold fixing during data path optimization,  that is controlled using the app option clock_opt.hold.effort.  
CCD의 설정 최적화는 홀드 타이밍에 대한 영향을 인식합니다. 그러나 설정 타이밍을 최적화하는 동안에도 홀드 위반은 여전히 발생할 수 있습니다. 이는 이후 데이터패스 최적화가 남아있는 홀드 위반을 쉽게 수정할 수 있기 때문에 허용됩니다. CCD에서 홀드를 우선순위로 두려는 상황이 발생할 수 있습니다. 예를 들어, 설계에서 홀드를 수정하는 것은 매우 어렵거나 일반적인 수단으로는 불가능할 수 있습니다. 설계가 면적이 중요하며 일반적인 홀드 버퍼 삽입으로 인한 페널티를 최소화해야 할 수도 있습니다. 표시된 애플리케이션 옵션은 CCD 지연 조정 또는 스큐 중 홀드 타이밍의 우선순위를 변경하는 데 사용됩니다. 기본적으로 낮은 설정은 설정에 가장 높은 우선순위를 부여합니다. 이 옵션을 중간, 높음 또는 매우로 변경함으로써 홀드의 중요도를 점차 증가시킬 수 있습니다. 홀드에 대한 노력 수준을 높이는 부작용은 설정 품질을 감소시킬 수 있습니다. 강조하자면, 이 옵션은 데이터패스 최적화 중 홀드 수정을 제어하지 않습니다. 이는 앱 옵션 clock_opt.hold.effort를 사용하여 제어됩니다.  
{: .notice--warning}  

## 1.18 CCD Focus on Sub-Critical Paths (CCD Flow)

- CCD focuses on the WNS path per path group, i.e. the critical paths
  - Sub-critical paths do not benefit from CCD optimization by default
- To focus on worst 300 sub-critical paths per path group:
  - `ccd.enable_top_wns_optimization` (Default: false)
- Use targeted CCD to prioritize CCD optimization for a few path groups or paths
  - For example, ICG or memory paths (details on next page)
- The above features apply to CCD operations in `place_opt` and `clock_opt` `build_clock`

By default, CCD focuses on the critical timing path in each path group. If you want CCD to also work on sub-critical timing paths you need to set this application option to true When you enable this feature, CCD limits you to 300 sub-critical paths per path group. This is a hard-coded number that you cannot adjust. There is an additional method if this doesn't meet your needs that we will cover on the next page. We call it targeted CCD optimization. Both CCD on sub-critical paths and targeted CCD features apply to CCD operations in place_opt as well as clock_opt during the build_clock stage.  
기본적으로 CCD는 각 경로 그룹의 중요한 타이밍 경로에 초점을 맞춥니다. CCD가 부분적으로 중요하지 않은 타이밍 경로에도 작동하도록 하려면 이 애플리케이션 옵션을 true로 설정해야 합니다. 이 기능을 활성화하면 CCD는 경로 그룹 당 최대 300개의 부분적으로 중요하지 않은 경로로 제한됩니다. 이는 조정할 수 없는 하드 코딩된 숫자입니다. 이 요구 사항을 충족시키지 못하는 경우 다음 페이지에서 다룰 추가적인 방법이 있습니다. 우리는 이를 "타겟팅된 CCD 최적화"라고 부릅니다. 부분적으로 중요하지 않은 경로에서의 CCD 및 타겟팅된 CCD 기능은 build_clock 단계에서의 place_opt 및 clock_opt에서의 CCD 작업에 모두 적용됩니다.  
{: .notice--warning}  

## 1.19 Targeted CCD (CCD Flow)

![2023-05-17-targeted-ccd.png]({{site.baseurl}}/assets/images/2023-05-17-targeted-ccd.png){: .align-center}  

Targeted CCD allows you to configure the CCD engine to prioritize certain path groups of your choice over others. These are path groups that would normally not be optimized by CCD because they are sub-critical, for example if your WNS is at -0.2 and these path groups are at -0.1, CCD would not optimize. But due to your design knowledge, you can force CCD to work on these paths anyway. In the example we're specifying that we want the tool to prioritize two path groups, one containing some critical ICGs, the other containing paths to a critical RAM. You can also force CCD to prioritize specific critical endpoints. To do this you first have to create a file which lists all of the different critical endpoints that you care about, and then use the targeted CCD endpoint file application option where the value of that application option is the name of that file. An important consideration to keep in mind is that if you specify both path groups and endpoints, then CCD only works on the endpoints that are part of both the path groups AND the endpoint list. As a word of caution, please only specify the most timing critical path groups and endpoints. Don't be tempted to specify all paths in your design, this would defeat the purpose and actually revert back to the default behaviour.  
타겟팅된 CCD를 사용하면 CCD 엔진을 다른 것보다 우선하는 원하는 경로 그룹을 구성할 수 있습니다. 이러한 경로 그룹은 일반적으로 부분적으로 중요하지 않기 때문에 CCD에 의해 최적화되지 않을 것입니다. 예를 들어, WNS가 -0.2이고 이러한 경로 그룹이 -0.1인 경우 CCD는 최적화하지 않을 것입니다. 그러나 설계 지식에 따라 CCD를 이러한 경로에 작동하도록 강제할 수 있습니다. 예시에서는 중요한 ICG를 포함한 경로 그룹과 중요한 RAM으로의 경로를 우선순위로 지정하도록 도구에게 알려주고 있습니다. 또한 CCD에게 특정한 중요한 엔드포인트에 우선순위를 부여할 수도 있습니다. 이를 위해 먼저 관심 있는 모든 다른 중요한 엔드포인트를 나열하는 파일을 작성한 다음, 타겟팅된 CCD 엔드포인트 파일 애플리케이션 옵션을 사용하여 그 파일의 이름을 값으로 지정해야 합니다. 유념해야 할 중요한 사항은 경로 그룹과 엔드포인트를 모두 지정하는 경우, CCD가 경로 그룹 및 엔드포인트 목록에 모두 속하는 엔드포인트에만 작동한다는 것입니다. 주의해야 할 점은 가장 타이밍이 중요한 경로 그룹과 엔드포인트만 지정하십시오. 설계의 모든 경로를 지정하지 마십시오. 그렇게 하면 목적을 달성하지 못하고 실제로 기본 동작으로 되돌아가게 됩니다.  
{: .notice--warning}  

## 1.20 Classic CTS and CCD Execution

- Clock tree synthesis, clock tree detail routing, data path optimization and signal global routing are all executed by: **`clock_opt`**
  - CCD is enabled by default (`clock_opt.flow.enable_ccd`)
- The three stages of `clock_opt` are:
  - `build_clock` → `route_clock` → `final_opto`
- You can control which stages are executed with `-from/-to`
  - For example, to perform only clock tree synthesis (CTS+CTO):
    - `clock_opt -to build_clock`

CTS buffers will have the prefix name `"CT_BUF_####"` by default.  
This can be modified with:  
`cts.common.user_instance_name_prefix`  
`opt.common.user instance_name prefix`  
{: .notice--info}

Classic clock tree synthesis and CCD are both executed using the same command: `clock_opt`. The choice of flow is controlled by the application option shown on the slide. The tool default is CCD. You will need to set the application option to false if you want the classic CTS flow. Both flows run through the same three stages that are `build_clock`, `route_clock`, `final_opto`. We recommend that at least in the early stages of your project that you run each stage independently. This allows you to stop and analyse the results before making any adjustments and re-running the stage. You can control what stage to run using the -from and -to options of the `clock_opt` command  
클래식 클럭 트리 합성과 CCD 모두 동일한 명령인 `clock_opt`를 사용하여 실행됩니다. 플로우 선택은 슬라이드에 표시된 애플리케이션 옵션에 의해 제어됩니다. 도구의 기본값은 CCD입니다. 클래식 CTS 플로우를 원하는 경우 애플리케이션 옵션을 false로 설정해야 합니다. 두 플로우 모두 `build_clock`, `route_clock`, `final_opto` 세 단계를 거칩니다. 프로젝트 초기 단계에서는 각 단계를 독립적으로 실행하는 것이 좋습니다. 이렇게 하면 조정 및 해당 단계 재실행 전에 결과를 중지하여 분석할 수 있습니다. `clock_opt` 명령의 -from 및 -to 옵션을 사용하여 실행할 단계를 제어할 수 있습니다.  
{: .notice--warning}  

## 1.21 Classic CTS Flow Stages (Classic CTS)

![2023-05-17-classic-cts-flow.png]({{site.baseurl}}/assets/images/2023-05-17-classic-cts-flow.png){: .align-center}  

- After the `build_clock` phase, the clock trees are global-routed. The global routing occurs during the “optimization” (CTO) part.  
If you enable the application option `cts.compile.enable_global_route`, global routing will also be performed during the “compile” (CTS) part.

Let's look at what happens in these three stages when we turn off CCD and run the classic CTS flow. The build_clock stage is going to build the clock tree buffer structure and perform inter-clock balancing for minimum skew. The result is a global routed clock tree. Remember that the initial clock tree construction doesn't use global routing by default, click on the notes button for more information. It then updates the IO latencies. The route_clock stage detail-routes the clock nets which we will talk about later. In the final_opto stage we perform global routing on all the signal nets and perform data path optimization to meet setup and hold timing and close logical DRC's. Remember that when the tool builds the clock tree using Classic CTS, it has no concept of data path timing, and when it's performing data path optimization, it never goes back and modifies the clock tree.  
만약 CCD를 끄고 클래식 CTS 플로우를 실행한다면, 세 단계에서 어떤 일이 발생하는지 살펴보겠습니다. 첫 번째로, build_clock 단계에서는 클럭 트리 버퍼 구조를 구축하고 최소 스키우를 위한 클럭 간 균형을 수행합니다. 결과적으로 글로벌 라우팅된 클럭 트리가 생성됩니다. 초기 클럭 트리 구성은 기본적으로 글로벌 라우팅을 사용하지 않습니다. 그런 다음 IO 지연 시간을 업데이트합니다. 두 번째로, route_clock 단계에서는 클럭 넷을 디테일하게 라우팅합니다. 이에 대해 나중에 더 자세히 이야기하겠습니다. 마지막으로, final_opto 단계에서는 모든 신호 넷에 대해 글로벌 라우팅을 수행하고 설정 및 홀드 타이밍을 충족시키고 논리 DRC를 해결하기 위한 데이터 패스 최적화를 수행합니다. 기억해야 할 점은 도구가 클래식 CTS를 사용하여 클럭 트리를 구축할 때 데이터 패스 타이밍에 대한 개념이 없으며, 데이터 패스 최적화를 수행할 때 클럭 트리를 다시 수정하지 않는다는 것입니다.  
{: .notice--warning}  

## 1.22 Classic CTS Flow: `clock_opt` vs. Atomic (Classic CTS)

![2023-05-17-clock-opt-vs-atomic.png]({{site.baseurl}}/assets/images/2023-05-17-clock-opt-vs-atomic.png){: .align-center}  

----------

- `"final_opto"` is the third and last stage of `clock_opt`
- Performs logic and placement optimization of non-clock logic to fix setup/hold timing and max tran/cap (DRC) violations
- Tries to minimize cell disturbances

The `build_clock` and `route_clock` stages each perform multiple tasks. While we recommend that you run the `build_clock` stage and then stop and analyse results before you continue, we acknowledge that it may be useful if you could break this down further into atomic commands. We can accommodate this because the `build_clock` stage splits into three atomic commands: `synthesize_clock_trees`, `balance_clock_groups` and `compute_clock_latency`. `synthesize_clock_trees` performs the CTS and CTO functions. By default, CTS only balances within a clock domain, and it is `balance_clock_groups` that performs the CTO function of inter clock balancing. Perhaps you have a particularly complex clock tree so you could choose to use these atomic commands to build and check an individual clock tree before continuing. The `route_clock` stage also has its task of clock net routing. `final_opto` does not have any underlying atomic commands.  
`build_clock` 단계와 `route_clock` 단계는 각각 여러 작업을 수행합니다. `build_clock` 단계를 실행한 후 결과를 중지하여 분석한 후 계속 진행하는 것을 권장하지만, 더 상세한 단계로 분할할 수 있다면 유용할 것으로 인식합니다. `build_clock` 단계는 `synthesize_clock_trees`, `balance_clock_groups` 및 `compute_clock_latency` 세 개의 원자적인 명령으로 분할될 수 있습니다. `synthesize_clock_trees`은 CTS 및 CTO 기능을 수행합니다. 기본적으로 CTS는 클럭 도메인 내에서만 균형을 조정하며, inter clock 균형 조정 기능을 수행하는 것은 `balance_clock_groups`입니다. 특히 복잡한 클럭 트리를 가지고 있다면, 이러한 원자적인 명령을 사용하여 개별 클럭 트리를 구축하고 확인하는 데 사용할 수 있습니다. `route_clock` 단계는 클럭 넷 라우팅 작업을 수행합니다. `final_opto`에는 기본적인 원자적인 명령이 없습니다.  
{: .notice--warning}  

## 1.23 CCD Flow Stages (CCD Flow)

![2023-05-17-ccd-flow-stages.png]({{site.baseurl}}/assets/images/2023-05-17-ccd-flow-stages.png){: .align-center}  

Concurrent Clock & Data optimization combines useful skew CTS and data-path optimization, in order to achieve best post-CTS results.  
Provides concurrent setup/hold timing and power optimization to achieve good overall QoR  

- Optimizes setup timing with lowest cost on clock power
- Optimizes hold timing to reduce hold buffer cost with minimal impact on clock buffer cost
- Optimizations occur across all active, timing-enabled scenarios

The clock optimizations during final_opto include on-route and off-route buffering, sizing and moves, buffer removal and reparenting.  

If we look at what happens for CCD we find the same three stages are used. However, with a CCD flow the `build_clock` and `final_opto` stages are different In the `build_clock` stage, the focus is on building timing-driven clock trees. In other words, the goal is now to meet timing and not to minimize skew. It also performs inter-clock balancing and again the goal is improve setup and hold timing plus logical DRCs and not to minimize skew. IO port clock latencies are updated at the end. The `route_clock` stage is the same as the classic CTS flow, all clocks are detail routed. In the `final_opto` stage when the tool is optimizing the data path logic to meet timing it will concurrently perform clock tree optimization to further improve setup or hold timing and address max transition and max capacitance violations. There are no atomic command equivalents when using CCD.  
CCD를 사용하는 경우에도 동일한 세 단계가 사용됩니다. 그러나 CCD 플로우에서는 `build_clock`와 `final_opto` 단계가 다릅니다. `build_clock` 단계에서는 타이밍 주도 클럭 트리를 구축하는 데 초점이 맞춰집니다. 다시 말해, 목표는 타이밍 충족이며, 스키우를 최소화하는 것이 목표가 아닙니다. 클럭 간 균형 조정도 수행되며, 설정 및 홀드 타이밍 및 논리 DRC 개선을 위한 목표입니다. 마지막에는 IO 포트 클럭 지연이 업데이트됩니다. `route_clock` 단계는 클래식 CTS 플로우와 동일하며, 모든 클럭이 디테일 라우팅됩니다. `final_opto` 단계에서 도구가 타이밍 충족을 위해 데이터 패스 로직을 최적화하는 동시에 클럭 트리 최적화를 수행하여 설정 또는 홀드 타이밍을 더욱 개선하고 max 전환 및 max 용량 위반을 처리합니다. CCD를 사용할 때는 원자적인 명령 상당자가 없습니다.  
{: .notice--warning}  

## 1.24 Post-CTS: Post-route Clock Tree Optimization (Classic CTS)

- Problems:
  - Possibility of clock tree QoR degradation (skew, latency, logical DRC) due to miscorrelation between global route (`build_clock`) and detail route (`route_clock`)
  - Clock tree QoR can further degrade due to coupling capacitance and crosstalk effects
- Solution: Post-route clock tree optimization
  - Works on **detail-routed clock** trees to address these degradations
  - Optimizations performed include on-route buffer insertion, buffer sizing, and gate sizing
  - `synthesize_clock_trees -postroute`
- Post-route CTO can also be performed after **signal** routing
- **Do not use if you have used CCD**

## 2.1 Analyzing the Clock Tree

## 2.2 Analyzing the Clock Tree

![2023-05-17-analyzing-the-clock-tree.png]({{site.baseurl}}/assets/images/2023-05-17-analyzing-the-clock-tree.png){: .align-center}  

After running the build_clock stage of clock_opt we recommend you analyse your clock tree before proceeding to the routing and data path optimization stages. As shown, there are a range of aspects: -Check that there are no unfixed DRCs -Check if the clock latency, skew, clock area and power achieved are reasonable for your design goals. -Check whether there is any routing congestion after CTS? Please remember that in the classic CTS flow, clock skew refers to global clock skew and in the CCD flow, the tool optimizes for timing and improves local skew.  
clock_opt의 build_clock 단계를 실행한 후 라우팅 및 데이터 패스 최적화 단계로 진행하기 전에 클럭 트리를 분석하는 것을 권장합니다. 아래는 분석해야 할 다양한 측면입니다: -해결되지 않은 DRC(Density Rule Check)가 있는지 확인하세요. -클럭 지연, 스키우, 클럭 영역 및 전력이 설계 목표에 합리적인지 확인하세요. -CTS 이후에 라우팅 혼잡도가 있는지 확인하세요. 클래식 CTS 플로우에서 클럭 스키우는 글로벌 클럭 스키우를 의미하며, CCD 플로우에서는 도구가 타이밍을 최적화하고 지역적인 스키우를 개선합니다.  
{: .notice--warning}  

## 2.3 Reporting Clock metrics

`report_clock_qor` is the go-to command to analyse the clock tree  

- Summary report
- Latency report
- Area report

The `report_clock_qor` command is your go-to command to analyse the clock tree. It can be used to show a summary of all your design's clocks… it can be used to display the shortest and longest latencies across your corners The latency histogram is quite useful for a high-level latency distribution And the area report can help you gain valuable insight. In the lab, you will get a chance to learn about a number of `report_clock_qor`'s options, so we won't go into details here.  
`report_clock_qor` 명령은 클럭 트리를 분석하는 데 사용되는 주요 명령입니다. 이 명령은 설계의 모든 클럭에 대한 요약을 보여주는 데 사용될 수 있습니다. 또한 각 코너에서 가장 짧은 지연과 가장 긴 지연을 표시하는 데 사용될 수 있습니다. 지연 히스토그램은 고수준의 지연 분포를 시각적으로 표시하는 데 유용합니다. 또한 영역 보고서를 통해 중요한 통찰력을 얻을 수 있습니다. 실습에서는 `report_clock_qor의` 다양한 옵션에 대해 자세히 배우게 될 것이므로 여기에서는 자세히 다루지 않겠습니다.  
{: .notice--warning}  

## 2.4 Analysis Using the CTS GUI

Menu: Window > New Clock Tree Analysis Window  

![2023-05-17-analyzing-using-cts-guipng]({{site.baseurl}}/assets/images/2023-05-17-analyzing-using-cts-guipng){: .align-center}  

There are also lots of wonderful things in the GUI that can help you analyse your clock tree. The CTS browser helps you to identify clock tree objects, properties and attributes. You can traverse your clock tree levels. The clock tree schematic shown on the lower left allows you to trace your clock paths and cross probe with the layout. The abstraction produced by the clock tree levelized graph or the latency graph are often the preferred way users approach analysing their clock tree.  
GUI에서는 클럭 트리를 분석하는 데 도움이 되는 다양한 기능이 있습니다. CTS 브라우저를 사용하면 클럭 트리 개체, 속성 및 속성을 식별할 수 있습니다. 클럭 트리 수준을 탐색할 수 있습니다. 왼쪽 하단에 표시된 클럭 트리 도표를 통해 클럭 경로를 추적하고 레이아웃과 교차 프로브를 수행할 수 있습니다. 클럭 트리 레벨별 그래프나 지연 그래프로 생성된 추상화는 사용자가 클럭 트리를 분석하는 선호하는 방법입니다.  
{: .notice--warning}  

## 2.5 GUI Analysis: Clock Tree Levelized and Latency Graphs

![2023-05-17-gui-analysis-clock-tree-levelized.png]({{site.baseurl}}/assets/images/2023-05-17-gui-analysis-clock-tree-levelized.png){: .align-center}  

Let's investigate these in a little more detail. The coloured objects conveniently tell you what cell type you are looking at without needing to zoom into the detail. For example, and by default the clock gating cells and multiplexors are red while sequential cells are blue. Most importantly, this graph will collapse multiple cells into one, making it much easier to visualize large clock trees. You will get a chance to explore these graphs in the lab.  
자세히 살펴보도록 하겠습니다. 색상이 지정된 개체들은 자세한 내용을 확대하지 않아도 보고 있는 셀 유형을 편리하게 알려줍니다. 예를 들어, 기본적으로 클럭 게이팅 셀과 멀티플렉서는 빨간색으로 표시되며, 순차 셀은 파란색입니다. 가장 중요한 것은 이 그래프에서 여러 셀을 하나로 축소하여 대규모 클럭 트리를 시각화하는 데 매우 유용하게 사용할 수 있다는 점입니다. 실습에서 이러한 그래프를 탐색하는 기회를 갖게 될 것입니다.  
{: .notice--warning}  

## 3.1 Post CTS Optimization

## 3.2 Pre-CTS: Ideal Clock Latencies

![2023-05-17-pre-cts-ideal-clock-latencies.png]({{site.baseurl}}/assets/images/2023-05-17-pre-cts-ideal-clock-latencies.png){: .align-center}  

Before you have built your clock trees, we describe the clock networks as being ideal. This means that by default, the delay between your clock port and all the clock pins are zero. You can model the expected clock network latency by applying a set_clock_latency constraint. In this example, we have applied a maximum set_clock_latency value of one onto the clock. This is then applied to the entire clock network. Notice that your internal clock network has a latency of one as well as any logic connected to your input port through a set input delay. Consequently, the data arrival time at the input port IN1 is going to be the specified input delay of 2.2 plus the ideal latency of 1.0 equals 3.2. An equivalent calculation is made for the output ports.  
클럭 트리를 구축하기 전에, 클럭 네트워크를 이상적으로 설명합니다. 기본적으로 클럭 포트와 모든 클럭 핀 간의 지연은 0입니다. set_clock_latency 제약을 적용하여 예상되는 클럭 네트워크 지연을 모델링할 수 있습니다. 이 예시에서는 클럭에 최대 set_clock_latency 값으로 1을 적용했습니다. 이 값은 전체 클럭 네트워크에 적용됩니다. 내부 클럭 네트워크와 입력 포트를 통해 연결된 로직도 동일한 1의 지연을 갖습니다. 따라서 입력 포트 IN1에서의 데이터 도착 시간은 지정된 입력 지연 2.2와 이상적인 1.0의 지연을 더한 3.2가 됩니다. 출력 포트에 대해서도 동일한 계산이 이루어집니다.  
{: .notice--warning}  

## 3.3 Post-CTS: Propagated Clock Sources / Updated Latencies

![2023-05-17-pre-cts-propaged-clock-sources.png]({{site.baseurl}}/assets/images/2023-05-17-pre-cts-propaged-clock-sources.png){: .align-center}  

During the first stage of the `clock_opt` command, at `build_clock`, the tool automatically applies a `set_propagated_clock` constraint to all clock sources. At this point the tool no longer needs to use the values from the `set_clock_latency` constraint that you had specified earlier. It means that the actual clock network latencies after the clock tree buffers have been inserted are going to be used. This only applies to the internal clock network. The I/O reference clocks are obviously not buffered, and continue to use `set_clock_latency`. But this latency is automatically changed to the median clock latency of all the input registers. So in our case, we got a calculated media latency of 0.92, and together with the unchanged input delay constraint, we now have a port data arrival of 3.12, instead of 3.2 as on the previous page. Please note that if a clock does not have a `set_clock_latency` constraint before clock_opt the automatic latency adjustment will apply a new network latency constraint on that clock.  
`clock_opt` 명령어의 첫 번째 단계인 `build_clock`에서 도구는 모든 클럭 소스에 자동으로 `set_propagated_clock` 제약을 적용합니다. 이 시점에서 도구는 이전에 지정한 `set_clock_latency` 제약의 값을 더 이상 사용하지 않습니다. 즉, 클럭 트리 버퍼가 삽입된 이후의 실제 클럭 네트워크 지연이 사용됩니다. 이는 내부 클럭 네트워크에만 적용됩니다. I/O 참조 클럭은 물론 버퍼링되지 않으며, 여전히 `set_clock_latency`를 사용합니다. 그러나 이 지연 시간은 자동으로 모든 입력 레지스터의 중앙 클럭 지연 시간으로 변경됩니다. 우리의 경우, 계산된 중앙 지연 시간은 0.92이며, 변경되지 않은 입력 지연 제약 조건과 함께, 이전 페이지에서의 3.2 대신 이제 3.12의 포트 데이터 도착 시간을 갖게 됩니다. 클럭_opt 이전에 클럭에 `set_clock_latency` 제약 조건이 없는 경우 자동으로 새로운 네트워크 지연 제약이 적용됩니다.  
{: .notice--warning}  

## 3.4 Post-CTS: Update Latencies in ALL Scenarios

- Only clocks in active scenarios are propagated and get updated latencies
- If not all scenarios were active during CTS, explicitly update clock network latencies in all scenarios immediately after CTS
  - This also makes any non-propagated clock sources propagated

```tcl
clock_opt -to route_clock
set_scenario_status -active true [all_scenarios]

# To make clocks propagated without latency updating:
# synthesize clock trees -propagate_only

compute_clock_latency
```

IO latency updates only apply to clocks in active scenarios during `clock_opt`. They are propagated and get updated latencies. If you invoke the `clock_opt` command with a subset of all your scenarios, then there will be scenarios that are still using ideal clock and non-updated clock latencies when it completes. To update the rest of those scenarios, right after `clock_opt`, activate all your scenarios and invoke `synthesize_clock_trees -propagate_only` followed by `compute_clock_latency`. This will make all the clocks propagated and update the ideal latency to match the median of the interface register latencies as we discussed previously. If you only wish to make the clocks propagated but do not want to update the IO latencies, then you would use `synthesize_clock_trees -propagate_only`. Do not run `compute_clock_latency`. I want to highlight that this is not recommended for CCD, because timing across all scenarios should be taken into account during `build_clock`. By disabling some scenarios, CCD will not be able to get a full picture of the timing.  
`clock_opt` 중에는 IO 지연 업데이트가 활성 시나리오의 클럭에만 적용됩니다. 이러한 업데이트는 전파되어 클럭의 지연 시간이 업데이트됩니다. `clock_opt` 명령을 모든 시나리오의 하위 집합으로 호출하는 경우, 완료된 시점에서 여전히 이상적인 클럭과 업데이트되지 않은 클럭 지연을 사용하는 시나리오가 있을 수 있습니다. 나머지 시나리오를 업데이트하려면 `clock_opt` 이후에 모든 시나리오를 활성화하고 `synthesize_clock_trees -propagate_only`를 호출한 다음 `compute_clock_latency`를 실행하면 됩니다. 이렇게 하면 모든 클럭이 전파되고 이상적인 지연 시간이 인터페이스 레지스터 지연의 중앙값과 일치하도록 업데이트됩니다. 클럭을 전파하기만 하고 IO 지연을 업데이트하고 싶지 않은 경우에는 `synthesize_clock_trees -propagate_only`를 사용하고 `compute_clock_latency`를 실행하지 않으면 됩니다. 그러나 이는 CCD에는 권장되지 않습니다. 왜냐하면 `build_clock` 단계에서 모든 시나리오의 타이밍을 고려해야 하기 때문입니다. 일부 시나리오를 비활성화하면 CCD가 타이밍의 전체적인 상황을 파악하지 못할 수 있습니다.  
{: .notice--warning}  

## 3.5 Auto-Update with Virtual Clocks

![2023-05-17-auto-update-with-virtual-clocks.png]({{site.baseurl}}/assets/images/2023-05-17-auto-update-with-virtual-clocks.png){: .align-center}  

By default, the `compute_clock_latency` command will copy the network latency of the specified `reference_clock` to the network latency of the `clocks_to_update`. If the early and late latency of the reference clock are different because of on chip variation (OCV), then the applied early and late latencies will also be different.  
When `-ocv_included` is specified, it will be assumed the OCV for the `clocks_to_update` will already be accounted for in the clocks’ source latency, and that you don’t want to apply extra OCV in the network latency. When `-ocv_included` is specified, the `compute_clock_latency` command will apply the single average of the early and late network latencies of the reference clock to the `clocks_to_update`.  

We can also automatically update latencies for IO ports that have been constrained with virtual clocks. In the example, we have our real clock called CLK that connects to the CLOCK port and our virtual clock called V_CLK that is not associated with a real port. Both are specified with a clock period of 4 and a pre-CTS clock latency of 1. In this instance, our data input, IN1 is constrained against the virtual clock. To enable the virtual clock latency to be updated we need to tell clock_opt wherever there is a relationship between a real clock and a virtual clock. This is done using the `set_latency_adjustment_options` command as shown. After clock_opt the latency of the virtual clock is automatically updated to the median value of the real tree as we've seen before.  
가상 클록으로 제약 조건이 있는 IO 포트의 지연 시간도 자동으로 업데이트할 수 있습니다. 이 예시에서는 실제 클록인 CLK가 CLOCK 포트에 연결되고 가상 클록인 V_CLK는 실제 포트와 연관되지 않습니다. 두 클록 모두 클록 주기가 4이고 CTS 이전 클록 지연이 1로 지정되었습니다. 이 경우 데이터 입력인 IN1은 가상 클록에 제약 조건이 걸려 있습니다. 가상 클록의 지연 시간을 업데이트하려면 실제 클록과 가상 클록 간의 관계가 있는 위치를 clock_opt에 알려주어야 합니다. 이는 `set_latency_adjustment_options` 명령을 사용하여 수행됩니다. clock_opt 이후에 가상 클록의 지연 시간은 이전과 마찬가지로 실제 트리의 중앙값으로 자동으로 업데이트됩니다.  
{: .notice--warning}  

## 3.6 Limiting Auto-Latency Update

- clock_opt always performs latency update
  - Occurs during the build_clock phase
  - Stand-alone command: compute_clock_latency
- You can configure I/O latency updates using `set_latency_adjustment_options`
  - By default, all I/Os are updated
- To disable automatic latency updates for some or for all clocks:
  - `set_latency_adjustment_options -exclude_clocks "CLK1 CLK2 ..."`

By default, `clock_opt` will always perform a latency update either during the `route_clock` phase or using the standalone command `compute_clock_latency`. If you do not want all the IO latencies to be updated you can configure what you want to exclude using the `set_latency_adjustment_options` command.  
기본적으로 `clock_opt`는 `route_clock` 단계에서 또는 독립적인 `compute_clock_latency` 명령을 사용하여 항상 지연 시간 업데이트를 수행합니다. 만약 모든 IO 지연 시간을 업데이트하고 싶지 않다면, `set_latency_adjustment_options` 명령을 사용하여 제외할 대상을 구성할 수 있습니다.  
{: .notice--warning}  

## 4.1 Post CTS Optimization

## 4.2 Post-CTS Optimization with GRE

![2023-05-17-post-cts-opt-with-gre.png]({{site.baseurl}}/assets/images/2023-05-17-post-cts-opt-with-gre.png){: .align-center}  

`clock_opt` `final_opto` performs global-route-based optimization, which gives better correlation with postroute timing and faster design closure. This greatly enhances the global route optimization that used to have to be run as an extra step after `final_opto`. You may see this global-route-based optimization referred to with the three-letter acronym "GRE". The flow looks like this: First, virtual route-based optimization is run. This addresses many of the larger timing violations introduced by the transition from ideal to computed clocks. Then, all signal nets in the design are global routed. After this, global route based optimization is run. This includes many of the same types of optimization techniques used during preroute optimization, such as sizing, cell relocation, CCD, and rebuffering of critical nets. But all of this optimization is aware of the RC delays due to the global routed nets. After global route based optimization completes, the global routing is incrementally rerun. In the next step of the flow after `clock_opt`, when routing is run, global routing is no longer needed because the design is already global routed coming out of `clock_opt`. `route_auto` will run only track assignment and detail routing. You may see a warning in the logfile that global routing is skipped during `route_auto`, but this is nothing to worry about. It is fully expected in the `clock_opt` global-route-based optimization flow.  
`clock_opt`의 `final_opto` 단계는 글로벌 라우팅 기반 최적화를 수행하여 postroute 타이밍과 더 좋은 상관관계를 제공하고 빠른 설계 완료를 가능하게 합니다. 이는 이전에 `final_opto` 이후에 추가 단계로 실행되던 글로벌 라우팅 최적화를 크게 향상시킵니다. 이 글로벌 라우팅 기반 최적화는 종종 세 글자 약어 "GRE"로 언급됩니다. 플로우는 다음과 같이 진행됩니다: 먼저, 가상 라우팅 기반 최적화가 실행됩니다. 이는 이상적인 클록에서 계산된 클록으로의 전환으로 인해 발생하는 많은 큰 타이밍 위반을 해결합니다. 그런 다음, 설계의 모든 신호 넷이 글로벌 라우팅됩니다. 이후에 글로벌 라우트 기반 최적화가 실행됩니다. 이는 사이징, 셀 이동, CCD 및 중요 넷의 리버퍼링과 같은 프리라우트 최적화에서 사용되는 동일한 종류의 최적화 기법을 포함합니다. 그러나 이 모든 최적화는 글로벌 라우팅된 넷에 의한 RC 지연을 고려합니다. 글로벌 라우트 기반 최적화가 완료되면 글로벌 라우팅이 점진적으로 다시 실행됩니다. `clock_opt` 이후 플로우의 다음 단계에서 라우팅이 실행되면 글로벌 라우팅은 더 이상 필요하지 않습니다. `route_auto`는 트랙 할당과 디테일 라우팅만 수행합니다. `route_auto` 중에 글로벌 라우팅이 건너뛰어진다는 로그 파일의 경고 메시지가 표시될 수 있지만, 이는 걱정할 필요가 없습니다. 이는 `clock_opt` 글로벌 라우팅 기반 최적화 플로우에서 완전히 예상되는 동작입니다.  
{: .notice--warning}  

## 4.3 Global Route (GR)

- Global Routing (GR) assigns nets to specific metal layers (in the preferred routing direction) and through specific global routing cells (GRC or Gcells)
- No metal traces are created
  - Uses colored zero-width lines to indicate global routes and assigned metal layers
- GR tries to avoid congested Gcells while minimizing detours
  - Congestion exists when more tracks are needed than available
  - Detours increase wire length (delay)
- GR also avoids:
  - P/G (rings/straps/rails)
  - Routing blockages

![2023-05-17-global-route.png]({{site.baseurl}}/assets/images/2023-05-17-global-route.png){: .align-center}  

The global router is run prior to the global-route-based optimization in `clock_opt`. During global routing, nets are assigned to specific metal layers following the preferred routing direction. These nets are not precisely routed the way the detail router will do. Instead, they are assigned into buckets called global routing cells, also called GRCs or Gcells. The width or height of one Gcell can accommodate several routing tracks, so global routing will assign multiple nets on top of each other through the same Gcell as an approximate routing location. The goal of global routing is to assign layers and route all the nets in the design through the Gcells while minimizing routing detours. Detours may be necessary in order to reduce routing congestion. Congestion occurs when the capacity of a Gcell is exceeded by the demand. Capacity is the number of routes a Gcell can hold, and demand is the number of global routes that actually pass through the Gcell. Global routing is both PG and blockage aware, so will only use legal, available Gcells for routing the nets.  
`clock_opt`에서 글로벌 라우팅 기반 최적화 이전에 글로벌 라우터가 실행됩니다. 글로벌 라우팅 중에는 선호하는 라우팅 방향을 따라 넷이 특정 금속 레이어에 할당됩니다. 이러한 넷들은 디테일 라우터가 수행하는 정확한 라우팅 방식과는 다릅니다. 대신, 글로벌 라우팅 셀 또는 GRC(Global Routing Cell) 또는 Gcell로 불리는 버킷에 할당됩니다. 하나의 Gcell의 폭 또는 높이는 여러 라우팅 트랙을 수용할 수 있으므로, 글로벌 라우팅은 대략적인 라우팅 위치로 같은 Gcell을 통해 여러 넷을 겹쳐서 할당합니다. 글로벌 라우팅의 목표는 Gcell을 통해 레이어를 할당하고 설계 내의 모든 넷을 글로벌 라우팅 셀을 통해 라우팅하는 동안 라우팅 우회를 최소화하는 것입니다. 라우팅 혼잡을 줄이기 위해 우회가 필요할 수 있습니다. 혼잡은 Gcell의 수용량을 넘는 수요로 발생합니다. 수용량은 Gcell이 수용할 수 있는 라우트의 수를 나타내고, 수요는 실제로 Gcell을 통과하는 글로벌 라우트의 수를 나타냅니다. 글로벌 라우팅은 PG(Placement Grid) 및 블록에 대해 인식하므로 넷을 라우팅하기 위해 법적이고 사용 가능한 Gcell만 사용합니다.  
{: .notice--warning}  

## 4.4 Global Route (GR)

- Global routing does not create metal shapes
  - You can select and interact with global routing in the GUI

![2023-05-17-global-route2.png]({{site.baseurl}}/assets/images/2023-05-17-global-route2.png){: .align-center}  

No actual metal routes are created in the database, though global routing topologies called "segments" can be queried. These segments have either horizontal or vertical direction and do have specific routing layers. When visualized in the GUI, a segment is a zero-width line that traverses through the middle of the Gcells.  
실제로 데이터베이스에는 실제 금속 라우트가 생성되지 않습니다. 그러나 "세그먼트"라고 불리는 글로벌 라우팅 토폴로지를 조회할 수는 있습니다. 이러한 세그먼트는 수평 또는 수직 방향을 가지며 특정한 라우팅 레이어를 갖습니다. GUI에서 시각화될 때, 세그먼트는 Gcell의 중앙을 통과하는 폭이 없는 선으로 표시됩니다.  
{: .notice--warning}  

## 4.5 Routing Setup

- Global Routing is a routing operation, you need to apply routing setup no later than before the `clock_opt` `final_opto` stage
  - Set up routing rules, guides, shields, redundant vias
  - Any pre-routes need to be handled beforehand (e.g., secondary PG routing)
  - Ensure you set up routing-related application options (`route.common.*`, `route.global.*`)
  - Detailed route setup discussed in the “Detail Signal Routing” module

```tcl
create_routing_rule ..
create_routing_guide ..

set_app_options -name route.common.* ...

set_app_options -name route.global.* ..
set_app_options -name route.global.effort_level -value high

clock_opt -from final_opto
```

Though detail routing of signal nets is not done during `clock_opt`, global routing is still a routing operation, and honors the design's routing constraints, such as non-default routing rules, layer assignments, routing guides, shielding rules, and redundant vias. All of these constraints should be applied before `clock_opt` `final_opto` is executed. Additionally, routing related application options, in particular any with the names `route.common.*` and `route.global.*`, need to be applied prior to `clock_opt` `final_opto` as well. Preroutes should be taken care of before `clock_opt` `final_opto`. This includes secondary PG pin detailed routing, which should be done using the `route_group` command. For higher effort global routing, the `route_global` command has a `-effort` option. This option cannot be used since you global routing is automatically run during `clock_opt` rather than separately with the `route_global` command. Instead, the `route.global.effort_level` application option can be used to configure the global routing effort.  
`clock_opt` 중에는 신호 넷의 디테일 라우팅이 수행되지는 않지만, 글로벌 라우팅은 여전히 라우팅 작업이며, 디자인의 라우팅 제약 조건을 준수합니다. 이는 비-기본 라우팅 규칙, 레이어 할당, 라우팅 가이드, 쉴딩 규칙 및 중복 비아 등의 라우팅 제약 조건을 의미합니다. 이러한 제약 조건은 `clock_opt` `final_opto가` 실행되기 전에 적용되어야 합니다. 또한, 라우팅 관련 애플리케이션 옵션 중에서 `route.common.*` 및 `route.global.*`과 같은 이름을 갖는 옵션도 `clock_opt` `final_opto` 이전에 적용되어야 합니다. `clock_opt` `final_opto` 이전에 Preroute를 처리해야 합니다. 이는 라우트 그룹 명령을 사용하여 수행되어야 하는 보조 PG 핀 디테일 라우팅을 포함합니다. 더 높은 노력의 글로벌 라우팅을 위해서는 `route_global` 명령에 `-effort` 옵션이 있습니다. 그러나 글로벌 라우팅은 `route_global` 명령으로 별도로 실행하는 것이 아니라 `clock_opt` 중에 자동으로 실행되므로 이 옵션은 사용할 수 없습니다. 대신, `route.global.effort_level` 애플리케이션 옵션을 사용하여 글로벌 라우팅의 노력 수준을 구성할 수 있습니다.  
{: .notice--warning}  

## 4.6 Timing Application Options Settings

- Align timing application options of clock opt final_opto with route
  - Do not enable `time.si_enable_analysis` during `clock_opt` `final_opto`, leave at default
  - Optionally, set advanced timer options for tighter timing correlation when practical

```tcl
# .. routing setup ..
# Set advanced timer options:

set_app_options -name time.delay_calc_waveform_analysis_mode -value full_design
set_app_options -name time.enable_ccs_rcv_cap -value true
reset_app_options time.si_enable_analysis

clock_opt -from final_opto

set_app_options -name time.si_enable_analysis -value true

route_auto ;# performs track assign and detail route
```

Often, timer settings are adjusted prior to running routing. Like other settings, these also should be applied prior to `clock_opt` `final_opto` to ensure that global routing integrated into the command sees a consistent timer setup with the track and detail routing that will follow `clock_opt`. There is one exception to this rule, which is the `time.si_enable_analysis` application option that enables crosstalk signal integrity analysis in the timer. This application option should remain false during `clock_opt` and only be enabled prior to running `route_auto` or `route_track` and `route_detail` after `clock_opt`. Enabling the SI analysis during the global-route-based optimization will increase runtime, and isn't guaranteed to have good correlation with the final SI picture, since tracks are not yet assigned during global routing and the crosstalk victim and aggressor nets can't be accurately paired up. This script shows an example of the timer setup for `clock_opt` `final_opto` and routing. Common advanced timer settings that are enabled for detail routing only (typically because of runtime impact) are shown here. The first application option `time.delay_calc_waveform_analysis` enables advanced waveform propagation in the timer. The second application option `time.enable_ccs_rcv_cap` enables the CCS receiver capacitance modeling. `time.si_enable_analysis` is left at its default value for `clock_opt` `final_opto`, and only set true prior to running route_auto.  
라우팅을 실행하기 전에 타이머 설정을 조정하는 경우가 많습니다. 다른 설정과 마찬가지로, 글로벌 라우팅이 포함된 `clock_opt` `final_opto` 이전에 적용되어야 합니다. 이렇게 하면 `clock_opt` 이후에 이어지는 트랙 및 디테일 라우팅과 일관된 타이머 설정을 보장할 수 있습니다. 그러나 이 규칙에는 하나의 예외가 있습니다. `time.si_enable_analysis` 애플리케이션 옵션은 타이머에서 크로스토크 신호 무결성 분석을 활성화합니다. 이 애플리케이션 옵션은 `clock_opt` 중에는 false로 유지되어야 하며, `clock_opt` 이후에 `route_auto` 또는 `route_track` 및 `route_detail`을 실행하기 전에만 활성화되어야 합니다. 글로벌 라우팅 중에 SI 분석을 활성화하면 실행 시간이 증가하며, 트랙이 아직 할당되지 않았기 때문에 크로스토크 피해자 및 공격자 넷을 정확하게 연결할 수 없으므로 최종 SI 이미지와의 일관성이 보장되지 않을 수 있습니다. 이 스크립트는 `clock_opt` `final_opto` 및 라우팅을 위한 타이머 설정의 예를 보여줍니다. 일반적으로 디테일 라우팅에서만 활성화되는 일반 고급 타이머 설정이 여기에 표시되어 있습니다. 첫 번째 애플리케이션 옵션인 `time.delay_calc_waveform_analysis`는 타이머에서 고급 웨이브폼 전파를 활성화합니다. 두 번째 애플리케이션 옵션인 `time.enable_ccs_rcv_cap`은 CCS 수신기 용량 모델링을 활성화합니다. `time.si_enable_analysis`는 `clock_opt` `final_opto`에서 기본값으로 유지되고, route_auto를 실행하기 전에만 true로 설정됩니다.  
{: .notice--warning}  

## 4.7 Scenario Alignment

![2023-05-17-scenario-alignment.png]({{site.baseurl}}/assets/images/2023-05-17-scenario-alignment.png){: .align-center}  

The active timing scenarios should also be aligned between `clock_opt` `final_opto` and routing and postroute optimization. Keeping consistent scenarios isn't just important for optimization, but also for routing. Typically, timing-driven routing is enabled for most flows, and the routing priorities given to nets can change dramatically with different scenario setups. Sometimes only one scenario is used for routing, even when multiple scenarios are used for postroute optimization. In this case, the best practice is to align the `clock_opt` `final_opto` scenarios with the postroute optimization or route_opt scenarios. And additionally, to set the `route.common.focus_scenario` application option to the scenario that is enabled for routing, so the global routing within `clock_opt` uses that scenario. This ensures that optimization sees consistent scenarios, and that routing during all phases focuses on the same focus scenario.  
`clock_opt` `final_opto`와 라우팅 및 포스트라우트 최적화 사이에는 활성 타이밍 시나리오도 일치시켜야 합니다. 일관된 시나리오 유지는 최적화뿐만 아니라 라우팅에도 중요합니다. 일반적으로 대부분의 플로우에서 타이밍 주도 라우팅이 활성화되며, 넷에 대한 라우팅 우선순위는 다른 시나리오 설정에 따라 크게 변경될 수 있습니다. 때로는 라우팅에 하나의 시나리오만 사용되는 경우가 있으며, 포스트라우트 최적화에는 여러 시나리오가 사용되는 경우도 있습니다. 이 경우, 가장 좋은 방법은 `clock_opt` `final_opto` 시나리오를 포스트라우트 최적화 또는 route_opt 시나리오와 일치시키는 것입니다. 추가로 `route.common.focus_scenario` 애플리케이션 옵션을 라우팅에 사용되는 시나리오로 설정하여, `clock_opt` 내의 글로벌 라우팅이 해당 시나리오를 사용하도록 합니다. 이렇게 하면 최적화가 일관된 시나리오를 인식하고, 모든 단계에서의 라우팅이 동일한 주요 시나리오에 초점을 맞출 수 있습니다.  
{: .notice--warning}  

## 4.8 GRE Optimization Overview

![2023-05-17-gre-opt-overview.png]({{site.baseurl}}/assets/images/2023-05-17-gre-opt-overview.png){: .align-center}  

- Good correlation between post-CTS and post-route optimization
- Simultaneously optimize for timing and congestion
  - Timing is global routing based
  - Optimization is congestion aware
  - Routing is timing driven
  - All optimization techniques supported
  - Layer promotion, rerouting and rebuffering

Reviewing the global-route-based optimization that we just discussed: the main purpose of this flow is to provide improved correlation between post-CTS and post-route optimization. This technology is on-by-default in 2020.09 with the acronym GRE. If you are a long-time user of Synopsys implementation tools, you may have previously seen an earlier version of this technology , packaged in a fouth stage of the `clock_opt` command called `global_route_opt`. That stage is now retired, and GRE provides similar but improved functionality built-in to `clock_opt` `final_opto`. Global-route-based optimization allows for simultaneous optimization of timing and congestion. All of the timing analysis is global-route based for more accurate RC delays. The optimization engines are congestion aware and use an up-to-date global route congestion map for a current understanding of the congestion picture. Timing-driven routing is enabled by default. Global route optimization uses the same basic toolbox of optimization techniques that are used throughout preroute optimization, only with added awareness of the global routing, both in terms of the RC delays as well as optimization's impact on routing and congestion. Layer-aware optimization techniques like layer promotion and rebuilding / rerouting buffer trees are empowered during global-route based optimization.  
전반적으로 다룬 글로벌 라우트 기반 최적화(GRE)에 대해 다시 한 번 리뷰해보겠습니다. 이 플로우의 주요 목적은 CTS 이후와 라우트 이후 최적화 사이의 상관 관계를 향상시키는 것입니다. 이 기술은 2020.09 버전부터 기본으로 활성화되었으며, GRE라는 약어로 불립니다. Synopsys의 구현 도구를 오랫동안 사용하신 분들은 이 기술의 이전 버전을 본 적이 있을 수도 있습니다. 그 이전에는 `clock_opt` 명령의 네 번째 단계인 `global_route_opt`에서 이 기술이 제공되었습니다. 그러나 해당 단계는 현재 폐기되었으며, GRE는 `clock_opt` `final_opto`에 내장된 비슷한 기능을 개선하여 제공합니다. 글로벌 라우트 기반 최적화는 타이밍과 혼잡도의 동시 최적화를 가능하게 합니다. 모든 타이밍 분석은 보다 정확한 RC 지연을 위해 글로벌 라우팅 기반으로 수행됩니다. 최적화 엔진은 혼잡도를 인식하며, 최신의 글로벌 라우트 혼잡도 맵을 사용하여 현재 혼잡도 상황을 파악합니다. 타이밍 주도 라우팅은 기본으로 활성화됩니다. 글로벌 라우트 최적화는 프리라우트 최적화 전반에서 사용되는 최적화 기법과 동일한 기본 도구 상자를 사용하지만, 글로벌 라우팅에 대한 인식과 RC 지연에 대한 최적화의 영향을 포함하여 라우팅 및 혼잡도에 대한 이해도를 높였습니다. 레이어 인식 최적화 기법인 레이어 프로모션 및 버퍼 트리의 재구성/재라우팅도 글로벌 라우트 기반 최적화에서 가능해졌습니다.  
{: .notice--warning}  

## 4.9 CCD Clock Power or Area Recovery (Classic CTS + CCD Flow)

![2023-05-17-ccd-clock-power-or-area-recovery.png]({{site.baseurl}}/assets/images/2023-05-17-ccd-clock-power-or-area-recovery.png){: .align-center}  

- The CCD engine can recover power or area from the clock network
  - Power/area recovered without any timing QoR impact
  - Optimization includes repeater removal, clock cell and register sizing
  - Applies to either the classic CTS or CCD flow
  - Runs during the `final_opto` stage of `clock_opt`
    - Can also be enabled during post-route optimization (`route_opt`)
- Choose between power or area recovery:
  - Power: Should use accurate SAIF files for accurate dynamic power calculation
  - Area: Use if accurate SAIF files are not available or if area is an optimization goal

The CCD engine is also used to recover power or area from your clock network without any impact on your timing quality of results. In the example shown CCD can still meet timing by removing a buffer to prepone the clock and by downsizing a register. These optimization techniques apply to both classic and CCD flows. They run during the `final_opto` stage of `clock_opt`. As a side note, you can also enable it during `route_opt`. Our recommendation is that if you have accurate switching activity then work on power recovery while if you don't, then choose area optimization.  
CCD 엔진은 타이밍 결과에 영향을 미치지 않으면서 클럭 네트워크에서 전력 또는 면적을 회수하는 데에도 사용됩니다. 표시된 예시에서 CCD는 버퍼를 제거하여 클럭을 미루고, 레지스터를 축소함으로써 여전히 타이밍을 충족시킬 수 있습니다. 이러한 최적화 기법은 클래식 및 CCD 플로우 모두에 적용됩니다. 이는 `clock_opt`의 `final_opto` 단계에서 실행됩니다. 부가적으로, `route_opt` 중에도 활성화할 수 있습니다. 우리의 권장사항은 정확한 스위칭 액티비티가 있는 경우 전력 회수에 집중하고, 그렇지 않은 경우 면적 최적화를 선택하는 것입니다.  
{: .notice--warning}  

## 4.10 Enable CCD Clock Power/Area Recovery

- When CCD clock power/area recovery is enabled:
  - In the CCD flow, useful skew is used both for timing and power/area improvements
  - In the classic CTS flow, useful skew is used only for power/area improvements
    - → global skew/latency can degrade

```tcl
set_app_options -name clock_opt.flow.enable_clock_power_recovery \
                -value area | power | auto | none
clock_opt
```

- Default setting auto:  
  - In CCD flow, power recovery is enabled if scenario configured for power, otherwise does area recovery
  - Innon-CCD flow, no recovery occurs (→ none)

Our application option `clock_opt.flow.enable_clock_power_recovery` is used to enable CCD power recovery. Its default setting is auto. In the CCD flow this means that it's going to focus on power recovery. In the non-CCD flow, nothing happens, so it's as though it was set to none. You can override that. If you're in a classic CTS flow, you can explicitly set it to power or area. Or, if you're in the CCD flow and you want to focus on area, you would explicitly set it to area. In the CCD flow, useful skew is used for both timing and power improvements. In other words, CCD is always going to try to work on improving timing, but you're adding power and area recovery to that. In the classic CTS flow, there is no equivalent of CCD's timing improvement. Therefore, in classic CTS, the CCD flow will only be used to improve power and area. But without damaging timing.  
우리의 애플리케이션 옵션 `clock_opt.flow.enable_clock_power_recovery`는 CCD 전력 회수를 활성화하는 데 사용됩니다. 이의 기본 설정은 auto입니다. CCD 플로우에서는 이는 전력 회수에 초점을 맞출 것입니다. CCD 플로우가 아닌 경우에는 아무런 동작도 수행되지 않으므로 none으로 설정된 것과 동일합니다. 이를 재정의할 수 있습니다. 클래식 CTS 플로우에서는 명시적으로 power 또는 area로 설정할 수 있습니다. 또는 CCD 플로우에서 면적에 초점을 맞추고자 하는 경우 area로 명시적으로 설정할 수 있습니다. CCD 플로우에서는 타이밍 및 전력 개선을 위해 유용한 스쿠이를 사용합니다. 다시 말해, CCD는 항상 타이밍 개선을 시도하지만 이에 전력 및 면적 회수를 추가합니다. 클래식 CTS 플로우에서는 CCD의 타이밍 개선과 같은 동등한 기능이 없습니다. 따라서 클래식 CTS에서는 CCD 플로우가 타이밍을 손상시키지 않으면서 전력과 면적을 개선하는 데만 사용됩니다.  
{: .notice--warning}  

## 4.11 CCD Clock Power Recovery: Leakage, Dynamic or Total

- Clock power recovery will reduce leakage, dynamic or total power based solely on the scenario configuration and `clock_opt.flow.enable_power`
  - Does not depend on `opt.power.*`
  - For example, to reduce total power
    - `set_scenario_status func.ss_125c -dynamic_power true -leakage_power true`
  - Change the scenario power status according to your requirements
  - For best results, make sure you allow CTS to size registers and ICGs (see CTS Setup module)
  - If you enable only `leakage_power`, no clock power recovery occurs during `clock_opt` `final_opto`, but only during `route_opt` if enabled there (application option: `route_opt.flow.enable_clock_power_recovery=power`)

Clock power recovery will reduce leakage, dynamic or total power based only on your scenario configuration and `clock_opt.flow.enable_power` In the past we used opt.power application options and there was a mode where you could choose your power recovery mode. They are no longer necessary or supported. Therefore, please adjust your scenario power status according to what you require. For best results, please ensure you allow CTS to size your registers and your integrated clock gating cells. We discuss this in more details in the "Setting Up CTS" module. Lastly, if you enable only `leakage_power` then no clock power recovery occurs during the `final_opto` stage of `clock_opt` but only during `route_opt`.  
클럭 전력 회수는 시나리오 구성 및 `clock_opt.flow.enable_power`에 따라 누설, 동적 또는 총 전력을 줄입니다. 이전에는 opt.power 애플리케이션 옵션을 사용하여 전력 회수 모드를 선택할 수 있는 기능이 있었습니다. 그러나 이는 더 이상 필요하지 않거나 지원되지 않습니다. 따라서 필요에 따라 시나리오 전력 상태를 조정해주시기 바랍니다. 최상의 결과를 얻기 위해서는 CTS에게 레지스터와 통합 클럭 게이팅 셀의 사이즈 조정을 허용해야 합니다. 이에 대해 더 자세히 "CTS 설정" 모듈에서 다루고 있습니다. 마지막으로, `leakage_power`만 활성화하는 경우에는 `clock_opt`의 `final_opto` 단계에서는 클럭 전력 회수가 발생하지 않고, `route_opt` 중에만 발생합니다.  
{: .notice--warning}  

## 4.12 Example Script

```tcl
open_lib design.dlib
open_block my_block/compiled

# CTS setup (→ Setting Up CTS module)
source clock_tree_balance.tcl
source clock_preexisting_cells.tcl
source clock_routing_rules.tcl
source clock_constraints.tcl

# Apply CTS configuration steps, as needed
set_scenario_status -active true [all_scenarios]
set_scenario_status { s2 s4 } -hold true
set_app_options -name time.remove_clock_reconvergence_pessimism -value true
set_app_options -name route.global.effort_level -value high
reset_app_options time.si_enable_analysis

# CCD is enabled by default, to disable set clock_opt.flow.enable_ccd to false
clock_opt -to route_clock

# Additional setup required?
clock_opt -from final_opto
```

here we present an example script to tie everything we've discussed in this section together. Our assumption is that you have successfully completed physical synthesis. After opening your compiled block you apply all your CTS setup steps that we discuss in detail in our next module. Next you will activate all your scenarios and ensure you have specified your hold scenarios. Then setup clock reconvergence pessimism removal, your global route effort levels etc Remember that by default, CCD is enabled. If you only want the classic CTS, you must make that choice next before you run `clock_opt`.  
다음은 이 섹션에서 논의한 모든 내용을 하나로 묶은 예시 스크립트를 제시합니다. 가정은 물리적 합성이 성공적으로 완료되었다는 것입니다. 컴파일된 블록을 열고, 다음 모듈에서 자세히 다루는 CTS 설정 단계를 적용합니다. 그다음에는 모든 시나리오를 활성화하고 홀드 시나리오를 지정하는 것을 확인합니다. 그런 다음 클럭 재수렴 비관적 제거, 글로벌 라우트 노력 수준 등을 설정합니다. 기본적으로 CCD가 활성화되어 있음을 기억하세요. 클래식 CTS만 사용하려는 경우에는 `clock_opt`를 실행하기 전에 해당 선택을 해야 합니다.  
{: .notice--warning}  
