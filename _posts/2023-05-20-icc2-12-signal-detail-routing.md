---
title: "IC Compiler II - 12 Signal Detail Routing"
excerpt: "Block-level Implementation"
date: 2023-05-20 07:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - IC Compiler II
---

## Objectives

- Describe the key routing steps in IC Compiler II
- List some key routing setup steps
- Route the signal nets using `route_auto`

## 1. IC Compiler II Unified Flow

![2023-05-20-icc2-flow.png]({{site.baseurl}}/assets/images/2023-05-20-icc2-flow.png){: .align-center}  

Just as a quick reminder, let's review the IC Compiler II flow. After performing the design and timing setup, you run the `place_opt` command which performs physical synthesis, placement and optimization. Next you run clock tree synthesis and optimization using the `clock_opt` command. That's where the clock trees are built optimized, the clocks routed and the signal nets are optimized and global routed. This unit is all about the signal detail routing stage. Using the `route_auto` command. Post-Route optimization will be discussed later. That's when we talk about the `route_opt` command. After the design has been routed and optimized, you continue to the Signoff phase.  
간단히 상기시키기 위해 IC Compiler II 플로우를 다시 리뷰해보겠습니다. 디자인 및 타이밍 설정을 수행한 후, `place_opt` 명령을 실행하여 물리적 합성, 배치 및 최적화를 수행합니다. 그 다음 `clock_opt` 명령을 사용하여 클럭 트리 합성과 최적화를 수행합니다. 이 단계에서 클럭 트리를 구축하고 최적화하며, 클럭이 라우트되고 신호 넷이 최적화되고 글로벌 라우트됩니다. 이 유닛은 신호의 상세 라우팅 단계에 대한 내용입니다. `route_auto` 명령을 사용합니다. 포스트라우트 최적화는 나중에 다룰 예정입니다. 이때 `route_opt` 명령에 대해 이야기합니다. 디자인이 라우트되고 최적화된 후에는 Signoff 단계로 이어집니다.
{: .notice--warning}  

## 2. IC Compiler II Routing Flow

![2023-05-20-icc2-routing-flow.png]({{site.baseurl}}/assets/images/2023-05-20-icc2-routing-flow.png){: .align-center}  

The “routing phase” involves several key steps:  

- Pre-routing checks and setup
- Signal routing
- Optimization and rerouting (routing update)

There are three main steps to the routing flow: performing pre routing checks to ensure that the design is ready to route and does not have issues that can cause routability problems; performing routing setup to control what happens during routing; and performing the signal routing. The routing flow is iterative; if you don't get the desired results, you might have to change your setup and rerun routing.  
라우팅 플로우에는 세 가지 주요 단계가 있습니다: 디자인이 라우팅 가능한 문제를 일으킬 수 있는 문제를 가지고 있지 않고 라우트 준비가 되어 있는지 확인하기 위해 라우팅 전 체크를 수행하는 것, 라우팅 중에 어떤 일이 발생하는지 제어하기 위해 라우팅 설정을 수행하는 것, 그리고 신호 라우팅을 수행하는 것입니다. 라우팅 플로우는 반복적인 과정입니다. 원하는 결과를 얻지 못한 경우 설정을 변경하고 다시 라우팅을 실행해야 할 수 있습니다.  
{: .notice--warning}  

## 3. Recommended Routing Flow

- Clock nets should be detail routed (during clock opt route_clock stage)
- Signal nets are global routed during the clock opt final_opto stage
- Signal net routing must next be completed with:
  - `route_auto`
  - Performs: **Track Assignment → Detail Routing**

![2023-05-20-recommended-routing-flow.png]({{site.baseurl}}/assets/images/2023-05-20-recommended-routing-flow.png){: .align-center}  

So how does the actual routing flow look like? Routing is really distributed across multiple commands. The first routing happens during `clock_opt` `build_clock`. That's when the clocks are actually global routed, and that's only the Clock Nets. During the `route_clock` stage that's when the clocks are detail routed. During the `clock_opt` `final_opto` stage, that's when we perform signal net routing, but that's only global routing no detailed route occurs there yet. But the nets are all global routed and optimized using the global route. So during the `route_auto` stage, we only perform track assignment and detail routing. These are two steps that we'll discuss here in this unit. Any setup for routing has to be done before you run any of these routing operations. But I'll talk more about the routing steps in a few minutes.  
라우팅 플로우는 실제로 여러 명령에 걸쳐 분산되어 진행됩니다. 첫 번째 라우팅은 `clock_opt` `build_clock` 단계에서 발생합니다. 그때 클럭이 실제로 글로벌 라우팅되며, 이는 클럭 넷에 대해서만 수행됩니다. `route_clock` 단계에서는 클럭이 상세하게 라우팅됩니다. `clock_opt` `final_opto` 단계에서는 신호 넷 라우팅이 수행되지만, 여기에서는 아직 상세한 라우팅은 이루어지지 않습니다. 그러나 모든 넷은 글로벌 라우트 및 최적화를 사용하여 라우트됩니다. 따라서 `route_auto` 단계에서는 트랙 할당과 상세 라우팅만 수행됩니다. 이 두 가지 단계에 대해 이번 유닛에서 자세히 다룰 것입니다. 라우팅을 위한 모든 설정은 이러한 라우팅 작업을 실행하기 전에 완료되어야 합니다. 그러나 라우팅 단계에 대해서는 곧 더 자세히 이야기하겠습니다.  
{: .notice--warning}  

## 4. Route Operations: Track Assignment

- Track Assignment (TA):
  - Assigns each net to a specific track within each global routing cell and creates the actual metal shapes
- Attempts to:
  - Make long, straight traces
  - Reduce the number of vias
- After Track Assign, there should be no global routes or GR vias left

![2023-05-20-track-assignment.png]({{site.baseurl}}/assets/images/2023-05-20-track-assignment.png){: .align-center}  

After TA finishes, all nets are routed, but not very carefully. There are many violations, particularly where the routing connects to pins. Detail routing works to correct those violations.
{: .notice--info}

----------

If track assignment can reduce the number of jogs and jumps in metal traces, this will generally improve timing (since each jump generally requires a via to jump to a higher or lower level metal layer). Reducing the number of vias is generally a plus for reliability and yield since their failure rate is slightly higher than that of a simple, straight metal track in a moder, planarized process. After track assignment, there should not be any left-over global routes. If this is the case, make sure that these nets don’t have the attribute `physical_status=locked`, which would prevent track assignment.

As I've already mentioned, global routing has already occurred for the signal nets. During track assign, the global routes are turned into actual metal traces. For each layer, the global routes within a global routing cell or GRC are assigned to specific tracks. The tracks were created during floor planning based on the rules from the tech file. If the net cannot be assigned a track maybe because the global routing cell was overcapacity, the track will be assigned to a neighboring GRC. Track assign is going to try to make the traces as long and straight as possible minimizing jogs and trying to minimize vias. By default it is allowed to do small jogs in the non preferred direction. If that can help reduce the amount of vias but only on layers that allow jogs. After track assignment there should be no global routes left. if any are left, you should check and find out what went wrong. If you do DRC checking after track assignment, you would likely see many violations particularly near pin connections. These violations will be corrected during detailed route. As shown on the right side after track assignment you can see the created metal shapes in the GUI layout window.  
이미 언급한 대로, 신호 넷에 대한 글로벌 라우팅이 이미 진행되었습니다. 트랙 할당 중에는 글로벌 라우트가 실제 금속 트레이스로 변환됩니다. 각 레이어에서 글로벌 라우팅 셀 또는 GRC 내의 글로벌 라우트가 특정 트랙에 할당됩니다. 트랙은 테크 파일의 규칙을 기반으로 플로어 플래닝 중에 생성되었습니다. 만약 넷이 트랙에 할당될 수 없는 경우(예: 글로벌 라우팅 셀이 용량을 초과한 경우), 트랙은 인접한 GRC에 할당됩니다. 트랙 할당은 트레이스를 가능한 한 길고 직선적으로 만들고, 조그를 최소화하고 via를 최소화하려고 노력합니다. 기본적으로 비선호 방향에서 작은 조그를 허용합니다. 이는 via의 양을 줄일 수 있지만 조그가 허용되는 레이어에서만 적용됩니다. 트랙 할당 후에는 더 이상 글로벌 라우트가 남아서는 안 됩니다. 남아 있는 경우, 어떤 문제가 발생했는지 확인하고 해결해야 합니다. 트랙 할당 이후에 DRC 체크를 수행하면 핀 연결 부근을 포함하여 많은 위반 사항이 나타날 수 있습니다. 이러한 위반 사항은 상세 라우트 중에 수정됩니다. 오른쪽에 표시된 것처럼 트랙 할당 후에는 GUI 레이아웃 창에서 생성된 금속 모양을 확인할 수 있습니다.  
{: .notice--warning}  

## 5. Route Operations: Detail Routing

- Detail route fixes physical design rule violations
- Performs multiple iterations with varying repair box sizes

![2023-05-20-route-operations-detail-routing.png]({{site.baseurl}}/assets/images/2023-05-20-route-operations-detail-routing.png){: .align-center}  

Detail routing fixes the physical design rule violations left after track assignment. Here are some examples of the types of fixes made by detail routing. At the top left of the figure, the red oval indicates a notch spacing violation caused by a small jog that connects to a via. To correct this violation, the detail router can jog further away from the via instead of next to it, which creates a much bigger notch that does not violate the notch spacing rule. At the top right of the figure, the red oval indicates a violation caused when two vias on the same net are placed close to each other. To fix this violation, the detail router can either move the vias or create a jog in the nonpreferred direction to keep everything on the same layer, as shown in this figure. At the bottom left of the figure, the red oval indicates a spacing violation caused when a fat metal shape is placed too close to a thin metal shape. Typically, the minimum spacing from a fat metal shape to a thin metal shaped is larger than the spacing between two thin metal shapes. To correct this violation, the detail router can increase the spacing between the fat and thin metal shapes. At the bottom right of the figure, the red oval indicates a spacing violation caused when two independent vias are placed next to each other. To fix this violation, the detail router can move the vias so that they are offset from each other. The detail router performs multiple iterations to fix the physical DRC violations. You can control the number of iterations that it performs.  
상세 라우팅은 트랙 할당 후에 남은 물리적 설계 규칙 위반을 수정합니다. 여기에는 상세 라우팅에서 수행되는 몇 가지 예시가 있습니다. 그림의 왼쪽 상단에는 빨간색 타원형이 표시되어 있으며, 이는 via에 연결되는 작은 조그로 인한 노치 간격 위반을 나타냅니다. 이 위반을 수정하기 위해 상세 라우터는 via 옆에 위치하는 대신 via에서 더 멀리 조그를 만들어 노치 간격 규칙을 위반하지 않는 훨씬 큰 노치를 생성할 수 있습니다. 그림의 오른쪽 상단에는 빨간색 타원형이 표시되어 있으며, 이는 같은 넷에 있는 두 개의 via가 서로 가깝게 배치되어 있는 위반을 나타냅니다. 이 위반을 수정하기 위해 상세 라우터는 via를 이동하거나 동일한 레이어에 모두 유지하기 위해 비선호 방향으로 조그를 만들 수 있습니다. 그림의 왼쪽 하단에는 빨간색 타원형이 표시되어 있으며, 이는 두꺼운 금속 모양이 얇은 금속 모양과 너무 가까이 배치되어 있는 간격 위반을 나타냅니다. 일반적으로 두꺼운 금속 모양과 얇은 금속 모양 사이의 최소 간격은 두 개의 얇은 금속 모양 사이의 간격보다 큽니다. 이 위반을 수정하기 위해 상세 라우터는 두꺼운 금속 모양과 얇은 금속 모양 사이의 간격을 증가시킬 수 있습니다. 그림의 오른쪽 하단에는 빨간색 타원형이 표시되어 있으며, 이는 서로 독립된 두 개의 via가 옆에 배치되어 있는 간격 위반을 나타냅니다. 이 위반을 수정하기 위해 상세 라우터는 via를 이동하여 서로 오프셋이 되도록 할 수 있습니다. 상세 라우터는 물리적 DRC 위반을 수정하기 위해 여러 번의 반복을 수행합니다. 반복 횟수를 제어할 수 있습니다.  
{: .notice--warning}  

## 6. Pre-Routing Checks and Setup

![2023-05-20-prerouting-checks-and-setup.png]({{site.baseurl}}/assets/images/2023-05-20-prerouting-checks-and-setup.png){: .align-center}  

## 7. Check for Routing “Readiness”

- Run checks to ensure that you are ready for routing:

```tcl
check_design -checks pre_route_state
```

- Open a Message Browser Window to analyze the violations in the EMS database

To perform the pre routing checks, run the `check_design -checks pre_route_stage` command. This command performs many general checks, as well as specific checks associated with routability. After running this command, open the message browser to evaluate the reported errors and warnings. You must fix any reported errors. For warnings, you can decide whether the issue is something that needs to be fixed.  
사전 라우팅 점검을 수행하려면 `check_design -checks pre_route_stage` 명령을 실행하세요. 이 명령은 일반적인 점검과 라우팅 가능성과 관련된 특정 점검을 수행합니다. 이 명령을 실행한 후에는 보고된 오류와 경고를 평가하기 위해 메시지 브라우저를 열어야 합니다. 보고된 오류는 반드시 수정해야 합니다. 경고의 경우에는 문제를 해결해야 할 사항인지 여부를 판단할 수 있습니다.  
{: .notice--warning}  

## 8. Routability Check

- The routability check calls check_routability to confirm that the design is ready for routing:
  - Blocked ports, out-of-boundary pins
    - A logical port can have several physical pins
    - A port is considered blocked if none of its pins are accessible
  - Pin access edge rules honored
  - Minimum-grid violations

The `check_design` command runs the `check_routability` command to perform the routability checks that confirm that the design is ready for routing. The command checks for issues such as blocked ports, which are ports that have no accessible pins; pins that are outside of the design boundary; pin access edge violations; and minimum grid violations.  
`check_design` 명령은 라우팅 가능성을 확인하기 위해 `check_routability` 명령을 실행하여 라우팅 점검을 수행합니다. 이 명령은 다음과 같은 사항을 점검합니다: 접근할 수 없는 핀이 있는 블록된 포트; 디자인 경계를 벗어난 핀; 핀 접근 가장자리 위반; 최소 그리드 위반 등의 문제점을 확인합니다.  
{: .notice--warning}  

## 9. Secondary PG Pin Connections

![2023-05-20-secondary-pg-pin-connections.png]({{site.baseurl}}/assets/images/2023-05-20-secondary-pg-pin-connections.png){: .align-center}  

Multivoltage designs contain level shifters that convert from a low voltage area to a high voltage area or vice versa. These level shifters have two power pins, so the cell can connect to the two different power supplies. The main rail VDD and VSS power pins, in the picture the long horizontal shapes at the top and bottom of the cell, are connected when the power mesh is built. However, the secondary power pin must be connected using secondary power-pin routing with the detail router. You should route these secondary PG pins before you perform general signal routing. To connect the secondary power pins, the router will look for the nearest strap that provides that power. By default there is no limit to the number of pins that can connect to the same tap point on that strap, which could have some IR drop implications. To limit the number of pins to connect to a single tap point on the strap, use the application option shown.  
다중 전압 설계에는 저전압 영역에서 고전압 영역 또는 그 반대로 변환하는 레벨 시프터가 포함됩니다. 이 레벨 시프터는 두 개의 전원 핀을 가지고 있으며, 셀은 두 가지 다른 전원 공급에 연결될 수 있습니다. 그림에서 볼 수 있는 것처럼, 셀의 상단과 하단에 있는 긴 수평 모양의 VDD 및 VSS 전원 핀은 전원 메시가 구성될 때 연결됩니다. 그러나 보조 전원 핀은 상세 라우터를 사용하여 연결해야 합니다. 일반 신호 라우팅을 수행하기 전에 이러한 보조 PG 핀을 라우팅해야 합니다. 보조 전원 핀을 연결하기 위해 라우터는 해당 전원을 제공하는 가장 가까운 스트랩을 찾습니다. 기본적으로 동일한 스트랩의 동일한 탭 지점에 연결할 수 있는 핀의 수에는 제한이 없으며, 이로 인해 일부 IR 드롭 영향이 발생할 수 있습니다. 스트랩의 단일 탭 지점에 연결할 핀 수를 제한하려면, 표시된 응용 옵션을 사용하세요.  
{: .notice--warning}  

## 10. NDRs for Secondary PG Routing

- It is not uncommon to use NDRs for Secondary PG
  - Restrict routing to certain layers
  - Use only certain vias
  - Specify width

```tcl
# default: false
set_app_options -value true \
  -name route.common.separate_tie off_from_secondary_pg

create_routing_rule sec_pg \
  -widths {M1 0.2 M2 0.2 M3 0.2} \
  -vias {...}

set_routing_rule {VDDH} -rule sec_pg \
  -min_routing_layer M2
  -max_routing_layer M3

route_group -nets {VDDH}
```

Nondefault routing rules are often used for routing secondary PG nets. For example, instead of routing the VDDH net using the minimum width rule, define a nondefault routing rule that makes it wider, apply this rule to all VDDH nets, and restrict the routing of these nets to specific metal layers. To perform the secondary PG net routing, use the `route_group -nets `command. I should comment on the application option at the top of the script - `separate_tie_off_from_secondary_pg` is used to tell the tool to not use the same NDR used for secondary PG routing for tie-high connections with the same VDDH net. Tie offs are generally OK to route using default routing.  
비표준 라우팅 규칙은 보조 PG 넷의 라우팅에 자주 사용됩니다. 예를 들어, VDDH 넷을 최소 폭 규칙으로 라우팅하는 대신, 이를 더 넓게 만드는 비표준 라우팅 규칙을 정의하고, 모든 VDDH 넷에 이 규칙을 적용하고, 이러한 넷을 특정 금속 레이어로 제한할 수 있습니다. 보조 PG 넷 라우팅을 수행하기 위해 `route_group -nets` 명령을 사용합니다. 스크립트 맨 위에 있는 응용 옵션 `separate_tie_off_from_secondary_pg`는 동일한 VDDH 넷에 대한 tie-high 연결에 대해 보조 PG 라우팅에 사용되는 동일한 비표준 라우팅 규칙을 사용하지 않도록 도구에 지시하는 데 사용됩니다. 타이 오프는 일반적으로 기본 라우팅을 사용하여 라우팅할 수 있습니다.  
{: .notice--warning}  

## 11. Inserting Redundant Vias on Signal Nets

![2023-05-20-inserting-redundant-vias-on-signal-nets.png]({{site.baseurl}}/assets/images/2023-05-20-inserting-redundant-vias-on-signal-nets.png){: .align-center}  

----------

DFM = Design for Manufacturability  

While the term “redundant via insertion” is commonly used, a better term to describe the optimization is “via optimization”. This is because “via optimization” does not rely only on replacing single-cut vias with multiple-cut via arrays (redundant vias). Single-cut vias can also be replaced with larger single-cut square vias, or with “bar” vias (rectangular shape), or with vias with better metal overlaps, which can also improve the yield of vias.  
As the redundant via rate increases, it becomes more difficult to converge on the routing design rules and you might see a reduction in signal integrity; therefore, you should use near 100% redundant via insertion only for those blocks that truly require such a high redundant via rate. In addition, achieving very high redundant via rates might require you to modify the floorplan utilization to allow enough space for the redundant vias.  

Depending on what technology you are using, you may want to insert redundant vias in your design, which is done to prevent manufacturing issues and increase reliability. Redundant via insertion replaces single-cut vias with multiple-cut via arrays or another single cut via with a better contact code with a larger area. By default, single-cut vias are replaced by two-cut via arrays. You can specify custom via mapping rules with the add_via_mapping command, which is described in the appendix of this module. There are three methods to perform redundant via insertion, which are shown in order of increasing conversion rate, as well as increasing runtime. By default, redundant via insertion is disabled. You must explicitly enable the method you want to use. When selecting the redundant via insertion method, you must trade off runtime and increased conversion rate. In general, the higher the conversion rate, the higher the runtime. The first method performs opportunistic redundant via insertion after detail routing. To enable this method, set the route.common.post_detailed_route_redundant_via_insertion application option to medium. With this method, the route_auto command performs track assignment and detail routing without considering redundant vias, and then performs redundant via insertion by inserting redundant vias wherever there is room. Because this method is opportunistic and the router doesn't change anything to help improve that, this method inserts the fewest redundant vias and uses the least runtime. The second method reserves space for redundant vias during detail routing and then inserts the redundant vias after detail routing. To enable this method, you must both set the route.common.post_detailed_route_redundant_via_insertion application option and set the route.common.concurrent_redundant_via_mode application option to reserve_space. With this method, during detail routing, the route_auto command saves space to insert the redundant vias but does not actually insert them, and then after detail routing, it inserts the redundant vias wherever there is room. Because the router reserved space during detail routing, there should be more space available, and therefore a higher conversion rate. The third method performs redundant via insertion during detail routing, and can achieve close to a 100% conversion rate. To enable this method, set the route.common.concurrent_redundant_via_mode application option to insert_at_high_cost. With this method, the route_auto command inserts redundant vias during each detail routing iteration. If the router cannot insert a redundant via, it iterates until it can do the insertion. Because redundant via insertion occurs in each detail routing iteration, and the number of iterations can increase to enable redundant via insertion, this method can greatly increase the routing runtime. However, if you are designing a chip that must be completely reliable and cannot fail, such as the chip for the braking system for an electric automobile or a chip for an airplane or rocket, you should use this method to ensure the required conversion rate. In most cases, the reserve space method is an effective way of balancing conversion rate and runtime. The next slide provides more information about this method.  
사용하는 기술에 따라 리디지터스 배이어 삽입을 원할 수 있습니다. 이 작업은 제조 과정에서 발생하는 문제를 방지하고 신뢰성을 높이기 위해 수행됩니다. 리디지터스 배이어 삽입은 단일 컷 배이어를 다중 컷 배이어 배열로 또는 더 큰 면적을 가진 더 나은 컨택트 코드를 가진 다른 단일 컷 배이어로 대체합니다. 기본적으로 단일 컷 배이어는 이중 컷 배이어 배열로 대체됩니다. add_via_mapping 명령을 사용하여 사용자 정의 배이어 매핑 규칙을 지정할 수 있으며, 이는 이 모듈의 부록에 설명되어 있습니다. 리디지터스 배이어 삽입은 변환 속도와 런타임 증가의 관점에서 증가하는 순서로 세 가지 방법으로 수행됩니다. 기본적으로 리디지터스 배이어 삽입은 비활성화되어 있으며, 원하는 방법을 명시적으로 활성화해야 합니다. 리디지터스 배이어 삽입 방법을 선택할 때는 런타임과 증가된 변환 속도를 고려해야 합니다. 일반적으로 변환 속도가 높을수록 런타임도 증가합니다. 첫 번째 방법은 상세 라우팅 후에 기회적인 리디지터스 배이어 삽입을 수행합니다. 이 방법을 활성화하려면 route.common.post_detailed_route_redundant_via_insertion 응용 옵션을 medium으로 설정하면 됩니다. 이 방법에서 route_auto 명령은 리디지터스 배이어를 고려하지 않고 트랙 할당과 상세 라우팅을 수행한 후, 여유 공간이 있는 곳에 리디지터스 배이어를 삽입합니다. 이 방법은 기회적이며 라우터가 개선을 위해 아무런 변경도 하지 않기 때문에 가장 적은 리디지터스 배이어를 삽입하며 런타임도 가장 적게 사용합니다. 두 번째 방법은 상세 라우팅 중에 리디지터스 배이어를 위한 공간을 보존하고, 상세 라우팅 후에 리디지터스 배이어를 삽입합니다. 이 방법을 활성화하려면 route.common.post_detailed_route_redundant_via_insertion 응용 옵션과 route.common.concurrent_redundant_via_mode 응용 옵션을 reserve_space로 설정해야 합니다. 이 방법에서 route_auto 명령은 상세 라우팅 중에 리디지터스 배이어를 삽입하기 위한 공간을 보존하지만 실제로 삽입하지 않으며, 상세 라우팅 후에 여유 공간이 있는 곳에 리디지터스 배이어를 삽입합니다. 상세 라우팅 중에 라우터가 공간을 예약했으므로 더 많은 공간이 확보되어 변환 속도가 높아집니다. 세 번째 방법은 상세 라우팅 중에 리디지터스 배이어 삽입을 수행하며, 거의 100%의 변환 속도를 달성할 수 있습니다. 이 방법을 활성화하려면 route.common.concurrent_redundant_via_mode 응용 옵션을 insert_at_high_cost로 설정하면 됩니다. 이 방법에서 route_auto 명령은 각 상세 라우팅 반복에서 리디지터스 배이어를 삽입합니다. 라우터가 리디지터스 배이어를 삽입할 수 없는 경우 삽입할 수 있을 때까지 반복합니다. 리디지터스 배이어 삽입이 각 상세 라우팅 반복에서 수행되므로 반복 횟수가 증가하여 라우팅 런타임이 크게 증가할 수 있습니다. 그러나 자동차의 브레이킹 시스템을 위한 칩이나 비행기 또는 로켓을 위한 칩과 같이 완벽하게 신뢰성이 보장되어야 하는 칩과 같이 완벽한 신뢰성이 필요한 칩을 설계하는 경우에는 이 방법을 사용하여 요구되는 변환 속도를 보장해야 합니다. 대부분의 경우 공간 예약 방법은 변환 속도와 런타임을 균형있게 조절하는 효과적인 방법입니다. 다음 슬라이드에서 이 방법에 대해 더 자세히 설명합니다.  
{: .notice--warning}  

## 12. Redundant Vias and Small Geometries

![2023-05-20-redundant-vias-and-small-geo.png]({{site.baseurl}}/assets/images/2023-05-20-redundant-vias-and-small-geo.png){: .align-center}  

The redundant via insertion methods I described all perform redundant via insertion for each detail routing iteration. To reduce runtime, you can use the reserve space method and, instead of performing redundant via insertion after each iteration, use the `add_redundant_vias` command to perform standalone redundant via insertion. You can reserve space for redundant vias both during detail routing, by setting the `route.common.concurrent_redundant_via_mode` application option to reserve_space, as mentioned previously, and during ECO routing, by setting the `route.common.eco_route_concurrent_redundant_via_mode` application option to reserve_space. Notice that post-detail-route redundant via insertion is disabled when you use this method. As shown here, the `route_auto`, `route_detail` and `route_opt` commands reserve space for the redundant vias, but do not actually insert them. After running each of these commands, you run the `add_redundant_vias` command to insert the redundant vias. This method results in a high conversion rate, with reduced runtime because it is run only once after all of the detail routing or ECO routing iterations.  
저가 설명한 리디지터스 배이어 삽입 방법은 모두 각 상세 라우팅 반복마다 리디지터스 배이어 삽입을 수행합니다. 런타임을 줄이기 위해, 리디지터스 배이어 삽입을 각 반복 후에 수행하는 대신 `add_redundant_vias` 명령을 사용하여 독립적인 리디지터스 배이어 삽입을 수행할 수 있습니다. 리디지터스 배이어 공간 예약은 상세 라우팅 중에 `route.common.concurrent_redundant_via_mode` 응용 옵션을 reserve_space로 설정함으로써 수행할 수 있습니다. 앞에서 언급한 대로, ECO 라우팅 중에도 `route.common.eco_route_concurrent_redundant_via_mode` 응용 옵션을 reserve_space로 설정하여 리디지터스 배이어를 위한 공간을 예약할 수 있습니다. 이 경우, 상세 라우팅 이후의 리디지터스 배이어 삽입은 비활성화됩니다. 이 예시에서 보여지듯이 `route_auto`, `route_detail` 및 `route_opt` 명령은 리디지터스 배이어를 위한 공간을 예약하지만 실제로 삽입하지 않습니다. 각 명령을 실행한 후에는 `add_redundant_vias` 명령을 실행하여 리디지터스 배이어를 삽입합니다. 이 방법은 상세 라우팅 또는 ECO 라우팅 반복 후에 한 번만 실행되므로 높은 변환 속도와 감소된 런타임을 제공합니다.  
{: .notice--warning}  

## 13. Enable Wire and Via Optimization

- Via and wire optimization is recommended when adding redundant vias
  - Attempts to make long straight traces and reduce the need for vias (using small jogs in non-preferred direction), Default: low
  - `set_app_options -value high -name route.detail.optimize_wire_via_effort_level`
  - Alternatively, use `optimize_routes` to run this same optimization stand-alone after `route_auto`

Wire and via optimization tries to make long, straight traces and reduce the need for vias by using small jogs in the nonpreferred direction. When you insert redundant vias, it is recommended that you perform this optimization, which is enabled by default. However, the default effort level is low. When you insert redundant vias, consider setting this application option to either `medium` or `high`. By default, wire and via optimization occurs for each detail routing iteration and can increase the routing runtime. Similar to redundant via insertion, you can disable wire and via optimization during detail routing and use the `optimize_routes` command to perform the optimization after running the `route_auto` command.  
와이어 및 비아 최적화는 긴, 직선적인 트레이스를 만들고 비아의 필요성을 줄이기 위해 비선호 방향으로 작은 점프(jog)을 사용합니다. 리디지터스 배이어를 삽입할 때는 기본적으로 이 최적화를 수행하는 것이 좋습니다. 그러나 기본 최적화 수준은 낮습니다. 리디지터스 배이어를 삽입할 때에는 이 응용 옵션을 `medium` 또는 `high`로 설정하는 것을 고려해보십시오. 기본적으로 와이어 및 비아 최적화는 각 상세 라우팅 반복마다 수행되며 라우팅 런타임을 증가시킬 수 있습니다. 리디지터스 배이어 삽입과 마찬가지로, 상세 라우팅 중에 와이어 및 비아 최적화를 비활성화하고 `route_auto` 명령을 실행한 후 `optimize_routes` 명령을 사용하여 최적화를 수행할 수 있습니다.  
{: .notice--warning}  

## 14. Concurrent Antenna Fixing: Layer Jumping

![2023-05-20-concurrent-antenna-fixing.png]({{site.baseurl}}/assets/images/2023-05-20-concurrent-antenna-fixing.png){: .align-center}  

An antenna violation can occur when you have an open connection between a driver pin and its load pin, and the load pin is connected to a large antenna made up of multiple metal layers. During the manufacturing process, the die is placed in a large electromagnetic field that is used to activate the etching. If the electromagnetic field of the antenna is large enough, the antenna can collect enough charge to zap the polysilicon gate and damage the chip. Antenna rules define the allowed ratio between the area of dangling pieces of wire before the connection is made and the area or circumference of the polysilicon gate. During detail routing, the tool checks for antenna rule violations and can fix them using one of two methods: layer jumping, which is also called layer hopping, or diode insertion. The recommendation is to use layer jumping during detail routing to fix most of the violations, and then use diode insertion, which is done during post-route optimization, to fix any remaining violations. Layer jumping is controlled by the three application options shown here, and their default settings enable layer jumping during detail routing, so you do not need to set these application options. However, you do need to load the antenna rules. Typically your library provider will give you a Tcl file containing the antenna rules and you need to source this file before routing your design. Diode insertion is usually performed during the final block preparation phase; however, it is disabled by default.  
안테나 위반은 드라이버 핀과 로드 핀 사이의 연결이 열린 상태이고, 로드 핀이 여러 금속 레이어로 이루어진 큰 안테나에 연결되어 있는 경우 발생할 수 있습니다. 제조 과정에서 다이는 에칭을 활성화하기 위해 큰 전자기장에 놓이게 됩니다. 안테나의 전자기장이 충분히 큰 경우, 안테나는 충분한 전하를 수집하여 폴리실리콘 게이트를 충격시켜 칩을 손상시킬 수 있습니다. 안테나 규칙은 연결이 이루어지기 전에 매달려있는 와이어의 면적과 폴리실리콘 게이트의 면적 또는 둘레 사이의 허용 비율을 정의합니다. 상세 라우팅 중에 도구는 안테나 규칙 위반을 확인하고, 레이어 점프(레이어 핍핑이라고도 함) 또는 다이오드 삽입이라는 두 가지 방법 중 하나를 사용하여 이를 수정할 수 있습니다. 권장하는 방법은 상세 라우팅 중에 레이어 점프를 사용하여 대부분의 위반을 수정한 후, 나머지 위반을 수정하기 위해 포스트 라우트 최적화 중에 다이오드 삽입을 사용하는 것입니다. 레이어 점프는 여기에 표시된 세 가지 응용 옵션에 의해 제어되며, 기본 설정은 상세 라우팅 중에 레이어 점프를 활성화하므로 이러한 응용 옵션을 설정할 필요가 없습니다. 그러나 안테나 규칙을 로드해야 합니다. 일반적으로 라이브러리 제공 업체는 안테나 규칙이 포함된 Tcl 파일을 제공하며, 라우팅하기 전에 이 파일을 소스로 지정해야 합니다. 다이오드 삽입은 일반적으로 최종 블록 준비 단계에서 수행되지만, 기본적으로 비활성화되어 있습니다.  
{: .notice--warning}  

## 15. Controlling Router Behavior: Routing Guides

![2023-05-20-controlling-routing-behavior.png]({{site.baseurl}}/assets/images/2023-05-20-controlling-routing-behavior.png){: .align-center}  

Routing guides change the default routing behavior in a specific region. They are honored by global routing as well as detail routing, and should be defined before you perform placement, which performs global routing. To create a routing guide, you use the `create_routing_guide` command. For example, assume that you need a horizontal route in a specific region. Metal layers 1, 3, 5, and 7 have a preferred direction of horizontal; however, there are no routing resources available on these layers in that region. To allow the router to route the horizontal wire on metal 6, which has a preferred direction of vertical, you would use the command shown here. The rectangular region is defined by the coordinates of its lower-left and upper-right corners. This command has many additional options. For information about these, see the user guide or man page.  
라우팅 가이드는 특정 영역에서 기본 라우팅 동작을 변경합니다. 이러한 가이드는 글로벌 라우팅과 상세 라우팅에서 존중되며, 전체 배치를 수행하는 전에 정의해야 합니다. 라우팅 가이드를 생성하기 위해 `create_routing_guide` 명령을 사용합니다. 예를 들어, 특정 영역에서 수평 경로가 필요한 경우를 가정해보겠습니다. 메탈 레이어 1, 3, 5, 7은 수평을 선호하는 방향을 가지고 있지만, 해당 영역에는 이러한 레이어에서의 라우팅 리소스가 없습니다. 수평 와이어를 수평 방향이 선호되지 않는 메탈 6에서 라우팅할 수 있도록 하려면 아래에 표시된 명령을 사용합니다. 사각형 영역은 왼쪽 아래 모서리와 오른쪽 위 모서리의 좌표로 정의됩니다. 이 명령에는 많은 추가 옵션이 있습니다. 이에 대한 자세한 정보는 사용자 가이드 또는 매뉴얼 페이지를 참조하십시오.  
{: .notice--warning}  

## 16. Preventing Routing: Routing Blockages

- To prevent signal routing in a certain area, define a routing blockage:

```tcl
create_routing_blockage
  -layers layers
  -boundary {list_of_points}
  [-net_types net_type_list]
  [-zero_spacing]
  [-reserve_for_top_level_routing]
  [-nets list_of_nets] ...
```

- Routing guides and blockages should be applied before place opt, as they are fully respected by global routing as well
- `-zero_spacing`: A zero-spacing routing blockage disables the minimum spacing rule between the routing blockage boundary and the net shapes
- `-reserve_for_top_level_routing`: Top-level routing uses the area reserved by this kind of routing blockages as a routing channel through the block

Routing blockages prevent signal routing on certain metal layers within a specific region. Routing blockages can be rectangular or rectilinear. Similar to routing guides, they are honored by global routing as well as detail routing, and should be defined before you run `place_opt`. To create a routing blockage, you use the `create_routing_blockage` command. By default, routing respects the same minimum spacing requirements defined in the techfile between two nets or a net and a routing blockage on the same layer. If you define a routing blockage with the zero-spacing option, then the minimum spacing rule is disabled, and the route can abut with the routing blockage.  
라우팅 차단은 특정 영역 내에서 특정 메탈 레이어에서의 신호 라우팅을 방지합니다. 라우팅 차단은 직사각형 또는 사각형의 형태를 가질 수 있습니다. 라우팅 차단은 글로벌 라우팅과 상세 라우팅에서 존중되며, `place_opt`를 실행하기 전에 정의해야 합니다. 라우팅 차단을 생성하기 위해 `create_routing_blockage` 명령을 사용합니다. 기본적으로, 라우팅은 동일한 레이어에서 두 개의 넷 또는 넷과 라우팅 차단 사이의 최소 간격 요구 사항을 존중합니다. 만약 zero-spacing 옵션으로 라우팅 차단을 정의하면, 최소 간격 규칙이 비활성화되고, 라우팅은 라우팅 차단과 접하게 됩니다.  
{: .notice--warning}  

## 17. Controlling Scope for Rerouting

- By default, the router is allowed to reroute any pre-routed nets (unless shapes are marked as “user route” bye.g. route custom ')
- To control the scope for rerouting, set the physical status attribute on the net:

```tcl
set_attribute <nets> physical_status
  unrestricted | minor_change | locked
```

- For example, to prevent rerouting on specific nets:

```tcl
set_attribute "netl net2" physical_status locked
```

By default, the router can reroute any nets, except those marked as user routes. An example of a net marked as a user route would be a net created by the `route_custom` command. The scope of change that the router can make to a net is controlled by the net's `physical_status` attribute. By default, this attribute is set to unrestricted, and the router can rip out and reroute the net. To prevent the router from making any change to the net, set its `physical_status` attribute to locked. To allow the router to make small changes to the net, set the attribute to minor_change. For example, if you created only a partial route from the source to the sink pins because you couldn't easily complete the route, you can set the attribute for that net to minor_change net. This setting allows the router to complete the net, but not remove it.  
기본적으로, 라우터는 사용자 경로로 표시된 것을 제외한 모든 넷을 다시 라우팅할 수 있습니다. 사용자 경로로 표시된 넷의 예는 `route_custom` 명령으로 생성된 넷입니다. 넷의 물리적 상태 속성에 의해 라우터가 넷에 가할 수 있는 변경 범위가 제어됩니다. 기본적으로 이 속성은 unrestricted로 설정되어 있어 라우터가 넷을 제거하고 다시 라우팅할 수 있습니다. 넷에 대한 어떠한 변경도 허용하지 않으려면 해당 넷의 `physical_status` 속성을 locked로 설정하십시오. 넷에 대한 작은 변경을 허용하려면 속성을 minor_change로 설정하십시오. 예를 들어, 소스 핀에서 싱크 핀까지의 부분적인 경로만 생성하고 경로를 쉽게 완성할 수 없는 경우 해당 넷의 속성을 minor_change 넷으로 설정할 수 있습니다. 이 설정은 라우터가 넷을 완성할 수 있지만 제거하지는 않습니다.  
{: .notice--warning}  

## 18. Timing Driven Routing

![2023-05-20-timing-driven-routing.png]({{site.baseurl}}/assets/images/2023-05-20-timing-driven-routing.png){: .align-center}  

Timing-driven routing prioritizes the timing critical nets and routes those nets first, which means that these nets get shorter, more direct routes than the noncritical nets. By default, routing is not timing-driven. There are application options that control whether each routing stage is timing-driven; by default, each of these application options is false. To enable timing-driven routing, set these application options to true. When timing-driven routing is enabled, the default effort level is high, which ensures that the timing critical nets get the highest priority. However, this can result in scenic routes for the other nets, which can increase the congestion. If you see many DRC violations or congestion when using timing-driven routing, consider reducing the effort level to medium or low.  
타이밍 주도 라우팅은 타이밍에 중요한 넷을 우선적으로 라우팅하여 이러한 넷이 비타이밍 넷보다 더 짧고 직접적인 경로를 가지도록 합니다. 기본적으로 라우팅은 타이밍 주도가 아닙니다. 각 라우팅 단계가 타이밍 주도인지를 제어하는 애플리케이션 옵션이 있으며, 기본적으로 이러한 애플리케이션 옵션은 모두 false로 설정되어 있습니다. 타이밍 주도 라우팅을 활성화하려면 이러한 애플리케이션 옵션을 true로 설정하십시오. 타이밍 주도 라우팅이 활성화되면 기본적인 노력 수준은 high로 설정되어 타이밍에 중요한 넷이 최우선 순위를 받게 됩니다. 그러나 이로 인해 다른 넷에 대해서는 경로가 지나치게 복잡해질 수 있으며 이는 혼잡도를 증가시킬 수 있습니다. 타이밍 주도 라우팅을 사용할 때 많은 DRC 위반 또는 혼잡도가 발생하는 경우 노력 수준을 medium이나 low로 낮추는 것을 고려해야 합니다.  
{: .notice--warning}  

## 19. Crosstalk Analysis

![2023-05-20-crosstalk-analysis.png]({{site.baseurl}}/assets/images/2023-05-20-crosstalk-analysis.png){: .align-center}  

Crosstalk, which is also called signal integrity, affects the timing of nets that are close to each other. For example, if two nets are next to each other, when one net is rising and the other is falling, it can slow down the victim net, which can affect hold time. If the nets change in the same direction, it could speed up the victim net, which can affect setup time. These timing effects are called delta delay and because they affect the net timing, they affect timing-driven routing. By default, crosstalk analysis is not enabled and routing and post-route optimization do not consider these timing effects. When you enable crosstalk analysis by setting the time.si_enable_analysis application option to true, the timer considers the delta delay, including the timing arrival windows, in the delay calculations. When doing crosstalk analysis, there is always an aggressor net and a victim net, which are close together. In the figure at the bottom of the slide, the top net is the aggressor and the bottom net is the victim. Crosstalk analysis with timing windows looks at the earliest and latest possible times when the aggressor and victim nets transition. If these windows overlap, the timer calculates a delta delay effect. If the windows do not overlap, it is never possible for the aggressor net to transition at the same time as the victim net, so there is no delta delay effect. If timing windows are not enabled, crosstalk analysis assumes that there is always a delta delay effect between an aggressor net and a victim net, resulting in more pessimistic delay calculations.  
크로스토크(또는 신호 무결성)는 서로 근접한 넷 간에 타이밍에 영향을 주는 현상입니다. 예를 들어, 두 개의 넷이 서로 인접해 있을 때 한 넷이 상승하는 동안 다른 넷이 하강하면 피해자 넷의 속도가 느려질 수 있어 홀드 타임에 영향을 줄 수 있습니다. 넷이 동일한 방향으로 변경되면 피해자 넷의 속도가 빨라질 수 있어 세트업 타임에 영향을 줄 수 있습니다. 이러한 타이밍 영향을 델타 딜레이라고 하며, 넷의 타이밍에 영향을 주기 때문에 타이밍 주도 라우팅에도 영향을 줍니다. 기본적으로 크로스토크 분석은 활성화되지 않으며, 라우팅 및 포스트 라우트 최적화에서는 이러한 타이밍 영향을 고려하지 않습니다. time.si_enable_analysis 애플리케이션 옵션을 true로 설정하여 크로스토크 분석을 활성화하면 타이머는 델타 딜레이를 포함한 딜레이 계산에서 타이밍 도착 창을 고려합니다. 크로스토크 분석 시 언제나 공격자 넷과 피해자 넷이 근접해 있습니다. 슬라이드 아래쪽의 그림에서 위쪽 넷이 공격자이고 아래쪽 넷이 피해자입니다. 타이밍 창을 고려하는 크로스토크 분석은 공격자와 피해자 넷이 전환되는 가능한 가장 일찍과 가장 늦은 시간을 확인합니다. 이러한 창이 겹치면 타이머는 델타 딜레이 효과를 계산합니다. 창이 겹치지 않으면 공격자 넷이 피해자 넷과 동시에 전환될 수 없으므로 델타 딜레이 효과가 없습니다. 타이밍 창이 비활성화된 경우, 크로스토크 분석은 공격자 넷과 피해자 넷 간에 항상 델타 딜레이 효과가 있다고 가정하므로 더 비관적인 딜레이 계산이 이루어집니다.  
{: .notice--warning}  

## 20. Crosstalk Prevention

![2023-05-20-crosstalk-prevention.png]({{site.baseurl}}/assets/images/2023-05-20-crosstalk-prevention.png){: .align-center}  

Crosstalk prevention avoids putting long, parallel, timing-critical nets on adjacent tracks, which decreases the chance of delta delay violations between these nets. By default, crosstalk prevention is not enabled. To enable crosstalk prevention, you must both enable crosstalk analysis, as described on the previous slide, and enable crosstalk prevention. You can enable crosstalk prevention in both the global routing and track assignment stages; however, the current recommendation is to enable it only for track assignment and not for global routing.  
크로스토크 예방은 길고 평행한 타이밍에 중요한 넷을 인접한 트랙에 배치하지 않음으로써 이러한 넷 간의 델타 딜레이 위반 가능성을 줄이는 것입니다. 기본적으로 크로스토크 예방은 활성화되지 않습니다. 크로스토크 예방을 활성화하려면 이전 슬라이드에서 설명한 대로 크로스토크 분석을 활성화하고 크로스토크 예방을 활성화해야 합니다. 크로스토크 예방은 글로벌 라우팅과 트랙 할당 단계에서 모두 활성화할 수 있지만, 현재의 권장 사항은 글로벌 라우팅에는 적용하지 않고 트랙 할당에만 적용하는 것입니다.  
{: .notice--warning}  

## 21. Static Noise

![2023-05-20-static-noise.png]({{site.baseurl}}/assets/images/2023-05-20-static-noise.png){: .align-center}  

When you enable signal integrity analysis, it enables static noise analysis in addition to delta delay analysis. When transitions occur on an aggressor net, it can cause a noise bump on the victim net, which can cause the wrong logic value to be captured. Static noise analysis identifies noise bump violations based on the defined noise margins. Typically these values come from the composite current source noise libraries, which are libraries that are characterized with noise information. However, noise analysis also supports user-defined noise margins, which are specified with the `set_noise_margin` command. To report static noise violations, run the `report_noise` command.  
신호 무결성 분석을 활성화하면 델타 딜레이 분석 외에 정적 잡음 분석도 활성화됩니다. 어그레서 넷에서 전환이 발생하면 피해자 넷에 잡음이 발생하여 잘못된 논리 값이 캡처될 수 있습니다. 정적 잡음 분석은 정의된 잡음 여유 공간에 기반하여 잡음 장애를 식별합니다. 일반적으로 이러한 값은 잡음 정보가 포함된 복합 전류 소스 잡음 라이브러리에서 가져옵니다. 그러나 잡음 분석은 사용자 정의 잡음 여유 공간도 지원하며, 이는 `set_noise_margin` 명령을 사용하여 지정할 수 있습니다. 정적 잡음 위반을 보고하려면 `report_noise` 명령을 실행하면 됩니다.  
{: .notice--warning}  

## 22. Align IC Compiler II and PrimeTime Settings

![2023-05-20-align-icc2-and-primetime.png]({{site.baseurl}}/assets/images/2023-05-20-align-icc2-and-primetime.png){: .align-center}  

To increase timing analysis accuracy and ensure good correlation between IC Compiler II timing analysis and PrimeTime signoff timing analysis, set the application options as shown on this slide. These application options enable the composite current source receiver capacitance model and composite current source waveform analysis during IC Compiler II timing analysis.  
IC Compiler II의 타이밍 분석 정확도를 향상시키고 PrimeTime 서명 타이밍 분석과의 좋은 상관관계를 보장하기 위해, 이 슬라이드에 나와 있는대로 어플리케이션 옵션을 설정하십시오. 이러한 어플리케이션 옵션은 IC Compiler II의 타이밍 분석 중에 composite current source 수신자 용량 모델과 composite current source 웨이브폼 분석을 가능하게 합니다.  
{: .notice--warning}  
