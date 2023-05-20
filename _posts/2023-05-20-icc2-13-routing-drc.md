---
title: "IC Compiler II - 13 Routing - DRC"
excerpt: "Block-level Implementation"
date: 2023-05-20 08:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - IC Compiler II
---

## Objectives

- Report and fix DRC violations
- Perform incremental detail route iterations
- List ways to fix DRC violations

## 1. IC Compiler II Flow​

![2023-05-20-icc2-flow-route-auto.png]({{site.baseurl}}/assets/images/2023-05-20-icc2-flow-route-auto.png){: .align-center}  

Let's have a quick look at the general IC Compiler II flow. We are still in the signal detail routing phase. You shouldn't perform `route_opt` or post-route optimization, if there is a large number of physical design rule violations, instead you need to determine the cause and run incremental detail route iterations.  
일반적인 IC Compiler II 플로우를 간단히 살펴보겠습니다. 우리는 여전히 신호 상세 라우팅 단계에 있습니다. 만약 많은 물리적 설계 규칙 위반이 있는 경우, `route_opt`나 포스트 라우트 최적화를 수행해서는 안 됩니다. 대신 원인을 파악하고 증분 상세 라우트 반복을 실행해야 합니다.  
{: .notice--warning}  

## 2. Routing and DRC Fixing

![2023-05-20-routing-and-drc-fixing.png]({{site.baseurl}}/assets/images/2023-05-20-routing-and-drc-fixing.png){: .align-center}  

Just as a reminder, we discussed in the routing unit that the actual routing step consists of 3 items - routing the signal nets, checking DRCs, and performing incremental routing. In this unit we will focus on DRC checks and incremental detail routing.  
우리는 라우팅 단계가 실제로는 3가지로 구성된다는 것을 논의했습니다 - 신호 넷의 라우팅, DRC(Density Rule Check) 검사, 그리고 증분 상세 라우팅. 이 유닛에서는 DRC 검사와 증분 상세 라우팅에 초점을 맞출 것입니다.  
{: .notice--warning}  

## 3. Route the Signal Nets

```tcl
route_auto
```

- Initial routing entails track assignment and detail routing - no post-route optimization
- To run the two phases individually:

```tcl
route_track
route_detail
```

- Modify routing or crosstalk options, routing guides, or via mapping rules and re-route, as needed

As discussed in the routing unit, the command `route_auto` performs track assignment and detail routing. Global routing has been performed previously in the CTS and Optimization stage. The `route_auto` command does not perform post-route optimization. It creates the routes but does not change any logic. If you prefer, you can perform track assignment and detail routing individually. Running in phases might be helpful, for example, to provide details on advanced node impacts that can be analyzed prior to detail routing.  
라우팅 유닛에서 논의한 대로, `route_auto` 명령은 트랙 할당과 상세 라우팅을 수행합니다. 글로벌 라우팅은 이전에 CTS(클럭 트리 합성) 및 최적화 단계에서 수행되었습니다. `route_auto` 명령은 포스트 라우트 최적화를 수행하지 않습니다. 논리를 변경하지 않고 경로를 생성합니다. 원하는 경우 트랙 할당과 상세 라우팅을 개별적으로 수행할 수도 있습니다. 단계별로 실행하는 것은 고급 노드 영향을 상세 라우팅 이전에 분석할 수 있어 도움이 될 수 있습니다.  
{: .notice--warning}  

## 4. Detail Route Iterations

- By default, route auto performs a maximum of 40 detail route iterations to fix DRCs
- If your technology requires more iterations, set the value using:

```tcl
route_auto -max_detail_route_iterations 60
```

By default, the route_auto command performs a maximum of 40 detail routing iterations to fix your DRC violations. In today's technologies, there are usually many DRC violations, so 40 iterations might not be enough. If necessary, you can increase the number of iterations. Remember, the more iterations, the longer the runtime. But this is likely preferable to manually fixing the violations.  
route_auto 명령은 기본적으로 최대 40회의 상세 라우팅 반복을 수행하여 DRC 위반을 해결합니다. 현재의 기술에서는 일반적으로 많은 DRC 위반이 발생하기 때문에 40회의 반복만으로는 충분하지 않을 수 있습니다. 필요한 경우 반복 횟수를 증가시킬 수 있습니다. 기억하세요, 반복 횟수가 많을수록 실행 시간이 길어집니다. 하지만 이는 위반 사항을 수동으로 수정하는 것보다는 선호될 수 있습니다.  
{: .notice--warning}  

## 5. Check Routing DRC Violations

![2023-05-20-check-routing-drc-violations.png]({{site.baseurl}}/assets/images/2023-05-20-check-routing-drc-violations.png){: .align-center}  

To perform detailed checking for DRC violations, use the `check_routes` command. This command checks for violations of physical design rules, minimum spacing rules, minimum width rules, any nondefault routing rules, and via rules, as well as shorts or opens, antenna violations, and voltage area violations. However, by default, it does not check for violations between prerouted nets, such as power nets. For example, if you have a vertical power strap that is placed too close to a vertical ground strap and you're violating the fat via metal rule, by default, the `check_routes` command does not detect the violation. Generally, you should have checked the PG network integrity previously using the command `check_pg_drc`. After running this command, you can bring up the error browser to display the detected violations. When you open the error browser, it shows you all the errors. When you click on a specific error type, there might be more than one of them, so they are expanded down below. When you click on a specific error, the display automatically zooms to that error in the layout window and provides details about the error in the error box.  
DRC 위반을 자세히 확인하려면 `check_routes` 명령을 사용하십시오. 이 명령은 물리적 설계 규칙, 최소 간격 규칙, 최소 폭 규칙, 비표준 라우팅 규칙 및 비아 규칙, 그리고 쇼트 또는 오픈, 안테나 위반 및 전압 영역 위반을 확인합니다. 그러나 기본 설정에서는 파워 넷과 같은 이미 라우팅된 넷 간의 위반을 확인하지 않습니다. 예를 들어, 수직 전원 스트랩이 수직 접지 스트랩과 너무 가까이 배치되어 펫 비아 금속 규칙을 위반하는 경우, 기본 설정에서 `check_routes` 명령은 위반을 감지하지 않습니다. 일반적으로, 미리 `check_pg_drc` 명령을 사용하여 PG 네트워크의 무결성을 확인해야 합니다. 이 명령을 실행한 후 오류 브라우저를 열어 검출된 위반 사항을 표시할 수 있습니다. 오류 브라우저를 열면 모든 오류가 표시됩니다. 특정 오류 유형을 클릭하면 여러 개의 오류가 있는 경우 아래로 확장됩니다. 특정 오류를 클릭하면 레이아웃 창에서 해당 오류로 자동으로 확대되며, 오류 상자에 오류에 대한 세부 정보가 제공됩니다.  
{: .notice--warning}  

## 6. Viewing Cell Internal Structures

![2023-05-20-view-cell-internal-structures.png]({{site.baseurl}}/assets/images/2023-05-20-view-cell-internal-structures.png){: .align-center}  

- By increasing the view level alone, cell internal structures will not be visible — important for understanding DRC violations
- You need to enable “Expanding cell types” for Standard and/or Hard Macro cells

Sometimes there's a violation to a piece of metal that's inside a cell. For example, you might have a metal one route that's outside a cell that is too close to a piece of metal one that's inside the cell. By default, the GUI does not display the information inside a cell because the view level is set to 0 by default. All cells are basically black boxes. To see what is inside a cell, you must increase the view level. But just increasing the view level won't make the information visible. You also need to expand the cell types for standard cells and hard macros by clicking this button over here to display the Level Controls dialog box, and then checking Standard and Hard Macro, which are off by default.  
가끔 셀 내부에 있는 메탈에 위반이 발생할 수 있습니다. 예를 들어, 셀 내부에 있는 메탈과 너무 가까이 위치한 셀 외부의 메탈 라인이 있을 수 있습니다. 기본적으로 GUI는 셀 내부의 정보를 표시하지 않습니다. 이는 기본적으로 뷰 레벨이 0으로 설정되기 때문입니다. 모든 셀은 사실상 블랙 박스입니다. 셀 내부를 볼려면 뷰 레벨을 높여야 합니다. 그러나 뷰 레벨을 높인다고 해서 정보가 표시되지는 않습니다. 또한 표준 셀과 하드 매크로의 셀 유형을 확장해야 합니다. 이를 위해 레벨 컨트롤 대화 상자를 표시하기 위해 이 버튼을 클릭하고, 그런 다음 기본적으로 꺼져 있는 표준 셀과 하드 매크로를 선택해야 합니다.  
{: .notice--warning}  

## 7. Layout versus Schematic Check

![2023-05-20-layout-versus-schematic-check.png]({{site.baseurl}}/assets/images/2023-05-20-layout-versus-schematic-check.png){: .align-center}  

There is also a `check_lvs` command, which is specifically for reporting opens and shorts as well as floating nets. The `check_routes` command also gives you information about shorts and opens, but it checks for everything else, so the runtime is much longer. If you want to find out more quickly if you have shorts or opens, run the `check_lvs` command first. The `check_lvs` command also reports where the opens, shorts, and floating net are located. Similar to the `check_routes` command, you can display the errors reported by the `check_lvs` command using the error browser. If you use the `-open_reporting detailed` option, the GUI displays very useful debugging information in the GUI. For example, the figure shows an open with arrows that indicate where this net is supposed to be connected, which makes it very easy to debug where your opens are. The arrows are displayed because detailed reporting was specified. Similarly, for detailed short reporting, the GUI display a short indicator to indicate exactly where the short occurs.  
`check_lvs` 명령어는 오픈, 쇼트, 그리고 플로팅 넷과 같은 오류를 보고하는 명령어입니다. `check_routes` 명령어도 오픈과 쇼트에 대한 정보를 제공하지만, 모든 다른 오류를 체크하므로 실행 시간이 훨씬 더 길어집니다. 따라서 오픈이나 쇼트가 있는지 빠르게 확인하려면 먼저 `check_lvs` 명령어를 실행하면 됩니다. `check_lvs` 명령어는 또한 오픈, 쇼트, 그리고 플로팅 넷의 위치를 보고합니다. `check_lvs` 명령어에서 보고된 오류를 확인하려면 error browser를 사용할 수 있습니다. `-open_reporting detailed` 옵션을 사용하면 GUI에서 매우 유용한 디버깅 정보를 표시합니다. 예를 들어, 그림에서는 화살표로 표시된 오픈이 나타나며, 이 화살표는 해당 넷이 연결되어야 할 위치를 나타내므로 오픈 위치를 디버깅하는 데 매우 유용합니다. 화살표는 자세한 보고가 지정되었기 때문에 표시됩니다. 마찬가지로, 자세한 쇼트 보고를 위해 GUI에는 쇼트가 발생하는 위치를 나타내는 쇼트 인디케이터가 표시됩니다.  
{: .notice--warning}  

## 8. Detail Routing Loops

- Run additional detail route iterations at any time to fix DRC violations

```tcl
route detail \
  -incremental true
  -max_number_iterations 50
  -coordinates { {llx1 lly1} {urx1 ury1} ... }
```

- With `-coordinates`, you can specify the routing area - multiple rectangles are supported

If you have remaining DRC violations, how do you fix them? You run incremental detail routing using the `route_detail -incremental` command If you do not specify the `-incremental true` option, the router rips out and reroutes the nets, which you do not want to do. Instead, you want to keep your detail routed nets and incrementally improve them. By default, the router performs incremental routing on the entire design. If you have a lot of violations and runtime is an issue, you might want to focus the routing on the regions that contain a high number of violations by specifying the coordinates of those regions. You can also control the number of detail routing iterations performed by the router.  
DRC 위반 사항이 남아 있다면 어떻게 해결할까요? `route_detail -incremental` 명령을 사용하여 증분(detail) 라우팅을 실행합니다. `-incremental true` 옵션을 지정하지 않으면 라우터는 넷을 제거하고 다시 라우팅하게 됩니다. 이렇게 되지 않도록 하기 위해 이미 적용된 detail 라우트를 유지하고 점진적으로 개선시키는 것이 목표입니다. 기본적으로 라우터는 전체 디자인에 대해 증분 라우팅을 수행합니다. 그러나 많은 위반 사항이 있고 런타임이 문제인 경우, 해당 영역의 좌표를 지정하여 라우팅을 집중할 수 있습니다. 또한 라우터가 수행하는 증분 라우팅 반복 횟수를 제어할 수도 있습니다.  
{: .notice--warning}  

## 9. Change Routing Options

![2023-05-20-change-routing-options.png]({{site.baseurl}}/assets/images/2023-05-20-change-routing-options.png){: .align-center}  

There are many detail route and common route options that affect the behavior of the router, and we did not cover them all. All of them start with the key category route and then a subcategory of either `.detail` or `.common`. The `route.detail` application options affect only detail routing, while the `route.common` application options affect all three stages of routing. To see all of these application options, run the `report_app_options` command with wildcards as shown here. To give you one example, the `route.detail.repair_shorts_over_macros_effort_level` controls the effort spent on fixing short violations that happen to occur over macros. If you look at the man page, you will find that the default setting off means that the router still repairs shorts over macros, but only at the detail route stage. The router does not rip out and redo global routing, track assignment, and detail routing. Instead, it performs incremental detail routing using a very small repair window to try to fix those shorts. If it cannot do so within that small window, it gives up. If you increase the effort level to low, medium, or high, you are increasing the size of that repair window. If you set the effort to high, the router might go a lot further away from the short location to make the changes. In addition, the low, medium, and high effort levels allow the nets to be completely ripped out and rerouted.  
라우터의 동작에 영향을 주는 다양한 디테일 라우트 및 공통 라우트 옵션이 있으며, 우리는 모든 옵션을 다루지 않았습니다. 이러한 옵션들은 route라는 키 카테고리로 시작하고, 그 다음에는 `.detail` 또는 `.common`의 하위 카테고리가 있습니다. `route.detail` 애플리케이션 옵션은 디테일 라우팅에만 영향을 미치고, `route.common` 애플리케이션 옵션은 라우팅의 모든 세 단계에 영향을 미칩니다. 이러한 애플리케이션 옵션을 모두 보려면, 이렇게 와일드카드를 사용하여 `report_app_options` 명령을 실행하면 됩니다. 한 가지 예를 들어 설명하면, `route.detail.repair_shorts_over_macros_effort_level`은 매크로 위에 발생하는 단락 위반을 수정하는 데 사용되는 노력 수준을 제어합니다. man 페이지를 참조하면, 기본 설정인 off는 라우터가 여전히 매크로 위의 단락을 수정하지만, 디테일 라우트 단계에서만 이를 수행합니다. 라우터는 전역 라우팅, 트랙 할당 및 디테일 라우팅을 다시 실행하지 않습니다. 대신, 매우 작은 복구 창을 사용하여 증분 디테일 라우팅을 수행하여 해당 단락을 수정하려고 시도합니다. 그러나 그 작은 창 안에서 수정을 수행할 수 없는 경우, 포기하게 됩니다. 노력 수준을 낮음, 중간, 높음으로 설정하면, 해당 복구 창의 크기를 늘리게 됩니다. 노력을 높게 설정하면, 라우터는 변경 사항을 적용하기 위해 단락 위치에서 상당히 멀리 이동할 수도 있습니다. 또한, 낮음, 중간, 높은 노력 수준은 넷이 완전히 제거되고 다시 라우팅되는 것을 허용합니다.  
{: .notice--warning}  

## 10. Rerouting Problematic Nets

![2023-05-20-rerouting-problematic-nets.png]({{site.baseurl}}/assets/images/2023-05-20-rerouting-problematic-nets.png){: .align-center}  

Sometimes you end up in a situation where incremental detail routing cannot fix your violations. If you end up with these stubborn DRC violations, sometimes you must manually rip out that net using the `remove_routes` command and then use the `route_eco` command to reroute it. ECO routing basically starts from scratch, redoing the global routing, track assignment, and detail routing. Oftentimes this allows the router to make a much smarter connection and fix the problem. ECO routing basically leaves all other routes alone. It won't touch them and it only routes whatever was unconnected or removed. Sometimes you can have a situation where you are removing many different routes. In that case, what you might want to do is remove one route and reroute it, and then remove the next route that might be near the other route and then reroute it, and so on. However, sometimes by doing this, rerouting the second or third can disturb the first net. To prevent this, you might want to lock down the `route_eco` nets as you route them. So this is just a reminder that you can change the physical status to locked on these `route_eco` nets so that any subsequent ECO routing doesn't undo them.  
때로는 증분 디테일 라우팅으로도 문제가 해결되지 않는 상황에 직면할 수 있습니다. 이러한 고집스러운 DRC 위반 사항이 발생하면, 수동으로 해당 넷을 `remove_routes` 명령을 사용하여 제거한 다음, `route_eco` 명령을 사용하여 재라우팅해야 합니다. ECO 라우팅은 기본적으로 전역 라우팅, 트랙 할당 및 디테일 라우팅을 다시 수행하여 처음부터 시작합니다. 이를 통해 라우터가 더 영리한 연결을 생성하고 문제를 해결할 수 있는 경우가 많습니다. ECO 라우팅은 다른 모든 라우트를 그대로 두고, 연결되지 않은 라우트나 제거된 라우트만 다시 라우팅합니다. 때로는 여러 다른 라우트를 제거해야 하는 상황이 발생할 수 있습니다. 그럴 경우 한 라우트를 제거하고 재라우팅한 다음, 다른 라우트를 제거하고 재라우팅하고 이를 반복할 수 있습니다. 그러나 이렇게 하면 두 번째 또는 세 번째 라우트를 재라우팅하는 과정에서 첫 번째 넷에 영향을 줄 수 있습니다. 이를 방지하기 위해 `route_eco` 넷을 재라우팅하는 동안 이러한 넷을 잠그는 것이 좋습니다. 따라서, 이는 `route_eco` 넷의 물리적 상태를 locked로 변경하여 재라우팅 과정에서 해당 넷을 수정하지 않도록 할 수 있다는 것을 상기시키는 것입니다.  
{: .notice--warning}  