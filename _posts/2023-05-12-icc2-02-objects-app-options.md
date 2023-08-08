---
title: "IC Compiler II - 02 Objects, Blocks and App Options"
excerpt: "Block-level Implementation"
date: 2023-05-12 18:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
---

## Objectives

After completing this module, you should be able to:

- Query objects and retrieve design information using attributes
- Perform block/library manipulation operations like, open the block, save the block, or library, block renaming, so on.
- Query and set application options to control IC Compiler II's two's behavior.

이 모듈을 완료한 후에는 다음을 할 수 있어야 합니다:

- 속성을 사용하여 객체를 쿼리하고 설계 정보를 검색할 수 있습니다.
- 블록/라이브러리 조작 작업을 수행할 수 있습니다. 예를 들어 블록 열기, 블록 또는 라이브러리 저장, 블록 이름 변경 등을 할 수 있습니다.
- IC Compiler II의 동작을 제어하기 위해 응용 프로그램 옵션을 쿼리하고 설정할 수 있습니다.

## Prerequisites

Before starting this module, you should be able to

- Load an existing design
- Use the GUI to interact with a design

We do not expect any prior knowledge of objects, blocks and app options

## Objects

![2023-05-12-objects.png]({{site.baseurl}}/assets/images/2023-05-12-objects.png)

ICC II provides a full set of object classes, and associated tickle commands to, create, remove, query, and modify all primary design data. For example, `cell`, `pin`, `net`, `clock`. There are `get` commands that allow you to access those objects. ​ For example, `get_nets *_clk`. The `*` here matches any string. So, this is just like a simple shell script match. We can list all available object class through command, `help_attribute` or `get_defined_attributes` `-return_classes`.  
ICC II는 모든 주요 설계 데이터를 생성, 제거, 조회 및 수정하기 위한 완전한 객체 클래스와 관련된 티클(tickle) 명령을 제공합니다. 예를 들어, 셀(`cell`), 핀(`pin`), 넷(`net`), 클럭(`clock`) 등이 있습니다. 이러한 객체에 액세스할 수 있는 `get` 명령도 제공됩니다. 예를 들어, `get_nets *_clk`와 같이 사용할 수 있습니다. 여기서 `*`는 어떤 문자열과도 일치합니다. 이는 셸 스크립트와 유사합니다. 우리는 `help_attribute` 또는 `get_defined_attributes` `-return_classes` 명령을 통해 사용 가능한 모든 객체 클래스를 나열할 수 있습니다.
{: .notice--warning}

## Querying Objects Associated with Objects

![2023-05-12-querying-objects.png]({{site.baseurl}}/assets/images/2023-05-12-querying-objects.png)

In the same context of previous slide, another very useful option with `get_*` command is, `-of_object`. By using this option, you can find the objects associated with, or connected to specific objects. For example, you can find the cells associated with other cells, or with pins, nets, bounds, voltage areas, etc.. Example shown in the slide, this command will return all cells which are associated with the net name called, `net_sdram_clk`. It is recommended to see the man page of each `get_*` command for detailed information.  
이전 슬라이드와 동일한 맥락에서, `get_*` 명령과 함께 매우 유용한 옵션 중 하나는 `-of_object`입니다. 이 옵션을 사용하면 특정 객체와 관련된 또는 연결된 객체를 찾을 수 있습니다. 예를 들어, 다른 셀 또는 핀, 넷, 바운드, 전압 영역 등과 관련된 셀을 찾을 수 있습니다. 슬라이드에 나온 예시의 명령은 `net_sdram_clk` 넷과 관련된 모든 셀을 반환합니다. 자세한 정보는 각 `get_*` 명령의 매뉴얼 페이지를 참조하는 것이 좋습니다.
{: .notice--warning}

## Querying Objects / Hierarchy

![2023-05-12-querying-objects-hier.png]({{site.baseurl}}/assets/images/2023-05-12-querying-objects-hier.png)

'`get_cells -hierarchical`' returns leaf and hierarchy cells with the name attribute matching the pattern, at all levels of hierarchy. The `get_flat_cells` command returns leaf cells with the full_name attribute matching the pattern. The same applies to, `get_flat_nets`, and `get_flat_pins`. `get_flat_cells` returns all leaf-cells in the current physical hierarchy, we can use '`get_flat_cells -hierarchical`' to return all leaf-cells in all physical hierarchies.  
'`get_cells -hierarchical`' 명령은 계층 구조의 모든 수준에서 패턴과 일치하는 이름 속성을 가진 리프 셀과 계층 셀을 반환합니다. '`get_flat_cells`' 명령은 패턴과 일치하는 full_name 속성을 가진 리프 셀을 반환합니다. 마찬가지로 '`get_flat_nets`'와 '`get_flat_pins`'도 동일한 원리로 작동합니다. '`get_flat_cells`'은 현재 물리적 계층 구조에서 모든 리프 셀을 반환하며, '`get_flat_cells -hierarchical`'를 사용하면 모든 물리적 계층 구조에서 모든 리프 셀을 반환할 수 있습니다.
{: .notice--warning}

## Matching Objects

![2023-05-12-matching-objects.png]({{site.baseurl}}/assets/images/2023-05-12-matching-objects.png)

By default, when using query commands, the query string needs to match the actual in-memory object name, and it will fail once it mismatched In ICC 2, we also support rule-based object matching, user can enable that firstly through, '`set_app_options -name design.enable_rule_based_query -value true`', and then set up query rule using '`set_query_rules`'. Notice: First, rule-based matching is enabled by default with `read_def`, in order to resolve object name mismatches. Second, Rule based query is more CPU-intensive, and should only be turned on when required, that is, when reading files that require translation.  
기본적으로 쿼리 명령을 사용할 때는 쿼리 문자열이 실제로 메모리에 있는 객체 이름과 일치해야하며, 일치하지 않을 경우 실패합니다. ICC 2에서는 규칙 기반 객체 일치도 지원합니다. 사용자는 먼저 '`set_app_options -name design.enable_rule_based_query -value true`'를 통해 규칙 기반 쿼리를 활성화할 수 있으며, 그런 다음 '`set_query_rules`'를 사용하여 쿼리 규칙을 설정할 수 있습니다. 주의사항: 첫째로, 객체 이름 불일치를 해결하기 위해 `read_def`와 함께 규칙 기반 일치가 기본적으로 활성화됩니다. 둘째로, 규칙 기반 쿼리는 CPU 부하가 많이 발생하므로, 번역이 필요한 파일을 읽을 때와 같이 필요한 경우에만 활성화해야 합니다.
{: .notice--warning}

## Enable Rule-Based Object Matching

![2023-05-12-rule-based-matching.png]({{site.baseurl}}/assets/images/2023-05-12-rule-based-matching.png)

It is the example for rule-based object matching.
{: .notice--warning}

## Querying Object by Location

![2023-05-12-querying-objects-by-loc.png]({{site.baseurl}}/assets/images/2023-05-12-querying-objects-by-loc.png)

Many `get_<object>` commands support the location query options, and the specified regions can be rectangular, or rectilinear polygons. For example: `get_cell -within geometry`.There has an alternative command in ICC2 that provides similar functionality:`get_objects_by_location -class <object_type>`.But by default, the behavior for `-within`, and `-intersect` are different with '`get_<object>`', for more detail on the difference, please review the man page. User can make the behavior for `-within` and `-intersect` same between the 2 commands with setting '`design.enable_icc_region_query`' as true.  
많은 '`get_<객체>`' 명령은 위치 쿼리 옵션을 지원하며, 지정된 영역은 직사각형 또는 직선 다각형일 수 있습니다. 예를 들어: `get_cell -within geometry`입니다. ICC2에는 비슷한 기능을 제공하는 대체 명령인 `get_objects_by_location -class <객체_유형>`도 있습니다. 그러나 기본적으로 `-within`과 `-intersect`의 동작은 '`get_<객체>`'와 다릅니다. 차이에 대한 자세한 내용은 매뉴얼 페이지를 참조해주십시오. 사용자는 '`design.enable_icc_region_query`'를 true로 설정하여 두 명령어의 `-within`과 `-intersect` 동작을 동일하게 할 수 있습니다.
{: .notice--warning}

## Attributes of Objects

![2023-05-12-attr-of-objects.png]({{site.baseurl}}/assets/images/2023-05-12-attr-of-objects.png)

Each object has a set of related attributes, user can review that through '`list_attributes -application -class objects`'.To learn about attributes of a specific class, we can do: m`an <class>_attributes`. For example: `man net_attributes`, it can show you the predefined attributes for nets.  
각 객체에는 관련 속성 집합이 있으며, 사용자는 '`list_attributes -application -class objects`'를 통해 이를 검토할 수 있습니다. 특정 클래스의 속성에 대해 알아보려면 '`man <class>_attributes`'를 사용할 수 있습니다. 예를 들어, '`man net_attributes`'를 실행하면 넷에 대한 미리 정의된 속성을 볼 수 있습니다.
{: .notice--warning}

## Retrieving and Setting Attribute Values

- To retrieve an attribute value of an object:

```tcl
get_attribute object_list attribute_name

# For example:
get_attribute [get_cells I_RAM_3] physical_status

# >>> placed
```

- To apply an attribute value on objects:

```tcl
set_attribute object_list attribute_name attribute_value

# For example:
set_attribute [get_cells I_RAM_3] physical_status fixed
```

We can retrieve an attribute value of an object through `get_attribute` command. For example: `get_attribute [get_cells I_RAM_3] physical_status`. This command will return the physical status of the cell, which is placed in this case.We can apply an attribute value on objects through `set_attribute` command. For example: `set_attribute [get_cells I_RAM_3] physical_status fixed`.  
객체의 속성 값을 `get_attribute` 명령을 통해 가져올 수 있습니다. 예를 들어, `get_attribute [get_cells I_RAM_3] physical_status`와 같이 사용할 수 있습니다. 이 명령은 해당 셀의 물리적 상태를 반환합니다. 이 경우에는 셀이 배치된 상태입니다.  
객체에 속성 값을 적용하기 위해 `set_attribute` 명령을 사용할 수 있습니다. 예를 들어, `set_attribute [get_cells I_RAM_3] physical_status fixed`와 같이 사용할 수 있습니다.
{: .notice--warning}

## Reporting Attributes of Objects

![2023-05-12-reporting-attrs.png]({{site.baseurl}}/assets/images/2023-05-12-reporting-attrs.png)

We can generate a list of attributes, and values of a currently selected object through '`report_attributes -application [get_selection]`', and user can turn on -`compact` option to improve readability of long object names.Note: The above `report_attributes` command is similar to doing an object query in the GUI, except that the GUI query only lists the attribute name and value, not the other columns. Example, type.Few attributes can have value of collection type.  We will discuss this in the next slide.  
현재 선택된 객체의 속성 및 값 목록을 '`report_attributes -application [get_selection]`'를 통해 생성할 수 있습니다. 긴 객체 이름의 가독성을 향상시키기 위해 -`compact` 옵션을 켤 수도 있습니다.  
참고: 위의 `report_attributes` 명령은 GUI에서 객체 쿼리를 수행하는 것과 유사하지만, GUI 쿼리는 속성 이름과 값만 나열하고 다른 열은 표시하지 않습니다. 예를 들어, type입니다. 일부 속성은 컬렉션 유형의 값을 가질 수도 있습니다. 이에 대해서는 다음 슬라이드에서 다룰 예정입니다.
{: .notice--warning}

## Accessing Cascaded Attributes

![2023-05-12-access-cascaded-attrs.png]({{site.baseurl}}/assets/images/2023-05-12-access-cascaded-attrs.png)

Two (and more) level attribute query is supported:  
`icc2_shell> get_attribute [get_cells U6 ] ref_block.lib.voltage_unit 1.00V`

Some object attributes have "sub-attributes", we called it as cascaded attributes. For example: the layer attribute of a shape object.To query these types of attributes, we will use the same `report_attributes` command with one more level query as shown in the slide. Access the cascaded pitch attribute directly by using `get_attribute` command as shown here.  
일부 객체 속성에는 "하위 속성"이 있을 수 있습니다. 이를 계단식 속성(cascaded attributes)이라고 합니다. 예를 들어, 도형 객체의 레이어 속성이 있습니다. 이러한 유형의 속성을 쿼리하기 위해 슬라이드에 나온 것과 같은 `report_attributes` 명령을 사용하되, 한 단계 더 깊은 쿼리로 사용합니다. 계단식 피치 속성에는 다음과 같이 `get_attribute` 명령을 사용하여 직접 액세스할 수 있습니다.
{: .notice--warning}

## Blocks and Design Libraries

- A block is the container of design data
- A design library stores blocks, together with other library-level data such as technology data

| Create a block from Verilog | Open an existing block |
|:----|:----|
| `create_lib` | `open_lib` |
| `read_verilog` | `open_block` |
| `...` | `...` |
| `save block or save_lib` | |
| `close_lib` | |

A block is the container of design data, and a design library stores blocks, together with other library-level data such as technology data.  
블록은 설계 데이터의 컨테이너이며, 설계 라이브러리는 블록과 함께 기술 데이터와 같은 다른 라이브러리 수준의 데이터를 저장합니다.
{: .notice--warning}

## Save the Design Library and Block

- It is good practice to save the design after each key design phase,
  - for example: design setup, placement, etc.
- Using a block label is a convenient way of doing this:
  - `save block -as ORCA/init_design`
- Saves the design library to disk (if not previously done with `save_lib`) and the block to a new name
  - By default, `save block -as` saves a new copy of a block to disk, but does not change the name of the block in memory
- Blocks with the same block name but different labels are independent

----------

Using a block label is a convenient way to save the design after each key design phase. For example: `save_block -as ORCA/init_design`.

- Using the block label to name a newer version of the block, instead of changing the block name itself, is very convenient for top-level integration: The top-level design, which references the block you are designing, can continue referencing the sub-block ORCA, even if you save the block with different labels.
- Using the above method, at the end of the physical design flow, a block will have many labels. For example: init_design, floorplan, placed, cts, routed. To specify which label should be used for the ORCA reference when it is instantiated in the top-level design: change_abstract -reference ORCA, -label routed. Saves the design library to disk, and the block to a new name.
- By default, `save_block -as`, saves a new copy of a block to disk, but does not change the name of the block in memory.

블록 레이블을 사용하면 각 주요 설계 단계 이후에 설계를 저장하는 편리한 방법입니다. 예를 들어, `save_block -as ORCA/init_design`과 같이 사용할 수 있습니다.

- 블록 레이블을 사용하여 블록의 이름 자체를 변경하는 대신 블록의 새 버전에 이름을 지정하는 것은 상위 수준 통합에 매우 편리합니다. 설계 중인 블록을 참조하는 상위 수준 설계에서는 블록 레이블 ORCA를 계속 참조할 수 있습니다. 블록을 다른 레이블로 저장하더라도 마찬가지입니다.
- 위의 방법을 사용하면 물리적 설계 플로우의 끝에서 블록에는 여러 레이블이 있게 됩니다. 예를 들어, init_design, floorplan, placed, cts, routed 등이 될 수 있습니다. 상위 수준 설계에서 ORCA 참조가 인스턴스화될 때 어떤 레이블을 사용해야 하는지 지정하려면 `change_abstract -reference ORCA`, `-label routed`와 같이 사용할 수 있습니다. 이 명령은 디자인 라이브러리를 디스크에 저장하고 블록을 새 이름으로 저장합니다.
- 기본적으로 `save_block -as`는 블록의 이름을 메모리에서 변경하지 않고 블록의 새 복사본을 디스크에 저장합니다.

## Copying then Saving

- Use the following method to create a copy of the block to a new name, then save it after optimization (RM method)

```tcl
open_lib ORCA_LIB.dlib
copy_block -from ORCA/init_design \
           -to ORCA/place opt
# Copied block is automatically opened, 
# so open_block is not needed, 
# but it is not automatically the current_block; 
# There is no current_block af this point:
current_block ORCA/place_opt
# ...
place_opt
# ...
save_block
exit
```

`copy_block` copies into memory, by default.  
Set `design.on_disk_operation` to `true` to copy to disk.
{: .notice--info}

See notes above for an example of copying to another library  

We can use '`copy_block -from, -to`', and '`save_block`' to create a copy of the block to a new name, then save it. Copied block is automatically opened, so `open_block` is not needed, but it is not automatically the `current_block`; There is no current_block at this point. User need to set the current block explicitly by using current block command. Note: `copy_block` copies into memory, by default. User can set app option: '`design.on_disk_operation`' to `true`, to copy to disk.  
'`copy_block -from, -to`'와 '`save_block`'을 사용하여 블록의 복사본을 새 이름으로 생성한 다음 저장할 수 있습니다. 복사된 블록은 자동으로 열리므로 `open_block`은 필요하지 않지만, 현재 블록은 자동으로 설정되지 않습니다. 이 시점에서는 현재 블록이 없습니다. 사용자는 `current_block` 명령을 사용하여 현재 블록을 명시적으로 설정해야 합니다. 참고: 기본적으로 `copy_block`은 메모리로 복사합니다. 사용자는 '`design.on_disk_operation`' 앱 옵션을 `true`로 설정하여 디스크로 복사할 수 있습니다.
{: .notice--warning}

## Application Options

- Application Options control the behavior and features of
IC Compiler II's commands
- For example, the application option `place_opt.flow.do_spg` controls whether the command `place_opt` takes SPG placement into account or not
- Application options use the following naming convention: `category.sub_category.option_name`
  - The sub_category field is optional

Application Options control the behavior and features of IC Compiler two's commands. For example: the application option: '`place_opt.flow.do_spg`', controls whether the command `place_opt` takes SPG placement into account or not. Application options use the following naming convention: `category.sub_category.option_name`. Category is the name of the engine affected by the application option, such as place, cts, and route. Some application option categories have subcategories to further refine the area affected by the application option. For example, the route category has the following subcategories: `common`, `detail`, `global`, and `track`.  
응용 프로그램 옵션은 IC Compiler II의 명령의 동작과 기능을 제어합니다. 예를 들어, '`place_opt.flow.do_spg`'라는 응용 프로그램 옵션은 `place_opt` 명령이 SPG(Standard Parasitic Extraction) 배치를 고려할지 여부를 제어합니다.  
응용 프로그램 옵션은 다음과 같은 네이밍 규칙을 사용합니다: `category.sub_category.option_name`. 여기서 category는 응용 프로그램 옵션에 영향을 받는 엔진의 이름(예: place, cts, route)입니다. 일부 응용 프로그램 옵션 카테고리에는 응용 프로그램 옵션에 영향을 받는 영역을 더 세분화하는 서브카테고리가 있을 수 있습니다. 예를 들어 route 카테고리에는 다음과 같은 서브카테고리가 있습니다: `common`, `detail`, `global`, tra`ck.
{: .notice--warning}

## Finding / Reporting Application Options

To find the application options for a specific engine, use `report_app_options` or `get_app_options`

- Timing-related application options start with time. To report all timing-related application options, use '`report_app_options time.*`'
- We can list all application options with changed (non-default) values by using command  `report_app_options -non_default`
- '`write_script -include app_options`' creates a directory called `wscript` in the current working directory. This includes, among others, a file called `design.tcl`, which contains all changed application option settings. This file can be edited and then sourced, as needed.

특정 엔진에 대한 응용 프로그램 옵션을 찾으려면 `report_app_options` 또는 `get_app_options`를 사용합니다.

- 타이밍 관련 응용 프로그램 옵션은 time으로 시작합니다. 모든 타이밍 관련 응용 프로그램 옵션을 보고하려면 '`report_app_options time.*`'를 사용합니다.
- 변경된 (기본값이 아닌) 값이 있는 모든 응용 프로그램 옵션을 나열하려면 '`report_app_options -non_default`' 명령을 사용할 수 있습니다.
- '`write_script -include app_options`' 명령을 사용하면 현재 작업 디렉토리에 `wscript`라는 디렉토리가 생성됩니다. 이 디렉토리에는 `design.tcl`이라는 파일이 포함되어 있으며, 모든 변경된 응용 프로그램 옵션 설정이 포함됩니다. 이 파일은 필요에 따라 편집한 후 소스로 사용할 수 있습니다.

## Setting an Application Option Value

There are several ways to set an application option value

```tcl
set_app_options -name time.remove_clock_reconvergence_pessimism \
                -value true

set_app_options -list {
  time.remove_clock_reconvergence pessimism true
  time.aocvm_enable analysis true }
# > Discouraged! Can cause unreadable and hard-to-debug scripts

# Query the value:
get_app_option_value -name time.remove_clock_reconvergence_pessimism true
```

There are several ways to set an application option value, for example:The first method is, set the required app option explicitly by using the command, `set_app_options`, and example is shown here. Other way of setting the app option is, use `-list` option, along with the `set_app_options`, where all required app options will be mentioned. But this method is not recommended because, it can cause unreadable, and hard-to-debug scripts. And user can query the value by using the command, `get_app_option_value`, as shown here.  
응용 프로그램 옵션 값을 설정하는 방법은 여러 가지가 있습니다. 첫 번째 방법은 `set_app_options` 명령을 사용하여 필요한 응용 프로그램 옵션을 명시적으로 설정하는 것입니다. 예시가 여기에 표시되어 있습니다.  
다른 응용 프로그램 옵션 설정 방법은 `set_app_options`와 함께 `-list` 옵션을 사용하는 것입니다. 이 방법은 모든 필요한 응용 프로그램 옵션이 나열됩니다. 그러나 이 방법은 가독성이 떨어지고 디버깅하기 어려운 스크립트를 유발할 수 있으므로 권장되지 않습니다.  
사용자는 `get_app_option_value` 명령을 사용하여 값을 쿼리할 수도 있습니다. 예시가 여기에 표시되어 있습니다.
{: .notice--warning}

## Or Use the GUI

![2023-05-12-or-use-gui.png]({{site.baseurl}}/assets/images/2023-05-12-or-use-gui.png)

Use the GUI to show or editing the value of the application options.It is recommended that to explore more on this during the lab sessions.  
GUI를 사용하여 응용 프로그램 옵션의 값을 표시하거나 편집할 수 있습니다. 랩 세션 중에 이에 대해 자세히 알아보는 것을 권장합니다.
{: .notice--warning}

## Scope of Application Options

![2023-05-12-scope-of-app-opt.png]({{site.baseurl}}/assets/images/2023-05-12-scope-of-app-opt.png)

- All application options have a system default value. Either with specified value, or with empty.
- Application options have either a block scope or a global scope, and the scope cannot be changed by the user.

- 모든 응용 프로그램 옵션은 시스템의 기본값을 가지고 있습니다. 이 값은 지정된 값이거나 비어 있을 수 있습니다.
- 응용 프로그램 옵션은 블록 범위(block scope) 또는 전역 범위(global scope) 중 하나를 가지며, 사용자는 범위를 변경할 수 없습니다.

## Block-Scoped Application Options

- Block-scoped application options apply to the current block,
and are saved with the block
  - Block value applied with: `set_app_options -name ... -value ...`
- Block value overrides system default value
- To report only changed block app options:
  - `report_app_options -non_default -block [current_block]`
- While it is possible to override the system default with a user default value, this **is not recommended**
  - Applied with: `set_app_options -as_user_default -name`
  - Since there is no practical advantage of doing this, and the setting is not saved with the block, this is **not recommended** - apply block value(s) instead

----------

블록 범위의 응용 프로그램 옵션은 현재 블록에 적용되며, 블록과 함께 저장됩니다.  

- 블록 값은 `set_app_options -name ... -value ...` 명령을 사용하여 적용할 수 있습니다.

블록 값은 시스템의 기본값을 덮어씁니다. 변경된 블록 응용 프로그램 옵션만 보고하려면 `report_app_options -non_default -block [current_block]`을 사용할 수 있습니다.  

시스템 기본값을 사용자 기본값으로 덮어쓰는 것도 가능하지만, 이는 권장되지 않습니다.  

- `set_app_options -as_user_default -name ...`과 같이 적용할 수 있습니다.
- 이는 실제로는 유용한 이점이 없으며, 설정이 블록과 함께 저장되지 않기 때문에 권장되지 않습니다. 대신 블록 값으로 적용할 수 있습니다.  

## Global-Scoped Application Options

- **Global-scoped** application options apply to **all blocks** loaded in the current tool session, and are **not** saved with the block(s)
  - Global value applied with: `set_app_ options -list|-name ...`
  - Can be specified before any block is opened
- Global value overrides system default value
- To report only changed global app options:
  - `report_app_options -non_default -global`
- Good practice: Include **global** application option settings in a file to be sourced upon tool start-up

----------

전역 범위의 응용 프로그램 옵션은 현재 도구 세션에 로드된 모든 블록에 적용되며, 블록과 함께 저장되지 않습니다.

- 전역 값은 `set_app_options -list|-name ...`과 같이 적용할 수 있습니다.
- 어떤 블록도 열리기 전에 지정할 수 있습니다.

전역 값은 시스템의 기본값을 덮어씁니다.

변경된 전역 응용 프로그램 옵션만 보고하려면 `report_app_options -non_default -global`을 사용할 수 있습니다. 전역 범위의 응용 프로그램 옵션 설정은 도구 시작 시 소스로 사용될 파일에 포함하는 것이 좋은 실천 방법입니다.  

참고: 전역 범위의 응용 프로그램 옵션에 대해 `-as_user_default` 옵션은 무시됩니다.

## Summary

You should now be able to

- Use attributes to retrieve design information
- Perform block/library manipulation operations(open, save, rename, copy)
- Query and set application options to control IC Compiler II's behavior

----------

이제 다음을 수행할 수 있어야 합니다:

- 속성을 사용하여 설계 정보를 검색할 수 있습니다.
- 블록 또는 라이브러리 조작 작업을 수행할 수 있습니다.
- IC Compiler II의 동작을 제어하기 위해 응용 프로그램 옵션을 쿼리하고 설정할 수 있습니다.
