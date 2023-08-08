---
title: "IC Compiler II - 06 Placement & Optimization Part 3"
excerpt: "Block-level Implementation"
date: 2023-05-16 06:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
---

## Objectives

- Design Fusion Advanced Logic Restructuring
- Auto Timing Control
- Buffering Aware Placement
- Enable two-pass placement flow
- Control cell max density
- ICG optimization
- Route-Driven Extraction (RDE) for Preroute Optimization

## Timing-Focused Setup Steps

![2023-05-16-timing-focused-setup-steps.png]({{site.baseurl}}/assets/images/2023-05-16-timing-focused-setup-steps.png){: .align-center}  

- Once congestion is acceptable, analyze timing and logical DRCs
- Perform the following setup steps, as applicable, to improve timing
  - Use Design Fusion
  - Auto Timing Control
  - Buffering Aware Placement
  - Enable two-pass placement flow
  - Control cell max density
  - Enable ICG optimization

Once you've made sure that your design's congestion is acceptable, you should analyze the design for timing and logical DRCs, that is for maximum transition and capacitance. There are number of techniques to improve the timing of the design if needed, which we will cover.  
디자인의 혼잡도가 허용 가능한 수준인지 확인한 후에는 최대 전이 시간과 용량에 대한 타이밍 및 논리적 DRC(Design Rule Check)를 위해 디자인을 분석해야 합니다. 필요한 경우 디자인의 타이밍을 개선하기 위한 여러 기법이 있으며, 이를 다룰 것입니다.  
{: .notice--warning}  

## Timing/DRC QoR Summary Report

![2023-05-16-timing-drc-qor-summary-report.png]({{site.baseurl}}/assets/images/2023-05-16-timing-drc-qor-summary-report.png){: .align-center}  

- Use `report_gor -summary` to obtain a summary of timing violations across all active scenarios, listed by type (setup, hold), and number of logical DRC violations
- Use `report_timing` for violating timing path details
- Use `report_constraints -all -max_tran|-max_cap` for violating DRC details

The quickest way to get an overview, over your timing numbers is to use, `report_qor -summary`. This will show you worst negative slack (WNS), total negative slack (TNS), and the number of violating endpoints (NVE) for every scenario you have defined in your design. Since we are running `place_opt`, it's important to focus on the setup scenarios since hold is not addressed during `place_opt`. To examine the violating paths in more detail, you can use `report_timing`. And in order to examine the logical DRC violations you can use the `report_constraints` command.  
타이밍 숫자에 대한 개요를 가장 빠르게 얻는 방법은 `report_qor -summary`를 사용하는 것입니다. 이를 통해 디자인에서 정의한 모든 시나리오에 대해 최악의 음의 슬랙(WNS), 총 음의 슬랙(TNS), 그리고 위반하는 엔드포인트의 수(NVE)를 확인할 수 있습니다. `place_opt`를 실행 중이므로 hold는 `place_opt` 동안 처리되지 않으므로 setup 시나리오에 초점을 맞추는 것이 중요합니다. 더 자세히 위반하는 경로를 조사하려면 `report_timing`을 사용할 수 있으며, 논리적 DRC 위반을 조사하려면 `report_constraints` 명령을 사용할 수 있습니다.  
{: .notice--warning}  

## Capture Comprehensive Design Metrics

```bash
icc2_shell> create_qor_snapshot -name place_opt
icc2_shell> report_qor_snapshot -display
# or later from Linux:
% firefox snapshot/index.html
```

- Setup/Hold Timing across all scenarios
- Numbers of violations (max tran/cap/...)
- Cell/buf/inv counts, memory/CPU stats
- Timing histograms
- Links to detailed reports

A good way to capture all design metrics is to use a QoR snapshot. By running the command `create_qor_snapshot`, ICC II will generate a number of reports that capture all the metrics listed below, and will create an html file, which can be browsed in a web browser like firefox, or reviewed within ICC II using `report_qor_snapshot`. As you can see from the screenshot, the reports are very detailed, and can be captured after every implementation stage. This capability was introduced in ICC II 2016.12-SP2.  
모든 디자인 메트릭을 캡처하기에 좋은 방법은 QoR 스냅샷을 사용하는 것입니다. `create_qor_snapshot` 명령을 실행하면 ICC II가 아래에 나열된 모든 메트릭을 포함하는 여러 보고서를 생성하고, html 파일을 생성하여 웹 브라우저(예: Firefox)에서 탐색하거나 `report_qor_snapshot`을 사용하여 ICC II 내에서 검토할 수 있습니다. 스크린샷에서 볼 수 있듯이, 보고서는 매우 자세하며, 모든 구현 단계 이후에 캡처할 수 있습니다. 이 기능은 ICC II 2016.12-SP2에서 소개되었습니다.  
{: .notice--warning}  

## Analyze Metrics in Depth

```tcl
create_qor_snapshot -name place_opt
report_qor_snapshot
query_gor_snapshot -name place_opt -display
```

![2023-05-16-analyze-metrics-in-depth.png]({{site.baseurl}}/assets/images/2023-05-16-analyze-metrics-in-depth.png){: .align-center}  

- Generates table from the QoR snapshot
- Click on the entries for more information
- Apply Filtering (false paths) → ignore outliers
- Select paths to highlight in the layout

After creating a QoR snapshot, you can also use the command `query_qor_snapshot` to analyze and visualize individual timing violations. As you can see, you can select paths from the display window and have the corresponding paths highlighted in the layout view. It is also possible to mask timing violations, if you believe that they are due to incorrect timing constraints. If you click on the false path button, ICC II will create a set false path exception and write the constraint to a file. These paths will then no longer affect the overall timing of your design. This is particularly useful when you are working with an early synthesis netlist, and might be using incomplete constraints.  
QoR 스냅샷을 생성한 후에는 `query_qor_snapshot` 명령을 사용하여 개별적인 타이밍 위반을 분석하고 시각화할 수도 있습니다. 스크린샷에서 볼 수 있듯이, 디스플레이 창에서 경로를 선택하고 레이아웃 뷰에서 해당 경로를 강조 표시할 수 있습니다. 또한, 만약 잘못된 타이밍 제약 조건 때문에 타이밍 위반이 발생했다고 생각한다면 타이밍 위반을 마스킹할 수도 있습니다. false path 버튼을 클릭하면, ICC II가 false path 예외를 생성하고 해당 제약 조건을 파일에 기록합니다. 이러한 경로들은 더 이상 디자인의 전체적인 타이밍에 영향을 주지 않습니다. 이는 초기 합성 넷리스트를 사용하고 완전하지 않은 제약 조건을 사용할 수 있는 경우에 특히 유용합니다.  
{: .notice--warning}  

## Design Fusion - Advanced Logic Restructuring: Timing, Area, Power

- Design Fusion: Uses the synthesis engine in ICC II to perform logic restructuring, ensuring the best QoR for area, timing and power
- Logic Restructuring is enabled during `place_opt` `final_opto` and `clock_opt` `final_opto`
- Expect to see timing and area improvements and secondarily, leakage power improvement

Logic restructuring is enabled during `place_opt` `final_opto` and `clock_opt` `final_opto` to ensure the best QoR for area, timing and power.  
논리 재구조화는 `place_opt`의 `final_opto` 및 `clock_opt`의 `final_opto` 단계에서 활성화되어 면적, 타이밍 및 전력에 대한 최상의 QoR(Quality of Results)를 보장합니다.  
{: .notice--warning}  

## Logic Restructuring for Area and Timing

![2023-05-16-logic-restructuring-for-area-timing.png]({{site.baseurl}}/assets/images/2023-05-16-logic-restructuring-for-area-timing.png){: .align-center}  

- Area restructuring finds functionally equivalent cones of logic that improve area/leakage without degrading timing
- Timing-based restructuring solutions are evaluated in parallel with other transforms like sizing and buffering, to find the best overall timing solution

Area restructuring finds functionally equivalent cones of logic, that improves area or leakage without degrading timing. Timing-based restructuring solutions are evaluated in parallel, with other transforms like sizing, and buffering to find best timing solution.  
면적 재구조화는 기능적으로 동등한 로직 콘들을 찾아 면적이나 누설을 개선하는 작업으로, 타이밍을 저하시키지 않습니다. 타이밍 기반 재구조화 솔루션은 사이징 및 버퍼링과 같은 다른 변환과 병렬로 평가되어 최상의 타이밍 솔루션을 찾습니다.  
{: .notice--warning}  

## Logic Restructuring for Power

- Logic cones of combinational gates are identified and remapped with equivalent implementations with better power and similar timing

![2023-05-16-logic-logic-restructuring-for-power.png]({{site.baseurl}}/assets/images/2023-05-16-logic-logic-restructuring-for-power.png){: .align-center}  

For power and timing logic cones of combinational gates are identified, and remapped with equivalent implementations. As shown here in the slide, ICC II can use Composition, Decomposition and rewiring to address it.  
전력과 타이밍을 위해 조합 게이트의 로직 콘들이 식별되고, 동등한 구현으로 재매핑됩니다. 이 슬라이드에서 보여지듯이, ICC II는 이를 해결하기 위해 조합, 분해 및 재연결을 사용할 수 있습니다.  
{: .notice--warning}  

## Define Fusion - Logic Restructuring for Area and Timing

```tcl
opt.common.advance_logic_restructuring_mode
# Default: none
```

- Enable logic restructuring using
  - Values: "none", "area", "timing", "power", "area_timing", "timing_power"
    - Enabling power restructuring will also enable area restructuring under the hood
    - See notes for details
- Requires the RTL2GDS Advanced Fusion Add-on key
  - `place_opt` and `clock_opt` commands will exit immediately when the feature is enabled but the necessary license is not available

To enable logic restructuring, use the app option, `opt.common.advance_logic_restructuring_mode`. The default value of it is, none. User can specify values for area, timing, power, both area and timing or timing and power. The features requires advance fusion key.  
논리 재구조화를 활성화하려면 `opt.common.advance_logic_restructuring_mode` 애플리케이션 옵션을 사용합니다. 기본값은 "none"입니다. 사용자는 "area", "timing", "power", "both area and timing" 또는 "timing and power"와 같은 값들을 지정할 수 있습니다. 이 기능은 고급 퓨전 키가 필요합니다.  
{: .notice--warning}  

## Restructuring - Wire Length Cost

- By default, restructuring moves will be rejected in case of wire length increase, which can be modified by:
  - `opt.common.advanced_logic_restructuring_wirelength_costing`
  - Default: high
  - medium: more area reduction; restriction on wire length increase is relaxed
  - none: maximum area reduction; no restriction on wire length increase
- Apply "medium" or "none" settings on designs with few routing challenges to get more benefits out of logic restructuring

If your design does not have congestion or routing issues, you can get the tool to accept the restructuring results, even if it increases the wire length by setting the app option, `opt.common.advanced_logic_restructuring_wirelength_costing`, to medium or none. Default is high.  
만약 디자인에 혼잡도나 라우팅 문제가 없다면, `opt.common.advanced_logic_restructuring_wirelength_costing` 애플리케이션 옵션을 medium 또는 none으로 설정하여 와이어 길이를 증가시키더라도 재구조화 결과를 도구가 수용하도록 할 수 있습니다. 기본값은 high입니다.  
{: .notice--warning}  

## Out-of-the-Box Placement Settings

![2023-05-16-out-of-the-box-placement.png]({{site.baseurl}}/assets/images/2023-05-16-out-of-the-box-placement.png){: .align-center}  

- Default placer settings are configured for best “out of the box” placement results, for example:
  - Timing-driven and buffering aware initial placement
(as compared to wire-length driven)
  - Weighted TNS driven placement (auto timing control)
    - No need for customized path groups, move bounds and net weights
  - Placement density and congestion-driven max utilization settings applied automatically throughout placement
- All of the above features are on by default

The first feature is Buffer Aware Timing Driven Placement for initial placement In the place_opt command flow, the initial placement is wirelength-driven. This can lead to poor placement of registers, and can lead to difficulties in closing timing at later stages. Running timing-driven placement as initial placement is problematic as well. As high fanout nets will dominate the timing and appear critical;  timing-driven will try to shorten nets unnecessarily, thus leading to sub-optimal timing and congestion results. Solution is buffering aware timing driven placement. Here, placer models the "buffered delays" of the netlist, and uses that timing information to run timing driven initial placement. There by helping timing Q o R. Auto Timing Control introduced many timing improvement techniques in placer. These techniques help in improving timing Q o R out, reduces the need for user to create bounds, and a large number of path groups, etc. This is by enhanced TNS-driven placement along with the focus on most critical paths. Combining benefits of wns and tns are: WNS-driven placer would only put focus on WNS path, leaving large number of paths unoptimized in case of a WNS bottleneck. TNS-driven placer would put similar weight on all violating paths. It can result in worse WNS. ATC puts the high effort on reducing slack of WNS, or near WNS paths, and very gradually reduces the effort as it works on lesser critical paths. So, does good at both WNS and TNS optimization. As part of ATC, placement is enhanced to focus on both WNS and TNS.  
첫 번째 기능은 초기 배치를 위한 버퍼 인식 타이밍 주도 배치입니다. place_opt 명령 플로우에서 초기 배치는 와이어 길이를 기반으로 이루어집니다. 이는 레지스터의 잘못된 배치를 야기하고, 나중에 타이밍을 닫는 데 어려움을 초래할 수 있습니다. 타이밍 주도 배치를 초기 배치로 실행하는 것도 문제가 됩니다. 고 팬아웃 넷은 타이밍을 지배하고 중요해 보이며, 타이밍 주도 배치는 불필요하게 넷을 줄이려고 시도하여 최적의 타이밍 및 혼잡도 결과를 낳을 수 있습니다. 이를 해결하기 위해 버퍼 인식 타이밍 주도 배치를 사용합니다. 여기서 배치 도구는 넷리스트의 "버퍼 지연"을 모델링하고 해당 타이밍 정보를 사용하여 타이밍 주도 초기 배치를 수행합니다. 이를 통해 타이밍 Q o R을 돕습니다. Auto Timing Control은 배치 도구에서 여러 개의 타이밍 개선 기술을 도입했습니다. 이러한 기술은 타이밍 Q o R을 향상시키는 데 도움이 되며, 사용자가 경계값이나 많은 경로 그룹 등을 생성할 필요를 줄입니다. 이는 TNS 주도 배치와 가장 중요한 경로에 초점을 맞춘 향상된 TNS 주도 배치를 통해 이루어집니다. WNS와 TNS의 장점을 결합하는 것은 다음과 같습니다. WNS 주도 배치는 WNS 경로에만 초점을 맞추고, WNS 병목 현상의 경우 많은 경로가 최적화되지 않은 상태로 남을 수 있습니다. TNS 주도 배치는 모든 위반 경로에 대해 유사한 가중치를 부여합니다. 이는 더 나쁜 WNS 결과를 초래할 수 있습니다. ATC는 WNS 슬랙 또는 근처 WNS 경로의 슬랙을 줄이는 데 높은 노력을 기울이고, 상대적으로 중요하지 않은 경로에 작업을 수행함에 따라 노력을 점진적으로 줄입니다. 따라서 WNS 및 TNS 최적화에서 양쪽에서 좋은 성과를 내게 됩니다. ATC의 일환으로 배치는 WNS 및 TNS에 모두 초점을 맞추도록 개선되었습니다.  
{: .notice--warning}  

## Enhanced Auto Density Control

- Release 2019.03 makes automatic density control more dynamic by adjusting the settings based on utilization and technology
  - From v2021.06, use single app_option to control Auto Density Control feature

![2023-05-16-enhanced-auto-density-control.png]({{site.baseurl}}/assets/images/2023-05-16-enhanced-auto-density-control.png){: .align-center}  

If you specify a max density, and utilization values by setting the application options, place.coarse.max density and/or place.coarse.congestion_driven_max_util ICC II will use the specified values. If you want manual control, you should set place.coarse.auto_density_control to false, then set the following application options: The first one is place.coarse.max_density. "By default, placement is uniform. Specifying a value of 0.75 for example, brings related cells closer, up to 75% cell density." This variable only affects course placement. The higher the value, the more the clumping. place.coarse.max_density is a global value, and is specified for the entire block: There is no way to specify different values for voltage areas, exclusive bounds, or plan groups, which may have different average densities than the average overall design density. You can use partial blockages to control the density of specified areas independently. The second app option is place.coarse.congestion_driven_max_util. The default value is 0.93, which allows the utilization of non-congested areas to increase up to 93% in order to reduce congestion in congested areas. Reducing the value to 0.85, for example, will keep more clumped cells together. This setting is respected when congestion removal is enabled which is controlled by place_opt.place.congestion effort, which is set to medium by default. During congestion removal, congestion_driven_max_util overrides the max_density percentage in areas where the cells are being moved to.  
만약 place.coarse.max_density 및/또는 place.coarse.congestion_driven_max_util 애플리케이션 옵션을 설정하여 최대 밀도 및 활용도 값을 지정한다면, ICC II는 지정된 값들을 사용합니다. 수동으로 제어하려면 place.coarse.auto_density_control을 false로 설정한 후 다음과 같은 애플리케이션 옵션을 설정해야 합니다. 첫 번째 옵션은 place.coarse.max_density입니다. "기본적으로 배치는 균일합니다. 예를 들어, 0.75의 값을 지정하면 관련된 셀들을 75% 셀 밀도까지 가깝게 배치합니다." 이 변수는 coarse placement에만 영향을 미칩니다. 값이 높을수록 셀들이 더 모여 들어갑니다. place.coarse.max_density는 전체 블록에 대해 전역 값으로 지정되며, 전체적인 디자인 밀도보다 평균적인 밀도가 다른 전압 영역, 배타적인 바운드 또는 플랜 그룹과 같은 영역에 대해 다른 값을 지정할 수 있는 방법은 없습니다. 특정 영역의 밀도를 독립적으로 제어하기 위해 partial blockages를 사용할 수 있습니다. 두 번째 애플리케이션 옵션은 place.coarse.congestion_driven_max_util입니다. 기본값은 0.93이며, 혼잡 지역의 혼잡도를 줄이기 위해 비혼잡 지역의 활용도가 최대 93%까지 증가할 수 있습니다. 예를 들어 값이 0.85로 줄이면 더 많은 클러스터링된 셀을 함께 유지할 수 있습니다. 이 설정은 혼잡 제거가 활성화된 경우에 적용되며, 이는 기본적으로 중간 값으로 설정된 place_opt.place.congestion effort에 의해 제어됩니다. 혼잡 제거 중에는 혼잡 지역에서 셀이 이동되는 경우에 max_density 비율을 재정의하는 congestion_driven_max_util이 적용됩니다.  
{: .notice--warning}  

## Two-Pass Placement Flow

- For macro-dominated designs, enabling the two-pass initial placement flow, in addition to GR-opto, can improve timing and congestion
- In the two-pass flow, the `initial_place` phase performs:
  1. Buffering aware timing-driven coarse placement
  2. Trial HENS
  3. Timing- and congestion-driven placement
     - The trial HFNS result is discarded
- Remaining `place_opt` phases remain unchanged
- Affects the `initial_place` stage → Applies to non-SPG flow only

```tcl
set_app_options -list {
    place_opt.initial_drc.global_route_based true
    place_opt.initial_place.two_pass true }
```

While an SPG flow is generally recommended for timing-critical designs, it is possible that a non-SPG flow with two-pass placement may achieve slightly better results. Therefore, if the Q o R after SPG placement is not acceptable, and you can afford the additional run time, try both flows and compare results. Note that, the Q o R difference between these two flows is entirely design dependent. If one design shows better results with a two-pass placement flow compared to and SPG flow, do not assume that this will be true for all designs.  
일반적으로 타이밍이 중요한 디자인에는 SPG(flow)가 권장되지만, 비-SPG(flow)와 2단계 배치를 사용한 경우에는 약간 더 좋은 결과를 얻을 수도 있습니다. 따라서, SPG 배치 이후의 Q o R이 받아들일 수 없는 경우 추가적인 실행 시간이 허용된다면, 두 가지 플로우를 모두 시도하고 결과를 비교해 보세요. 그러나 두 플로우 사이의 QoR 차이는 완전히 디자인에 따라 다를 수 있습니다. 하나의 디자인이 SPG 플로우 대비 2단계 배치 플로우에서 더 좋은 결과를 보인다고 해서 모든 디자인에 대해 동일한 결과를 기대하지 마십시오.  
{: .notice--warning}  

## Optimizing the Clock Gate Enable Logic

![2023-05-16-optimizing-clock-gate-enable-logic.png]({{site.baseurl}}/assets/images/2023-05-16-optimizing-clock-gate-enable-logic.png){: .align-center}  

- Pre-CTS, by default, ICG clock pins are assigned the same latency as that of register clock pins
  - Enable-logic optimization will be **optimistic**
  - May lead to ICG **timing violations** after CTS
- Common solutions:
  - Apply **annotated latencies** on ICG clock pins
  - Apply **path margins** on ICG enable pins
- Disadvantages with these approaches:
  - Time consuming and error prone
  - Applied values are **static**
    - ICG placement is not driven to improve (increase) ICG clock pin latencies
    - Optimization of enable logic does not see ICG latency changes from ICG placement changes

By default at the pre-CTS stage, ICG clock pins are assigned the same latency as that of register clock pins. That may lead to ICG timing violation after the CTS stage. Applying a positive path margin, by `set_path_margin`, to ICG enable pins reduces the required path delay time, to these enable pins by the margin amount. This may allow optimization to see, and try to fix the violating enable timing paths.  
기본적으로 CTS 이전 단계에서 ICG(clock gating) 클럭 핀은 레지스터 클럭 핀과 동일한 지연 시간을 할당받습니다. 그러나 이는 CTS 단계 이후에 ICG 타이밍 위반을 초래할 수 있습니다. `set_path_margin`을 사용하여 ICG enable 핀에 양의 경로 여유(margin)를 적용하면, 해당 enable 핀의 경로 지연 시간이 여유분 만큼 감소합니다. 이는 최적화가 위반하는 enable 타이밍 경로를 파악하고 수정을 시도할 수 있게 합니다.  
{: .notice--warning}  

## Integrated Clock Gate Latency Estimation

![2023-05-16-integrated-clock-gate-latency-estimation.png]({{site.baseurl}}/assets/images/2023-05-16-integrated-clock-gate-latency-estimation.png){: .align-center}  

- Integrated clock gate latency estimation is on by default during `place_opt`
- Clock gate latency estimation is integrated into the pre-CTS optimization flow
  - Ensures that the clock gate latencies are accurate throughout the pre-CTS flow by updating latencies after necessary steps such as:
    - Placement
    - Integrated multibit banking
    - Clock gate optimization steps (merging, splitting, ...)
  - Allows more accurate datapath and useful skew optimization of enable control registers and combinational logic

By using this new feature, there is no need to manually apply ICG latency estimates The new recommended methodology for best ICG enable path timing Q o R is now on by default. The latency estimation engine considers clock NDR and buffer or inverter settings to ensure best correlation to post-CTS latencies. The clock-gate latencies are updated at each move of the ICG inside placer iterations. Placer sees the complete effect of ICG movement and gives best ICG location or timing considering data path delay, and clock network delay at the same time. The integrated clock gate latency estimation offsets, user latency values on clock gating cell instances, and it ignores set clock gate latency settings, (which were used before the introduction of CG estimation).  
이 새로운 기능을 사용하면 ICG 지연 예측을 수동으로 적용할 필요가 없습니다. 최상의 ICG enable 경로 타이밍 Q o R을 위한 새로운 권장 방법론이 기본 설정으로 사용됩니다. 지연 예측 엔진은 클럭 NDR 및 버퍼 또는 인버터 설정을 고려하여 CTS 이후 지연과 가장 일치하도록 보장합니다. 클럭 게이트의 지연은 플레이서(Placer) 반복 내에서 ICG 이동마다 업데이트됩니다. 플레이서는 ICG 이동의 전체 효과를 볼 수 있으며, 데이터 경로 지연과 클럭 네트워크 지연을 동시에 고려하여 최상의 ICG 위치 또는 타이밍을 제공합니다. 통합된 클럭 게이트 지연 예측은 클럭 게이팅 셀 인스턴스에 대한 사용자 지연 값을 보정하며, CG 추정 도입 이전에 사용되던 클럭 게이트 지연 설정을 무시합니다.  
{: .notice--warning}  

## Latency Estimation - Prerequisites

- As the feature uses CTS settings for latency estimation, set up the following before the place _opt command
  - Clock NDRs (set_clock_routing_rules)
  - CTS buffer and inverter library cell list (set_lib_cell_purpose -include cts)
- Do not use trial-clock-tree or optimize-ICG features, they will disable ICG latency estimation

```tcl
reset_app_options {
  place_opt.flow.optimize_icgs
  place_opt.flow.trial_clock_tree }
```

The latency estimation feature uses CTS settings to estimate the latency. It is important to perform some CTS setup, before invoking the place_opt command. The two important setups are, Clock Non default routing rules, and specific CTS buffer and inverter library cell list.  
지연 예측 기능은 CTS 설정을 사용하여 지연을 추정합니다. place_opt 명령을 실행하기 전에 CTS 설정을 수행하는 것이 중요합니다. 두 가지 중요한 설정은 "Clock Non default routing rules" 및 특정 CTS 버퍼 및 인버터 라이브러리 셀 목록입니다.  
{: .notice--warning}  

## Honoring User-Annotated Latency

- In special circumstances such as an ICG driving a clock mesh or an H-tree that has not been built yet, you might want to annotate clock latency on the ICG and have it honored by the tool

```tcl
# Set the required clock latency for ICGs in all scenarios:
current_scenario s1
set_clock_latency -clock [get_clocks {clk1}] -0.5 [get_pins {arch_ICG/CK}]
set_clock_latency -clock [get_clocks {clk1}] 0 [get_pins {arch_ICG/GCLK}]
current_scenario s2
set_clock_latency -clock [get_clocks {clk1}] -0.25 [get_pins {arch_ICG/CK}]
set_clock_latency -clock [get_clocks {clk1}] 0 [get_pins {arch_ICG/GCLK}]

# Set the dont_estimate_clock_latency attribute on the clock pin of the ICGs:
set_attribute [get_pins {arch_ICG/CK}] dont_estimate_clock_latency true
```

In special circumstances such as an ICG driving a clock mesh or an H-tree that has not been built yet, you might want to annotate clock latency on the ICG and have it honored by the tool. The clock latency is set for ICGs in all scenarios.  
특정한 상황에서는 아직 구축되지 않은 클럭 메시나 H-트리를 구성하는 ICG를 주행할 때, 클럭 지연을 ICG에 주석으로 표시하고 도구가 이를 적용하도록 설정할 수 있습니다. 클럭 지연은 모든 시나리오에서 ICG에 대해 설정됩니다.  
{: .notice--warning}  

## Reporting Estimated Latencies

- User-specified clock latency constraints, if any, on clock gates are not modified by this feature:
  - Instead, the feature uses `set_clock_latency -offset` when annotating the estimated clock latencies
- The latency offsets can be reported using the `write_script` command
  - Latency of the `icg1/CP` pin is 0.3ns - 0.1333385ns = 0.1666615ns

```tcl
# -origin user
set_clock_latency 0.5 [get_clocks {clk}]
# -origin user
set_clock_latency -clock [get_clocks {clk}] 0.3 [get_pins {icg1/CP}]
# -origin estimated
set_clock_latency -clock [get_clocks {clk}] -0.1333385 -offset [get_pins {icg1/CP}]
# -origin estimated
set_clock_latency -clock [get_clocks {clk}] -0.0346993 -offset [get_pins {icg2/CP}]
```

You can have user specified latencies, and also allow the tool to do its latency estimation. As the tool performs the latency estimation, it uses the offset values, and these offset values will not modify user specified latencies. Use the `write_script` command, to report the latency offsets. Refer to the notes section for details.  
사용자 지정 지연 값을 가질 수 있으며, 도구가 지연 예측을 수행하도록 허용할 수도 있습니다. 도구가 지연 예측을 수행하는 경우, 오프셋 값이 사용되며 이러한 오프셋 값은 사용자 지정 지연 값을 수정하지 않습니다. `write_script` 명령을 사용하여 지연 오프셋을 보고할 수 있습니다. 자세한 내용은 노트 섹션을 참조하십시오.  
{: .notice--warning}  

## Pre-route/Post-route Correlation

- `place_opt` uses 2-layer (H/V) virtual routing to calculate net lengths
- By default, uses the average RC of all H/V layers
- Modern nodes exhibit large RC variation across layers
  - Predicting required vias without routing is also difficult
- Miscorrelation results in sub-optimal pre-route buffering
  - Under / over-buffering
- Two techniques to handle miscorrelation:
  - Layer promotion
    - Assigns critical nets to higher less resistive layers
  - Route Driven Extraction (RDE)
    - Computes a statistical model for VR nets based on actual global routing data

By default, `place_opt` uses 2-layer virtual routing to calculate net lengths. It uses the average RC of all Horizontal or Vertical layers. Metal layer resistance varies substantially in smaller technology nodes, impacting pre-route R estimation, especially for long nets. This can lead to: increased violations and buffer count, and reduced correlation with post-route results. ICC2 tool will use 2 techniques to mitigate these issues. First technique is Layer promotion, where critical nets are assigned to higher less resistive layers. Second technique is route driven extraction, where tool computes a statistical model for VR nets based on actual global routing data.  
기본적으로 `place_opt`는 넷 길이를 계산하기 위해 2층 가상 라우팅을 사용합니다. 모든 수평 또는 수직 레이어의 평균 RC 값을 사용합니다. 작은 기술 노드에서는 금속 레이어의 저항이 상당히 변동하므로 긴 넷의 경우에 특히 사전 라우팅 저항(R) 추정에 영향을 줍니다. 이로 인해 다음과 같은 문제가 발생할 수 있습니다: 위반 및 버퍼 수의 증가, 후 라우트 결과와의 상관 관계 감소. ICC2 도구는 이러한 문제를 완화하기 위해 2가지 기술을 사용합니다. 첫 번째 기술은 레이어 프로모션(Layer promotion)으로, 중요한 넷은 저항이 더 적은 상위 레이어로 할당됩니다. 두 번째 기술은 라우트 주도 추출(route driven extraction)로, 도구는 실제 글로벌 라우팅 데이터를 기반으로 VR 넷에 대한 통계적 모델을 계산합니다.  
{: .notice--warning}  

## Pre-Route Layer Optimization

- Automatically assigns timing-critical nets to higher layers using min layer constraints during `final_opto` (feature is always on)
  - Reduces the number of buffers needed for the same path
  - May also “demote” nets to lower layers that were promoted previously
- Nets will always be rebuffered once promoted / demoted to achieve the best buffering solution
- Minimum layer constraints are carried all the way to post-route optimization
  - Layer assignments can be output by `write_script` in the file `design.tcl`

----------

The layer optimization also looks for long repeater chains that have been buffered previously, assigns the net to higher layers and rebuffers, thereby reducing the number of buffers and possibly improving its timing. This optimization occurs in the `final_opto` stage.  

`write_script` will capture layer assignments in the file `design.tcl`, for example:

```tcl
set_routing_rule -rule_is_user false -min_routing_layer "M7" -min_layer_mode \
soft -min_layer_mode_soft_cost low [get_nets {pidsel}]

set_routing_rule -rule_is_user false -min_routing_layer "M5" -min_layer_mode \
soft -min_layer_mode_soft_cost low [get_nets {pgnt_n}]
...
```

Note that this feature is not controlled using `place_opt.flow.optimize_layers` — this app option controlled the
layer optimization algorithm used in earlier releases of ICC II.  

The layer optimization also looks for long repeater chains that have been buffered previously, assigns the net to higher layers and rebuffers, thereby reducing the number of buffers and possibly improving its timing. This optimization occurs in the final_opto stage. Refer the note section for design.tcl example.  
레이어 최적화는 이전에 버퍼링된 긴 리피터 체인을 찾아서, 해당 넷을 더 높은 레이어로 할당하고 재버퍼링을 수행함으로써 버퍼의 수를 줄이고 타이밍을 개선할 수 있습니다. 이 최적화는 final_opto 단계에서 수행됩니다. design.tcl 예제를 참조하십시오.  
{: .notice--warning}  

## Route-Driven Extraction (RDE) for Preroute Optimization

- RDE uses global routing on the placed design to capture:
  - Detailed layer occupancy beyond min/max layer usage
  - GR topology and length
  - Via resistance
  - Cell pin geometries
- Once data is captured, all virtual routes will use the GR RC model for RC extraction
  - Ensures close parasitic and timing correlation to actual routing
  - Tends to result in better area and power with similar post-route timing QoR
  - Does not increase full flow run time

To improve correlation, the tool can perform global routing, perform extraction based on the global routes and use this parasitic information when performing optimization.  
상관 관계를 개선하기 위해 도구는 글로벌 라우팅을 수행하고, 글로벌 라우트를 기반으로 추출을 수행하며, 이 패러사이틱 정보를 최적화 수행 시 사용할 수 있습니다. 이를 통해 결과의 상관 관계를 향상시킬 수 있습니다.  
{: .notice--warning}  

## RDE in the Pre-Route Flow

![2023-05-16-rde-in-preroute-flow.png]({{site.baseurl}}/assets/images/2023-05-16-rde-in-preroute-flow.png){: .align-center}  

- RDE capture occurs twice in the flow
  - At the beginning of `place_opt` and `clock_opt` `final_opto`
- RDE data is stored with the design, and is reused for any subsequent optimizations
- RDE is enabled by default for 16nm and below
  - `opt.common.enable_rde`
  - To enable for other technologies, set the application option to true

If the tool detects that the technology used by the design is an advanced technology node that is less than 16 nm, the tool enables this feature by default. To enable it for any technology, set the `opt.common.enable_rde` application option to true. When enabled, the tool performs route-driven extraction during the `final_opto` stage of the `place_opt` and `clock_opt` commands.  
도구는 설계에 사용된 기술이 16nm보다 작은 고급 기술 노드인 경우 이 기능을 기본적으로 활성화합니다. 모든 기술에 대해 이를 활성화하려면 `opt.common.enable_rde` 애플리케이션 옵션을 true로 설정하면 됩니다. 활성화되면 도구는 `place_opt` 및 `clock_opt` 명령의 `final_opto` 단계에서 라우트 주도 추출을 수행합니다.  
{: .notice--warning}  

## Via Ladder Insertion in Pre-route Flow

- Via ladder is a stacked via that starts from the pin layer and extends into the upper routing layers.
- Usage of via ladders during optimization improves the performance and electromigration robustness of a design
- Command used is `insert_via_ladders`
- `insert_via_ladders` command inserts via ladders on pins with via ladder constraints. For this `set_via_ladder_rules` or `set_via_ladder_constrains` commands are used
- Via ladder rules are specified in ViaRule section of the technology file or using `create_via_rule`

Via ladder is a stacked via, that starts from the pin layer, and extends into the upper routing layers. Usage of via ladders during optimization, improves the performance, and electromigration robustness of a design. `insert_via_ladders` command, inserts via ladders on pins with via ladder constraints. Via ladder rules are specified in ViaRule section of the technology file, or using `create_via_rules`.  
Via ladder는 핀 레이어에서 시작하여 상위 라우팅 레이어로 확장되는 적층 비아입니다. 최적화 과정에서 Via ladder를 사용하면 설계의 성능과 전자이동성 강도를 향상시킬 수 있습니다. `insert_via_ladders` 명령은 Via ladder 제약 조건을 갖는 핀에 Via ladder를 삽입합니다. Via ladder 규칙은 기술 파일의 ViaRule 섹션이나 `create_via_rules`를 사용하여 지정할 수 있습니다.  
{: .notice--warning}  

## Placement and Optimization Application Options Summary

- The previous pages showed several useful placement and optimization application options that can help produce better QoR
- Depending on the design’s specific goals, you may want to consider other available options:
  - `report_app_options {place.* place_opt.* opt.*}`

![2023-05-16-placement-opt-app-options.png]({{site.baseurl}}/assets/images/2023-05-16-placement-opt-app-options.png){: .align-center}  

- Apply any QoR control commands to the initial design and re-run `place_opt`

Depending on the design's specific goals, you may explore the available options with `report_app_options {place.* place_opt.* opt.*}`. Apply any QoR control commands to the initial design and re-run `place_opt`.  
디자인의 구체적인 목표에 따라, `report_app_options {place.* place_opt.* opt.*}`를 사용하여 가능한 옵션을 탐색할 수 있습니다. 초기 디자인에 QoR 제어 명령을 적용하고 `place_opt`를 다시 실행하십시오.  
{: .notice--warning}  

## Summary

You should now be able to

- Apply pre-placement setup to control:
  - Design and flow requirements
  - Congestion, timing, area and power QoR
- Perform placement and related optimizations
- Analyze congestion, timing, power and area

You should now be able to apply pre-placement setup to control design and flow requirements. Perform placement and related optimizations. You should be able to analyze congestion, timing, power and area.  
이제 디자인 및 플로우 요구사항을 제어하기 위해 사전 배치 설정을 적용할 수 있어야 합니다. 배치와 관련된 최적화를 수행할 수 있어야 합니다. 혼잡도, 타이밍, 전력 및 면적을 분석할 수 있어야 합니다.  
{: .notice--warning}  
