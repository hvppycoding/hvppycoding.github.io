---
title: "IC Compiler II - 08 Design Setup"
excerpt: "Block-level Implementation"
date: 2023-05-16 07:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - IC Compiler II
---

## Objectives

- Create a design library
- Load the netlist and power intent (UPF)
- Apply the floorplan
- Load the scan chain definitions (scan-DEF)
- Connect PG and enable tie cells

## IC Compiler II Flow

![2023-05-16-iccii-flow-design-setup.png]({{site.baseurl}}/assets/images/2023-05-16-iccii-flow-design-setup.png){: .align-center}  

The flow chart show the general ICCII flow, in this module we will focus on Design Setup.  
플로우 차트는 일반적인 ICCII 플로우를 보여줍니다. 이 모듈에서는 디자인 설정에 초점을 맞출 것입니다.  
{: .notice--warning}  

## Design Setup

![2023-05-16-design-setup.png]({{site.baseurl}}/assets/images/2023-05-16-design-setup.png){: .align-center}  

For design setup, first we need design library. It acts like a container for your design data like RTL source code, constraints, tech related information, RC models. We will discuss how to create the orange colored box present at the center of the slide.Design library has references to the all CLIBS or NDMs of Std cells, IP blocks of the design.While creating the design library user can also read technology information in the form of tech file. As well as RC model files.The solid arrow indicates that the data from these files are saved in the design block, design library or reference library. This means that once this data is loaded, and the design database is saved, the files are no longer needed or used, when the design and/or library is closed and subsequently re-opened. The dotted-line arrow indicates that only "pointers" to this data are saved in the design library. This means that after this data is loaded, even after the design database is saved, the reference libraries must remain available to be re-loaded, when the design, the design library and/or the Fusion Compiler session is closed and subsequently re-opened. The technology file and TLUPlus files can be loaded into a technology-only reference library, and/or in the design library. Now, come to the design part, once you read the gate level netlist, then the block will be created inside the design library. Apart from these inputs, additionally you will also provide, other inputs like Power intent information, Floorplan data, Timing setup information through SDC and some user defined Tcl files and Scan chain information through SCAN-DEF. This completes your design setup process.  
디자인 설정을 위해서는 먼저 디자인 라이브러리가 필요합니다. 이 라이브러리는 RTL 소스 코드, 제약 조건, 기술 관련 정보, RC 모델과 같은 디자인 데이터를 저장하는 컨테이너 역할을 합니다. 이 슬라이드의 중앙에 있는 주황색 상자를 생성하는 방법에 대해 논의할 것입니다. 디자인 라이브러리에는 디자인의 모든 CLIB 또는 NDM (표준 셀) 및 IP 블록에 대한 참조가 포함됩니다. 디자인 라이브러리를 생성하는 동안 사용자는 기술 파일 형식으로 기술 정보를 읽을 수도 있습니다. 또한 RC 모델 파일도 읽을 수 있습니다. 실선 화살표는 이러한 파일에서의 데이터가 디자인 블록, 디자인 라이브러리 또는 참조 라이브러리에 저장된다는 것을 나타냅니다. 즉, 이러한 데이터가 로드되고 디자인 데이터베이스가 저장된 후에는 해당 파일이 더 이상 필요하지 않거나 사용되지 않습니다. 점선 화살표는 디자인 라이브러리에 이러한 데이터에 대한 "포인터"만 저장된다는 것을 나타냅니다. 즉, 이러한 데이터가 로드된 후에도 디자인 데이터베이스가 저장된 후에도 참조 라이브러리가 다시 로드되어야 한다는 것을 의미합니다. 기술 파일과 TLUPlus 파일은 기술 전용 참조 라이브러리와/또는 디자인 라이브러리에 로드될 수 있습니다. 이제 디자인 부분으로 넘어가면, 게이트 레벨 넷리스트를 읽은 후에 디자인 라이브러리 내에 블록이 생성됩니다. 이 외에도 파워 인텐트 정보, 플로어플랜 데이터, SDC를 통한 타이밍 설정 정보 및 사용자 정의 Tcl 파일, SCAN-DEF를 통한 스캔 체인 정보와 같은 추가 입력 데이터를 제공해야 합니다. 이로써 디자인 설정 프로세스가 완료됩니다.  
{: .notice--warning}  

## Create a "Container": The Design Library

Create a design library

- Specify the technology and cell libraries
- Created in memory only (by default)

```tcl
lappend search_path /x/y/clibs
create_lib orca.dlib        \
    -technology abc14_9m.tf \
    -ref_libs {abc14_std.ndm abc14_srams.ndm abc14_ip.ndm}
```

- If `-use_technology_lib` is used, the technology library **must** be listed in `-ref_libs`
- If you set `lib.setting.on_disk_operation` to `true`, `create_lib` writes to disk upon creation

The first step in data-setup is to create the design library. This entails giving the library a user-defined name, and specifying the technology library and  reference libraries to be used with this design library. The design library saves "pointers" to the reference and technology libraries (saved as absolute paths). When a design library is closed, and subsequently re-opened, these libraries must be available in the UNIX directory structure, to be re-loaded. Note that when you use a separate technology library, even though that library is specified with the `-use_technology_lib` option, it has to also be listed in `-ref_libs` along with all the other reference libraries. By default, the design library created in memory only, user can set `lib_setting_on_disk_operation` to true, and then `create_lib` writes to disk upon creation.  
데이터 설정의 첫 번째 단계는 디자인 라이브러리를 생성하는 것입니다. 이 단계에서는 라이브러리에 사용자 정의 이름을 지정하고, 이 디자인 라이브러리와 함께 사용할 기술 라이브러리 및 참조 라이브러리를 지정합니다. 디자인 라이브러리는 참조 및 기술 라이브러리에 대한 "포인터"를 저장합니다(절대 경로로 저장됨). 디자인 라이브러리가 닫히고 다시 열릴 때, 이러한 라이브러리들은 UNIX 디렉토리 구조에서 사용 가능해야 하며 다시 로드되어야 합니다. 별도의 기술 라이브러리를 사용하는 경우에도, 해당 라이브러리는 `-use_technology_lib` 옵션으로 지정되지만 모든 참조 라이브러리와 함께 `-ref_libs`에 명시되어야 합니다. 기본적으로, 디자인 라이브러리는 메모리에만 생성됩니다. 사용자는 `lib_setting_on_disk_operation`을 true로 설정하고, `create_lib`이 생성 시에 디스크에 작성되도록 설정할 수 있습니다.  
{: .notice--warning}  

## Block Compression

- Blocks can be compressed to reduce a design library’s disk size

- To enable compression, set the following before creating the design library 
  - All blocks will be compressed when saved
  - `set_app_options -name lib.setting.compress_design_lib -value true`
- Alternatively, explicitly compress the current block or all blocks when saving:
  - `save_block -compress`
  - `save_lib -compress`
  - Compression may increase the `save_` command execution time

Blocks can be compressed to reduce a design library's disk size. To enable compression, we can set the following app option before creating the design library, and all blocks will be compressed when saved: Alternatively, user can explicitly compress the current block or all blocks when saving by using save block or save lib command with -compress option respectively.  
블록은 디자인 라이브러리의 디스크 크기를 줄이기 위해 압축될 수 있습니다. 압축을 활성화하려면 디자인 라이브러리를 생성하기 전에 다음과 같은 앱 옵션을 설정하면 됩니다. 이렇게 하면 모든 블록이 저장될 때 압축됩니다. 또는 현재 블록이나 모든 블록을 저장할 때 -compress 옵션을 사용하여 명시적으로 압축할 수도 있습니다.  
{: .notice--warning}  

## Reference Library Paths

- When creating a design library, reference libraries can be specified in two ways
- Libraries found using `search_path`, stored as **absolute paths**

```tcl
lappend search_path ../libs
create_lib orca.dlib    \
    -ref_libs {
      abc14_lvt_std.ndm 
      abc14_hvt_std.ndm ...
    }
```

- Libraries specified with path, stored **as given** (here: **relative paths**)

```tcl
create_lib orca.dlib    \
    -ref_libs {
      ../libs/abc14_lvt_std.ndm
      ../libs/abc14_hvt_std.ndm ...
    }
```

When creating a design library, reference libraries can be specified in two ways: •Libraries can be found using legacy app variable search_path, during this case, reference libraries are stored as absolute paths. •Another way of specifying the path for reference libraries is by using their complete path as shown here. These types of paths are called as relative paths. It is completely acceptable to rely on the search_path for some libraries, while including relative and/or absolute paths with other libraries, for example: In a hierarchical design, the "sub-block" libraries might be specified with a relative path, and the standard cell and hard macro libraries are stored as an absolute path, either using the search_path variable, or included in the ref_libs name. If the design library in this example is moved, as long as the "block" libraries are moved along with it, all references will be resolved (linked to a reference library).  
디자인 라이브러리를 생성할 때, 참조 라이브러리는 두 가지 방법으로 지정할 수 있습니다. • 라이브러리는 기존의 앱 변수 search_path를 사용하여 찾을 수 있으며, 이 경우 참조 라이브러리는 절대 경로로 저장됩니다. • 참조 라이브러리의 경로를 지정하는 또 다른 방법은 완전한 경로를 사용하는 것입니다. 이러한 유형의 경로는 상대 경로라고도 합니다. 일부 라이브러리에는 search_path에 의존하고, 다른 라이브러리에는 상대 경로와/또는 절대 경로를 포함하는 것이 완전히 허용됩니다. 예를 들어, 계층적 디자인에서 "하위 블록" 라이브러리는 상대 경로로 지정할 수 있고, 표준 셀 및 하드 매크로 라이브러리는 절대 경로로 저장되어 있으며, search_path 변수를 사용하거나 ref_libs 이름에 포함됩니다. 이 예에서 디자인 라이브러리가 이동되더라도 "블록" 라이브러리가 함께 이동되면 모든 참조가 해결되어 (참조 라이브러리에 연결됨)됩니다.  
{: .notice--warning}  

## Moving Design and/or Reference Libraries

- With **absolute** reference library paths:
  - If the design library moves, nothing changes - libraries still resolve
  - If the reference libraries move, you need to update the search path and rebind
- With **relative** reference library paths:
  - Reference libraries need to move along with the design library
  - If they do not move together, you need to update the reference library paths

In the real time scenarios, the location of a design library can be changed for example, a user copies someone's design library to their local working directory. During such situations how reference libraries will be resolved. Let us consider with absolute reference library paths, where reference library paths are specified by using search path variable: •If the design library moves, nothing changes - libraries still resolve •If the reference libraries move, you need to update the search path and user need to perform rebinding operation Now let us consider With relative reference library paths, where user will mention the complete path for reference libraries: •In such cases, Reference libraries need to move along with the design library •If they do not move together, you need to update the reference library paths  
실제 시나리오에서는 디자인 라이브러리의 위치가 변경될 수 있습니다. 예를 들어, 사용자가 다른 사람의 디자인 라이브러리를 로컬 작업 디렉토리로 복사하는 경우입니다. 이러한 상황에서 참조 라이브러리는 어떻게 해결되는지 살펴보겠습니다. 절대 참조 라이브러리 경로를 사용하는 경우를 고려해 보겠습니다. 여기서 참조 라이브러리 경로는 search path 변수를 사용하여 지정됩니다. •디자인 라이브러리가 이동하더라도 아무 변화가 없습니다. 라이브러리는 여전히 해결됩니다. •참조 라이브러리가 이동하면, search path를 업데이트해야 합니다. 사용자는 rebinding 작업을 수행해야 합니다. 이제 상대적인 참조 라이브러리 경로를 고려해 보겠습니다. 사용자는 참조 라이브러리를 위한 완전한 경로를 지정합니다. •이러한 경우 참조 라이브러리는 디자인 라이브러리와 함께 이동해야 합니다. •함께 이동하지 않는 경우, 참조 라이브러리 경로를 업데이트해야 합니다.  
{: .notice--warning}  

## Rebind a Design Library

```tcl
open_lib $DESIGN_LIBRARY
# Error: File '/old_location/saed32_hvt_std.ndm' cannot be found using search path of: ''. (FILE-002)

set search_path "/new_location"
set_ref_libs -rebind
report_ref_libs
save_lib
```

With the example, as you see after the first command, it may be that some libraries are no longer at the "old_location", in which case an Error is reported. If other libraries are still available at the old location, they will continue to be used. To mitigate this error, user needs to update the search path variable with new location and then perform rebind. After rebinding, make sure to check "report_ref_libs", which should show the new absolute paths to the reference libraries, and make sure to save the library with recent updates.  
예제에서 보시는 바와 같이 첫 번째 명령을 실행한 후에는 "old_location"에 일부 라이브러리가 더 이상 존재하지 않을 수 있습니다. 이 경우 오류가 보고됩니다. 다른 라이브러리는 여전히 이전 위치에서 사용될 것입니다. 이 오류를 완화하기 위해 사용자는 검색 경로 변수를 새 위치로 업데이트한 다음 rebinding 작업을 수행해야 합니다. rebinding 작업 이후에는 "report_ref_libs"를 확인하여 참조 라이브러리의 새로운 절대 경로가 표시되는지 확인하고, 최신 업데이트가 포함된 라이브러리를 저장해야 합니다.  
{: .notice--warning}  

## Restricting Cells in Specific Modules/Regions

- Specific modules or design regions may need to be restricted to use a sub-set of all available reference library cells, for example:
  - Only cells with a double site height
  - Only high or low speed cells
  - No ultra-LVth cells allowed
- Use `set_target_library_subset` for these purposes
  - See following example

Specific modules or design regions may need to be restricted to use a sub-set of all available reference library cells, for example: •Only cells with a double site height •Only high or low speed cells •No ultra-LVth cells allowed We can use `set_target_library_subset` for these purposes.  
특정 모듈 또는 디자인 영역은 사용 가능한 참조 라이브러리 셀 중 일부만 사용해야 할 수 있습니다. 예를 들어 다음과 같은 경우에는 특정 셀 서브셋을 사용합니다: •더블 사이트 높이를 갖는 셀만 •고속 또는 저속 셀만 •초저전압(LVth) 셀은 허용하지 않음. 이러한 목적으로 `set_target_library_subset`을 사용할 수 있습니다.
{: .notice--warning}  

## Target Library Subset Example

- Sub-design `Sub1`/`suba` is timing-critical and can use LVT cells in addition to SVT/HVT cells, while the rest of the design uses only SVT/HVT cells

![2023-05-16-target-lib-subset-example.png]({{site.baseurl}}/assets/images/2023-05-16-target-lib-subset-example.png){: .align-center}  

```tcl
set_target_library_subset -top {lib_HVT lib_SVT}
set_target_library_subset -objects Sub1/suba {lib_HVT lib_SVT lib_LVT}
```

Pre-existing LVT cells at top are not affected, unless they are optimized  

Let us consider the hierarchical design Top as shown in the slide. In this nested hierarchical design, where sub-design suba is timing critical block, so user as to enable the usage of Low VT cells in addition to regular VT/High VT. The piece of code shows, how user can accomplish the above requirement with the help of set target library subset command.  
Hierarchical design는 Top으로 표시된 슬라이드에 나와 있듯이, 이 중첩된 계층적 디자인에서 suba라는 하위 디자인이 타이밍에 중요한 블록임을 가정해 보겠습니다. 따라서 사용자는 일반 VT/고 VT 외에도 Low VT 셀의 사용을 허용해야 합니다. 아래의 코드 조각은 set target library subset 명령을 사용하여 위의 요구 사항을 충족하는 방법을 보여줍니다.  
{: .notice--warning}  

## Perform Automated Design Setup

