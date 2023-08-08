---
title: "IC Compiler II - 05 Placement & Optimization Part 2"
excerpt: "Block-level Implementation"
date: 2023-05-16 05:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
---

Welcome to the Placement and Optimization module, used in IC Compiler II block level implementation.  
{: .notice--warning}  

## Objectives

After completing this module, you should be able to: ·Perform congestion related optimizations ·Run global route-based HFS and DRC fixing ·Apply Per-layer congestion and pin access checking and optimization ·Use congestion-driven restructuring (CDR)  
이 모듈을 완료한 후에는 다음을 수행할 수 있어야 합니다:
·혼잡 관련 최적화 수행하기 ·전역 경로 기반 HFS 및 DRC 수정 실행하기 ·레이어별 혼잡 및 핀 접근 확인 및 최적화 적용하기 ·혼잡에 기반한 리스트럭처링 (CDR) 사용하기  
{: .notice--warning}  

## Congestion-Focused Setup Steps

![2023-05-16-congestion-focused-setup.png]({{site.baseurl}}/assets/images/2023-05-16-congestion-focused-setup.png){: .align-center}  

- After running `place_opt` analyze congestion and cell/pin density
- Perform the following setup steps, as applicable, to improve routability
  - Increase placement and congestion effort levels
  - Enable global route-based HFS and DRC Fixing
  - Apply keepout margins and placement blockages
  - Enable pin access checking and optimization
  - Enable cell expansion using GR congestion maps
  - Run congestion-driven restructuring (CDR)

Once you've set up the design, and performed your flow setup, you would run place_opt. As the first item, you would make sure that the congestion along with pin and cell density is acceptable. If you discover congestion issues, then there are a number of approaches to alleviate the congestion. Following slides has some setup changes for the congestion alleviation.  
디자인을 설정하고 플로우 설정을 완료한 후, place_opt를 실행해야 합니다. 첫 번째로 확인해야 할 사항은 혼잡도와 핀 및 셀 밀도가 허용 가능한 수준인지 확인하는 것입니다. 혼잡 문제가 발견된 경우, 혼잡을 완화하기 위한 몇 가지 방법이 있습니다. 다음 슬라이드에는 혼잡 완화를 위한 몇 가지 설정 변경 내용이 있습니다.  
{: .notice--warning}  

## Global-Route Congestion Analysis

After any placement run, first analyze congestion  

- Low effort level is recommended for faster runtime(default is medium)
- Results tend to be pessimistic

```tcl
route_global -congestion_map_only true -effort_level low
```

If congestion is borderline, re-analyze with a higher effort level  

- More accurate results helps to prevent fixing non-existent congestion problems

```tcl
route_global -congestion_map_only true -effort_level medium|high|ultra
```

After running place_opt, it is best to generate a low-effort congestion map. This will not only be generated faster, but the results tend to be pessimistic, which for the first congestion analysis, is a good thing. If this pessimistic map doesn't show any issues, then you are safe. If you have borderline congestion, then you should run the congestion analysis with increasing efforts. Of course, the higher the effort is you have to use for analysis, the higher the effort will be for final routing. If you don't remember how the congestion map is calculated, then please review the floorplanning unit, where this is explained in detail.  
place_opt를 실행한 후에는 저렴한 혼잡도 맵을 생성하는 것이 가장 좋습니다. 이렇게 하면 빠르게 생성될 뿐만 아니라 결과는 비관적인 경향을 가지며, 첫 번째 혼잡도 분석에는 좋은 결과를 제공합니다. 만약 이 비관적인 맵에서 문제가 나타나지 않는다면 안전합니다. 경계선 혼잡도가 있는 경우에는 노력을 증가시켜 혼잡도 분석을 실행해야 합니다. 물론, 분석에 사용하는 노력이 높을수록 최종 라우팅에 필요한 노력도 증가합니다. 혼잡도 맵이 어떻게 계산되는지 기억이 나지 않는다면, 이에 대해 자세히 설명된 폴로 플래닝 유닛을 다시 확인해 주세요.  
{: .notice--warning}  

## Global Route Congestion Map

![2023-05-16-global-route-congestion-map.png]({{site.baseurl}}/assets/images/2023-05-16-global-route-congestion-map.png){: .align-center}  

A global route congestion map to determine whether you will have any routing problems with the current macro placement, or if you have any large congestion hotspots that might pose challenges down the road. A congestion map is a predictor of routability. Routing a design can take a long time, generating a congestion map is a very quick operation in comparison. It uses quick global routing to estimate where you might run into trouble later during routing. To generate a congestion map, you select global route congestion from the pull down menu, then click on reload. Global routing routes the design by dividing it up into global routing cells or GRC's.  
현재 매크로 배치에서 라우팅 문제가 발생할 수 있는지, 또는 나중에 도로에서 도전적인 혼잡 지점이 있는지 확인하기 위해 전역 경로 혼잡도 맵을 사용합니다. 혼잡도 맵은 라우팅 가능성의 예측자입니다. 디자인의 라우팅은 오랜 시간이 걸릴 수 있지만, 혼잡도 맵 생성은 그에 비해 매우 빠른 작업입니다. 빠른 전역 라우팅을 사용하여 라우팅 중에 문제가 발생할 수 있는 위치를 예측합니다. 혼잡도 맵을 생성하려면 드롭다운 메뉴에서 전역 경로 혼잡도를 선택한 후, 리로드를 클릭하면 됩니다. 전역 라우팅은 디자인을 전역 라우팅 셀 또는 GRC(Global Routing Cells)로 분할하여 라우팅합니다.  
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

Congestion calculates the track usage for each layer and displays it in the GUI. Two methods will be supported in IC Compiler II. Sum of overflow for each layer, that is the new default. Total demand minus total supply, which is the old default in previous releases. The new default was introduced because of too much optimism in the old method. To explain how the numbers on the GRC edges are calculated, here's an example. Every edge will be labeled with an overflow number or "signed congestion overflow", followed by the total supply. In the example here +4 divided by 20. In order to calculate the supply, the tool will add the available routing resources or tracks of all layers crossing that edge. For the vertical edge in this example, we consider only horizontal routing layers, M1, M3 and M5. These are the layers that will cross this edge. The available supply of routing tracks for these layers is 8, 7 and 5 respectively. The demand is shown next - this represents the number of nets that need to cross this edge, here 10, 9 and 1. The overflows are calculated layer by layer. For M1, the supply is 8, the demand is 10, therefore the overflow is plus 2. For M3 the overflow is plus 2 as well, and for M5 it is minus 4, the demand is only 1 vs. a supply of 5. Negative overflows are counted as zero. To calculate the final overflow number, we add only positive overflows, which gives us +4 as the final number. So why do we ignore the available routing resources on M5? As mentioned, congestion analysis is a predictor of routability. In this particular case, those 4 signals could be routed, but the router is going to have to work harder. Routing will have to bump the routes all the way up to metal 5, which means that there's going to be a lot of vias. Vias that have to be put in and take up more space which in turn can cause more congestion issues etc. So in general, we have found that this way of calculating congestion correlates quite well to the final routing.  
혼잡도는 각 레이어의 트랙 사용량을 계산하고 GUI에서 표시합니다. IC Compiler II에서는 두 가지 메서드를 지원합니다. 각 레이어의 오버플로 합계인 새로운 기본값과 이전 버전에서의 기본값인 총 수요에서 총 공급을 뺀 값입니다. 새로운 기본값은 이전 방법의 과도한 낙관성 때문에 도입되었습니다. GRC(전역 라우팅 셀) 엣지의 숫자가 어떻게 계산되는지 설명하기 위해 여기 예를 들어보겠습니다. 각 엣지는 오버플로 숫자 또는 "부호 있는 혼잡도 오버플로"로 레이블이 지정되며, 그 뒤에는 총 공급이 표시됩니다. 이 예에서는 +4가 20으로 나눠진 형태입니다. 공급을 계산하기 위해 도구는 해당 엣지를 가로지르는 모든 레이어의 사용 가능한 라우팅 리소스 또는 트랙을 추가합니다. 이 예에서는 수직 엣지에 대해 가로 라우팅 레이어인 M1, M3 및 M5만 고려합니다. 이러한 레이어가 이 엣지를 가로지를 것입니다. 이러한 레이어의 라우팅 트랙의 사용 가능한 공급량은 각각 8, 7 및 5입니다. 수요는 다음과 같이 표시됩니다. 이 엣지를 가로지나야 하는 넷의 수입니다. 이 예에서는 각각 10, 9 및 1입니다. 오버플로는 레이어별로 계산됩니다. M1의 경우, 공급량은 8이고, 수요는 10이므로 오버플로는 양의 2입니다. M3의 경우도 양의 2이며, M5의 경우에는 마이너스 4입니다. 수요는 1에 대해 공급은 5입니다. 음수 오버플로는 0으로 간주됩니다. 최종 오버플로 숫자를 계산하기 위해 양수 오버플로만을 더하면 최종 숫자인 +4가 나옵니다. 그렇다면 M5의 사용 가능한 라우팅 리소스를 왜 무시할까요? 앞서 언급한 대로, 혼잡도 분석은 라우팅 가능성을 예측하는 것입니다. 이 경우에는 그 4개의 신호가 라우팅될 수 있지만, 라우터는 더 많은 작업을 수행해야 합니다. 라우팅은 메탈 5까지 루트를 밀어야 하므로, 많은 수의 비아가 필요합니다. 비아를 추가하면 공간을 더 많이 차지하게 되며, 이는 다시 더 많은 혼잡 문제를 일으킬 수 있습니다. 따라서 일반적으로 이러한 방식으로 혼잡도를 계산하는 것이 최종 라우팅과 상당히 일치하는 것으로 발견되었습니다.  
{: .notice--warning}  

## Increase Placement/Congestion Effort

- Increasing the congestion effort to `high` (default is `medium`) generally improves congestion, at a cost of runtime and possible timing degradation
  - Relevant during **`final_place`**, and during **`initial_place`** when using two-pass placement (discussed later)
- Increasing the **`final_place`** effort level to high may improve congestion
  - May also improve timing

```tcl
set_app_options -name place_opt.place.congestion_effort -value high
set_app_options -name place_opt.final_place.effort -value high
```

The easiest way to deal with congestion is to increase the effort ICC II spends on congestion removal. As shown, change the congestion effort to high, which will affect the placer in the final_place stage, as well as the initial_place stage if you are using two-pass placement. For the most part, increasing the effort for congestion removal will lead to longer runtimes.  
혼잡을 처리하는 가장 쉬운 방법은 ICC II가 혼잡 제거에 소비하는 노력을 증가시키는 것입니다. 예시에서 보이듯이, 혼잡 노력을 높은 값으로 변경하면 최종 배치 단계에서 플레이서에 영향을 미칠 뿐만 아니라, 두 번의 패스 배치를 사용하는 경우 초기 배치 단계에도 영향을 미칩니다. 대부분의 경우, 혼잡 제거를 위한 노력을 증가시키면 실행 시간이 더 오래 걸릴 수 있습니다.  
{: .notice--warning}  

## Global Route-Based HFS and DRC Fixing(GR-opto)

![2023-05-16-gr-opto.png]({{site.baseurl}}/assets/images/2023-05-16-gr-opto.png){: .align-center}  

- For macro-dominated designs, GR-opto can reduce congestion and improve routability
  - Global routing is used to improve buffer placement for high-fanout and DRC-violating nets - reduces congestion for nets crossing numerous channels
    - Timing analysis(net delay modeling) is still based on virtual route

- Benefits the **`initial_drc`** stage (Intended for non-SPG flow)

```tcl
set_app_options -name place_opt.initial_drc.global_route_based \
                -value true
# default: false
```

If the design is macro dominated – the picture on the right shows what we mean by that – then it's recommended to take advantage of global routing for high-fanout net synthesis. Global routing gives you a much more accurate picture of the routing as compared to the default virtual routing, which will allow for more accurate buffering. In general, you will not be using this with an SPG netlist, since the initial_drc step is skipped for SPG designs.  But you can always choose to disable the SPG setting in ICC2, and run global route based initial_drc anyway.  
디자인이 매크로 중심인 경우 - 오른쪽의 그림은 이를 의미하는 것을 보여줍니다 - 고 팬아웃 넷 합성을 위해 전역 라우팅을 활용하는 것이 권장됩니다. 전역 라우팅은 기본 가상 라우팅에 비해 라우팅에 대해 훨씬 정확한 그림을 제공하여 더 정확한 버퍼링을 가능하게 합니다. 일반적으로 SPG(Specialized Packed Guide) 넷리스트와 함께 사용하지 않을 것입니다. 왜냐하면 SPG 디자인에는 초기 DRC 단계가 건너뛰어지기 때문입니다. 하지만 ICC2에서 SPG 설정을 비활성화하고 전역 라우트 기반의 초기 DRC를 실행하는 것을 선택할 수도 있습니다.  
{: .notice--warning}  

## Keepout Margins and Placement Blockages

Apply new, or modify existing keepout margins and/or placement blockages  

![2023-05-16-keepout-margins-blockages.png]({{site.baseurl}}/assets/images/2023-05-16-keepout-margins-blockages.png){: .align-center}  

In some cases, you might have to adjust keepout margins and placement blockages. Normally, these things are already taken care of during floorplanning, but sometimes these have to be updated if you discover congestion issues during place_opt.  
일부 경우에는 keepout 여유 공간과 배치 차단을 조정해야 할 수도 있습니다. 일반적으로 이러한 사항들은 이미 폴로 플래닝 과정에서 처리되지만, 경우에 따라서는 place_opt 중에 혼잡 문제가 발견되었을 때 업데이트해야 할 수도 있습니다.  
{: .notice--warning}  

## Cell and Pin Density Analysis

- In addition to global route congestion, the GUI provides powerful tools to analyze cell and pin density
  - Help to further understand the causes of congestion

![2023-05-16-cell-and-pin-density-analysis.png]({{site.baseurl}}/assets/images/2023-05-16-cell-and-pin-density-analysis.png){: .align-center}  

Global route congestion maps can be very useful, but it is just as important to look at cell and pin density maps as well, and see if and how they correlate with congestion. Congestion can be cell density related, but it doesn't have to. Depending on the analysis, it is easy to determine what technique is best at alleviating your congestion issues.  
전역 경로 혼잡도 맵은 매우 유용할 수 있지만, 셀 밀도 맵과 핀 밀도 맵을 살펴보고 혼잡과의 상관 관계를 확인하는 것도 매우 중요합니다. 혼잡은 셀 밀도와 관련이 있을 수 있지만, 그렇지 않을 수도 있습니다. 분석에 따라 어떤 기술이 혼잡 문제를 완화하는 데 가장 적합한지 쉽게 판단할 수 있습니다.  
{: .notice--warning}  

## Is High Cell Density Causing Congestion?

![2023-05-16-high-cell-density-causing-congestion.png]({{site.baseurl}}/assets/images/2023-05-16-high-cell-density-causing-congestion.png){: .align-center}  

- The placer may create high density areas when optimizing for timing, which can cause congestion
- Look at a cell density map
  - If cell density hotspots coincide with congestion hotspots → Lower cell density in that area

To optimize for timing, the placer will try to place cells as close as possible, in order to minimize the interconnect. Dense placement can cause congestion in some cases, and in order to establish whether it is the dense placement. You need to look at both the congestion and cell density maps and check if there is correlation. If density is the cause of the congestion, then you will need to lower the density in that area.  
타이밍을 최적화하기 위해 플레이서는 가능한 한 셀을 가까이 배치하여 인터커넥트를 최소화하려고 합니다. 밀집된 배치는 일부 경우에 혼잡을 일으킬 수 있으며, 밀집 배치가 원인인지 확인하기 위해 혼잡도 맵과 셀 밀도 맵을 함께 살펴봐야 합니다. 밀도가 혼잡의 원인인 경우 해당 영역의 밀도를 낮추어야 합니다.  
{: .notice--warning}  

## Limit Cell Density with Partial Blockages

![2023-05-16-limit-cell-density.png]({{site.baseurl}}/assets/images/2023-05-16-limit-cell-density.png){: .align-center}  

```tcl
# Define an area with a maximum utilization of 40%
create_placement_blockage -boundary { {10 20} {100 160} } \
    -type partial -blocked_percentage 60
# Max utilization 30%, allow buffers and inverters only
create_placement_blockage -boundary { {10 20} {100 160} } \
    -type allow_buffer_only -blocked_percentage 70
# Max utilization 80% partial blockage, no registers allowed
create_placement_blockage -boundary { {10 20} {100 160} } \
    -type register -blocked_percentage 20
```

A quick way to lower the cell density in a certain area is to use partial blockages. There are several types of partial blockages, which allow you to reduce the number of cells within the blockage. In the first example, `create_placement_blockage` is used to create a partial blockage, with a blocked percentage of 60. This means that the maximum utilization in that area can be up to 40%. In the second example, the blocked percentage is increased to 70%, and at the same time we only allow buffers in the area of the blockage. This placement blockage is generally used to reserve areas for buffering only, often seen in channels between macros. In the last example, the blocked percentage is 20%, and here we specify that we do not want to allow registers. All other cells can be placed here up to a utilization of 80%.  
특정 영역의 셀 밀도를 낮추는 빠른 방법은 부분 차단을 사용하는 것입니다. 부분 차단에는 여러 유형이 있으며, 이를 통해 차단 내의 셀 수를 줄일 수 있습니다. 첫 번째 예에서는 `create_placement_blockage`를 사용하여 차단의 차단 비율을 60으로 설정합니다. 이는 해당 영역의 최대 활용도가 최대 40%가 될 수 있음을 의미합니다. 두 번째 예에서는 차단 비율을 70%로 증가시키고, 동시에 차단 영역에서는 버퍼만 허용합니다. 이러한 배치 차단은 일반적으로 버퍼링 전용 영역을 예약하는 데 사용되며, 매크로 간 채널에서 자주 볼 수 있습니다. 마지막 예에서는 차단 비율이 20%이며, 여기에 레지스터를 허용하지 않는다고 지정합니다. 다른 모든 셀은 최대 80%의 활용도로 배치할 수 있습니다.  
{: .notice--warning}  

## Optimizing Pin Access

Enable pin density awareness during coarse placement

- ICC II will automatically calculate a maximum pin density based on the design and try to respect it during cell placement

```tcl
place.coarse.pin_density_aware
```

- Enable pin optimization during legalization
  - Improves the routability of areas with high pin densities by redistributing the cells

```tcl
place.legalize.optimize_pin_access_using_cell_spacing
```

Affects the "Classic Legalizer" only.  
The Advanced Legalizer performs advanced pin access optimization by default  
{: .notice--danger}

More on pin density. The coarse placer can be configured to calculate a maximum pin density value based on your design. This setting is useful for smaller nodes as well, and it does incur some extra runtime. In addition, you can enable pin optimization during legalization. Note that this is not during placement, but afterwards when the cells need to be legalized. There are always several ways of how to legalize the cell, if you enable pin optimization, the legalizer will make sure that pin access improves. The default here is false, enable it if you see issues with pin accessibility.  
핀 밀도에 대해 더 알아보겠습니다. Coarse 플레이서는 설계에 기반한 최대 핀 밀도 값을 계산할 수 있도록 구성할 수 있습니다. 이 설정은 작은 노드에도 유용하며, 약간의 추가 실행 시간이 소요됩니다. 또한, Legalization 중에 핀 최적화를 활성화할 수 있습니다. 이는 배치 중이 아닌, 셀을 Legalization할 때에 적용됩니다. 셀의 Legalization 방법은 항상 여러 가지가 있으며, 핀 최적화를 활성화하면 Legalizer는 핀 접근성을 개선합니다. 여기서 기본 설정은 false이며, 핀 접근성에 문제가 있는 경우에만 활성화하십시오.  
{: .notice--warning}  

## Optimizing Pin Access for Advanced Node

For advanced node, enable pin cost awareness during coarse placement  

- Pin access difficulties do NOT only depend on pin density, but also locality of the connectivity
- Develop a better model for predicting pin access difficulties that introduces "pin cost" to consider more details of the cell context than just pin density and that is tightly integrated with the coarse placement engine

```tcl
place.coarse.pin_cost_aware
```

- `set_technology` automatically enable Pin Cost Aware Placement for supported nodes, so users does not need to set to true by manual.
  - When Pin Cost Aware Placement enabled, Pin Density Aware Placement is disabled automatically.

![2023-05-16-pin-access-for-adv-node.png]({{site.baseurl}}/assets/images/2023-05-16-pin-access-for-adv-node.png){: .align-center}  

We introduce a new feature for Pin Access improvement on Advanced node, which is Pin Cost Aware Placement. Pin access is not only depending on pin density, but also depending on connectivity, so we develop models to predict pin access for each  advanced node, We call it as Pin Cost Aware Placement. This Pin Cost Aware placement works only with `set_technology`, so `set_technology` automatically enable Pin Cost Aware Placement for supported nodes. For now, we support these technologies, 7+, 7, 6, 5, s5 , s4 and s3 As a note, when Pin Cost Aware Placement is enabled, Pin Density Aware Placement is automatically disabled  
저희는 고급 노드에서 핀 접근성 향상을 위한 새로운 기능을 소개합니다. 이 기능은 핀 밀도뿐만 아니라 연결성에도 의존하므로, 각 고급 노드에 대한 핀 접근성을 예측하기 위해 모델을 개발했습니다. 이를 'Pin Cost Aware Placement'라고 부릅니다. Pin Cost Aware Placement는 `set_technology`와 함께만 작동하며, 지원되는 노드에 대해 `set_technology`를 자동으로 활성화합니다. 현재는 다음 기술을 지원하고 있습니다: 7+, 7, 6, 5, s5, s4 및 s3. 참고로, Pin Cost Aware Placement가 활성화되면 Pin Density Aware Placement는 자동으로 비활성화됩니다.  
{: .notice--warning}  

## Congestion Modeling Enhancements

- Two placer enhancements improve overall congestion by enhancing the routing model and cell expansion behavior
  - Support for soft congestion map information to help designs that heavily use layer promotions and soft NDRs to meet QoR goals
  - Bidirectional cell expansion for reducing lower-layer congestion by considering the directionality (H / V) of the routing resource shortage

Two placer enhancements improve overall congestion by enhancing the routing model and cell expansion behavior Support soft congestion map information to help designs that heavily use layer promotions and soft NDRs. And bidirection, both horizontal and vertical, cells expansion, for reducing lower-layer congestion by considering the directionality (H / V) of the routing resource shortage  
두 가지 플레이서 향상 기능이 전체 혼잡도를 개선하는데 도움이 됩니다. 첫 번째는 라우팅 모델과 셀 확장 동작을 향상시켜 레이어 프로모션과 소프트 NDR을 많이 사용하는 디자인을 지원하는 것입니다. 이는 혼잡도 맵 정보를 부드럽게 지원하여 디자인의 레이어 프로모션과 소프트 NDR을 개선하는 데 도움이 됩니다. 두 번째는 양방향(수평 및 수직)으로 셀을 확장하여 라우팅 리소스 부족의 방향성(H/V)을 고려하여 하위 레이어 혼잡도를 감소시키는 것입니다. 이를 통해 라우팅 리소스 부족의 방향성을 고려하여 수평 및 수직으로 셀을 확장함으로써 하위 레이어 혼잡도를 감소시킬 수 있습니다.  
{: .notice--warning}  

## Placer Support for Soft Congestion Map

- The GR congestion map includes soft demands for routing resources
  - User-added soft NDRs, layer optimization demands/promotions
  - Can greatly affect the final detail routing and with that routing DRCs
- To enable the use of the soft congestion information enable

```tcl
route.global.export_soft_congestion_maps
# default: false
```

- Once enabled, placement uses the GR maps to perform cell expansion to create room for additional routing resources
  - Soft resource requirements are given a lower priority
  - Once enabled, look for `PLACE-079` messages in your log

In release 17.09, the router added support for exporting and measuring the impact of additional resources needed to apply soft routing constraints. The additional demand can sometimes lead to route DRCs during detail route. The placer uses GR congestion information to perform cell expansion to help increase routing resources.  Starting in 19.03, the placer will now see and process the additional information from the soft constraints.   This will allow the placer to compensate for the additional resource, so there are fewer surprises during detail route. While processing the soft constraints, the placer gives them a lower priority to ensure real route congestion is addressed before creating more space for the "nice to have" soft rules.  
릴리스 17.09에서 라우터는 부가적인 자원이 필요한 소프트 라우팅 제약 조건의 적용 영향을 내보내고 측정하는 기능을 추가했습니다. 추가적인 수요는 상세 라우트 중에 라우트 DRCs를 유발할 수도 있습니다. 플레이서는 GR 혼잡도 정보를 사용하여 셀 확장을 수행하여 라우팅 리소스를 증가시키는 데 도움이 됩니다. 19.03부터 플레이서는 소프트 제약 조건에 대한 추가 정보를 보고 처리할 수 있습니다. 이를 통해 플레이서는 추가 자원을 보상함으로써 상세 라우트 중 놀라운 상황이 줄어들도록 할 수 있습니다. 소프트 제약 조건을 처리하는 동안 플레이서는 "있으면 좋을 것"이라는 소프트 규칙에 대한 공간을 만들기 전에 실제 라우트 혼잡도가 해결되도록 우선순위를 낮게 지정합니다.  
{: .notice--warning}  

## Bidirectional Cell Expansion

![2023-05-16-bidirectional-cell-expansion.png]({{site.baseurl}}/assets/images/2023-05-16-bidirectional-cell-expansion.png){: .align-center}  

Cell expansion: The placer bloats cell area to create empty spaces for additional routing resources  

- 18.06 emphasizes lower routing layers (→ pin connections → better GR-DR correlation), but only performed horizontal expansion

The placer can now expand in both directions  

```tcl
place.coarse.congestion_expansion_direction
# default: horizontal
```

- Use both to enable bidirectional expansion
  - Helpful with heavy congestion mainly in the horizontal routing layers
  - It is not recommended to use the both value for all designs
    - Vertical expansion is generally more costly because cells need to snap to rows

Congestion-driven placement use cell expansion technique to create empty spaces for additional routing resources, and it expanded horizontal by default, and it is good enough for most cases. If the case, there is heavy congestion mainly in the horizontal routing layers, you try to set this app_option to both, to have cell expansion for both directions, horizontal and vertical. Again, It is not recommended to use the both value setting for all designs, Because Vertical expansion is generally more costly because cells need to snap to rows  
혼잡도 기반 배치는 셀 확장 기술을 사용하여 추가 라우팅 리소스를 위한 빈 공간을 만들며, 기본적으로 수평으로 확장됩니다. 이는 대부분의 경우에는 충분합니다. 그러나 수평 라우팅 레이어에서 주로 혼잡이 심한 경우에는 이 app_option 값을 "both"로 설정하여 수평 및 수직 모두에 대해 셀 확장을 수행해 볼 수 있습니다. 다시 한 번, 모든 디자인에 대해 "both" 값을 사용하는 것은 권장되지 않습니다. 왜냐하면 수직 확장은 일반적으로 셀을 행에 정확히 맞추어야 하기 때문에 비용이 더 많이 듭니다.  
{: .notice--warning}  

## Congestion-Driven Restructuring(CDR)

![2023-05-16-cdr.png]({{site.baseurl}}/assets/images/2023-05-16-cdr.png){: .align-center}  

- Designs with complex AOI/OAI logic structures can cause many net crossings, thereby creating substantial core congestion hotspots
- CDR identifies tangled nets that drive input pins of commutative and associative logic trees((N)AND/OR/XOR trees), reorders and places them more optimally
  - Alleviates congestion
  - Reduces wire length

Congestion driven restructuring is a feature that can identify sub-optimal wiring in And-Or-Invert or Or-And-Invert logic trees based on the placement of the involved cells. Based on the placement, ICC II will re-connect the cells without changing the functionality, for example it could swap the nets connected to input A and B of an XOR gate if the two nets were crossed. By performing this operation on a large scale, congestion can be drastically reduced. As you can see in the pictures, the left set of pictures shows what classic congestion removal does: Move cells away from the congestion hot-spot, to provide the nets in that area with more routing resources. The cell density map confirms that. The design is still congested though With CDR, the actual problem is solved, the netlist is untangled, and you can see that the congestion is gone, and the cell density is much more homogenous. Untangling the nets also leads to less wire length overall.  
혼잡에 기반한 리스트럭처링은 관련된 셀의 배치를 기반으로 And-Or-Invert 또는 Or-And-Invert 논리 트리에서 부적절한 배선을 식별할 수 있는 기능입니다. 배치를 기반으로 ICC II는 기능을 변경하지 않고 셀을 다시 연결할 수 있습니다. 예를 들어, 두 개의 넷이 교차된 경우 XOR 게이트의 입력 A와 B에 연결된 넷을 교체할 수 있습니다. 이러한 작업을 대규모로 수행함으로써 혼잡도를 크게 줄일 수 있습니다. 그림에서 왼쪽의 사진 세트에서는 클래식한 혼잡도 제거 방법이 수행되어 혼잡 지점에서 셀을 이동하여 해당 영역의 넷에 더 많은 라우팅 리소스를 제공합니다. 셀 밀도 맵이 이를 확인합니다. 그러나 여전히 디자인은 혼잡합니다. CDR을 사용하면 실제 문제가 해결되고, 넷리스트가 해결되며 혼잡이 사라지며 셀 밀도가 훨씬 균일해집니다. 넷을 해제하는 것은 전체적으로 더 짧은 와이어 길이를 유도합니다.  
{: .notice--warning}  

## CDR Setup

Congestion-Driven Restructuring is enabled by default  

```tcl
place.coarse.cong_restruct 
# Default: on
```

- By default, one iteration of placement and rewiring will be run during the **`initial_place`** stage of `place_opt`
- To increase the number of iterations (by default 1), use:

```tcl
place.coarse.cong_restruct_iterations 
# Reasonable values: 1-3
```

Control the CDR effort level using:  

```tcl
place.coarse.cong_restruct_effort
# Default: medium
```

- Controls the size of the netlist subset that is made available to CDR
- Allowed values: low, **medium**, high, ultra
  - Higher efforts consider trees with fewer input pins

Starting with the 2016.12-SP2 release, CDR is enabled by default. When you run place_opt, several iterations of placement and rewiring will take place during the initial_place stage. You can control the number of iterations by using the app option shown here. You can increase the effort for CDR by using the application option shown.  
2016.12-SP2 릴리스부터 CDR은 기본적으로 활성화되어 있습니다. place_opt를 실행할 때, 초기 배치 단계에서 여러 번의 배치 및 재배선(iterations)이 이루어집니다. 여기에 표시된 애플리케이션 옵션을 사용하여 반복 횟수를 제어할 수 있습니다. 또한, CDR을 위한 노력을 증가시키려면 표시된 애플리케이션 옵션을 사용할 수 있습니다.  
{: .notice--warning}  

## CDR with SPG

CDR is not supported when using SPG `place_opt`  

- With SPG, the **`initial_place`** stage, where CDR is performed, is skipped

If you want to use CDR in an SPG flow, use the following flow  

- The `create_placement` command starts by performing CDR on the existing placement, followed by incremental placement (configure iterations as needed)

```tcl
set_app_options -name place_opt.flow.do_spg -value true
set_app_options -name place.coarse.cong_restruct_iterations -value 2

create_placement
set_app_options -name place_opt.flow.do_spg -value false
place_opt -from initial_drc
```

Reminder: Apply all congestion-focused controls to the initial design and re-run `place_opt`
{: .notice--primary}

The example shown here is only recommended if you want to force the use of CDR in an SPG flow. In an SPG flow, create_placement starts by performing restructuring/pin reordering (honoring the current SPG placement), after which it performs incremental placement. Overall, 3 iterations of CDR occur. This is controllable with place.coarse.cong_restruct_iterations. After the first initial-placement it is important to turn off SPG, otherwise the place_opt command would not perform the initial_drc stage.  
이 예시는 SPG(Specialized Packed Guide) 플로우에서 CDR을 강제로 사용하고자 할 때에만 권장됩니다. SPG 플로우에서는 create_placement 단계에서 restructuring/pin reordering(현재 SPG 배치를 고려함)를 수행한 후 증분적인 배치를 진행합니다. 전체적으로 CDR의 3회 반복이 발생합니다. 이는 place.coarse.cong_restruct_iterations를 통해 제어할 수 있습니다. 첫 번째 초기 배치 이후에는 SPG를 해제하는 것이 중요합니다. 그렇지 않으면 place_opt 명령은 초기 DRC 단계를 수행하지 않습니다.  
{: .notice--warning}  

## Clock Nets Congestion Improvement

- Two placer enhancements improve congestion for Clock Nets
  - Placement to accounts for the total width, spacing and length (area) of NDR (Non-Default Routing Rules) nets as it optimizes for the cost of nets, rather than only the net length
  - Improve the placement of ICG’s within these structures by ensuring that they are placed close to the flops/latches that they drive

2 new features in  v2020.09, to improve congestion related to Clock Nets, explain about these 2 features next slides.  
2020.09 버전에서 도입된 두 가지 새로운 기능은 클럭 넷과 관련된 혼잡을 개선하기 위한 것으로, 이러한 두 가지 기능에 대해 다음 슬라이드에서 설명합니다.  
{: .notice--warning}  

## Clock WL Aware Placement

- Overview
  - Clock tree tends to consume Congestion/Routability as well as Power because of the complex structure of clock tree and the high toggle rates of clock nets
- Placement determines the location of the registers in the design, which can play a big part in clock tree wirelength
  - To improve the placement of registers in respect to the clock trees, more emphasis is added to the high fanout clock nets during placement
- Benefits
  - It is generally desirable to produce a placement with short clock net wire length for better design QoR
  - Reduce clock wirelength and improve congestion/routability/power for clock nets
- Usage in flow
  - This feature is OBD in v2021.06-SP3.

Tend to increase congestion and power by complex structure of clock tree and high toggle rate of the clock nets. Placement place registers to shorten wirelength of clock nets to improve the congestion/power. And also, this feature improve the placement of elements attached to any net with an NDR by adding additional recognition of the total number of tracks required to route these nets. We have seen cases where, as an example, clock-tree nets carrying NDR's are involved in congestion that is seen only after CTS is inserted, that can be better managed earlier in the flow with an improved understanding of the total area used (example:  extra width, spacing, shielding requirements) by the nets that carry an NDR, rather than linear net length alone. This feature is OBD from v2021.06-SP3, so no need to set any app_option to enable this feature.  
클럭 트리의 복잡한 구조와 클럭 넷의 고 토글 비율은 혼잡도와 전력을 증가시키는 경향이 있습니다. 플레이서는 클럭 넷의 와이어 길이를 단축시키기 위해 레지스터를 배치하여 혼잡도와 전력을 개선합니다. 또한, 이 기능은 NDR(Non-Destructive Read)을 가진 모든 넷에 연결된 요소들의 배치를 개선하기 위해 필요한 총 트랙 수를 추가로 인식함으로써 넷에 대한 더 나은 이해를 제공합니다. CTS(클럭 트리 합성)가 삽입된 이후에만 확인되는 혼잡도로 인해 클럭 트리 넷에 NDR이 포함된 경우가 있으며, 선형 넷 길이만 고려하는 것보다 NDR을 가진 넷이 사용하는 총 면적(예: 추가 폭, 간격, 보호 요구 사항)을 더 잘 파악함으로써 초기 플로우에서 더욱 효과적으로 관리할 수 있습니다. 이 기능은 v2021.06-SP3부터 OBD(On by Default)로 제공되므로, 이 기능을 활성화하기 위해 별도의 애플리케이션 옵션을 설정할 필요가 없습니다.
{: .notice--warning}  

## Sequential Array Aware ICG Placement

![2023-05-16-seq-array-icg.png]({{site.baseurl}}/assets/images/2023-05-16-seq-array-icg.png){: .align-center}  

- Overview
  - The figure on the right shows a 2-dimentional array of sequential elements here ICG’s are used as a gate for the write-enable to each row of storage
  - These sequential array can be difficult to place without improved handling of these ICG’s and the NDR’s on the clock-nets attached to these ICGs.
  - Current placer often creates clusters of ICG cells to drive bigger clusters registers with long clock wires
- Benefits
  - For arrays of sequential elements (latch-arrays, synthesized storage structures/RAM’s), we can improve the placement of ICG’s within these structures by ensuring that they are placed close to the flops/latches that they drive
  - This has been shown to reduce congestion related to the ICG’s of these structures
- UI

```tcl
place.coarse.seq_array_icg_aware
# Default: false
```

By ID'ing 2-dimensional storage arrays like latch-arrays, lookup tables, synthesized RAM's, etc, the placer can generate more-efficient placements by ensuring that the ICG's are kept close to the sequential elements they drive within these structures. This feature helps to improve situations where customers see clumps of these ICG's that are located closer to each other, than the flops/latches that they drive. As the clock nets are likely carrying NDR's, the clustering of these ICG's also causes clustering of nets that are carrying potentially large NDR's, and routability problems can arise The picture above attempts to show how these structures are identified. By ID'ing parallel "rows" of storage elements that share the same inputs across each "bit" of the rows, the placer attempts to distill that the intent is to keep the ICG's close to the flops, In such structrures, these ICG's serve a predominantly functional use as a write enable to each row, rather than being inserted for power savings. By keeping the ICG's of these structures close to the flops/latches, we improve the efficiency and routing friendliness of these structures related to the clock nets in to and out of these ICG's.  
Latch-arrays, lookup tables, synthesized RAMs, 등과 같은 2차원 저장 배열을 식별함으로써 플레이서는 이러한 구조 내에서 순차 요소와 밀접하게 연결되는 ICG(Internal Clock Gate)들이 유지되도록 하여 보다 효율적인 배치를 생성할 수 있습니다. 이 기능은 고객이 이러한 ICG들이 그들이 제어하는 플립플롭/라치보다 서로 더 가까운 위치에 형성되어 있는 그룹을 관찰하는 상황을 개선하는 데 도움이 됩니다. 클럭 넷이 아마도 NDR을 전달하고 있을 가능성이 있으므로, 이러한 ICG들의 집합은 잠재적으로 큰 NDR을 전달하는 넷들의 집합을 형성하고, 라우팅 가능성 문제가 발생할 수 있습니다. 위의 그림은 이러한 구조를 식별하는 방법을 보여주려고 시도한 것입니다. 각 "비트"의 행에서 동일한 입력을 공유하는 저장 요소들의 병렬 "행"을 식별함으로써 플레이서는 ICG를 플립플롭에 가까이 유지하는 의도를 추론하려고 합니다. 이러한 구조에서 이러한 ICG들은 주로 각 행에 대한 쓰기 가능성을 기능적으로 제공하기 위해 삽입되며 전력 절약을 위해 삽입되는 것이 아닙니다. 이러한 구조의 ICG를 플립플롭/라치에 가깝게 유지함으로써 클럭 넷과 관련된 이러한 구조의 효율성과 라우팅 편의성을 개선합니다.  
{: .notice--warning}  

## Power- and Area-Focused Setup Steps

![2023-05-16-power-area-focused-setup.png]({{site.baseurl}}/assets/images/2023-05-16-power-area-focused-setup.png){: .align-center}  

- Analyze power dissipation and area
- Perform the following setup steps, as applicable, to reduce power dissipation and area
  - Increase power and area optimization effort levels
  - Enable pre-route layer optimization
  - Enable Route-Driven Extraction (RDE)
  - Enable low-power placement

Let's look at power- and area-focused setup steps now. In most cases you will be looking at analyzing and improving timing first, that's what the diagram shows, but for this workshop, I will show you the power and area focused setup now, and the timing setup steps in part 3 of the placement unit. First, we will have a look at the power reporting available in ICC2. After that we will have a look at increasing the effort levels, enabling pre-route layer optimization, using global route layer binning, as well as enabling low-power placement.  
이제 전력 및 면적에 초점을 맞춘 설정 단계를 살펴보겠습니다. 대부분의 경우, 먼저 타이밍을 분석하고 개선하는 것이 일반적이지만, 이 워크샵에서는 이제 전력 및 면적에 초점을 맞춘 설정 단계를 보여드리고, 배치 유닛의 3부에서 타이밍 설정 단계를 안내해 드리겠습니다. 먼저, ICC2에서 제공하는 전력 보고 기능을 살펴보겠습니다. 그 다음으로, 노력 수준을 높이고, 프리 라우트 레이어 최적화를 활성화하며, 글로벌 라우트 레이어 binning을 사용하고, 저전력 배치를 활성화하는 방법을 알아보겠습니다.  
{: .notice--warning}  

## Power Analysis

- Report power dissipation:

![2023-05-16-power-analysis.png]({{site.baseurl}}/assets/images/2023-05-16-power-analysis.png){: .align-center}  

- Control power **reporting** using the application options `power.*`

Use the `report_power` command to report power dissipation. The command will report internal, switching, leakage, as well as total power categorized by io pads, memory, registers etc. `report_power_calculation` is used to understand how ICC II calculated the power for a certain pin, net or cell. If you specify a cell, the command will compute the internal, switching and leakage power consumed by it. When specifying a net, you will see the switching power, and when you specify a pin, you will see the internal power only. You can use the `verbose` switch to see all the details, including caps, toggle rate, voltage etc. Finally, Power reporting is controlled by the application options `power.*`. It's important to mention right here that `power.*` options are not used for power optimization; they are used for power reporting only. Using these options, you can for example change the effort used to propagate switching activity, the leakage calculation mode, etc.  
power dissipation를 보고하기 위해 `report_power` 명령을 사용합니다. 이 명령은 io pads, 메모리, 레지스터 등에 따라 내부, 스위칭, 누설 및 총 전력을 보고합니다. `report_power_calculation`은 특정 핀, 넷 또는 셀에 대해 ICC II가 전력을 계산하는 방식을 이해하는 데 사용됩니다. 셀을 지정하면 해당 셀이 소비하는 내부, 스위칭 및 누설 전력을 계산합니다. 넷을 지정하면 스위칭 전력을 볼 수 있고, 핀을 지정하면 내부 전력만 볼 수 있습니다. `verbose` 스위치를 사용하면 용량, 토글 속도, 전압 등을 포함한 모든 세부 정보를 볼 수 있습니다. 마지막으로, 전력 보고는 애플리케이션 옵션인 `power.*`에 의해 제어됩니다. 여기에서 중요한 점은 `power.*` 옵션은 전력 최적화에 사용되는 것이 아니라 전력 보고에만 사용된다는 것입니다. 이러한 옵션을 사용하여 스위칭 활동 전파에 사용되는 노력, 누설 계산 모드 등을 변경할 수 있습니다.  
{: .notice--warning}  

## Area Analysis

- Report area details:

![2023-05-16-area-analysis.png]({{site.baseurl}}/assets/images/2023-05-16-area-analysis.png){: .align-center}  

To find out about cell counts and area, use the command `report_design`. Here you will find counts and area of leaf cells, standard cells, hard and soft macros, etc. On the bottom, you can see the entire core area, chip area, and total site row area.  
셀 개수와 면적에 대한 정보를 얻으려면 `report_design` 명령을 사용합니다. 여기에서는 리프 셀, 표준 셀, 하드 매크로, 소프트 매크로 등의 개수와 면적을 확인할 수 있습니다. 하단에는 전체 코어 면적, 칩 면적 및 총 사이트 행 면적을 볼 수 있습니다.  
{: .notice--warning}  

## Dynamic Power-Driven Placement

- Dynamic power-driven placement reduces dynamic power by moving cells to shorten high switching (toggle rate) activity signal nets
  - Available in all placement stages, including initial placement

![2023-05-16-dynamic-power-driven-placement.png]({{site.baseurl}}/assets/images/2023-05-16-dynamic-power-driven-placement.png){: .align-center}  

- Best results achieved with SAIF switching activity

```tcl
# Low Power effort is enabled with set_qor_strategy
set_qor_strategy -stage pnr -metric total_power
set_scenario_status -dynamic_power true func_ss125c
read_saif design_sim.saif -scenario func_ss125c
place_opt
report_power -corners [all_corners]
```

Dynamic power-driven placement (also called enhanced LPP or eLPP) is available for all placement stages across `place_opt` and `clock_opt`. It optimizes the placement of cells connected to high toggle rate signal (non-clock) nets with the purpose of shortening these net lengths. Shortening high toggle rate nets, which reduces dynamic power dissipation, may lengthen low toggle-rate nets, which in turn increases power dissipation and path delay. Power-aware placement ensures that such trade-offs result in a net reduction of estimated dynamic power dissipation, while not worsening setup timing. Note that this enhanced LPP supersedes the originally available low power placement, which was enabled using the application option `place.coarse.low_power_placement`.  
Dynamic power-driven placement (또는 Enhanced LPP 또는 eLPP)은 `place_opt`와 `clock_opt` 전체적으로 사용할 수 있습니다. 이는 고 토글 비율 신호(클록이 아닌) 넷에 연결된 셀의 배치를 최적화하여 이러한 넷의 길이를 단축시킴으로써 동적 전력 소모를 줄이는 것을 목적으로 합니다. 고 토글 비율 넷의 길이를 단축시킴으로써 동적 전력 소모를 감소시킬 수 있지만, 그로 인해 저 토글 비율 넷의 길이가 늘어나 동적 전력 소모와 경로 지연이 증가할 수 있습니다. 전력을 고려한 배치는 이러한 트레이드오프가 예상 동적 전력 소모의 감소로 이어지면서도 설정 타이밍을 악화시키지 않도록 보장합니다. 이러한 향상된 LPP는 원래 사용 가능한 저전력 배치를 대체하며, 이는 `place.coarse.low_power_placement` 애플리케이션 옵션을 사용하여 활성화되었습니다.  
{: .notice--warning}  

## New Power Optimization Features

- Power optimization flow is significantly enhanced:
  - Bottleneck Aware Global Sizing technology introduced to optimize TNS with minimum power cost
  - New netlist pre-conditioning technology for both combinational and sequential cells to address suboptimalities (ex: excessive use of high power lib cells) in incoming netlist
  - Significant revamp of power optimization flow under-the-hood to improve power from `place_opt` to `route_opt`
  - Enhanced total power aware logic restructuring
  - New technologies for reducing switching power from pin capacitance
- To enable new power optimization features, use:

```tcl
set_qor_strategy -stage <pnr|synthesis> -metric <total_power|leakage_power> \
        -mode extreme_power
```

New power optimization features can be enabled by using the command shown here. Enhanced power optimization flow includes bottleneck aware global sizing technology to optimize TNS with minimum power cost. New netlist pre-conditioning technology for both combinational and sequential cells to address suboptimalities in the incoming netlist. Significant revamp of power optimization flow under the hood to improve power from place_opt to route_opt. Eahanced total power logic restructuring. Technologies to reduce switching power from pin capacitance.  
새로운 전력 최적화 기능은 여기에 표시된 명령을 사용하여 활성화할 수 있습니다. 향상된 전력 최적화 플로우에는 최소 전력 비용으로 TNS(Total Negative Slack)를 최적화하기 위한 병목지점 인식 글로벌 사이징 기술이 포함됩니다. 수용된 넷리스트에서 부적합한 부분을 해결하기 위한 조합 및 순차 셀에 대한 새로운 넷리스트 사전 조건 기술도 있습니다. place_opt에서 route_opt까지의 전력 최적화 플로우를 크게 개선하기 위한 중요한 개편이 이루어졌습니다. 총 전력 로직 재구조화가 강화되었으며, 핀 용량으로부터 스위칭 전력을 감소시키는 기술도 포함되어 있습니다.  
{: .notice--warning}  

## Summary

Continue with part 3 to learn about  

- Design Fusion Advanced Logic Restructuring
- Auto Timing Control
- Buffering Aware Placement
- Enable two-pass placement flow
- Control cell max density
- ICG optimization
- Route-Driven Extraction (RDE) for Preroute Optimization

This is the end of part two of the placement unit. Now continue with the part three for further learning. Where we will discuss about Design Fusion Advanced Logic Restructuring, Auto Timing Control, Buffering Aware Placement, two-pass placement flow, Control cell max density, ICG optimization, and Route-Driven Extraction (RDE) for Preroute Optimization.  
이것으로 배치 유닛의 두 번째 부분이 끝났습니다. 이제 계속해서 세 번째 부분으로 진행하여 추가적인 학습을 진행하겠습니다. 세 번째 부분에서는 Design Fusion Advanced Logic Restructuring, Auto Timing Control, Buffering Aware Placement, two-pass placement flow, Control cell max density, ICG optimization, 그리고 Preroute Optimization을 위한 Route-Driven Extraction (RDE)에 대해 논의할 것입니다.  
{: .notice--warning}  
