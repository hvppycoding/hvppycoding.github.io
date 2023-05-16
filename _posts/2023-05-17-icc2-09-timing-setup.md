---
title: "IC Compiler II - 09 Timing Setup"
excerpt: "Block-level Implementation"
date: 2023-05-17 04:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - IC Compiler II
---

## Objectives

After completing this module, you should be able to

- Perform MCMM setup: Define the corners, modes and scenarios required for analysis and optimization; Load the MCMM constraints
- Apply timing and optimization controls
- Confirm implementation phase readiness with a zero-interconnect timing sanity check
- Handle infeasible timing paths

## IC Compiler II Flow

![2023-05-17-icc2-flow.png]({{site.baseurl}}/assets/images/2023-05-17-icc2-flow.png){: .align-center}  

## Multiple Modes and Corners

- Today's chips, have to operate in multiple modes

![2023-05-17-multiple-modes.png]({{site.baseurl}}/assets/images/2023-05-17-multiple-modes.png){: .align-center}  

- and across multiple PVT corners

![2023-05-17-multiple-pvt-corners.png]({{site.baseurl}}/assets/images/2023-05-17-multiple-pvt-corners.png){: .align-center}  

Today's designs, today's chips, have to operate in multiple modes and multiple corners. A mode is something like a functional mode, a low power mode, test mode. A PVT or process voltage and temperature corner is, for example, high-temperature slow, max and leakage, high-temperature fast. The design has to be able to time in all of these corners and across all these modes.  
현재의 디자인 및 칩은 다중 모드와 다중 코너에서 작동해야 합니다. 모드는 기능 모드, 저전력 모드, 테스트 모드와 같은 것입니다. PVT(Pressure Voltage Temperature) 또는 공정 전압 및 온도 코너는 예를 들어 고온 슬로우, 최대 및 leakage, 고온 패스트와 같은 것입니다. 디자인은 모든 이러한 코너와 모드를 통해 정확한 타이밍을 가져야 합니다.  
{: .notice--warning}  

## Concurrent MCMM Optimization

- ICC II allows concurrent optimization under multiple corner and mode combinations, called scenarios

![2023-05-17-scenarios.png]({{site.baseurl}}/assets/images/2023-05-17-scenarios.png){: .align-center}  

- Improves each violation in a scenario, while trying not to cause/increase a violation in another scenario

![2023-05-17-concurrent-mcmm-optimization.png]({{site.baseurl}}/assets/images/2023-05-17-concurrent-mcmm-optimization.png){: .align-center}  

ICC II allows to concurrently optimize design under multiple corner mode combinations called scenarios. A scenario is defined as a combination of a mode with a corner, plus some additional constraints. Concurrent MCMM optimization will watch for all scenarios at the same time, and the goal is that fixing setup in Scenario 1, for example, will not degrade hold in Scenario n. That's the whole idea about the concurrent MCMM optimization. You're looking at all the corners, all the modes, at the same time.  
ICC II는 시나리오라고 불리는 다중 코너 모드 조합 아래에서 디자인을 동시에 최적화할 수 있도록 합니다. 시나리오는 모드와 코너의 조합과 추가 제약 조건으로 정의됩니다. 동시 MCMM(Multi-Channel Multi-Mode) 최적화는 동시에 모든 시나리오를 모니터링하며, 예를 들어 시나리오 1에서 설정을 수정하는 것이 시나리오 n의 홀드를 저하시키지 않도록 목표로 합니다. 이것이 동시 MCMM 최적화의 전체 아이디어입니다. 모든 코너와 모드를 동시에 고려합니다.  
{: .notice--warning}  

## Creating Modes, Corners and Scenarios

- Define modes and corners first
- Define scenarios, comprised of applicable mode+corner combinations

![2023-05-17-creating-modes-corners-scenarios.png]({{site.baseurl}}/assets/images/2023-05-17-creating-modes-corners-scenarios.png){: .align-center}  

```tcl
create_mode func
create_mode test

create_corner ss125c ; # Commoncorner to both modes

create_scenario -mode func -corner ss125c
create_scenario -mode test -corner ss125c
```

In IC Compiler II, you define modes, corners, and scenarios, which are comprised of mode+corner combinations. If you look down here, in the example, it is creating two modes, func and test. Then creating a single corner, and after that it creates two scenarios which share a corner, ss125c. There's a lot of performance improvements that can be achieved through this simple sharing of corners.  
IC Compiler II에서는 모드, 코너 및 시나리오를 정의하며, 이는 모드와 코너의 조합으로 구성됩니다. 아래의 예시에서는 두 가지 모드(func 및 test)를 생성한 후 단일 코너를 생성합니다. 그리고 그 후에는 ss125c라는 코너를 공유하는 두 개의 시나리오를 생성합니다. 이처럼 코너를 공유함으로써 많은 성능 향상을 이뤄낼 수 있습니다.  
{: .notice--warning}  

## Current Corner + Current Mode ↔ Current Scenario

- The current corner and mode determines the current scenario, and vice versa:
  - Switching current corner or mode changes current scenario
  - Switching current scenario changes current corner and/or mode

```tcl
# icc2_shell >
current_corner
# {ss_m40c}
current_mode
# {func}
current_scenario
# {func.ss_m40c}
current_corner ff_125c
# {ff_125c}
current_scenario
# {func.ff_125c}
current_scenario test.ff_m40c
# {test.ff_m40c}
current_mode
# {test}
current_corner
# {ff_m40c}
```

The current mode/corner/scenario affects reporting and constraint loading only. Optimization occurs concurrently among active scenarios.
{: .notice--info}

Corners, modes, scenarios are all objects. When you change a corner and the mode, this will affect the scenario you are currently in. Here in the first example , current_corner, current_mode, current_scenario, this all matches. Let's look at the next example here. Changing the corner to ff_125c. Automatically, the scenario is adjusted. You don't have to do anything. The next example shows that if the scenario is changed, test.ff_m40c, current_mode and current_corner follow.  
코너, 모드, 시나리오는 모두 객체입니다. 코너와 모드를 변경하면 현재 사용 중인 시나리오에 영향을 미칩니다. 첫 번째 예시에서는 current_corner, current_mode, current_scenario이 모두 일치합니다. 다음 예시를 살펴보겠습니다. 코너를 ff_125c로 변경하면 시나리오가 자동으로 조정됩니다. 아무런 조치를 취할 필요가 없습니다. 다음 예시에서는 시나리오가 변경되면(test.ff_m40c), current_mode와 current_corner가 따라가는 것을 보여줍니다.  
{: .notice--warning}  

## Constraint Classification

- Timing constraints are classified into four categories:

![2023-05-17-constraint-classification.png]({{site.baseurl}}/assets/images/2023-05-17-constraint-classification.png){: .align-center}  

When you apply constraints to your design, these constraints will be applied within modes, corners, and scenarios. It's good to know and to understand that there are modal constraints, like a create_clock or a set_case_analysis command. These affect the mode and will be stored in the mode object that you created using create_mode. Then, there are corner constraints-for example, set_process_number,  set_voltage or set_timing_derate. These will be stored in the corner object created by create_corner. Finally, there are scenario constraints-for example, set_input_delay, set_driving_cell or set_output_delay. Why is set_input_delay a scenario constraint and not a corner or a mode constraint? Because set_input_delay references a clock and includes a delay, so it has a component of the mode which is the clock and a component of a corner which is the delay. Therefore, it becomes a scenario constraint. There are also some global constraints-for example, set_ideal_network. These don't fall into any of these three categories right here.  
디자인에 제약 조건을 적용할 때, 이러한 제약 조건은 모드, 코너 및 시나리오 내에서 적용됩니다. create_clock 또는 set_case_analysis와 같은 모드 제약 조건이 있다는 사실을 알고 이해하는 것이 좋습니다. 이러한 제약 조건은 모드에 영향을 주며, create_mode를 사용하여 생성한 모드 객체에 저장됩니다. 그런 다음, set_process_number, set_voltage 또는 set_timing_derate와 같은 코너 제약 조건이 있습니다. 이러한 제약 조건은 create_corner로 생성한 코너 객체에 저장됩니다. 마지막으로, set_input_delay, set_driving_cell 또는 set_output_delay와 같은 시나리오 제약 조건이 있습니다. 왜 set_input_delay가 코너나 모드 제약 조건이 아니라 시나리오 제약 조건인가요? 왜냐하면 set_input_delay는 클록을 참조하고 지연을 포함하므로 클록이라는 모드 구성 요소와 지연이라는 코너 구성 요소를 갖기 때문입니다. 따라서 이는 시나리오 제약 조건이 됩니다. 또한 set_ideal_network와 같은 일부 전역 제약 조건도 있습니다. 이들은 여기에서 언급된 세 가지 범주에 속하지 않습니다.  
{: .notice--warning}  

## Scenario Constraints

- PrimeTime, Design Compiler and ICC use only scenarios
  - There is no concept of a mode or a corner
  - Constraints are either scenario-specific or global
  - Scenario constraints are typically in a single file
    - Contains scenario, modal and corner timing constraints
    - For example: Scenario `func_ss_125c`: `func_ss125c.sdc`
- ICC II has modes and corners, which are shareable among scenarios
- It is recommended to split SDC constraints into mode-, corner- and scenario-specific constraint files
  - Improves constraint loading efficiency (explained next), but is not required
- ICC II provides a few commands to simplify the transition to separate scenario/mode/corner files

In Prime Time or Design Compiler or IC compiler tools, there's no concept of modes and corners. There are only scenarios. And so, for those tools, your constraints are either scenario specific or they're global. There's no further subclassification of mode or corner or scenario specific. So typically for these tools, one single large file contains all of the constraints. For example, create_clocks, set_input_delay, set_output_delay, set_process_number, set_voltage etc, all of these constraints, including the global constraints, will be applied in one large file, and you will have one file for each of scenarios. So, for one specific scenario for example func_ss_125, you can have an SDC file that contains all of the constraint's scenario specific plus the global. If you have N scenarios, you will have N different constraint files. ICC II has modes and corners. And these modes in corners are sharable amongst the scenarios. So, one corner file, can be shared across multiple scenarios. Therefore, it is recommended to use scenarios, split up constraints into separate files. one contains the mode specific constraints only. Another one that contains the corner specific constraints. 3rd one that contains the scenario specific ones and then possible 4th one that contains the global constraint. And the reason we recommend that is because this will improve the constraint loading efficiency. The IC Compiler II provides a few commands to simplify the transition to separate scenario/mode/corner files".  
Prime Time, Design Compiler, 및 IC Compiler와 같은 도구에서는 모드와 코너의 개념이 없습니다. 대신, 시나리오만 존재합니다. 이 도구들에서는 제약 조건이 시나리오별로 구체적이거나 전역적입니다. 모드나 코너, 특정 시나리오에 대한 더 구체적인 하위 분류는 존재하지 않습니다. 일반적으로 이러한 도구들에서는 모든 제약 조건이 하나의 큰 파일에 포함됩니다. create_clocks, set_input_delay, set_output_delay, set_process_number, set_voltage 등 모든 제약 조건과 전역 제약 조건이 하나의 큰 파일에 적용되며, 각 시나리오별로 하나의 파일이 생성됩니다. 예를 들어, func_ss_125와 같은 특정 시나리오에 대해 시나리오별 제약 조건과 전역 제약 조건이 모두 포함된 SDC 파일을 사용할 수 있습니다. N개의 시나리오가 있다면 N개의 다른 제약 조건 파일이 생성됩니다. 반면, ICC II는 모드와 코너의 개념을 가지고 있습니다. 이러한 모드와 코너는 여러 시나리오에서 공유될 수 있습니다. 따라서 하나의 코너 파일은 여러 시나리오에서 공유될 수 있습니다. 그러므로, 시나리오를 사용하고, 제약 조건을 별도의 파일로 분할하는 것이 권장됩니다. 모드별 제약 조건만 포함된 파일, 코너별 제약 조건만 포함된 파일, 시나리오별 제약 조건만 포함된 파일, 그리고 전역 제약 조건이 포함된 파일로 제약 조건을 분할합니다. 이렇게 분할함으로써 제약 조건 로딩 효율이 향상됩니다. IC Compiler II는 별도의 시나리오/모드/코너 파일로의 전환을 간편하게 도와주는 몇 가지 명령어를 제공합니다. 이를 통해 제약 조건의 관리를 간소화하고 효율적으로 수행할 수 있습니다.  
{: .notice--warning}  

## Loading Constraints

- For the most efficient scenario setup, you should split constraints into separate files
  - These constraints are then loaded only once, even if shared across scenarios
  - Faster load time with less memory required
- After the modes, corners, and scenarios are defined, populate them with constraints
  - This ensures that no constraints are lost (put into defau/t mode/corner/scenario)

![2023-05-17-loading-constraints.png]({{site.baseurl}}/assets/images/2023-05-17-loading-constraints.png){: .align-center}  

Once you've created the corners, the modes, and scenarios, it is time to apply constraints. For most efficient and correct scenario setup, you should split those constraints into separate files-into a file for mode, into a file for corner, and one for scenario constraints. When you apply these constraints to the corners, the modes, and the scenarios, they are only loaded once even if they're shared across multiple scenarios. Let's consider an example right here. On the left, you see current_scenario M1_C1, and on the right, current_scenario M1_C2. Just looking at the naming, these two scenarios share the same mode, M1. When the constraints on the left is applied here, It reads corner constraints, mode constraints, and scenario constraints. On the right, it only applies corner constraints for C2 and scenario constraints. You don't have to apply the mode constraints again because these were applied already. Obviously, if you have a large number of scenarios and modes and corners, this can really reduce the time involved in applying these constraints.  
코너, 모드, 그리고 시나리오를 생성한 후에는 제약 조건을 적용해야 합니다. 가장 효율적이고 정확한 시나리오 설정을 위해, 제약 조건을 별도의 파일로 분리하여 모드, 코너, 그리고 시나리오 제약 조건을 저장하는 것이 좋습니다. 이러한 제약 조건을 코너, 모드, 그리고 시나리오에 적용할 때, 여러 시나리오 간에 공유되는 경우에도 한 번만 로드됩니다. 이곳에서 예시를 살펴보겠습니다. 왼쪽에는 current_scenario M1_C1가 있고, 오른쪽에는 current_scenario M1_C2가 있습니다. 네이밍을 보면 이 두 시나리오는 같은 모드인 M1을 공유합니다. 왼쪽의 제약 조건이 여기에 적용되면, 코너 제약 조건, 모드 제약 조건, 그리고 시나리오 제약 조건을 읽습니다. 오른쪽에서는 C2의 코너 제약 조건과 시나리오 제약 조건만 적용됩니다. 이미 적용된 모드 제약 조건을 다시 적용할 필요가 없습니다. 분명히, 시나리오, 모드, 코너의 수가 많은 경우 이러한 제약 조건을 적용하는 시간을 크게 줄일 수 있습니다.  
{: .notice--warning}  

## Minimize Modes and Corners with Scenario Analysis

- If you inherit mixed mode, corner and scenario constraints:
  - Create unique modes and corners for each scenario, can remove identical scenarios
  - Execute `remove_duplicate_timing_contexts` to find the minimum set of modes and corners, remove the duplicates and re-assign the scenarios
- Execute `write_script` to capture the mode-, corner-, scenario-constraints

![2023-05-17-minimize-scenario.png]({{site.baseurl}}/assets/images/2023-05-17-minimize-scenario.png){: .align-center}  

Let us see how we can minimize modes and corners with scenario analysis. As you can see in the example, there are 2 scenarios S1 with corner C1 and mode M1 and S2 with corner C2 and mode M2. Now you can execute the command remove_duplicate_timing_contexts." This command will analyze all the modes and corners and scenarios and try to see if there are any repetitions, if there are any that are identical. If there are modes that are identical, the tool will delete the duplicate mode and will make sure that the original scenario points to one of the modes. For example, if the tool finds out that M1 and M2 are identical, it will delete the mode M2 and make sure to reassign scenario S2 to mode M1. After all, the scenarios still need valid modes and corners to operate correctly. Again, after you run remove_duplicate_timing_contexts, you can use write_script to write out the final clean constraints for mode, corner, and scenario.  
시나리오 분석을 통해 모드와 코너를 최소화하는 방법을 살펴보겠습니다. 예시에서 보시다시피, 코너 C1과 모드 M1을 갖는 S1 시나리오가 있고, 코너 C2와 모드 M2를 갖는 S2 시나리오가 있습니다. 이제 "remove_duplicate_timing_contexts" 명령을 실행할 수 있습니다. 이 명령은 모든 모드, 코너 및 시나리오를 분석하고, 중복되는 부분이 있는지 확인합니다. 중복된 모드가 있을 경우, 도구는 중복된 모드를 삭제하고 원래 시나리오가 하나의 모드를 가리키도록 합니다. 예를 들어, 도구가 M1과 M2가 동일하다는 사실을 발견하면, M2 모드를 삭제하고 S2 시나리오를 M1 모드에 재할당합니다. 그래도 시나리오는 올바른 모드와 코너가 필요하므로 제대로 작동하기 위해서입니다. "remove_duplicate_timing_contexts"를 실행한 후에는 write_script를 사용하여 최종적으로 정리된 모드, 코너 및 시나리오의 제약 조건을 작성할 수 있습니다.  
{: .notice--warning}  

## Check Timing Constraints

- The following commands are available for checking constraints:
  - **`check_timing`** to check clock crossing, missing input/output delays, ...
  - **`report_exceptions`** to identify paths with single-cycle timing exceptions: false paths, multi-cycle paths, asynchronous min- or max-delay paths
  - **`report_case_analysis`** to confirm the correct mode settings
  - **`report_disable_timing`** to identify paths with disabled timing arcs (these will not be optimized for timing)

```tcl
check_timing
foreach_in_collection mode [all_modes] {
  current_mode $mode
  report_exceptions
  report_case_analysis
  report_disable_timing
}
```

After you have applied all your scenarios and modes and corners correctly, there are ways in IC Compiler II to check whether you have constrained everything in your design, whether your modes actually constrain the design entirely, or whether there are still clocks that should be defined, registers that are unclocked, etc. Of course, the best way to check constraint completeness and correctness is to use Galaxy Constraint Analyzer. That is a very complete tool that can really have a deep look at your constraints and provide a lot of assistance for correcting constraints. In ICC II, there are some commands that can report problems-check_timing can check clock crossings, can check missing input or output delays; `report_exceptions` can identify paths with single-cycle timing exceptions like false paths, multi-cycle paths, asynchronous min- or max-delay paths; `report_case_analysis` can show you all the mode settings, so you'll have to confirm yourself whether these are correct or not. Finally, `report_disable_timing` identifies paths with disabled timing arcs. Notice, down here, how to run these commands. check_timing will automatically run through all your scenarios. You don't have to change the current mode or anything. The other commands, `report_exceptions`, `case_analysis`, and `disable_timing`, all operate on the current mode, so you need to make sure to iterate through all modes. Here you see another thing in ICC II, that the modes are all collection objects. When you iterate through [all_modes], you need to use the function `foreach_in_collection`.  
모든 시나리오, 모드, 코너를 올바르게 적용한 후에는 IC Compiler II에서 설계의 제약 조건이 모두 적용되었는지, 모드가 실제로 설계를 완전히 제약하는지, 정의되어야 할 클록이 여전히 존재하는지, 클록이 정의되지 않은 레지스터 등이 있는지 확인할 수 있는 방법이 있습니다. 제약 조건의 완전성과 정확성을 확인하는 가장 좋은 방법은 Galaxy Constraint Analyzer를 사용하는 것입니다. 이는 제약 조건을 깊이 있게 살펴보고 제약 조건을 수정하는 데 많은 도움을 제공하는 매우 전문적인 도구입니다. ICC II에서는 몇 가지 명령어를 사용하여 문제를 보고할 수 있습니다. check_timing은 클록 크로싱을 확인하고 입력 또는 출력 딜레이가 누락된 부분을 확인할 수 있습니다. `report_exceptions`는 거짓 경로, 다중 사이클 경로, 비동기 최소 또는 최대 딜레이 경로와 같은 단일 사이클 타이밍 예외를 식별할 수 있습니다. `report_case_analysis`는 모드 설정을 모두 보여주므로 직접 확인하여 올바른지 여부를 판단해야 합니다. 마지막으로, `report_disable_timing`은 비활성화된 타이밍 아크가 있는 경로를 식별합니다. 아래에서 이러한 명령어를 실행하는 방법을 볼 수 있습니다. `check_timing`은 자동으로 모든 시나리오를 실행합니다. 현재 모드를 변경하거나 다른 작업을 수행할 필요가 없습니다. `report_exceptions`, `case_analysis` 및 `disable_timing`은 모두 현재 모드에서 동작하므로 모든 모드를 반복적으로 실행해야 합니다. 또한 ICC II에서 모드는 모두 컬렉션 객체입니다. [all_modes]를 반복하는 경우 `foreach_in_collection` 함수를 사용해야 합니다.  
{: .notice--warning}  

## Reports Usually Apply to Current Scenario

- Most `report_` commands apply to the current scenario
- Some commands have `-scenario`, `-corner`, and/or `-mode` options to modify what is reported

When you run any reports in IC Compiler II, typically they apply to the current scenario. Here is just a quick example. If you look at all scenarios, the current scenario is set to func.ss_125c. report_case_analysis will report based on the current scenario. To be more precise, it's actually reporting information for the current mode. The current mode in this case is going to be func. report_constraints -all_violators reports violations based on the current scenario shown here: func.ss_125c. If the current scenario is changed, report_case_analysis will give different numbers obviously. The mode is changed. And report_constraints -all_violators also reports what's active in the current scenario. There are a lot of commands that do have -scenario, -corner, or -mode switches which allow you to select a scenario, a mode, or a corner.  
IC Compiler II에서 보고서를 실행할 때 일반적으로 현재 시나리오에 적용됩니다. 간단한 예를 살펴보겠습니다. 모든 시나리오를 살펴보면, 현재 시나리오가 func.ss_125c로 설정되어 있습니다. report_case_analysis는 현재 시나리오를 기준으로 보고서를 작성합니다. 보다 정확하게 말하면, 실제로는 현재 모드에 대한 정보를 보고합니다. 이 경우 현재 모드는 func가 될 것입니다. report_constraints -all_violators는 현재 시나리오인 func.ss_125c에 기반하여 위반 사항을 보고합니다. 현재 시나리오가 변경되면 report_case_analysis는 당연히 다른 결과를 제공합니다. 모드가 변경되었고, report_constraints -all_violators도 현재 시나리오에서 활성화된 내용을 보고합니다. -scenario, -corner 또는 -mode 스위치가 있는 많은 명령어가 있으며, 이를 사용하여 특정 시나리오, 모드 또는 코너를 선택할 수 있습니다.  
{: .notice--warning}  

## Controlling MCMM Timing Reports

```tcl
# Report the single worst setup timing path among all
# path groups of all active setup-enabled scenarios:
report_timing

# Report the single worst setup timing path from among 
# the listed and active corners, modes or scenarios, respectively:
report_timing -corners "C1 C4"
report_timing -modes FUNC*
report_timing -scenarios "S1 S2 S5 S6"

# Report the worst setup timing path for each and every 
# active corner, mode, scenario or path group respectively:
report_timing -report_by corner|mode|scenario|group

# Report the worst setup timing path for each listed and 
# active corner, mode or scenario, respectively:
report_timing -report_by corner -corners "C1 C4"
report_timing -report_by mode -modes FUNC*
report_timing -report_by scenario -scenarios "S1 S2 S5 S6"
```

The command `report_timing` will report the single worst setup timing path among all path groups of all active setup enabled scenarios. If you want to report the worst setup timing path from the listed and active corners, modes or scenarios, use `-corners`, `-modes`, or `-scenarios` respectively. By using the command `report_timing -report_by corner|mode|scenario|group`, use can report the worst setup timing path for each and every active corner, mode, scenario or path group respectively. Report the worst setup timing path for each listed and active corner, mode or scenario, respectively by using the command `report_timing` with option `-report_by` shown here.  
`report_timing` 명령은 모든 활성 setup이 가능한 시나리오의 모든 경로 그룹 중에서 가장 나쁜 설정 타이밍 경로를 보고합니다. 나열된 활성 코너, 모드 또는 시나리오에서 가장 나쁜 설정 타이밍 경로를 보고하려면 각각 `-corners`, `-modes` 또는 `-scenarios`를 사용하십시오. `report_timing -report_by corner|mode|scenario|group` 명령을 사용하여 각 활성 코너, 모드, 시나리오 또는 경로 그룹에 대한 가장 나쁜 설정 타이밍 경로를 보고할 수 있습니다. 이 명령은 나열된 활성 코너, 모드 또는 시나리오에 대한 가장 나쁜 설정 타이밍 경로를 각각 보고하기 위해 `report_timing` 명령과 `-report_by` 옵션을 사용합니다.  
{: .notice--warning}  

## Conrtolling Scenario Analysis / Optimization

- The `create_scenario` command creates a scenario, makes it active, and enables the following analysis types:
  - Setup and hold timing
  - Leakage and dynamic power
  - Max transition, max and min capacitance
- Placement, CTS and routing optimizations occur simultaneously on all active scenarios, for their enabled analyses
- Use `set_scenario_status` to limit the analysis types or to make scenarios inactive before compile
  - See examples on next page

As we discussed before scenarios are created by using the command `create_scenario`, but it also makes that scenario active and enables all analysis types in that scenario as shown here". This needs to be remembered: that all analysis types are enabled by default for all scenarios created. Placement, CTS, routing optimization: they occur simultaneously on all active scenarios for their enabled analyses. The `set_scenario_status` command is used after a scenario is created to set the desired analysis types. You create the scenario using `create_scenario`. Then you modify it with `set_scenario_status`. Let's look at a couple of examples.  
이전에 논의한 대로, 시나리오는 `create_scenario` 명령을 사용하여 생성됩니다. 그러나 이 명령은 해당 시나리오를 활성화하고 해당 시나리오의 모든 분석 유형을 활성화합니다. 기억해야 할 사항은 생성된 모든 시나리오에 대해 기본적으로 모든 분석 유형이 활성화되어 있다는 것입니다. 배치, CTS, 라우팅 최적화는 활성화된 분석에 대해 모든 활성 시나리오에서 동시에 수행됩니다. `set_scenario_status` 명령은 시나리오 생성 후 원하는 분석 유형을 설정하는 데 사용됩니다. `create_scenario`를 사용하여 시나리오를 생성한 다음 `set_scenario_status`를 사용하여 수정합니다. 몇 가지 예를 살펴보겠습니다.  
{: .notice--warning}  

## Examples: Modifying Scenario Status

```tcl
create_scenario -mode mFUNC -corner cSLOW
create_scenario -mode mTEST -corner cSLOW
create_scenario -mode mFUNC -corner cFAST
create_scenario -mode mTEST -corner cFAST

# Disable setup timing analysis and optimization for the FAST corner scenarios;
# Disable leakage power analysis and optimization for the TEST mode scenarios:
set_scenario_status *cFAST -setup false

set_scenario_status mTEST* -leakage_power false

# For placement consider only SLOW corner scenarios:
set_scenario_status *cFAST -active false
place_opt

# For post-route analysis activate all scenarios and enable all analyses:
set_scenario_status * -active true -all
```

Here in this example, four scenarios are created. You see here those are mode corner combinations. You also notice that a -name option is not used for create_scenario. If the name is not provided, then by default create_scenario will assign a word composed of the mode and the corner, so this scenario will be called mFUNC::cSLOW. Here in this example, `set_scenario_status` is used to turn off setup analysis for fast scenarios. Also,  leakage power optimization is turned off in test mode related scenarios. For `place_opt`, all fast scenarios are deactivated. If you want to focus on the slow corners because during `place_opt` you don't care for hold. For post-route analysis here, activate all scenarios and all analyses. Specify `set_scenario_status`. This star `*` will match all scenarios, `-active true` activates all scenarios, and `-all` enables all analyses. Finally, an extensive example here, `set_scenario_status`, fast scenarios, activate all analyses, but then turn off max transition and max capacitance.  
이 예시에서는 네 개의 시나리오가 생성됩니다. 여기서는 모드와 코너의 조합입니다. 또한, create_scenario에 -name 옵션이 사용되지 않았음을 알 수 있습니다. 이름이 제공되지 않은 경우, create_scenario는 기본적으로 모드와 코너로 구성된 단어를 할당합니다. 따라서 이 시나리오는 mFUNC::cSLOW로 불릴 것입니다. 이 예시에서는 `set_scenario_status`를 사용하여 빠른 시나리오에서 setup 분석을 비활성화합니다. 또한, 테스트 모드 관련 시나리오에서는 leakage power 최적화를 비활성화합니다. `place_opt`의 경우, 모든 빠른 시나리오를 비활성화합니다. hold에 대해서는 신경 쓰지 않기 때문에 느린 코너에 초점을 맞추려는 것입니다. 여기서 후처리 분석을 위해 모든 시나리오와 분석을 활성화합니다. `set_scenario_status`를 지정합니다. 별표 `*`는 모든 시나리오와 일치하며, `-active true`는 모든 시나리오를 활성화하고, `-all`은 모든 분석을 활성화합니다. 마지막으로, `set_scenario_status`, 빠른 시나리오를 활성화하고 모든 분석을 활성화한 후, max transition과 max capacitance를 비활성화하는 방법을 자세히 살펴보겠습니다.  
{: .notice--warning}  

## Reporting Scenario Status Settings

![2023-05-17-report-scenario-status-settings.png]({{site.baseurl}}/assets/images/2023-05-17-report-scenario-status-settings.png){: .align-center}  

You can use report_scenarios to get a very nice summary of the settings of all your scenarios and whether these scenarios are active or not. Here you see the name of the scenarios, modes, and corners; whether they're active; and then finally whether the analysis types are true or false; for setup, hold, leakage, and dynamic power, max_tran, max_cap, and min_cap. You can also use get_scenarios with a -filter switch to find certain scenarios. For example, here if you want to have all scenarios that are active and used for setup timing analysis. Here scenarios are filtered for active and hold. I can use -filter active, max_tran, max_cap, and min_cap. This is a very nice way in scripts to verify that the correct scenarios are active.  
report_scenarios를 사용하여 모든 시나리오의 설정 및 해당 시나리오가 활성화되어 있는지 여부에 대한 아주 좋은 요약 정보를 얻을 수 있습니다. 여기에서는 시나리오, 모드, 코너의 이름을 볼 수 있으며, 해당 시나리오가 활성화되어 있는지 여부와 설정된 분석 유형(setup, hold, leakage, dynamic power, max_tran, max_cap, min_cap)에 대한 정보도 확인할 수 있습니다. get_scenarios를 -filter 스위치와 함께 사용하여 특정 시나리오를 찾을 수도 있습니다. 예를 들어, 여기에서는 setup 타이밍 분석에 사용되는 모든 활성 시나리오를 찾으려면 -filter active, max_tran, max_cap, min_cap을 사용할 수 있습니다. 이는 스크립트에서 올바른 시나리오가 활성화되어 있는지 확인하는 아주 좋은 방법입니다.  
{: .notice--warning}  

## Defining Corner PVT by Operating Condition

- ICC II accepts set_operating_conditions to determine the PVT values for each corner

```tcl
set_operating_conditions Worst -library saed32lvt_ss0p95v125c.db
```

- This is an indirect method, relying on an operating condition model name and library to determine the PVT numbers

Now that we've talked about the mechanics of setting up scenarios or using scenarios or activating scenarios, Let's talk a little bit about defining the corner PVT. ICC II will accept a set_operating_conditions command to determine the PVT for each corner. As in Design Compiler NXT, use set_operating_conditions followed by the name of the operating condition. This operating condition name is in a table in your .dbs. Here are the operating conditions. In order to find the actual PVT numbers-which are process, temperature, and voltage-we have to find the name and then look up the information from that table. In order to find other libraries that correspond to that same PVT, use the numbers to look up the nominal PVT of other libraries. That's a different table that's in the .db. It says "nominal PVTs," and that contains the values for which the library is characterized. Essentially, look up the name. Through the name, find the numbers. By using the numbers, find the other libraries that match this corner's specification. This is an indirect method because you're relying on a name to find the numbers.  
시나리오를 설정하거나 사용하거나 활성화하는 방법에 대해 이야기했으므로, 이제 코너 PVT를 정의하는 것에 대해 약간 더 이야기해 보겠습니다. ICC II는 set_operating_conditions 명령을 사용하여 각 코너의 PVT를 결정할 수 있습니다. Design Compiler NXT와 마찬가지로 set_operating_conditions 다음에 운영 조건의 이름을 사용합니다. 이 운영 조건 이름은 .dbs 파일의 테이블에 있습니다. 여기에 운영 조건이 있습니다. 실제 PVT 숫자를 찾기 위해서는-이는 공정, 온도 및 전압을 나타내는 숫자를 찾기 위해서는-해당 테이블에서 정보를 찾아야 합니다. 해당 동일한 PVT에 해당하는 다른 라이브러리를 찾으려면 이 숫자를 사용하여 다른 라이브러리의 정상 PVT를 찾습니다. 이는 .db에 있는 다른 테이블인 "nominal PVTs"라는 테이블입니다. 해당 테이블에는 라이브러리가 특성화된 값을 포함하고 있습니다. 기본적으로 이름을 찾습니다. 이름을 통해 숫자를 찾습니다. 숫자를 사용하여 이 코너의 사양과 일치하는 다른 라이브러리를 찾습니다. 이는 이름을 사용하여 숫자를 찾는 간접적인 방법입니다.  
{: .notice--warning}  

## Defining PVT Directly - Recommended

- It is highly recommended to use the direct method for clarity and ease of use:

```tcl
current_corner ss125c
set_process_number 0.99
set_voltage 0.75
set_voltage 0.95 -object_list VDDH
set_temperature 125
```

It is highly recommended to use the direct method in IC Compiler II. You can specify the process voltage and temperature numbers directly, and it's using these commands as shown here: `set_process_number`, `set_voltage`, and `set_temperature`. `set_voltage`. The `set_process_number` and `set_temperature`, these are added in ICC II. By using it, you can pick the exact PVT from the list.  
IC Compiler II에서는 직접적인 방법을 사용하는 것이 권장됩니다. 직접적으로 공정, 전압 및 온도 값을 지정할 수 있으며, 다음과 같은 명령을 사용합니다: `set_process_number`, `set_voltage` 및 `set_temperature`입니다. `set_voltage`. `set_process_number` 및 `set_temperature`는 ICC II에서 추가된 기능입니다. 이를 사용하여 목록에서 정확한 PVT를 선택할 수 있습니다.  
{: .notice--warning}  

## Reminder: NDM Cell Library Panes

![2023-05-17-ndm-cell-lib-panes.png]({{site.baseurl}}/assets/images/2023-05-17-ndm-cell-lib-panes.png){: .align-center}  

- The CLIB merges logic and physical models
  - Each `.db` characterized for a PVT corner is represented as a “pane”

You are familiar with this NDM (New Data Model) cell library concept. This slide is remainder of the same concept which is discussed in the Module 4 NDM Cell libraries. Here the diagram shows the process involved in the creation of CLIBs/NDMs by the Library manager. It will take tech file, .db files which are characterized for each PVT corners and .frame view files as input, and produces CLIBs. These CLIBs will merges logic and physical models together. As mentioned before,  each .db characterized for a PVT corner is represented as a "pane"  
NDM (New Data Model) 셀 라이브러리 개념에 대해 알고 계십니다. 이 슬라이드는 모듈 4 NDM 셀 라이브러리에서 논의된 내용의 나머지입니다. 여기에서 다이어그램은 라이브러리 매니저가 CLIBs/NDM을 생성하는 과정을 보여줍니다. 이 과정에서는 기술 파일, 각 PVT 코너에 대해 특성화된 .db 파일, .frame 뷰 파일이 입력으로 사용되며, CLIBs가 생성됩니다. 이러한 CLIBs는 논리 및 물리 모델을 함께 병합합니다. 이전에 언급한 대로, 각 PVT 코너에 대해 특성화된 각 .db 파일은 "패널"로 표현됩니다.  
{: .notice--warning}  

## Reporting Library Details: `report_lib`

![2023-05-17-report-lib.png]({{site.baseurl}}/assets/images/2023-05-17-report-lib.png){: .align-center}  

There is a report_lib command that can provide a lot of information about your reference libraries. The report_lib command, displays the process number, different voltages for the rails, the temperature, and a lot of other information-for example, the source .db file. The tool displays where the timing information came from, which .db file was used to create the NDM.  
report_lib 명령은 참조 라이브러리에 대한 다양한 정보를 제공할 수 있습니다. report_lib 명령을 사용하면 공정 번호, 레일의 다른 전압, 온도 및 다른 많은 정보(예: 소스 .db 파일)를 표시할 수 있습니다. 이 도구는 타이밍 정보가 어디에서 나왔는지, NDM을 생성하는 데 어떤 .db 파일이 사용되었는지를 표시합니다.  
{: .notice--warning}  

## VT Matching

- If the PVT does not match, the VT resolution function finds the closest VT match between specified and available library panes
  - As a result, if there is at least one timing model available for a cell instance, whatever the nominal PVT, that cell instance will be timed

![2023-05-17-vt-matching.png]({{site.baseurl}}/assets/images/2023-05-17-vt-matching.png){: .align-center}  

Let's move on. Now that we know how to create corners, now that we know how to use the direct method-set_voltage, _temperature, and _process_number-what does IC Compiler II do if a correct match is not found? This is a fundamental behavior change with regards to other tools. IC Compiler II will always find a library and match to it even if the numbers don't line up. Let's look at this example right here. We have four libraries. Or you can say we have four panes. This is what it's called in IC Compiler II NDM terminology. We have four panes that specify four different numbers here. Voltage, temperature, and process numbers are listed. set up four corners: a, b, c, and d. They're shown here. Now these circles right here, they signify the corner definition, and the colored circles correspond to the actual libraries listed on the left. If look at corner a definition, this has a voltage of 1.1, temperature 125, and process number of 1. By specifying these corner conditions, if look over to timing panes, there is a perfect match. That's why you see the corner a circle right on top of the purple circle right here. This is a perfect match. What about corner b? If we look at the voltage and the temperature and try to find these values in that combination over here, we're not going to be successful. In this case, IC Compiler II will find a closest VT match. Voltage goes along the Y-axis. Temperature goes the along the X-axis right here. Essentially, what the tool does is it will plot this number in the graph and find the closest X-Y match, and that happens to be this yellow library right here. The temperature is 0 and voltage 1.1. The tool will continue. There is no scaling that is performed. VT matching means that it will pick this library, and all timing will happen using these numbers of this library. If you have enabled scaling groups in Library Manager, in that case, matching doesn't occur because the libraries are enabled for scaling, so you will always get a number that is scaled to the specific voltage, temperature values you specify when you define your corner. This is talking about matching. This is not when using scaling. Scaling is one thing; matching is a different thing. Here in corner c, we have the same situation. Again, I have a voltage that doesn't exist. The tool will find the closest match. This last example here is interesting. Here we have a match with the voltage and temperature. But we don't have the match with the process number. The tool will still find the match. The process number is essentially ignored, and it will pick a library that covers the voltage and the temperature. Now all this happens. The tool will not error out. If you don't want this behavior-if you say that, no, and you want the tool to abort if there are any mismatches in the corner definitions as compared to the libraries available-then all you have to do is use the command set_message_info, and you configure that to stop on the warning message called PVT-030. This is VT matching.  
이제 코너를 생성하는 방법을 알았으므로, 직접 방법(set_voltage, set_temperature 및 set_process_number)을 사용할 때 IC Compiler II가 정확한 일치를 찾지 못한 경우에는 어떻게 동작하는지 알아보겠습니다. 이는 다른 도구와 비교하여 기본 동작 변경입니다. IC Compiler II는 숫자가 일치하지 않더라도 항상 라이브러리를 찾아서 일치시킵니다. 이 예제를 살펴봅시다. 우리는 네 개의 라이브러리 또는 네 개의 패널이 있다고 할 수 있습니다. 이것은 IC Compiler II의 NDM 용어로 불립니다. 여기에는 전압, 온도 및 공정 번호가 나열되어 있습니다. a, b, c 및 d 네 개의 코너를 설정합니다. 코너 정의를 나타내는 이 원들은 여기에 표시됩니다. 색칠된 원들은 실제로 왼쪽에 나열된 라이브러리와 대응됩니다. 코너 a 정의를 살펴보면 전압이 1.1, 온도가 125, 공정 번호가 1인 것을 알 수 있습니다. 이 코너 조건을 지정하면 타이밍 패널에 완벽한 일치가 있습니다. 따라서 여기에서 보라색 원 위에 코너 a 원이 정확하게 위치해 있습니다. 이것은 완벽한 일치입니다. 그렇다면 코너 b는 어떨까요? 전압과 온도를 살펴보고 이 조합에서 해당 값을 찾아보면 성공하지 못할 것입니다. 이 경우 IC Compiler II는 가장 가까운 VT 일치를 찾을 것입니다. 전압은 Y축을 따라 온도는 X축을 따라 이 숫자를 그래프에 표시하고, 가장 가까운 X-Y 일치를 찾습니다. 이 경우에는 노란색 라이브러리입니다. 온도는 0이고 전압은 1.1입니다. 도구는 계속 진행합니다. 스케일링은 수행되지 않습니다. VT 일치는 이 라이브러리를 선택하고 모든 타이밍은 이 라이브러리의 숫자를 사용하여 수행됩니다. 라이브러리 매니저에서 스케일링 그룹을 사용하도록 설정한 경우, 일치가 발생하지 않습니다. 라이브러리가 스케일링에 대해 활성화되었기 때문에 지정한 전압, 온도 값에 맞추어 스케일된 숫자를 얻게 됩니다. 이것은 일치가 아닌 매칭에 대한 이야기입니다. 매칭은 일이고, 스케일링은 다른 일입니다. 여기에서 코너 c의 경우도 동일합니다. 다시 한번 일치하는 전압이 없습니다. 마지막 예제는 흥미롭습니다. 여기서는 전압과 온도와 일치하지만 공정 번호가 일치하지 않습니다. 도구는 여전히 일치를 찾습니다. 공정 번호는 사실상 무시되며, 전압과 온도를 포함하는 라이브러리를 선택합니다. 이 모든 과정은 도구에서 오류가 발생하지 않습니다. 이 동작을 원하지 않는다면, 코너 정의와 라이브러리의 일치 여부에 관계없이 도구가 중단되도록 하려면 set_message_info 명령을 사용하여 경고 메시지 PVT-030을 중지하도록 설정하면 됩니다. 이것이 VT 매칭입니다.
{: .notice--warning}  

## Use Process Labels to Help the Matching

- Use process labels to augment the PVT matching,
  - for example when:
    - Slow (ss) and fast (ff) process corners have the same P values

![2023-05-17-process-labels-help-matching.png]({{site.baseurl}}/assets/images/2023-05-17-process-labels-help-matching.png){: .align-center}  

- Closest-match does not pick the pane that you want (example shown next)

Let's look at another example right here. What if there are two libraries or two timing panes that have the same process, temperature, and voltage? This is something that one encounters pretty often, and notice here that one is the fast library and one is the slow library. This can be due to differently characterized extractions for the nets, for example, or for the metals inside the cell. It cannot be differentiated between these two by just using process, temperature, and voltage.  
이 경우, 두 개의 라이브러리나 타이밍 패인이 같은 공정, 온도, 전압을 갖는 경우를 살펴보겠습니다. 이는 종종 발생하는 상황이며, 여기에서 하나는 빠른 라이브러리이고 다른 하나는 느린 라이브러리입니다. 이는 예를 들어 넷의 추출 방법이나 셀 내부의 금속에 대한 서로 다른 특성으로 인해 발생할 수 있습니다. 프로세스, 온도, 전압만 사용하여 이 둘을 구별할 수 없습니다. 이러한 경우, 라이브러리를 구별하기 위해 추가적인 정보나 제약 조건이 필요합니다. 이는 다른 명명 규칙을 사용하거나 고유한 라이브러리를 식별하기 위해 추가적인 매개변수를 지정하는 것을 포함할 수 있습니다.  
{: .notice--warning}  

## Process Label Specification

- Process labels are defined
  - In liberty lib using process_label within the operating_conditions definition
  - Using the set_process_label command in Library Compiler
  - Using the lib.configuration.process_label_mapping application option when using library configuration
    - See notes for details
- Next, process labels are specified as part of the design’s corner constraints

```tcl
create_corner c_slow
set_process_number -corner c_slow 1.0
set_process_label -corner c_slow slow
```

- The process label, when specified, supersedes the process number, and limits VT matching to only the set of panes with the specified label

----------

Notes:  

![2023-05-17-process-labels-help-matching-note.png]({{site.baseurl}}/assets/images/2023-05-17-process-labels-help-matching-note.png){: .align-center}  

Process labels can be defined in 3 ways shown here. The first way is, process labels are part of your library using process_label within the operating condition definition.Next method is they can be set by using set_process_label command  in  library compiler or library manager tool. Finally, if you are using the library configuration, you can use app option lib.configuration.process_label_mapping to apply certain process labels to certain libraries. Notice that process labels are specified as part of the design's corner constraints. For example, once you have created the corner c_slow, you will provide the process number for the created corner. Next, you will create a process label wrt the corner in the form string, here you have defined a string slow. Process label becomes a 4th piece of information to differentiate between dbs. When you specified the process label it supersedes the process number, and limits VT matching to only the set of panes with the specified label.  
프로세스 레이블은 여러 가지 방법으로 정의할 수 있습니다. 첫 번째 방법은 프로세스 레이블을 라이브러리의 일부로 정의하는 것입니다. 이는 운영 조건 정의 내에서 process_label을 사용하여 수행할 수 있습니다. 두 번째 방법은 library compiler 또는 library manager 도구에서 set_process_label 명령을 사용하여 설정할 수 있습니다. 마지막으로, 라이브러리 구성을 사용하는 경우 lib.configuration.process_label_mapping 애플리케이션 옵션을 사용하여 특정 프로세스 레이블을 특정 라이브러리에 적용할 수 있습니다. 프로세스 레이블은 디자인의 코너 제약 조건의 일부로 지정됩니다. 예를 들어, 코너 c_slow를 생성한 후, 생성된 코너에 대해 프로세스 번호를 지정할 것입니다. 다음으로, 문자열 형식으로 프로세스 레이블을 생성합니다. 여기에서는 "slow"라는 문자열을 정의했습니다. 프로세스 레이블은 dbs를 구분하는 4번째 정보가 됩니다. 프로세스 레이블을 지정하면 프로세스 번호를 무시하고, 지정된 레이블이 있는 패인 집합에 대해서만 VT 매칭이 제한됩니다.  
{: .notice--warning}  

## Process Label Exercise

![2023-05-17-process-labels-exercise.png]({{site.baseurl}}/assets/images/2023-05-17-process-labels-exercise.png){: .align-center}  

```tcl
create_corner c_slow
set_process_number 1.0
set_voltage -object VDDG 0.9
set_temperature 125
create_scenario -name s_func_slow -mode m_func -corner c_slow
```

Q. Scenario `s_func_slow` will be used for setup optimization: Which pane is selected?  

```tcl
## add a process label
set_process_label -corner c_slow slow
```

Q. Which pane is selected now?  

Let's look at this example. Consider the table at the top, it has five timing panes numbered from 0 to 4. Look at the process number. It's all 1.0. In the other column process labels are defined, and they're slow, fast, slow, slow, and fast. Then it has voltages and temperatures. Again, temperature is the same, so it doesn't matter for our exercise right here. Table also displays the names of the Liberty files. Here lets create a corner c_slow with a process number and voltage of 0.9, and use create_scenario -name s_func_slow -mode m_func -corner c_slow. Scenario s_func_slow is going to be used for setup optimization. Which pane is going to be selected? Among the five panes, which pane is going to be selected for func_slow? In order to answer this question, all we have to do is look at the process number, voltage, and temperatures. If we look at the process numbers and Temperature are same so it doesn't matter. Lets look at the voltage? Do I have a voltage of 0.9? So the pane number 1 is a perfect match. Check the name of the scenario it is a slow scenario, and in the table it used a fast library. In order to make sure the selection of the right pane, consider some other information. Here now it uses set_process_label -corner c_slow slow. Now which pane is going to be selected? We look at the same numbers here. We said that 0.9 was the perfect match, so now that the slow label overrides, we have to look at the slow panes. This is 0, 2, and 3. Among these, the closest match is pane number 0 right here with a voltage of 0.88, which is closest to 0.9. Ultimately, pane 0 is going to be selected, and that is probably the correct choice since this is a slow library for a slow scenario.  
이 예시를 살펴봅시다. 상단의 표는 0부터 4까지 번호가 매겨진 다섯 개의 타이밍 패인을 가지고 있습니다. 프로세스 번호를 살펴보면 모두 1.0입니다. 다른 열에는 프로세스 레이블이 정의되어 있으며, slow, fast, slow, slow, fast로 나열되어 있습니다. 그리고 전압과 온도가 표시되어 있습니다. 다시 한 번 온도는 동일하므로 이 연습에는 중요하지 않습니다. 표에는 Liberty 파일의 이름도 표시되어 있습니다. 이제 프로세스 번호와 전압이 0.9인 코너 c_slow를 생성하고, create_scenario -name s_func_slow -mode m_func -corner c_slow를 사용하여 scenario s_func_slow를 생성해 봅시다. s_func_slow 시나리오는 setup 최적화에 사용될 것입니다. func_slow에는 5개의 패인 중 어떤 패인이 선택될까요? 이 질문에 답하기 위해서는 프로세스 번호, 전압 및 온도를 살펴보면 됩니다. 프로세스 번호와 온도를 살펴보면 동일하므로 중요하지 않습니다. 전압을 살펴봅시다. 0.9의 전압이 있나요? 그렇다면 패인 번호 1이 완벽한 매치입니다. 시나리오의 이름을 살펴보면 slow 시나리오이지만, 표에서는 fast 라이브러리를 사용합니다. 올바른 패인을 선택하기 위해 다른 정보도 고려해야 합니다. 이제 set_process_label -corner c_slow slow을 사용합니다. 이제 어떤 패인이 선택될까요? 동일한 숫자들을 살펴봅니다. 0.9가 완벽한 매치라고 했으므로 이제 slow 레이블을 우선시하여 slow 패인을 살펴보아야 합니다. 이는 0, 2, 3입니다. 이 중에서 가장 가까운 매치는 전압이 0.9에 가장 가까운 패인 번호 0입니다. 최종적으로 패인 0이 선택될 것이며, 이는 올바른 선택일 것입니다. 왜냐하면 이는 slow 시나리오를 위한 slow 라이브러리이기 때문입니다.  
{: .notice--warning}  

## Reporting ALL Corners with PVT Mismatches

![2023-05-17-reporting-all-corners.png]({{site.baseurl}}/assets/images/2023-05-17-reporting-all-corners.png){: .align-center}  

For example, if you have 6 timing panes, 2 of them have process label slow and 4 of them process label fast. VT resolution is only applied to the 2 slow timing panes, since the process label which has selected here in the example was slow. The other 4 timing panes with  process label fast will be ignored during the VT matching process. The best way to validate the corner setup is report_pvt. This report will show you any mismatches that occur for process, that is for temperature and for voltage. In this example, as you can see there are 4 voltage and 12 temperature mismatches. Later in the detailed section you will have the entries. This is the entry 15 which is shown here. Here you can look for `*`. The line with the `*` will highlight the mismatches. In this case the user has specified the temp is 95 degrees, but the closest effective temp which means the library temp that was matched is 125 degrees. IC Compiler 2 could not find the library with the temp 95 and therefore you have this temp mismatch.  
예를 들어, 타이밍 패인이 6개 있고 그 중 2개의 프로세스 레이블이 slow이고 4개의 프로세스 레이블이 fast인 경우, VT 해상도는 예제에서 선택된 slow 프로세스 레이블에 대해서만 적용됩니다. 다른 4개의 fast 프로세스 레이블을 가진 타이밍 패인은 VT 매칭 과정에서 무시됩니다. 코너 설정을 검증하는 가장 좋은 방법은 report_pvt를 사용하는 것입니다. 이 보고서는 프로세스, 즉 온도와 전압에 대한 일치하지 않는 사항을 보여줍니다. 이 예제에서는 전압에 대한 4개의 불일치와 온도에 대한 12개의 불일치가 있음을 확인할 수 있습니다. 이후에는 세부 항목이 표시됩니다. 여기에 표시된 항목 15를 살펴보면, `*`을 찾을 수 있습니다. `*`가 있는 줄은 불일치를 강조합니다. 이 경우, 사용자가 온도를 95도로 지정했지만, 일치된 라이브러리 온도인 효과적인 온도는 125도입니다. IC Compiler II는 온도가 95인 라이브러리를 찾을 수 없었으므로 온도 불일치가 발생한 것입니다.  
{: .notice--warning}  

## Selective Library Loading

To get started, let's discuss the topic of selective library loading. In unit 4, which discusses NDM cell library creation, libraries are no longer a single monolithic file, but instead a directory that contains files for each pane, as well as for the physical library. In the past, when there was only a single NDM library, all timing panes were always loaded into memory, which can use up a lot of memory if you have a high pane count. IC Compiler 2 one can now be configured to load only the required panes. This in turn means that you can create one-size-fits-all cell libraries, meaning you can read in all your timing DBs without worrying about what you will be using in the project or not.  
시작하기 전에 선택적 라이브러리 로딩에 대해 이야기해 보겠습니다. NDM 셀 라이브러리 생성을 다루는 4단원에서는 라이브러리가 더 이상 단일 단일 파일이 아니라 각 패인을 위한 파일과 물리 라이브러리를 포함한 디렉토리로 구성되는 것을 설명하고 있습니다. 예전에는 단일 NDM 라이브러리만 있을 때 모든 타이밍 패인이 항상 메모리에 로드되었으며, 이는 패인 수가 많은 경우 많은 메모리를 사용할 수 있습니다. IC Compiler II에서는 필요한 패인만 로드할 수 있도록 구성할 수 있습니다. 이는 한 번에 모든 타이밍 DB를 읽어들여 프로젝트에서 사용할지 여부에 대해 걱정하지 않고도 일괄적으로 셀 라이브러리를 생성할 수 있다는 것을 의미합니다.  
{: .notice--warning}  

## Using Selective Library Loading

```tcl
# Configure PVTs before you open or create your design library
set_pvt_configuration -voltages {0.75 0.95} -temperatures {-40 125} \
  -process_numbers 1.0 -process_labels {slow fast}
create lib -ref_libs {all_panes.ndm ...} ...
read_verilog ...
# Flow continues as usual. Library in ICC II appears to only contain the
# timing panes that match above PVT configuration
```

In order to use selective loading, you're going to be using the command set_pvt_configuration. If you remember, that is exactly the same command you used in Library Manager to select the DBs that should go into the NDM cell library. Let's have a look at a small example here to explain how this really works. Assume that you have an NDM cell library with 132 panes. Your design on the other hand only uses eight of those panes. The first thing you want to do once you start ICC2 is set up the PVT configuration. Even before you open the library, before you create a library, you need to set the configuration correctly. If you look at the example here, the voltages .75 and .95, temperatures -40, 125, process numbers 1, and process labels slow and fast are selected. Now when you create the library using the create_lib command and you specify the reference libraries, ICC2 will only load the panes that you have specified. It will not load anything else. If you were to do a report_lib after this, you would only see the panes that match the PVT configuration. It's as if the library doesn't even contain those panes. If you have a look at the log file when you open the library or create the library, you will see information messages telling you exactly what panes are being loaded.  
선택적 로딩을 사용하려면 set_pvt_configuration 명령을 사용해야 합니다. 기억하시다시피, 이 명령은 Library Manager에서 NDM 셀 라이브러리에 포함되어야 할 DB를 선택하는 데 사용한 명령과 정확히 동일합니다. 작은 예제를 통해 어떻게 동작하는지 설명해 보겠습니다. 132개의 패인이 있는 NDM 셀 라이브러리가 있다고 가정해 보겠습니다. 반면에 설계에서는 그 중 8개의 패인만 사용합니다. ICC2를 시작하기 전에 PVT 구성을 설정해야 합니다. 라이브러리를 열기 전이나 라이브러리를 생성하기 전에 구성을 올바르게 설정해야 합니다. 예제를 살펴보면, 전압으로 .75와 .95, 온도로 -40과 125, 프로세스 번호로 1, 그리고 프로세스 라벨로 slow와 fast가 선택되어 있습니다. 그리고 create_lib 명령을 사용하여 라이브러리를 생성할 때 참조 라이브러리를 지정하면 ICC2는 지정된 패인만 로드하게 됩니다. 다른 패인은 로드되지 않습니다. 이후 report_lib를 수행하면 PVT 구성과 일치하는 패인만 표시됩니다. 마치 라이브러리에 해당 패인이 포함되어 있지 않은 것처럼 보입니다. 라이브러리를 열거나 생성할 때 로그 파일을 확인해 보면 로드되는 패인에 대한 정보 메시지가 표시됩니다.  
{: .notice--warning}  

## Specify TLUplus Parasitic RC Models

- Specify the appropriate TLUplus model for **each corner**
  - TLUplus models must have been previously loaded into a technology-only library, or into the design library (shown below)

```tcl
# If the TLUplus models have not been loaded into a technology library,
# they can be loaded into the design library:

read_parasitic_tech -tlup $TLUPLUS_MAX_FILE -name maxTLU
read_parasitic_tech -tlup $TLUPLUS_MIN_FILE -name minTLU

# Specify the TLUplus model for each corner;
# If the TLUplus models were loaded into the tech lib use the -library option:
set_parasitic_parameters -corner c_slow \
    -library ${techlib} -early spec maxTLU -late_spec maxTLU
set_parasitic parameters -corner c_fast \
    -library ${techlib} -early spec minTLU -late_spec minTLU
```

In addition to the tech file and CLIBs you also need to specify the TLU plus files. These files are used for RC extraction. If using the technology only library then TLU plus files are usually part of that, if not, then these file are read into the IC compiler 2 tool  by using the `read_parasitic_tech` command, there you will specify the name of the parasitic file and as well as internal name which can be used in later operations in the flow. In the example shown here, you have read two TLU+ files with the names maxTLU and minTLU. In later stage, once corners has been setup, use `set_parasitic_parameter` command to apply maxTLU and minTLU models to a specific corner.  
기술 파일과 CLIB 외에도 TLU 플러스 파일을 지정해야 합니다. 이 파일들은 RC 추출에 사용됩니다. 기술 전용 라이브러리를 사용하는 경우 TLU 플러스 파일이 일반적으로 포함되어 있지만, 그렇지 않은 경우 IC Compiler 2 도구에서 `read_parasitic_tech` 명령을 사용하여 이러한 파일을 읽어들입니다. 이 명령에서는 파라미터 파일의 이름과 나중에 플로우에서 사용할 수 있는 내부 이름을 지정합니다. 여기서 보여지는 예제에서는 maxTLU와 minTLU라는 두 개의 TLU+ 파일을 읽어들였습니다. 후속 단계에서 코너가 설정된 후에는 `set_parasitic_parameter` 명령을 사용하여 maxTLU와 minTLU 모델을 특정 코너에 적용할 수 있습니다.  
{: .notice--warning}  

## OCV Effects on Timing

- PVT variation across the die, or “on-chip variation” (OCV), causes timing variations
- OCV can cause real timing violations to be missed, if not considered during analysis and optimization — consider the following extreme example:

![2023-05-17-ocv-effects-on-timing.png]({{site.baseurl}}/assets/images/2023-05-17-ocv-effects-on-timing.png){: .align-center}  

- Process variations are random in nature
  - Can vary from transistor to transistor
- Voltage and temperature variations are systematic
  - Increase with distance between related cells

Let's move on to the next topic. How does on-chip variation or OCV affect your timing? OCV is essentially PVT variation across your die, rooted in the fact that there will be slightly different temperatures and voltages across your die, as well as device-fabrication based process variations. These variations can cause real timing violations if they're not taken into consideration during analysis and optimization. It's important to understand that process variations are random in nature, and can vary from transistor to transistor, because of fabrication variation - any cell in a timing path could be slightly faster or slightly slower. On the other hand, voltage and temperature variations across the die are systemic in nature - some areas on the chip might be warmer or have slightly different voltages; these variations increase with the distance between the related cells.  
다음 주제로 넘어가겠습니다. 칩 내 변동 또는 OCV가 타이밍에 어떤 영향을 미치는지 살펴봅시다. OCV는 칩 내 PVT 변동을 의미하며, 칩 내에서 약간 다른 온도와 전압, 그리고 장치 제작 공정에 기인한 공정 변동이 발생합니다. 이러한 변동은 분석과 최적화 과정에서 고려되지 않으면 실제 타이밍 위반이 발생할 수 있습니다. 공정 변동은 무작위적인 성질을 가지며, 트랜지스터마다 다를 수 있습니다. 이는 제작 변동으로 인해 타이밍 경로의 어떤 셀이 약간 빠를 수도, 약간 느릴 수도 있다는 것을 의미합니다. 반면에 칩 내 전압과 온도 변동은 체계적인 성질을 가지며, 칩 내의 일부 영역은 더 따뜻하거나 약간 다른 전압을 가질 수 있습니다. 이러한 변동은 관련된 셀들 사이의 거리가 멀어질수록 증가합니다.  
{: .notice--warning}  

## Traditional Timing Derate Method for Modeling OCV

Commonly only one library is available per SLOW or FAST corner  
→ Apply estimated derating to model the combined effects of PVT variations

- LATE derate: Slows down launch paths for setup, and capture paths for hold
  - `set_timing_derate -late 1.04`
- EARLY derate: Speeds up capture paths for setup, and launch paths for hold
  - `set_timing_derate -early 0.92`
- Can focus on data or clock logic, net or cell delays, etc.

![2023-05-17-traditional-timing-derate-method.png]({{site.baseurl}}/assets/images/2023-05-17-traditional-timing-derate-method.png){: .align-center}  

In order to model OCV delay variations, the most commonly used approach is set_timing_derate. A late derate is done using set_timing_derate -late. This slows down the launch paths for setup and capture paths for hold. If you look down here in the diagram, just a quick reminder as to what a launch path is; that's the red path right from the input Clk port through the data path cells to the register. The capture path is the path from the Clk port to the Clk pin of the capturing register. With the set_timing_derate -early, it speed up the capture paths for setup and the launch paths for hold. Let's essentially making it more difficult for setup and for hold to meet timing by changing the delays of the launch versus the capture paths. If you look at the picture on the bottom left, the actual library arrival time is shown, that's the value that is found using the db. If there is a 4% late timing derate, that is set_timing_derate -late 1.04, this slows down the logic. An 8% early timing derate, that is set_timing_derate -early 0.92, speeds up the logic. Using set_timing_derate, you can derate the data versus the clock logic, net versus cell delays, etc.  
OCV 지연 변동을 모델링하기 위해 가장 일반적으로 사용되는 방법은 set_timing_derate입니다. set_timing_derate를 사용하여 late derate를 수행합니다. 이는 setup에 대한 런치 패스를 늦추고, hold에 대한 캡처 패스를 늦춥니다. 아래의 다이어그램에서 런치 패스와 캡처 패스의 예시를 확인할 수 있습니다. 런치 패스는 입력 Clk 포트에서 데이터 패스 셀을 통해 레지스터로 이어지는 빨간색 경로입니다. 캡처 패스는 Clk 포트에서 캡처 레지스터의 Clk 핀으로 이어지는 경로입니다. set_timing_derate -early를 사용하면, setup에 대한 캡처 패스와 hold에 대한 런치 패스를 가속화합니다. 이는 런치 패스와 캡처 패스의 지연을 변경하여 setup과 hold의 타이밍 충족을 어렵게 만듭니다. 아래 왼쪽의 그림에서는 실제 라이브러리 도착 시간이 표시되어 있으며, 이 값은 DB를 사용하여 찾아집니다. 만약 4%의 late timing derate, 즉 set_timing_derate -late 1.04을 사용한다면, 로직이 느려집니다. 8%의 early timing derate, 즉 set_timing_derate -early 0.92을 사용한다면, 로직이 가속화됩니다. set_timing_derate를 사용하면 데이터 대 클럭 로직, 넷 대 셀 지연 등을 derate할 수 있습니다.  
{: .notice--warning}  

## Derate Factor in Timing Report

```tcl
set_timing_derate -early 0.92
set_timing_derate -late 1.04
report_timing -path full_clock -derate
```

To see the derate information in a timing report, use the -derate switch. As you can see from the setup timing report, there's a derate of 1.04 for the launch path, and 0.92 for the capture path.  
OCV 지연 정보를 타이밍 보고서에서 확인하려면 -derate 스위치를 사용합니다. setup 타이밍 보고서를 살펴보면, 런치 패스에 대한 1.04의 derate와 캡처 패스에 대한 0.92의 derate를 확인할 수 있습니다.  
{: .notice--warning}  

## Timing Pessimism with OCV Analysis

The “min-max” OCV delay range of BF1 in the SLOW corner is 90-100 ps:  

1. Assume the above OCV is modelled by early/late derating: If the launch and capture clock paths each contain ten BF 1 buffers, setup timing analysis assumes:  
   - a. 900 ps for launch path; 1000 ps for capture path
   - b. 1000 ps for launch path; 900 ps for capture path
2. On actual silicon, the more realistic total delay for ten BF1 buffers will be between 900ps and 1000ps. Therefore, OCV timing analysis is:
   - a. Optimistic
   - b. Pessimistic
3. The closer the cells are physically placed to each other:
   - a. The larger their systematic V/T variations can be
   - b. The smaller their systematic V/T variations can be
4. Since process (P) variation is random, its effects are smaller in a path with:
   - a. 30 BF1 cells
   - b. 3 BF1 cells

Before we move on to other variation models, let's discuss a few questions regarding OCV analysis and how it relates to timing pessimism. Assume that I have a timing path that consists of 10 buffers. The delay variation or OCV delay range is between 90 and 100 picoseconds per buffer. If we use set_timing_derate, and the launch and capture paths each contain 10 buffers, what does setup timing analysis assume?  Will it use 900 picoseconds for launch and 1,000 for capture, or will it use 1,000 for launch and 900 for capture? The answer here would be B. The launch path will be slowed down. Since we use constant derating, every buffer will use the slower delay, and in a chain of 10 buffers, we end up with 1,000 picoseconds for launch, while the capture path will be the opposite, so 90 ps, which adds up to 900 ps. Question 2 - On actual silicon, the more realistic total delay of 10 buffers will be between 900 and 1,000 picoseconds, and, therefore, is OCV timing analysis optimistic or pessimistic? It's very unlikely that all buffers get to their max delay all at the same time versus their fastest delay. Therefore, this is pessimistic. Question 3 - the closer cells are physically placed to each other-do we have more or less systemic on-chip variation? The answer again here is B, less on-chip variation. The closer cells are together, the less variation we'll get in temperature and voltage. Finally, since process variation is random in nature as discussed, are the effects smaller in a path with 30 or 3 cells? The answer here would have to be A. The more samples you have, the more the samples will cancel each other out, and the more predictable the delay will become.  
다른 변동 모델에 대해 이동하기 전에 OCV 분석과 타이밍 비관성과의 관련성에 대해 몇 가지 질문을 논의해 보겠습니다. 10 개의 버퍼로 구성된 타이밍 경로가 있다고 가정해 봅시다. 딜레이 변동 또는 OCV 딜레이 범위는 각 버퍼당 90에서 100 피코초사이입니다. set_timing_derate를 사용하고, 발생 경로와 캡처 경로가 각각 10 개의 버퍼를 포함한다면, setup 타이밍 분석은 무엇을 가정할까요? 발생 경로에는 900 피코초, 캡처 경로에는 1000 피코초를 사용할까요, 아니면 발생 경로에는 1000 피코초, 캡처 경로에는 900 피코초를 사용할까요? 여기서는 B입니다. 발생 경로는 느려집니다. 일정한 디레이트를 사용하기 때문에 모든 버퍼는 더 느린 딜레이를 사용하며, 10 개의 버퍼 연쇄에서 발생 경로에는 1000 피코초가 되고, 캡처 경로는 그 반대인 90 피코초가 되어 900 피코초가 됩니다. 질문 2 - 실제 실리콘에서 10 개의 버퍼의 보다 현실적인 총 딜레이는 900에서 1000 피코초 사이가 될 것입니다. 따라서 OCV 타이밍 분석은 낙관적인가 비관적인가요? 모든 버퍼가 동시에 최대 딜레이에 도달하는 것은 매우 불가능하기 때문에 이는 비관적입니다. 질문 3 - 더 가까이 위치한 셀일수록 칩 내 시스템적인 온칩 변동이 더 많이 발생합니까 아니면 덜 발생합니까? 여기서도 B입니다. 셀이 서로 가까이 위치할수록 온도와 전압의 변동이 적어집니다. 마지막으로, 프로세스 변동은 무작위성을 가지기 때문에, 30 개 또는 3 개의 셀로 구성된 경로에서의 영향은 더 작을까요? 여기서는 A입니다. 샘플이 많을수록 샘플들은 상쇄되어 딜레이가 예측 가능해집니다.
{: .notice--warning}  

## AOCV Calculates Derating Based on Depth and Distance

![2023-05-17-aocv-derating-based-on-depth-dist.png]({{site.baseurl}}/assets/images/2023-05-17-aocv-derating-based-on-depth-dist.png){: .align-center}  

Since `set_timing_derate` can be very pessimistic, other approaches are used for timing derate. Advanced on-chip variation, or AOCV, provides a more realistic variable derating, based on path depth and distance. The random Process variation becomes a function of the logical path depth, and the systemic temperature and voltage variation is modeled by measuring the physical distance of the path cells and nets. On the bottom right you can see a table that shows the derate factor applied based on the depth of the timing path. For a depth of 5, the variable AOCV derate factor is only 1.056, vs. an assumed constant derate factor of 1.15. As you can see, for very short paths (1 and 2) the variable derate can actually be larger than the constant derate value. On the bottom it is an AOCV depth+distance variation table. The factor decreases with increasing path depth, and increases with increasing distance. The table is an example of such a file that is read into ICCII.  
`set_timing_derate`가 매우 비관적일 수 있으므로 타이밍 디레이트에는 다른 접근 방식이 사용됩니다. 고급 온칩 변동(AOCV)은 경로의 깊이와 거리에 기반한 더 현실적인 가변 디레이팅을 제공합니다. 무작위 프로세스 변동은 논리적인 경로의 깊이에 따라 함수적으로 결정되며, 체계적인 온도와 전압 변동은 경로 셀과 넷의 물리적인 거리를 측정하여 모델링됩니다. 오른쪽 아래에서는 타이밍 경로의 깊이에 따라 적용된 디레이트 팩터를 보여주는 테이블을 볼 수 있습니다. 깊이가 5인 경우, 가변 AOCV 디레이트 팩터는 1.056에 불과하며, 가정된 고정 디레이트 팩터인 1.15보다 작습니다. 매우 짧은 경로(1과 2)의 경우 가변 디레이트가 실제로 고정 디레이트 값보다 큰 경우도 있습니다. 아래쪽에는 AOCV 깊이+거리 변동 테이블이 있습니다. 팩터는 경로의 깊이가 증가함에 따라 감소하고 거리가 증가함에 따라 증가합니다. 이 테이블은 ICCII에 읽히는 파일의 예입니다.  
{: .notice--warning}  

## Enabling Advanced On-Chip Variation Modeling

- AOCVM reduces excessive timing margin or pessimism
- If you have AOC VM tables, use them!

If AOCV tables are available, it is recommended that you use them. They will reduce excessive pessimism, especially in the smaller technology nodes. To enable AOCV Modeling, use the application option time.aocvm_enable_analysis. Distance analysis is enabled using a separate application option as shown. Read the AOCV tables using read_ocvm. You can read different tables into different corners.  
AOCV 테이블이 사용 가능하다면 해당 테이블을 사용하는 것이 권장됩니다. 특히 작은 기술 노드에서 과도한 비관적인 결과를 줄일 수 있습니다. AOCV 모델링을 활성화하기 위해 time.aocvm_enable_analysis 응용 프로그램 옵션을 사용합니다. 거리 분석은 별도의 응용 프로그램 옵션을 사용하여 활성화됩니다. read_ocvm을 사용하여 AOCV 테이블을 읽을 수 있습니다. 다른 테이블을 다른 코너에 읽을 수 있습니다.  
{: .notice--warning}  

## Pessimism in Graph-Based Analysis (GBA) AOCV

![2023-05-17-pessimism-in-gba-aocv.png]({{site.baseurl}}/assets/images/2023-05-17-pessimism-in-gba-aocv.png){: .align-center}  

There is one problem with AOCVM and graph-based timing analysis. Graph based analysis, or GBA, calculates the timing of a cell once, regardless of whether the cell is involved in multiple shared paths or not. If different transition times feed into the common cell, GBA uses the worst transition for example. The same applies to AOCV. If a cell is shared among a short and a long path, the derate factor ICCII picks must be the worst factor, so for AOCV this will be the path with 3 cells. This means that for the timing path on the bottom that has a depth of 39, ICCII will still use the derate factor of 1.097, instead of 1.034 which would correspond to that path. Path-based analysis, or PBA, in Primetime uses the actual path-specific derate factor. PBA has a higher runtime penalty though. So in conclusion, it's important to understand that this pessimism will be part of the timing calculation in ICCII.  
AOCVM 및 그래프 기반 타이밍 분석에는 한 가지 문제가 있습니다. 그래프 기반 분석(GBA)은 셀이 여러 공유 경로에 참여하더라도 해당 셀의 타이밍을 한 번만 계산합니다. 만약 서로 다른 전환 시간이 공통 셀에 입력된다면, GBA는 가장 나쁜 전환 시간을 사용합니다. AOCV에도 동일한 원리가 적용됩니다. 셀이 짧은 경로와 긴 경로에서 공유된다면, ICCII가 선택하는 derate factor는 가장 나쁜 factor여야 합니다. 따라서 AOCV에서는 3개 셀을 포함하는 경로의 derate factor가 선택됩니다. 이는 아래에 표시된 깊이가 39인 타이밍 경로에도 해당되므로, ICCII는 여전히 1.097의 derate factor를 사용하게 됩니다. 이는 해당 경로에 해당하는 1.034의 factor를 사용하지 않음을 의미합니다. Primetime의 경로 기반 분석(PBA)은 실제 경로별 derate factor를 사용합니다. 하지만 PBA는 실행 시간이 더 오래 걸립니다. 따라서 ICCII에서는 이러한 비관적인 점이 타이밍 계산의 일부로 반영됨을 이해하는 것이 중요합니다.  
{: .notice--warning}  

## Parametric On-Chip Variation Modeling of Random Variation

![2023-05-17-pocv.png]({{site.baseurl}}/assets/images/2023-05-17-pocv.png){: .align-center}  

The latest approach used to calculate random P variation is called Parametric on-chip variation, or POCV. POCV doesn't rely on path depth, but instead uses a Gaussian distribution for each cell delay. Each cell has a mean delay m and a standard deviation sigma, which is also sometimes called sensitivity. To calculate the path delay, the individual delay distributions are added statistically. This entirely eliminates GBA-pessimism, and simplifies the timing analysis and optimization.  
최신 접근 방식인 Parametric on-chip variation(POCV)은 랜덤 P variation을 계산하는 데 사용됩니다. POCV는 경로의 깊이에 의존하는 것이 아니라, 각 셀 딜레이에 대해 가우시안 분포를 사용합니다. 각 셀은 평균 딜레이 m과 표준 편차 sigma(때로는 민감도라고도 함)를 가지고 있습니다. 경로 딜레이를 계산하기 위해 개별 딜레이 분포가 통계적으로 더해집니다. 이는 GBA 비관적인 요소를 완전히 제거하고, 타이밍 분석과 최적화를 간소화합니다.  
{: .notice--warning}  

## POCV Input Data: Sigma(σ) - Two Formats

There are two ways to provide the sigma value to ICCII. On the left, there's the single coefficient file, in which every library cell or group of library cells will have a certain POCV coefficient, characterized at a particular input transition and output load. To calculate the sigma value, the coefficient is multiplied with the mean delay. On the right, you can see an LVF table. LVF stands for liberty variation format, and provides a mechanism in the normal liberty dbs to directly provide sigma values as a function of input transition and output load, so very similar to the table lookup for a normal delay value. LVF becomes more important for the smaller technology nodes.  
ICCII에 시그마 값을 제공하는 두 가지 방법이 있습니다. 왼쪽에는 단일 계수 파일이 있습니다. 이 파일에서는 각 라이브러리 셀 또는 그룹의 POCV 계수가 특정 입력 전환과 출력 부하에서 특성화됩니다. 시그마 값을 계산하기 위해 계수는 평균 딜레이와 곱해집니다. 오른쪽에는 LVF 테이블이 표시됩니다. LVF는 라이브러리 DB에서 시그마 값을 입력 전환과 출력 부하의 함수로 직접 제공하는 Liberty Variation Format의 약어입니다. 이는 일반적인 딜레이 값에 대한 테이블 조회와 매우 유사한 방식입니다. LVF는 작은 기술 노드에서 더 중요해집니다.  
{: .notice--warning}  

## POCV 관련 내용 생략

## Perform a 'Timing Sanity Check'

- Before starting placement it is important to ensure that the design is not over-constrained
  - Constraints should match the design’s specification
- Report ‘ZIC’ setup timing before placement
  - Check for unrealistic or incorrect constraints
  - Investigate large zero-interconnect timing violations

```tcl
set_app_options -list {
  time.delay_calculation_style zero_interconnect
  time.high_fanout_net_pin_capacitance OpF
  time.high_fanout_net_threshold 32
}
update_timing -full
report_constraints -all_violators -max_delay
report_qor -summary -include setup
report_timing -slack_lesser_than 0
```

Once you have completed your timing setup, before you embark on placement and optimization, you should check that the synthesis netlist is in good shape. In zero-interconnect timing mode, ICCII will calculate the timing based on zero net capacitance. With that, you check for unrealistic or incorrect constraints, and fix them before starting `place_opt`. Have a look at the example script: the application option to set this mode is called `time.delay_calculation_style`. It is set to zero_interconnect. If your netlist contains unbuffered high fanout nets, then these will cause excessive timing failures, as even with all nets having zero capacitance, the sum of all the input pin capacitances connected to the high-fanout net will be huge, causing large delays. In order to mask pin capacitances of nets with a fanout of 100 or more, use the two application options shown. After an "`update_timing -full`", you can analyze the design for timing using your favorite commands.  
플레이스먼트와 최적화를 시작하기 전에 타이밍 설정이 완료되었는지 확인하기 위해 합성 넷리스트가 적합한지 확인해야 합니다. ICCII에서는 제로 인터커넥트 타이밍 모드로 동작하여 넷 용량을 제로로 가정하여 타이밍을 계산합니다. 이를 통해 현실적이지 않거나 잘못된 제약 조건을 확인하고 `place_opt`를 시작하기 전에 수정할 수 있습니다. 예제 스크립트를 살펴보십시오. 이 모드를 설정하는 애플리케이션 옵션은 `time.delay_calculation_style`이라고 합니다. 이 옵션은 zero_interconnect로 설정됩니다. 넷리스트에 비버퍼화된 고 팬아웃 넷이 포함되어 있다면, 이러한 넷들은 모든 넷의 용량을 제로로 설정해도, 고 팬아웃 넷에 연결된 모든 입력 핀의 용량의 합이 크기 때문에 큰 딜레이를 발생시켜 과도한 타이밍 실패를 초래할 수 있습니다. 팬아웃이 100 이상인 넷의 핀 용량을 마스킹하기 위해 표시된 두 개의 애플리케이션 옵션을 사용하십시오. "`update_timing -full`" 이후에는 원하는 명령을 사용하여 타이밍을 분석할 수 있습니다.  
{: .notice--warning}  

## Timing Constraints Feasibility

- Are your constraints clean?
  - Realistic 10 constraints
  - All necessary timing exceptions applied
- If not, many optimization cycles will be wasted on possibly infeasible timing paths
- Recommendation:
  - Identify and handle infeasible paths before `place_opt`

While constraining the design, make sure constraints are realistic. For example, inputs and outputs ports can be optimistic and conservative, you might over-constrain these ports which can cause situations where timing cannot meet.  Or you might not apply a false path or a multi-cycle path in a particular area that can also cause a situation where timing is never met. If this is the scenario then many optimization cycles will be wasted on possibly infeasible timing paths. So, the recommendation is, to identify and handle such infeasible paths.  
디자인을 제약할 때, 제약 조건이 현실적인지 확인하세요. 예를 들어, 입력 및 출력 포트는 낙관적이고 보수적일 수 있습니다. 이러한 포트를 과도하게 제약할 경우 시간이 충족되지 못하는 상황이 발생할 수 있습니다. 또는 특정 영역에서 false path나 multi-cycle path를 적용하지 않을 경우에도 시간이 충족되지 않는 상황이 발생할 수 있습니다. 이러한 시나리오라면, 많은 최적화 주기가 불가능한 타이밍 경로에 낭비될 수 있습니다. 따라서 권장하는 방법은 불가능한 경로를 식별하고 처리하는 것입니다.  
{: .notice--warning}  

## Handling Infeasible Paths

- Infeasible paths are paths that are impossible to meet timing(arrival time at start point > required time at endpoint):
  - Missing false path or multicycle path constraints
  - Unreasonable input or output delay constraints, even when cell and net delays are set to zero

```tcl
compute_infeasible_path_overrides
```

- Identifies infeasible timing paths
- Generates negative path margins
- Applies the generated path margins to prevent optimization of these paths

The infeasible paths are paths that are impossible to meet timing. If the arrival time of the data at the start point of the path is greater than the required time at the endpoint, that is an infeasible path. If you have missing false paths or missing multicycle path constraints or if you have unreasonably large input delay or output delay constraints, you can end up in a situation like that. So even when your cell and net delay is 0 there's no way to meet those constraints. Identify these paths with the `compute_infeasible_path_overrides` command. It will identify the infeasible timing paths. It then generates negative path margins, that can be applied to these generated path margins to prevent the optimization of these paths.  
불가능한 경로란 타이밍을 충족할 수 없는 경로입니다. 경로의 시작점에서 데이터의 도달 시간이 종점에서 요구되는 시간보다 큰 경우, 이는 불가능한 경로입니다. 만약 false path나 multicycle path 제약 조건이 누락되었거나 입력 지연이나 출력 지연 제약 조건이 지나치게 크다면, 이와 같은 상황이 발생할 수 있습니다. 따라서 셀과 넷 지연이 0이더라도 이러한 제약 조건을 충족시킬 방법은 없습니다. `compute_infeasible_path_overrides` 명령을 사용하여 이러한 경로를 식별하세요. 이 명령은 불가능한 타이밍 경로를 식별합니다. 그런 다음 이러한 경로의 최적화를 방지하기 위해 생성된 경로 여유를 음수로 적용할 수 있습니다.  
{: .notice--warning}  

## Infeasible Paths Example

- The following example identifies infeasible paths in the current scenario and writes out `set_path_margin` exceptions to `out.tcl`
  - After verifying/adjusting the path margin exceptions apply them by sourcing `out.tcl`
  - In report: arrival = startpoint arrival time, required = endpoint required time

```tcl
compute_infeasible_path_overrides -verbose -output out.tcl
```

Here is an example of applying the `compute_infeasible_path_overrides` command with -verbose and -output options. And writing the results to an out.tcl file. In the GUI notice, there are two infeasible paths shown in this example. So the first path starts in a primary input port called IN1 and it ends at the D pin of a register called ff7_reg inside inst1. It displays the infeasibility in the infeasibility column. The arrival column denotes the arrival time of the inputs. Here, the input data arrives at 1.44, but it's expected to be at the D pin at 1.2. This is an example where we have an over constraint set input delay and it's arriving later than when it is expected at the endpoint, so the infeasibility is 0.24. In the second example, we have a launching Flip Flop, the Clock pin being captured by the D pin of another register and at the launch clock, the clock happens at 3.52, which is later than when it's supposed to arrive at the D pin. So again, this is infeasible. To meet the timing we would need a negative delay. In the out.tcl file, notice the two `set_path_margin` commands, both of them are set to -10,000. That is hard-coded. It always sets -10,000 as setup override. It applies a comment with the keyword `SYNOPSYS_INFEASIBLE_PATH_OVERRIDE`. And it applies it to a specific scenario and a particular path within the scenario.  So runs this on a scenario-by-scenario basis. The out.tcl file is editable, you can source this file.  
`compute_infeasible_path_overrides` 명령을 -verbose와 -output 옵션과 함께 적용하는 예제입니다. 그리고 결과를 out.tcl 파일에 작성합니다. GUI에서는 이 예제에서 두 개의 불가능한 경로가 표시됩니다. 첫 번째 경로는 IN1이라는 기본 입력 포트에서 시작하여 inst1 내부의 ff7_reg라는 레지스터의 D 핀에서 끝납니다. 불가능성은 불가능성 열에서 표시됩니다. arrival 열은 입력의 도달 시간을 나타냅니다. 여기에서 입력 데이터는 1.44에 도달하지만 D 핀에서는 1.2에 도달해야 합니다. 이는 입력 지연이 지나치게 제약되어 있고 종점에서 기대 시간보다 늦게 도달한 예제입니다. 따라서 불가능성은 0.24입니다. 두 번째 예제에서는 런칭 플립 플롭으로 클록 핀이 다른 레지스터의 D 핀에 의해 캡처되고 런칭 클록에서 클록이 3.52에 발생합니다. 이는 D 핀에서 도달해야 하는 시간보다 늦게 발생하므로 다시 한 번 불가능한 상황입니다. 타이밍을 충족시키기 위해서는 음수 지연이 필요합니다. out.tcl 파일에서는 두 개의 `set_path_margin` 명령을 볼 수 있습니다. 두 명령 모두 -10,000으로 설정되어 있습니다. 이는 하드코딩되어 있으며 항상 -10,000을 setup override로 설정합니다. `SYNOPSYS_INFEASIBLE_PATH_OVERRIDE`라는 키워드와 함께 주석을 적용하고 특정 시나리오와 시나리오 내의 특정 경로에 적용합니다. 따라서 시나리오별로 실행합니다. out.tcl 파일은 편집 가능하며 이 파일을 소스로 사용할 수 있습니다.  
{: .notice--warning}  

## Infeasible Path Command Details

So, the compute_infeasible_path_override command applies to the current scenario by default. Run it for each of the corners or/and modes or scenarios. By default, it identifies the worst 10,000 paths per path group. This command will look at the worst 10,000 paths, but for example, if there are 12,000 violations then the 10,001 up to the 12,000 violations, those will not get a path margin unless the number is not increased with the -max_paths or -nworst option. By default, the command does not apply the path margin. It is recommended to write out the path margins to a file, then check, correct, and source. But if `-apply_override_constraints` is used, then it will apply them during the command execution. The generated `set_path_margin` command will apply this timing exception, which can be reported with `report_exceptions`. It always relaxes the path timing by 10,000 time units.  
compute_infeasible_path_override 명령은 기본적으로 현재 시나리오에 적용됩니다. 각각의 코너 또는/및 모드 또는 시나리오에 대해 실행하세요. 기본적으로 이 명령은 경로 그룹당 최악의 10,000개 경로를 식별합니다. 이 명령은 최악의 10,000개 경로를 확인하지만 예를 들어 12,000개의 위반 사항이 있는 경우 10,001번부터 12,000번까지의 위반 사항은 -max_paths 또는 -nworst 옵션을 사용하여 경로 수를 늘리지 않는 한 경로 여유를 받지 못합니다. 기본적으로 이 명령은 경로 여유를 적용하지 않습니다. 경로 여유를 파일로 작성한 다음 확인하고 수정한 후에 소스로 적용하는 것이 권장됩니다. 그러나 `-apply_override_constraints` 옵션을 사용하면 명령 실행 중에 경로 여유가 적용됩니다. 생성된 `set_path_margin` 명령은 타이밍 예외를 적용하며 `report_exceptions`로 보고할 수 있습니다. 항상 경로 타이밍을 10,000개의 시간 단위로 완화시킵니다.  
{: .notice--warning}  

## Removing Infeasible Paths

- Generated path margin exceptions are annotated with a “comment” attribute value of `SYNOPSYS_INFEASIBLE_PATH_OVERRIDE`
- To remove these generated path margins:
  - `remove_path_margin -comment SYNOPSYS_INFEASIBLE_PATH_OVERRIDE`

If you have applied and sourced the out.tcl file and then you later decide to remove all infeasible paths then to remove the infeasible path, there is a remove_path_margin command. The way to easily identify all of the tool-applied path margins is to use the -comment option. The tool applied path margins have a -comment of `SYNOPSYS_INFEASIBLE_PATH_OVERRIDE`, it will remove all of these tool path margins, identified by this comment.  
만약 이전에 out.tcl 파일을 적용하고 소스로 사용하여 불가능한 경로에 경로 여유를 추가했다면, 나중에 이를 제거하기로 결정한 경우 remove_path_margin 명령을 사용할 수 있습니다. 도구에서 적용된 경로 여유를 쉽게 식별하는 방법은 -comment 옵션을 사용하는 것입니다. 도구에서 적용된 경로 여유는 `SYNOPSYS_INFEASIBLE_PATH_OVERRIDE`라는 주석이 있습니다. 이 주석으로 식별된 도구 경로 여유를 모두 제거할 수 있습니다.  
{: .notice--warning}  

## Additional Timing Settings

- Use `report_app_options` to find additional timing application options
- Use the man pages to learn more: `report_app_options time.*`

Use `report_app_options` with a key category in a wild card that will list all of the app options. Find application options that affect timing analysis, it starts with the key category time. You can also use the man page to learn more about them.  
`report_app_options` 명령을 사용하여 와일드카드를 포함한 키 카테고리를 가진 모든 앱 옵션을 나열합니다. 타이밍 분석에 영향을 주는 앱 옵션을 찾으려면 키 카테고리가 "time"으로 시작하는 옵션을 찾습니다. 또한 man 페이지를 사용하여 더 자세히 알아볼 수도 있습니다.  
{: .notice--warning}  
