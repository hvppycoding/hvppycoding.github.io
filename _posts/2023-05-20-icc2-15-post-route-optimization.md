---
title: "IC Compiler II - 15 Post-route Optimization"
excerpt: "Block-level Implementation"
date: 2023-05-20 10:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
---

## Objectives

- Check whether the design is ready for post-route optimization
- Optimize the post-route design using `route_opt` or `hyper_route_opt`

## 1. IC Compiler II Flow​

![2023-05-20-icc2-flow-post-route-opt.png]({{site.baseurl}}/assets/images/2023-05-20-icc2-flow-post-route-opt.png){: .align-center}  

Let's take another look at the full flow diagram for IC Compiler II. After initially setting up the design, you ran the `place_opt` command to synthesize the netlist and perform early placement, buffering, and preroute optimization steps. Once that QoR is relatively clean, you proceeded to run clock tree synthesis to build the clock trees in the design, and then further optimize the design to account for the timing change due to computed clock delays. Then signal routing was performed. At this point in the flow, your design has been legally placed and optimized with clock trees fully built, and all clock and signal nets have been routed. Now the design is ready to begin post-route optimization with the `route_opt` or the `hyper_route_opt` command.  
다시 한 번 IC Compiler II의 전체 플로우 다이어그램을 살펴보겠습니다. 디자인을 초기 설정한 후 `place_opt` 명령을 실행하여 넷리스트를 합성하고 초기 배치, 버퍼링 및 프리루트 최적화 단계를 수행했습니다. QoR이 상당히 깨끗해진 후에는 클럭 트리 합성을 실행하여 디자인 내의 클럭 트리를 구축하고, 계산된 클럭 지연에 따른 타이밍 변경을 반영하여 디자인을 더욱 최적화했습니다. 그런 다음 신호 라우팅이 수행되었습니다. 이 시점에서 디자인은 법적으로 배치되어 최적화되었으며 클럭 트리가 완전히 구축되었고, 모든 클럭 및 신호 넷이 라우팅되었습니다. 이제 디자인은 `route_opt` 또는 `hyper_route_opt` 명령을 사용하여 후처리 최적화를 시작할 준비가 되었습니다.  
{: .notice--warning}  

## 2. Post-Route Optimization Phase Goal

- The goal of the post-route optimization phase is to achieve design convergence with best out-of-box QoR
  - Intelligent tradeoff of accuracy and runtime
    - Solves many design violations very quickly and the last few problems with high accuracy
  - Scalability
    - Optimizes a large number of design problems simultaneously
    - Solver-based optimization on QoR metrics of setup, hold, maximum transition, area, and power

The post route optimization has a goal of providing out-of-the-box QoR that converges on your design goals. It trades off accuracy versus runtime as it runs. In the initial stages of optimization, it solves the easy design violations quickly. Then towards the end it focuses on the more challenging violations with more accuracy. `route_opt` and `hyper_route_opt` both are scalable, using multithreading technology to efficiently tackle a large number of design problems at once. It has many algorithmic solvers at its disposal to address all QoR issues, including setup and hold timing, max transition and capacitance fixing, as well as reducing area and power.  
포스트 라우트 최적화는 사용자의 디자인 목표에 수렴하는 즉시 사용 가능한 품질 결과(QoR)를 제공하는 것을 목표로 합니다. 실행 중에 정확성과 실행 시간 사이의 균형을 조절합니다. 최적화의 초기 단계에서는 쉬운 디자인 위반을 빠르게 해결합니다. 그리고 마지막 단계에서는 더 도전적인 위반 사항에 대해 보다 정확하게 집중합니다. `route_opt`와 `hyper_route_opt`는 모두 확장 가능하며, 멀티스레딩 기술을 사용하여 한 번에 많은 수의 디자인 문제를 효율적으로 해결합니다. QoR 문제를 해결하기 위해 다양한 알고리즘 솔버를 사용할 수 있으며, 설정 및 보유 시간, 최대 전이 및 커패시턴스 수정, 면적 및 전력 감소를 포함한 모든 QoR 문제에 대응할 수 있습니다.  
{: .notice--warning}  

## 3. IC Compiler II Post-Route Optimization Flow

![2023-05-20-icc2-post-route-opt-flow.png]({{site.baseurl}}/assets/images/2023-05-20-icc2-post-route-opt-flow.png){: .align-center}  

The “post-route optimization phase” involves several key steps:  

- Pre-postroute optimization checks and setup
- Optimization
- ECO route

Here is an outline of the three major steps of the postroute optimization flow. We must begin with a fully legalized and routed design. Before proceeding with `route_opt`, you should ensure that the design is ready, by performing a number of checks and also setting up your design constraints for postroute optimization. Once the design state is validated and all necessary constraints and configurations are applied, `route_opt` or `hyper_route_opt` can be run. Post-route optimization does both optimization as well as ECO routing. You can optionally run additional ECO routing after optimization if many routing DRCs still remain. After the postroute optimization flow is complete, you are ready to proceed with the signoff flow where things like sign-off quality timing and extraction, functional and timing ECOs, filler cell insertion, and metal fill are done.  
포스트 라우트 최적화 플로우의 세 가지 주요 단계 개요입니다. 먼저 완전히 합법화되고 라우팅된 디자인으로 시작해야 합니다. `route_opt`를 진행하기 전에, 다양한 체크를 수행하고 포스트 라우트 최적화를 위한 디자인 제약 조건을 설정하여 디자인이 준비되었는지 확인해야 합니다. 디자인 상태가 확인되고 필요한 제약 조건과 설정이 적용된 경우, `route_opt` 또는 `hyper_route_opt를` 실행할 수 있습니다. 포스트 라우트 최적화는 최적화와 ECO(Engineering Change Order) 라우팅을 모두 수행합니다. 최적화 후에 여전히 많은 라우팅 DRC(Design Rule Check)가 남아 있는 경우 선택적으로 추가 ECO 라우팅을 실행할 수 있습니다. 포스트 라우트 최적화 플로우가 완료된 후, 시그너프 플로우로 진행하여 시그너프 품질 타이밍과 추출, 기능 및 타이밍 ECO, 필러 셀 삽입, 금속 필 적용 등을 수행할 준비가 됩니다.  
{: .notice--warning}  

## 4. IC Compiler II Post-Route Optimization Flow

- Clock and signal nets have been routed
- Afterwards, post-route optimization is performed:
- `route_opt`, or `hyper_route_opt`
  - Concurrently optimizes:
    - Timing and max-tran/max-cap (by default)
    - Clock tree, power (optionally)
    - Uses PrimeTime delay calculation and fusion extraction and, optionally StarRC Extraction
  - Updates routes affected by optimizations using ECO routing

As previously mentioned, it is critical that all clock and signal nets have been fully routed before running `route_opt` or `hyper_route_opt`. post route optimization relies on the fully routed nets for accurate RC delays to undertand the true postroute timing. During optimization, many optimization goals are concurrently optimized. Setup and hold timing optimization, max transition fixing, and max capacitance fixing are all done by default. You can optionally enable clock tree optimization including CCD, as well as leakage or total power optimization by enabling the appropriate settings. By default, the optimization engine uses PrimeTime for delay calculation, and can optionally use StarRC extraction data. This provides better correlation between the timing and RCs seen by `route_opt`, and the sign-off engines that will be used for final validation of your flow results. Post route optimization is routing aware, and automatically considers routing impact for the optimization moves it makes, and patches up broken routing topologies resulting from any logic that is inserted or removed, or cells that have been resized or moved. `route_opt` and `hyper_route_opt` is very different from preroute optimization, it is not just the same optimization techniques with routing patch-up afterwards. Postroute optimization is more cautious than preroute optimization, in order to avoid excessive disturbance to existing routing topologies and placement, which when changed can significantly impact the timing.  
이전에 언급했듯이, `route_opt` 또는 `hyper_route_opt`를 실행하기 전에 모든 클록 및 신호 넷이 완전히 라우팅되었는지 확인하는 것이 매우 중요합니다. 포스트 라우트 최적화는 정확한 RC 딜레이를 이해하기 위해 완전히 라우팅된 넷에 의존합니다. 최적화 중에는 여러 최적화 목표가 동시에 최적화됩니다. 설정 및 홀드 타이밍 최적화, 최대 전이 수정, 최대 커패시턴스 수정이 기본적으로 수행됩니다. 필요한 경우 적절한 설정을 활성화하여 CCD를 포함한 클록 트리 최적화, 누설 또는 총 전력 최적화를 선택적으로 활성화할 수 있습니다. 기본적으로 최적화 엔진은 PrimeTime을 사용하여 딜레이를 계산하며, 선택적으로 StarRC 추출 데이터를 사용할 수도 있습니다. 이는 `route_opt`에서 볼 타이밍 및 RC와 플로우 결과의 최종 검증에 사용될 시그너프 엔진 간에 더 나은 상관관계를 제공합니다. 포스트 라우트 최적화는 라우팅에 대한 인식이 있으며, 최적화 동작에 대한 라우팅 영향을 자동으로 고려하며, 삽입 또는 제거된 로직, 크기 조정 또는 이동된 셀로 인해 손상된 라우팅 토폴로지를 수정합니다. `route_opt`와 `hyper_route_opt`는 프리라우트 최적화와 매우 다릅니다. 라우팅 패치업을 한 후에 동일한 최적화 기술만 적용하는 것이 아닙니다. 포스트 라우트 최적화는 타이밍에 중대한 영향을 미칠 수 있는 기존 라우팅 토폴로지와 플레이스먼트에 과도한 변경을 피하기 위해 프리라우트 최적화보다 더 조심스럽게 진행됩니다.  
{: .notice--warning}  

## 5. Pre-postroute Checks and Setup

![2023-05-20-pre-postroute-checks-and-setup.png]({{site.baseurl}}/assets/images/2023-05-20-pre-postroute-checks-and-setup.png){: .align-center}  

Let's discuss in more detail the checks and setup required after completing routing but before running the `route_opt` flow. You will learn about a number of different topics, ranging from legality and DRC checks, to design constraint adjustments, to enabling certain features like power optimization and PrimeTime delay calculation.  
라우팅을 완료한 후 `route_opt` 플로우를 실행하기 전에 수행해야 하는 체크 및 설정에 대해 좀 더 자세히 논의해봅시다. 여러 가지 다른 주제에 대해 알아볼 것인데, 이는 합법성 및 DRC 체크부터 디자인 제약 조건 조정, 전력 최적화 및 PrimeTime 딜레이 계산과 같은 특정 기능 활성화에 이르기까지 다양합니다.  
{: .notice--warning}  

## 6. Check for Placement Legality

![2023-05-20-check-for-placement-legality.png]({{site.baseurl}}/assets/images/2023-05-20-check-for-placement-legality.png){: .align-center}  

The `check_legality` command reports placement legalization violations. Ideally, post route optimization should not run until you have a fully legalized design. Some violations are more serious than others. In this report, there are some user fixed cells with pg_drcs. This is not so critical since the legalizer engine isn't trying to move these cells. You could proceed with `route_opt` despite this type of violation. Most important is to avoid legality violations on movable cells. These are cells that the legalizer engine previously attempted to place in a legal location. If there are violations, then it indicates that the previous call to the legalizer was unsuccessful in legalizing the design, whether inside of clock_opt final_opto stage, or manually called with legalize_placement. This can indicate serious design issues, like overutilization of the entire design, or a particular voltage area or macro channel. Post route optimization is unlikely to converge on a solution if moveable cells could not be fully legalized beforehand. It will deliver worse results and runtime will increase as the tool struggles to find a legalization solution.  
`check_legality` 명령은 플레이스먼트 합법화 위반을 보고합니다. 이상적으로는 완전히 합법화된 디자인이 있을 때까지 포스트 라우트 최적화를 실행하지 않아야 합니다. 일부 위반 사항은 다른 것보다 심각할 수 있습니다. 이 보고서에서는 pg_drcs와 함께 일부 사용자 고정 셀이 있습니다. 이는 합법화 엔진이 이러한 셀을 이동시키려고 하지 않기 때문에 크게 중요하지 않습니다. 이러한 유형의 위반에도 불구하고 `route_opt`를 실행할 수 있습니다. 가장 중요한 것은 이동 가능한 셀에서의 합법성 위반을 피하는 것입니다. 이러한 셀은 이전에 합법적인 위치로 배치하려고 시도한 셀입니다. 위반 사항이 있는 경우 이는 이전의 합법화 호출이 디자인을 합법화하는 데 실패했음을 나타냅니다. 이는 전체 디자인이나 특정 전압 영역 또는 매크로 채널과 같은 심각한 디자인 문제를 나타낼 수 있습니다. 이동 가능한 셀이 사전에 완전히 합법화되지 않은 경우 포스트 라우트 최적화는 해결책에 수렴하기 어렵습니다. 결과가 악화되며 도구는 합법화 솔루션을 찾기 위해 노력함에 따라 실행 시간이 증가할 것입니다.  
{: .notice--warning}  

## 7. Check for Clean Routing

- Apply check routes to confirm that the design is ready for `route_opt` and `hyper_route_opt`
  - Clock and signal nets are routed
  - No open nets
  - Reasonable physical DRC violations

`check_routes` should also be run to check the routing DRCs that remain after the detail routing that was done just prior to `route_opt` or `hyper_route_opt`. It's important that all clock and signal nets have been routed. There should be no open DRC violations. Beyond that, the overall DRC picture should be reasonable. This is a judgement call on your part, based on your own study and analysis of not just the `check_routes` report, but also the GUI layout view with the DRC error browser open to view the locations and causes of the remaining DRCs. You may have some DRCs related to power grids or fixed cell placement that you expect and will be cleaning up later. You can still proceed with `route_opt` despite these DRCs, though it can slow down the runtime of the incremental routing. But, be very concerned about DRCs related to excessive cell or pin density in parts of the floorplan, and routing congestion. These are design and optimization issues, and if they are showing up here even before `route_opt`, they will remain. `route_opt` is not intended to resolve overutilization and congestion problems. These should be addressed earlier in the flow before proceeding with postroute optimization.  
`route_opt` 또는 `hyper_route_opt` 이전에 수행된 디테일 라우팅 이후에 남은 라우팅 DRC를 확인하기 위해 `check_routes`를 실행해야 합니다. 모든 클록과 신호 넷이 라우팅되었는지 확인하는 것이 중요합니다. 개방된 DRC 위반 사항이 없어야 합니다. 이외에도 전반적인 DRC 상황이 합리적이어야 합니다. 이는 `check_routes` 보고서뿐만 아니라 DRC 오류 브라우저를 통해 위치 및 원인을 확인할 수 있는 GUI 레이아웃 뷰의 공부 및 분석에 기초한 판단에 의존합니다. 전력 그리드나 고정 셀 배치와 관련된 몇 가지 DRC가 있을 수 있으며, 이를 나중에 정리할 예정입니다. 이러한 DRC가 있더라도 `route_opt`를 진행할 수 있지만, 이는 증분 라우팅의 실행 시간을 늦출 수 있습니다. 그러나 층간의 과도한 셀 또는 핀 밀도, 라우팅 혼잡과 관련된 DRC에 대해서는 매우 주의해야 합니다. 이는 디자인 및 최적화 문제이며, `route_opt` 이전에도 이러한 문제가 나타나는 경우 그대로 남게 될 것입니다. `route_opt`는 과다 사용 및 혼잡 문제를 해결하기 위한 것이 아닙니다. 이러한 문제는 포스트 라우트 최적화를 진행하기 전에 플로우의 이전 단계에서 해결되어야 합니다.  
{: .notice--warning}  

## 8. Adjust Design Constraints and Scenarios for Post-Route

- Apply `compute_clock_latency` to update your block's timing constraints
  - Calculates and updates the I/O latencies of real and virtual clock objects for all active mode and corner combinations
  - This should have occurred before `clock_opt -from final_opto`
- Apply post route design constraints like clock uncertainty, max_transition, etc. to match with signoff criteria
- Apply all scenarios for post-route optimization
- Align with scenario setup for signoff check
- Enable leakage and dynamic scenario for power optimization as needed

After verifying that the placement legality and routing DRCs are good enough to proceed with postroute optimization, it is time to set up the design constraints. It is recommended to run the `compute_clock_latency` command to update the block's I/O latencies based on the actual latencies of the clocks, to ensure the most accurate I/O timing. Even though this was done earlier after clock tree synthesis, additional preroute optimization and routing of signal nets may have impacted clock latencies enough to justify running this command again. Also adjust other design constraints to match with the signoff criteria. This might include reducing clock uncertainties and tweaking `max_transition` constraints. Generally you should reduce earlier pessimism in design constraints, since now the optimization is working with routed clock nets, PrimeTime accurate delay calculation, and optionally StarRC extraction for even better correlation with sign-off timing numbers. Make any scenario adjustments needed for post-route optimization, including alignment of scenario settings with those used in sign-off, and enabling any additional leakage and dynamic scenarios needed for power optimization.  
플레이스먼트 합법성과 라우팅 DRC가 충분히 좋아서 포스트 라우트 최적화를 진행할 준비가 되었다면, 이제 디자인 제약 조건을 설정해야 합니다. 실제 클록의 지연 시간을 기반으로 블록의 I/O 지연 시간을 업데이트하기 위해 `compute_clock_latency` 명령을 실행하는 것이 권장됩니다. 이는 가장 정확한 I/O 타이밍을 보장하기 위해 수행됩니다. 클록 트리 합성 이후에 이미 수행되었지만, 추가적인 프리라우트 최적화 및 신호 넷의 라우팅으로 인해 클록 지연이 충분히 변경되어 이 명령을 다시 실행할 가치가 있습니다. 또한 서명 기준과 일치하도록 다른 디자인 제약 조건을 조정해야 합니다. 이는 클록 불확실성을 줄이고 `max_transition` 제약 조건을 조정하는 것을 포함할 수 있습니다. 일반적으로 라우트된 클록 넷, PrimeTime 정확한 딜레이 계산 및 선택적으로 StarRC 추출과 더욱 높은 상관성을 위해 이전에 비관적으로 설정된 디자인 제약 조건을 줄이는 것이 좋습니다. 포스트 라우트 최적화에 필요한 시나리오 조정을 수행하고, 시그너프에서 사용된 설정과 시나리오 설정을 일치시키며, 전력 최적화에 필요한 추가적인 leakage 및 dynamic 시나리오를 활성화하는 등 필요한 조정을 수행하세요.  
{: .notice--warning}  

## 9. Enable Clock Tree Optimizations in Post-Route Optimization

![2023-05-20-enable-clock-tree-opt-in-post-route-opt.png]({{site.baseurl}}/assets/images/2023-05-20-enable-clock-tree-opt-in-post-route-opt.png){: .align-center}  

Several `route_opt` application options control clock tree optimizations. `route_opt.flow.enable_ccd` is used to enable concurrent clock and data optimization in `route_opt`. It is off by default, but it is highly recommended to enable it for the first iteration of `route_opt`. If CCD is enabled, then by default clock power recovery will also be enabled. But you can even enable clock power recovery without CCD by using the `route_opt.flow.enable_clock_power_recovery` app option. Also, clock logical DRC fixing is enabled by default with CCD, but the `route_opt.flow.enable_ccd_clock_drc_fixing` app option is also available to independently control clock DRC fixing if you choose to disable CCD. So in summary, if you choose to enable postroute CCD, everything else here is set up automatically.  
`route_opt` 애플리케이션 옵션 중 몇 가지는 클록 트리 최적화를 제어합니다. `route_opt.flow.enable_ccd`는 `route_opt`에서 동시 클록 및 데이터 최적화를 활성화하는 데 사용됩니다. 기본적으로 비활성화되어 있지만, `route_opt`의 첫 번째 반복에서는 활성화하는 것이 매우 권장됩니다. CCD가 활성화되면 기본적으로 클록 전력 복구도 활성화됩니다. 그러나 `route_opt.flow.enable_clock_power_recovery` 앱 옵션을 사용하여 CCD 없이도 클록 전력 복구를 활성화할 수도 있습니다. 또한, CCD와 함께 클록 논리 DRC 수정도 기본적으로 활성화되지만, 필요에 따라 CCD를 비활성화하는 경우 `route_opt.flow.enable_ccd_clock_drc_fixing` 앱 옵션을 사용하여 독립적으로 클록 DRC 수정을 제어할 수도 있습니다. 요약하자면, 포스트 라우트 CCD를 활성화하려면 나머지 설정은 자동으로 설정됩니다.  
{: .notice--warning}  

## 10. Enable Power Optimization in Post-Route Optimization

- To enable power optimization:

```tcl
set_app_options -name route_opt.flow.enable_power -value true
# Default: false
```

- Power optimization, once enabled, is based on the scenario configuration
  - Active scenario with leakage+dynamic: total power optimization
  - Active scenario with leakage only: leakage power optimization
  - If neither leakage or dynamic power-activated scenarios exist, no power optimization occurs

Power optimization in `route_opt` is enabled with the `route_opt.flow.enable_power` app option. By default it is also set to false. Enabling power optimization with this app option is a necessary step, but not the only required setup for `route_opt` power optimization. The specific behavior of the power optimization is controlled by your scenario setup from the `set_scenario_status` command. Any scenarios enabled for leakage and dynamic power will have `route_opt` total_power optimization performed. If a scenario is enabled for only leakage power, then route_opt will only perform leakage optimization for it. If neither leakage nor dynamic power is enabled for the scenario, then `route_opt` will not optimize power for it at all.  
`route_opt`에서 전력 최적화를 활성화하려면 `route_opt.flow.enable_power` 앱 옵션을 사용합니다. 기본적으로 false로 설정되어 있습니다. 이 앱 옵션을 사용하여 전력 최적화를 활성화하는 것은 필요한 단계이지만, `route_opt` 전력 최적화를 위한 유일한 요구 사항은 아닙니다. 전력 최적화의 구체적인 동작은 `set_scenario_status` 명령을 통해 설정한 시나리오 설정에 따라 제어됩니다. leakage 및 dynamic 전력에 대해 활성화된 시나리오는 `route_opt`에서 total_power 최적화를 수행합니다. leakage 전력에 대해서만 활성화된 시나리오의 경우 해당 시나리오에 대해 leakage 최적화만 수행합니다. 시나리오에 leakage 및 dynamic 전력이 모두 비활성화된 경우 `route_opt`는 해당 시나리오에 대해 전력 최적화를 수행하지 않습니다.  
{: .notice--warning}  

## 11. Use PrimeTime Delay Calculation

![2023-05-20-use-pt-delay-calculation.png]({{site.baseurl}}/assets/images/2023-05-20-use-pt-delay-calculation.png){: .align-center}  

`route_opt` and `hyper_route_opt` uses integrated PrimeTime delay calculation by default. This improves the correlation with sign-off timing done in PrimeTime during the signoff flow, which will reduce ECO iterations and give overall faster timing closure. In order to use the integrated PrimeTime delay calculation, you must use CLIBs, which are generated by the Library Manager tool from verion O-2018.06 or later. To further ensure correlation with PrimeTime sign-off timing, ensure that these key timer app options are aligned with the PrimeTime settings. You should enable CCS receiver caps, and also use advanced waveform propagation by settings the app option to full_design. Enabling crosstalk analysis is absolutely essential for accurate timing. And enabling SI timing windows will ensure there isn't too much pessimism in identified victim and aggressor nets for crosstalk analysis.  
`route_opt`와 `hyper_route_opt`는 기본적으로 통합된 PrimeTime 딜레이 계산을 사용합니다. 이는 Sign-off 플로우에서 PrimeTime을 통해 수행되는 Sign-off 타이밍과의 상관성을 향상시키며, ECO 반복을 줄이고 전체적인 타이밍 클로저를 더 빠르게 달성할 수 있습니다. 통합된 PrimeTime 딜레이 계산을 사용하려면 버전 O-2018.06 이상의 Library Manager 도구에서 생성된 CLIB를 사용해야 합니다. PrimeTime Sign-off 타이밍과의 상관성을 보다 확실하게 하기 위해 이러한 핵심 타이머 앱 옵션을 PrimeTime 설정과 일치시켜야 합니다. CCS 수신기 용량을 활성화하고, 전체 디자인으로 앱 옵션을 설정하여 고급 웨이브폼 전파를 사용해야 합니다. 크로스토크 분석을 활성화하는 것은 정확한 타이밍을 위해 절대적으로 필요합니다. 또한 SI 타이밍 윈도우를 활성화하여 크로스토크 분석을 위한 식별된 피해자 및 공격자 넷에 너무 많은 비관적인 설정이 없도록 해야 합니다.  
{: .notice--warning}  

## 12. Enable Path Based Optimization

- Use Path-Based post-route optimization technology to reduce pessimism, improve area and power, and reduce the number of timing ECO loops
  - PBA driven CCD optimization is available since R-2020.09 release
  - `set_app_options -name time.pba_optimization_mode -value path`
- Dependencies
  - Design is detail-routed
  - PT delay calculation is enabled

With PrimeTime delay calculation in `route_opt`, you can also enable path-based optimization, which is less pessimistic than the graph-based analysis earlier in the preroute optimization flow. Here, the application option is configured for path, meaning that path based recalculation is done path by path as needed. You can also set this app option to exhaustive for exhaustive path-based analysis. This is even less pessimistic, but increases runtime. Since the R-2020.09 release, CCD in `route_opt` also supports path-based optimization.  
`route_opt`에서 PrimeTime 딜레이 계산을 사용하면, 이전의 프리라우트 최적화 플로우에서 사용되었던 그래프 기반 분석보다 비관적이지 않은 경로 기반 최적화를 활성화할 수도 있습니다. 여기서 앱 옵션은 path로 구성되며, 필요에 따라 경로별로 경로 기반 재계산이 수행됩니다. 더욱 비관적이지 않은 완전한 경로 기반 분석을 위해 이 앱 옵션을 exhaustive로 설정할 수도 있습니다. 이는 더욱 비관적이지 않지만 실행 시간이 늘어납니다. R-2020.09 릴리스부터는 `route_opt`의 CCD도 경로 기반 최적화를 지원합니다.  
{: .notice--warning}  

## 13. Use Fusion Extraction / In-design StarRC

![2023-05-20-use-fusion-extraction.png]({{site.baseurl}}/assets/images/2023-05-20-use-fusion-extraction.png){: .align-center}  

Since the Q-2019.12-SP4 release, `route_opt` uses Fusion extraction by default, which provides deep API-level integration of the StarRC core within the tool's extraction engine. This provides automatic sign-off quality RC extraction for `route_opt`. Used in combination with the integrated PrimeTime delay calculation, `route_opt` has extremely good correlation with final sign-off timing. The `extract.starrc_mode` app option controls this behavior. fusion_adv is the default for fusion extraction. `in_design` uses the older in-design StarRC extraction technology, and none uses the TLUPlus for extraction, but use the nxtgrd files instead. Therefore, fusion extraction requires providing the StarRC nxtgrd files through a config file. This is provided using the `set_starrc_in_design` command.  
Q-2019.12-SP4 이후, `route_opt`은 기본적으로 Fusion 추출을 사용합니다. 이는 StarRC 코어와 도구의 추출 엔진 간의 깊은 API 수준의 통합을 제공합니다. 이를 통해 `route_opt`에 자동으로 시그너프 품질의 RC 추출이 가능해집니다. 통합된 PrimeTime 딜레이 계산과 함께 사용될 때, `route_opt`은 최종 시그너프 타이밍과 매우 우수한 상관성을 갖게 됩니다. `extract.starrc_mode` 앱 옵션으로 이 동작을 제어할 수 있습니다. fusion_adv가 Fusion 추출의 기본값입니다. `in_design`은 이전의 in-design StarRC 추출 기술을 사용하며, none은 TLUPlus를 사용하지만 nxtgrd 파일을 사용합니다. 따라서, Fusion 추출을 사용하려면 StarRC nxtgrd 파일을 구성 파일을 통해 제공해야 합니다. 이는 `set_starrc_in_design` 명령을 사용하여 제공됩니다.
{: .notice--warning}  

## 14. Recommended Post-Route Optimization Solution

![2023-05-20-recommended-post-route-opt.png]({{site.baseurl}}/assets/images/2023-05-20-recommended-post-route-opt.png){: .align-center}  

For post route optimization, the `hyper_route_opt` is the new and recommended solution for better OOTB QoR convergence. It performs 4 integrated iterations of optimization, eco and timing update in a single command. This integrated post-route optimization command delivers single pass convergence with better timing and total power, it also include the new MOO-CCD and power-aware optimization engines, has much close integration between optimization, CCD, and ECO routing, and includes an integrated true footprint size only optimization to further improve convergence. To run the `hyper_route_opt`, you must all of the setup previously discussed before the `hyper_route_opt`. This includes your constraint and scenario setup, PrimeTime delay calculation and StarRC extraction settings. Enable CCD, power optimization, and path-based optimization. Now call the `hyper_route_opt`. after that, if there is still violations left, you can use any round of targeted endpoint (TEP) `route_opt` to further improve QoR.  
포스트 라우트 최적화를 위해서는 `hyper_route_opt`이 새롭고 권장되는 솔루션입니다. 이 명령은 최적화, ECO 및 타이밍 업데이트를 통합된 4회 반복으로 수행합니다. 이 통합된 포스트 라우트 최적화 명령은 개선된 타이밍 및 총 전력을 가진 단일 패스 수렴을 제공하며, MOO-CCD 및 전력 주의 최적화 엔진을 포함하고 있으며, 최적화, CCD 및 ECO 라우팅 간의 통합이 매우 밀접하며, 수렴을 더욱 개선하기 위해 통합된 실제 풋프린트 크기 최적화도 포함하고 있습니다. `hyper_route_opt`를 실행하려면 이전에 설명한 모든 설정을 완료해야 합니다. 이에는 제약 조건 및 시나리오 설정, PrimeTime 딜레이 계산 및 StarRC 추출 설정이 포함됩니다. CCD, 전력 최적화 및 경로 기반 최적화를 활성화해야 합니다. 그런 다음 `hyper_route_opt`를 호출하세요. 그 이후에도 여전히 위반 사항이 남아 있다면, 목표 지점(TEP)을 대상으로 한 추가적인 `route_opt` 라운드를 사용하여 QoR을 더욱 개선할 수 있습니다.  
{: .notice--warning}  

## 15. Alternative Post-Route Optimization Solution with `route_opt`

![2023-05-20-alternative-post-route-opt.png]({{site.baseurl}}/assets/images/2023-05-20-alternative-post-route-opt.png){: .align-center}  

Alternatively, you can use the `route_opt` to perform the post-route optimization, with 3-pass `route_opt`. The idea of the three pass `route_opt` flow is to incrementally move towards more focused and limited optimizations in each pass. The first pass has the most opportunity to improve gross design violations, but also has the most potential to disturb the design. Each subsequent pass is more tightly controlled, to fine-tune the remaining violations without introducing new violations from making large optimization moves. Enable CCD, power optimization, and path-based optimization before calling the first `route_opt`. then for the second `route_opt`, CCD will be turned off. This will limit the extent of the optimization changes introduced in this iteration of `route_opt`. For the third `route_opt`, the `route_opt.flow.size_only_mode` app option is used to disable even more route_opt optimization techniques. Now only cell sizing will be used to clean up the remaining violations and recover more power or area. The `equal_or_smaller` setting of this app option allows only for swapping to lib_cells with a footprint that is the same size or smaller than that used for the current cell instance. By doing so, placement legalization changes can be avoided, which also avoids many changes to existing routing topologies.  
대안으로, 3회 반복하는 `route_opt`를 사용하여 포스트 라우트 최적화를 수행할 수 있습니다. 3회 반복하는 `route_opt` 플로우의 아이디어는 각각의 반복에서 더 집중적이고 제한된 최적화로 점진적으로 진행하는 것입니다. 첫 번째 반복은 총체적인 디자인 위반 사항을 개선할 가장 큰 기회를 제공하지만, 동시에 디자인에 큰 변화를 일으킬 가능성도 가장 큽니다. 이후의 각 반복은 더욱 엄격하게 제어되어, 큰 최적화 이동으로 인한 새로운 위반 사항을 도입하지 않으면서 남은 위반 사항을 세밀하게 조정합니다. 첫 번째 `route_opt`를 호출하기 전에 CCD, 전력 최적화 및 경로 기반 최적화를 활성화해야 합니다. 두 번째 `route_opt`에서는 CCD가 비활성화됩니다. 이는 `route_opt`의 이번 반복에서 도입되는 최적화 변경의 범위를 제한합니다. 세 번째 `route_opt`에서는 `route_opt.flow.size_only_mode` 앱 옵션을 사용하여 더 많은 `route_opt` 최적화 기법을 비활성화합니다. 이제 남은 위반 사항을 정리하고 더 많은 전력 또는 면적을 회복하기 위해 셀 크기 조정만 사용됩니다. 이 앱 옵션의 `equal_or_smaller` 설정은 현재 셀 인스턴스에 사용된 것과 동일한 크기 또는 작은 풋프린트를 가진 lib_cells로만 스왑될 수 있도록 허용합니다. 이렇게 함으로써, 플레이스먼트 합법화 변경을 피할 수 있으며, 기존의 라우팅 토폴로지에 대한 많은 변경도 피할 수 있습니다.  
{: .notice--warning}  

## 16. Targeted Endpoint Post-route Optimization

![2023-05-20-targeted-endpoint-post-route-opt.png]({{site.baseurl}}/assets/images/2023-05-20-targeted-endpoint-post-route-opt.png){: .align-center}  

After `route_opt` or `hyper_route_opt` closure flow applied, you can use targeted endpoint optimization to optimize a list of endpoints to close the remaining violation, to push for WNS and TNS further. The list of target endpoints can be derived from either a native timing report, or a PrimeTime timing report. Use the `set_route_opt_target_endpoints` command to specify that endpoint file. You can also specify the target endpoints through other command options pointing to a collection of endpoint objects, then run `route_opt` to do the optimization.  
`route_opt` 또는 `hyper_route_opt` 클로저 플로우를 적용한 후에는 남은 위반을 해결하고 WNS 및 TNS를 더욱 향상시키기 위해 대상 엔드포인트 최적화를 사용할 수 있습니다. 대상 엔드포인트 목록은 네이티브 타이밍 보고서 또는 PrimeTime 타이밍 보고서에서 유도할 수 있습니다. `set_route_opt_target_endpoints` 명령을 사용하여 해당 엔드포인트 파일을 지정하세요. 또는 엔드포인트 객체의 컬렉션을 가리키는 다른 명령 옵션을 통해 대상 엔드포인트를 지정한 다음 `route_opt`를 실행하여 최적화를 수행할 수도 있습니다.  
{: .notice--warning}  

## 17. Optimization and ECO Route

![2023-05-20-opt-and-eco-route.png]({{site.baseurl}}/assets/images/2023-05-20-opt-and-eco-route.png){: .align-center}  

ECO routing is always necessary after optimizations have been performed. You can control the ECO routing performed during `route_opt`, or also call additional routing iterations after `route_opt` to clean up more routing DRC violations.  
ECO 라우팅은 최적화가 수행된 후에 항상 필요합니다. `route_opt` 중에 수행되는 ECO 라우팅을 제어할 수 있으며, `route_opt` 이후에 추가적인 라우팅 반복을 호출하여 추가적인 라우팅 DRC 위반을 정리할 수도 있습니다.  
{: .notice--warning}  

## 18. Physical DRC Convergence in Post-Route Optimization

![2023-05-20-physical-drc-convergence-in-post-route-opt.png]({{site.baseurl}}/assets/images/2023-05-20-physical-drc-convergence-in-post-route-opt.png){: .align-center}  

`route_opt` and `hyper_route_opt` does perform routing patch-up itself. But by default, it only does five or 10 iterations of route_eco, which may not be enough to reduce DRC violations fully in some cases. You can increase the number of iterations run inside of optimization by using the application option shown. You can also call additional routing iterations by using the `route_detail` command directly. This may be necessary for final DRC cleanup after all optimization iterations are complete.  
`route_opt`와 `hyper_route_opt`는 라우팅 패치업을 수행합니다. 그러나 기본적으로 route_eco의 반복 횟수는 다섯 번이나 열 번으로 설정되어 있어서 모든 경우에 완전히 DRC 위반을 줄이는 데 충분하지 않을 수 있습니다. 앱 옵션을 사용하여 최적화 내에서 실행되는 반복 횟수를 증가시킬 수 있습니다. 또한 `route_detail` 명령을 직접 사용하여 추가적인 라우팅 반복을 호출할 수도 있습니다. 이는 모든 최적화 반복이 완료된 후 최종 DRC 정리에 필요할 수 있습니다.  
{: .notice--warning}  
