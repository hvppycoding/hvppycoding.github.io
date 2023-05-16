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

- If you used Design Compiler for synthesis, consider using Design Compiler’s write_icc2_files to create all necessary inputs for a seamless transfer of data to ICC II (Verilog, UPF, floorplan, timing constraints, settings, ...)

```bash
dcnxt_shell-topo> write_icc2_files -output ORCA_DC_icc2 \
    -golden_floorplan ORCA_TOP.fp/floorplan.tcl
```

- The golden floorplan is the one created by ICC Il write floorplan - when specified, Design Compiler only writes incremental information (→ std cell placement, layer constraints)
- In ICC II, the following sets up the entire design:

```bash
icc2_shell> source ORCA_DC_icc2/ORCA.icc2_script.tcl
```

If you used Design Compiler for synthesis, consider using Design Compiler's `write_icc2_files` command  to create all necessary inputs for a seamless transfer of data to ICC II as shown here. It will dump out all collaterals required for ICC2 tool to perform placement and routing which includes Verilog file, UPF, floorplan, timing constraints, settings, etc. … The golden floorplan is the one created by ICC II `write_floorplan` - when specified, Design Compiler only writes incremental information ( std cell placement, layer constraints) In ICCII, the following sets up the entire design: `Icc2_shell> source DC_icc2/ORCA.icc2_script.tcl`  
만약 합성에 Design Compiler를 사용했다면, 다음과 같이 Design Compiler의 `write_icc2_files` 명령을 사용하여 ICC II로 데이터를 원활하게 전송하기 위해 필요한 모든 입력을 생성하는 것을 고려해보십시오. 이 명령은 ICC2 도구가 배치 및 라우팅을 수행하는 데 필요한 모든 자료들(Verilog 파일, UPF, 플로어플랜, 타이밍 제약 조건, 설정 등)을 출력할 것입니다. 골든 플로어플랜은 ICC II의 `write_floorplan`으로 생성된 플로어플랜입니다. 이 경우, Design Compiler는 증분 정보만 작성합니다(표준 셀 배치, 레이어 제약 조건 등). ICCII에서는 다음과 같이 디자인 전체를 설정합니다: `Icc2_shell> source DC_icc2/ORCA.icc2_script.tcl`  
{: .notice--warning}  

## Read the Netlist and Create a Design

![2023-05-16-read-the-netlist-and-create-design.png]({{site.baseurl}}/assets/images/2023-05-16-read-the-netlist-and-create-design.png){: .align-center}  

Let us see how to read the gate level netlist into the design library. Read Verilog is the command used to read the gate level netlist file into the design library. Once you read in the netlist file, design view of the block will be created inside the design library with the block name as shown here. The `-top` option of `read_verilog` defines the current design. If not specified, the last module in the RTL becomes the current design. `link_block` tries to resolve all instantiated sub-design or leaf cell references; A sub-design reference is resolved by locating its "design" in memory (a design is created for each Verilog "module");  Leaf cells are resolved by finding them in one of the CLIBs. If an instance cannot be resolved, a warning is issued.  
게이트 레벨 넷리스트를 디자인 라이브러리에 읽어들이는 방법을 살펴보겠습니다. Read Verilog은 게이트 레벨 넷리스트 파일을 디자인 라이브러리에 읽어들이는 명령입니다. 넷리스트 파일을 읽으면, 블록의 디자인 뷰가 디자인 라이브러리 내에 블록 이름으로 생성됩니다. `read_verilog`의 `-top` 옵션은 현재 디자인을 정의합니다. 지정되지 않으면 RTL의 마지막 모듈이 현재 디자인이 됩니다. `link_block`은 모든 인스턴스화된 하위 디자인 또는 리프 셀 참조를 해결하려고 시도합니다. 하위 디자인 참조는 메모리에서 해당 "디자인"(각각의 Verilog "module"에 대해 디자인이 생성됨)을 찾아 해결합니다. 리프 셀은 CLIB 중 하나에서 찾아내어 해결합니다. 인스턴스를 해결할 수 없는 경우 경고가 발생합니다.  
{: .notice--warning}  

## Linking

- Linking is still considered “successful” if there are missing references:

```bash
icc2_shell> link_block
Using libraries: ORCA_TOP.dlib saed32_1p9m_tech saed32_hvt_std saed32_rvt_std ...
Linking block ORCA_TOP.dlib:ORCA_TOP.design
Warning: Unable to resolve reference to 'SRAMLP2RW64x32' in module 'SDRAM_TOP'. (LNK-005)
Information: Block ‘ORCA_TOP.dlib:ORCA_TOP.design’ has 4 unbound instances. (LNK-073)
Information: Block ‘ORCA_TOP.dlib:ORCA_TOP.design’ has 1 unresolved references. (LNK-074)
Design ‘ORCA TOP' was successfully linked.
```

- If you want ICC II to cause a Tcl error instead: `set_message_info -id LNK-005 -stop_on`
- A successful link allows you to continue with the flow if you choose
  - For example, you may decide to turn unresolved instances into black boxes
- To report linking issues later, use:

```tcl
report_design_mismatch -verbose
get_cells -hierarchical -filter is_unbound
```

Linking is still considered "successful" if there are missing references, it can allows user to continue with the flow. For example, user can turn these unresolved instances into black boxes through: create_blackbox To report linking issues later, use: `report_design_mismatch -verbose` Note that: even though linking might be successful, this does not mean that `place_opt` will run. You need to resolve unbound reference, for example, by creating black boxes for them, otherwise `place_opt` will abort.  
참조가 누락된 경우에도 링킹은 여전히 "성공적으로" 간주됩니다. 이는 사용자가 플로우를 계속할 수 있도록 허용합니다. 예를 들어, 사용자는 다음을 통해 이러한 해결되지 않은 인스턴스를 블랙박스로 전환할 수 있습니다: create_blackbox. 나중에 링킹 문제를 보고하려면 다음을 사용하십시오: `report_design_mismatch -verbose`. 참고할 점은 링킹이 성공적이더라도 `place_opt`가 실행되지 않는다는 것입니다. 미결된 참조를 해결해야 하며, 이를 위해 해당 인스턴스에 대해 블랙박스를 생성하는 등의 조치를 취해야 합니다. 그렇지 않으면 `place_opt`가 중단됩니다.  
{: .notice--warning}  

## Load nxtgrd or TLUPlus Parasitic RC Models

- nxtgrd or TLUPlus files (used for parasitic net RC modeling) must be loaded as follows:

```tcl
# Add the -layermap option to map itf to tf layer names(e.g. cm1 → M1)
# Name used later during corner setup
read_parasitic_tech -tlup $TLUPLUS_MAX_FILE -name maxTLU
read_parasitic_tech -tlup $TLUPLUS_MIN_FILE -name minTLU
get_parasitic_techs {maxTLU minTLU}
report_lib -parasitic_tech orca.dlib
```

- With some advanced nodes, nxtgrd files are used instead of TLUPlus
  - The nxtgrd files contain the TLUPlus data
  - You use the same option `-tlup` to specify the nxtgrd file
- TLUPlus and nxtgrd files <20MB are copied into the design library, otherwise pointers to those files are stored in the design library

Load nxtgrd or TLUPlus Parasitic RC Models RC models are fed into the tool in two ways, either in the form of nxtgrd file or in the form of TLU plus file. nxtgrd file is the output of Synopsys extraction tool called StarRC. TLU plus is another form of loading the RC parasitic to the tool. Read parasitic tech is the used to load the parasitic information to the tool. User can give the symbolic name with the option `-name`.  These name options can be used later in the corner setup. With `-tlup` option user can mention min and max TLU plus files. To know about the parasitics information of the design, user can use get parasitic techs and report lib commands as shown here. With some advanced nodes, nxtgrd files are used instead of TLUPlus These nxtgrd files contain the TLUPlus data and user can use the same option `-tlup` to specify the nxtgrd file If an nxtgrd/TLUPlus file is 20MB or smaller, it is copied into the design library (DLIB). Otherwise, only a pointer to the file is stored!  
RC 모델은 nxtgrd 파일 또는 TLU Plus 파일의 형태로 도구로 입력됩니다. nxtgrd 파일은 Synopsys 추출 도구인 StarRC의 출력입니다. TLU Plus는 RC 파라사이틱을 도구로 로딩하는 또 다른 방법입니다. Read parasitic tech은 파라사이틱 정보를 도구에 로드하는 데 사용됩니다. 사용자는 `-name` 옵션과 함께 심볼릭 이름을 지정할 수 있습니다. 이러한 이름 옵션은 나중에 코너 설정에서 사용할 수 있습니다. `-tlup` 옵션을 사용하여 사용자는 최소 및 최대 TLU Plus 파일을 지정할 수 있습니다. 디자인의 파라사이틱 정보를 알고 싶다면 get parasitic techs 및 report lib 명령을 사용할 수 있습니다. 일부 고급 노드에서는 TLUPlus 대신 nxtgrd 파일이 사용됩니다. 이 nxtgrd 파일에는 TLUPlus 데이터가 포함되어 있으며, 사용자는 동일한 `-tlup` 옵션을 사용하여 nxtgrd 파일을 지정할 수 있습니다. nxtgrd/TLUPlus 파일의 크기가 20MB 이하인 경우, 디자인 라이브러리 (DLIB)에 복사됩니다. 그렇지 않으면 파일에 대한 포인터만 저장됩니다!  
{: .notice--warning}  

## Placement Sites and Symmetry

![2023-05-16-placement-stes-and-symmetry.png]({{site.baseurl}}/assets/images/2023-05-16-placement-stes-and-symmetry.png){: .align-center}  

The technology file does not contain both the symmetry requirements for cell placement and the default site for floorplan initialization. User needs to set the site attributes: symmetry and is_default through set_attribute. The is_default site attribute is used when creating the default site arrays/rows with initialize_floorplan, when multiple site definitions exist. If no default site is defined, the first site definition is selected. Please refer the notes section for an example.  
기술 파일에는 셀 배치를 위한 대칭 요구 사항과 플로어플랜 초기화를 위한 기본 사이트(default site)가 모두 포함되어 있지 않습니다. 사용자는 set_attribute를 통해 사이트 속성(symmetry 및 is_default)을 설정해야 합니다. is_default 사이트 속성은 여러 사이트 정의가 있는 경우 initialize_floorplan으로 기본 사이트 배열/행을 생성할 때 사용됩니다. 기본 사이트가 정의되지 않은 경우 첫 번째 사이트 정의가 선택됩니다. 예제에 대해서는 참고 사항을 참조하십시오.  
{: .notice--warning}  

## Routing Direction

- The technology file does not contain information on the preferred routing direction
- Specify the routing directions using attributes (vendor supplied):

```tcl
set_attribute [get_layers {M1 M3 M5 M7}] \
    routing direction horizontal
set_attribute [get_layers {M2 M4 M6 M8}] \
    routing direction vertical
```

The technology file does not contain information on the preferred routing direction, user needs to specify the routing directions using set_attribute command shown here. Routing directions do not depend on the technology, they depend on the physical standard cell libraries. Cell designers design cells in a certain way, and decide how routing should be done in order to achieve the highest yields.  
기술 파일에는 선호되는 라우팅 방향에 대한 정보가 포함되어 있지 않습니다. 사용자는 다음과 같이 set_attribute 명령을 사용하여 라우팅 방향을 지정해야 합니다. 라우팅 방향은 기술에 의존하는 것이 아니라 물리적 표준 셀 라이브러리에 의존합니다. 셀 디자이너들은 셀을 특정 방식으로 설계하고, 가장 높은 수율을 달성하기 위해 어떻게 라우팅이 이루어져야 하는지 결정합니다.  
{: .notice--warning}  

## PRF Equivalent Support

In this slide we show the way to define  PRF attribute  PRF stands for physical_rule_file. With loading PRF into design, we don't need to use "set_attribute" anymore. Here is an example to use PRF define site_def's "symmetry" and "is_default" attribute. For more PRF info, please refer to Library Manager user guide.  
이 슬라이드에서는 PRF 속성(physical_rule_file)을 정의하는 방법을 보여줍니다. PRF를 디자인에 로드하면 더 이상 "set_attribute"을 사용할 필요가 없습니다. 다음은 PRF를 사용하여 site_def의 "symmetry" 및 "is_default" 속성을 정의하는 예입니다. PRF에 대한 자세한 정보는 Library Manager 사용자 가이드를 참조하십시오.  
{: .notice--warning}  

Continuation of the previous slide, Here is an example to use PRF define layer's "routing_direction" attribute.  
이전 슬라이드의 연장선으로, 다음은 PRF를 사용하여 레이어의 "routing_direction" 속성을 정의하는 예입니다.
{: .notice--warning}  

## Load the Power Intent

- Unified Power Format (UPF) is an IEEE standard (1801)
- UPF defines the entire “power intent and structure” of a multi-voltage design
- The netlist that was read in will already contain multi-voltage (MV) components from Synthesis, like level shifters (LS), isolation cells (ISO), retention registers

IC Compiler II by default is MV-aware and MCMM-aware.  That is, even if you don't load a UPF, internally ICC II will create a default UPF file, which all the ICC II engines will be based on. The default UPF only creates one power domain, VDD/VSS SN (Supply Net), SP (Supply Port) and  CSN (connect_supply_net).  If your design has double VDD/VSSDVDD/DVSS, then the default UPF will not work.  In that case, you can set the few application options. To know about the app options please refer note section. The netlist that was read in will already contain multi-voltage (MV) components from Synthesis, like level shifters (LS), isolation cells (ISO), retention registers.  
IC Compiler II는 기본적으로 MV(Multi-Voltage) 및 MCMM(Multi-Channel Multi-Mode)을 지원합니다. 즉, UPF를 로드하지 않아도 내부적으로 ICC II는 모든 엔진이 기반으로 하는 기본 UPF 파일을 생성합니다. 기본 UPF는 하나의 파워 도메인(VDD/VSS SN, SP 및 CSN)만 생성합니다. 디자인이 더블 VDD/VSSDVDD/DVSS를 가지고 있는 경우, 기본 UPF는 작동하지 않습니다. 이 경우, 일부 응용 프로그램 옵션을 설정할 수 있습니다. 응용 프로그램 옵션에 대해서는 참고 사항을 참조하십시오. 읽은 넷리스트는 이미 합성에서 MV 구성 요소(레벨 쉬프터, 아이솔레이션 셀, 리텐션 레지스터 등)를 포함하고 있을 것입니다.  
{: .notice--warning}  

## Load and Commit the UPF

- To load the power intent file(s)

```tcl
load_upf TOP.upf
load_upf -scope I_B B.upf
```

- After loading all UPF files

```tcl
commit_upf
```

- References all UPF objects and performs primary power/ground checks
- Associates strategies for instantiated power management (PM) cells
- It is recommended to run this command after loading the UPF to ensure that everything is resolved
  - `commit_upf` is run automatically when invoking `check_mv_design` or `place_opt`

You can read multiple UPF files using `load_upf`. The files can reference objects that don't already exist, if they are defined in a subsequent file you load. `commit_upf` will then make sure that everything is resolved correctly. Note that `commit_upf` is run automatically when the user invoking check_mv_design or place_opt command.  
여러 개의 UPF 파일을 `load_upf`를 사용하여 읽을 수 있습니다. 파일은 이미 존재하지 않는 객체를 참조할 수 있습니다. 단, 해당 객체가 이후에 로드하는 파일에서 정의되어 있는 경우입니다. `commit_upf`는 모든 것이 올바르게 해결되도록 보장합니다. `commit_upf`는 사용자가 check_mv_design 또는 place_opt 명령을 실행할 때 자동으로 실행됩니다.  
{: .notice--warning}  

## Support of Golden-UPF and UPF'/UPF'' Flows

![2023-05-16-golden-upf-flow.png]({{site.baseurl}}/assets/images/2023-05-16-golden-upf-flow.png){: .align-center}  

UPF'/UPF'' flow: •All UPF changes performed by Design Compiler are captured in a UPF' file •All UPF' changes performed by ICC II are captured in a UPF'' file Golden UPF flow: •User UPF is preserved as golden UPF (G-UPF) •All intent changes captured in supplemental UPF  (S-UPF) •You can optionally use the Verilog netlist to store all PG connectivity information, making connect_supply_net commands unnecessary in the UPF files. This can significantly simplify and reduce the overall size of the UPF files. •All name changes captured in name mapping file Design Compiler generates supply connectivity information, that is normally captured in the UPF' file. For the Golden UPF flow, that supply connectivity information can either be captured in the Supplemental-UPF file (without a "PG Netlist"), or the user has the option to capture it in the "PG Netlist", in which case that information is not needed in the S-UPF file. save_upf will write full or supplemental UPF based on the setting of  mv.upfof mv.upf.enable_golden_upf app option.  
UPF'/UPF'' 플로우:
Design Compiler에서 수행한 모든 UPF 변경 사항은 UPF' 파일에 기록됩니다. ICC II에서 수행한 모든 UPF' 변경 사항은 UPF'' 파일에 기록됩니다. Golden UPF 플로우: 사용자 UPF는 골든 UPF(G-UPF)로 보존됩니다. 모든 의도 변경 사항은 보충 UPF(S-UPF)에 기록됩니다. Verilog 넷리스트를 사용하여 모든 전원 그라운드(PG) 연결 정보를 저장할 수도 있으며, 이로 인해 UPF 파일에서 connect_supply_net 명령이 필요하지 않을 수 있습니다. 이를 통해 UPF 파일의 전체 크기를 크게 간소화하고 줄일 수 있습니다. 모든 이름 변경 사항은 이름 매핑 파일에 기록됩니다. Design Compiler는 전원 연결 정보를 생성하며, 일반적으로 UPF' 파일에 기록됩니다. Golden UPF 플로우에서는 해당 전원 연결 정보를 보충 UPF 파일(S-UPF)에 기록하거나, "PG Netlist" 없이 보충 UPF 파일(S-UPF)에 필요하지 않은 경우 "PG Netlist"에 기록할 수 있습니다. save_upf는 mv.upf.enable_golden_upf 앱 옵션의 설정에 따라 전체 또는 보충 UPF를 작성합니다.
{: .notice--warning}  

## Load a DEF Floorplan File

If the floorplan was saved with write_def which is (not a  recommended method), load it with read def command as shown here: •By default, the tool ignores cells that exist in the DEF file but not in the netlist like tap and end-cap cells. User can use -add_def_only_objects all option  to add these cells to the netlist •Use DEF version 5.8 or later when possible (for example, colored tracks introduced in version 5.8)  
플로어플랜이 권장되지 않는 방법으로 write_def로 저장된 경우 다음과 같이 read def 명령을 사용하여 로드합니다: •기본적으로 도구는 넷리스트에는 없지만 DEF 파일에 있는 tap 및 end-cap 셀과 같은 셀을 무시합니다. 사용자는 -add_def_only_objects all 옵션을 사용하여 이러한 셀을 넷리스트에 추가할 수 있습니다. •가능한 경우 DEF 버전 5.8 이상을 사용하십시오 (예: 버전 5.8에서 도입된 색상 트랙).
{: .notice--warning}  

## Load the Floorplan

![2023-05-16-load-floorplan.png]({{site.baseurl}}/assets/images/2023-05-16-load-floorplan.png){: .align-center}  

If write_floorplan was used in ICC II to save the floorplan, we can load the floorplan with: source command shown here. Refer the note section for more information on floorplan data need to be conveyed to ICC2.  
ICC II에서 floorplan을 저장하기 위해 write_floorplan을 사용한 경우, 다음과 같이 source 명령을 사용하여 floorplan을 로드할 수 있습니다. floorplan 데이터가 ICC2에 전달되어야 하는 자세한 정보에 대해서는 참고 사항을 참조하십시오.  
{: .notice--warning}  

## Load Design Compiler SPG Information

- In Design Compiler, if SPG was used, you will need to import:
  - Standard cell placement
  - Layer soft constraints
- Design Compiler’s `write_icc2_files` creates all necessary inputs including standard cell placement and layer constraints:

```tcl
# dcnxt_shell-topo>
write_icc2 files -output ORCA_DC_icc2 \
  -golden_floorplan ORCA_TOP.fp/floorplan.tcl
```

- Load the placement and layer constraints:

```tcl
# icc2_shell> 
source ORCA_DC_icc2/ORCA.floorplan/floorplan.tcl
```

In Design Compiler, if SPG was used, you will need to import the following extra information to ICC2, thay are: •Standard cell placement •Layer soft constraints Design Compiler's `write_icc2_files` command creates all necessary inputs including standard cell placement and layer constraints as shown here. And then we can load the placement and layer constraints by sourcing floorplan.tcl file.  
Design Compiler를 사용할 경우, SPG (Global 커플링이 포함된 표준 파라사이틱 추출)를 사용한 경우에는 다음과 같은 추가 정보를 ICC2로 가져와야 합니다: •표준 셀 배치 •레이어 소프트 제약 조건 Design Compiler의 `write_icc2_files` 명령은 여기에 표시된 대로 표준 셀 배치와 레이어 제약 조건을 포함한 모든 필요한 입력을 생성합니다. 그리고 우리는 floorplan.tcl 파일을 소스로 사용하여 배치 및 레이어 제약 조건을 로드할 수 있습니다.  
{: .notice--warning}  

## Spefcify Unused Routing Layers

- By default ICC IIuses all metal layers defined in the technology file
- If using fewer metal layers this can result in:
  - Optimistic congestion analysis pre-route
  - Inaccurate delay calculations due to inaccurate parasitic RC calculations
- Specify the unused or “ignored” layers for accurate congestion and timing analysis prior to routing

```tcl
set_ignored_layers -max_routing_layer M7
report_ignored_layers
```

By default ICC II uses all metal layers defined in the technology file If using fewer metal layers this can result in: •Optimistic congestion analysis pre-route •Inaccurate delay calculations due to inaccurate parasitic RC calculations We can specify the unused or "ignored" layers for accurate congestion and timing analysis prior to routing with the help of `set_ignored_layers` command showed in the example. This command is useful if you have a technology file which defines, for example, nine metal layers, but you plan to use fewer layers, for example, only up to metal 7. The example above ignores the layers above M7 for RC and congestion estimation. It also prevents the router from routing on layers above M7. This is common for example, if you are reserving the upper layers for top-level routing, or using the top layers for the power-mesh only.  
ICC II는 기본적으로 기술 파일에 정의된 모든 메탈 레이어를 사용합니다. 그러나 더 적은 메탈 레이어를 사용하는 경우 다음과 같은 결과가 발생할 수 있습니다: •사전 라우팅 기준으로 과도한 혼잡도 분석 •정확하지 않은 RC 파라사이틱 계산으로 인한 부정확한 지연 시간 계산. `set_ignored_layers` 명령을 사용하여 라우팅 이전에 정확한 혼잡도 및 타이밍 분석을 위해 사용하지 않는 레이어를 지정할 수 있습니다(예시에서 보여준 대로). 이 명령은 예를 들어 기술 파일이 아홉 개의 메탈 레이어를 정의하지만 사용할 레이어는 메탈 7까지로 계획하는 경우 유용합니다. 위의 예시는 RC 및 혼잡도 추정을 위해 M7 이상의 레이어를 무시합니다. 또한 라우터가 M7 이상의 레이어에 라우팅하지 못하도록 합니다. 이는 예를 들어 상위 레이어를 최상위 라우팅에 보존하거나 전원 메쉬에만 상위 레이어를 사용하는 경우와 같이 일반적으로 사용됩니다.
{: .notice--warning}  

## Load Scan Chain Identification Information

- Load SCAN-DEF to help ICC II identify the scan chains and their re-ordering buckets
- Will be used during placement to re-order the scan chains to reduce congestion

```tcl
read_def ORCA_scan.def
```

![2023-05-16-load-scan-chain-info.png]({{site.baseurl}}/assets/images/2023-05-16-load-scan-chain-info.png){: .align-center}  

ICCII gets the scan chain information from a SCANDEF file, which specifies the scan chain components and their pins used for the scan path, along with reordering constraints. To get the best quality of result (QoR) for scan designs and enable repartitioning and reordering of the scan chains, you must annotate the scan chain information by loading the SCANDEF file,  and it will be used during placement to re-order the scan chains to reduce congestion.  
ICCII는 SCANDEF 파일에서 스캔 체인 정보를 가져옵니다. 이 파일은 스캔 경로에 사용되는 스캔 체인 구성 요소 및 핀들과 재정렬 제약 조건을 지정합니다. 스캔 디자인의 최상의 결과 품질(QoR)을 얻고 스캔 체인의 재분배와 재정렬을 가능하게 하려면 SCANDEF 파일을 로드하여 스캔 체인 정보를 주석으로 추가해야 합니다. 이 정보는 배치 과정에서 스캔 체인을 재정렬하여 혼잡도를 줄이는 데 사용됩니다.  
{: .notice--warning}  

## Connect PG Pins to Supply Nets

- Synthesis netlist has no P/G supply nets or connections
- Must explicitly connect P/G pins to supply nets
  - These are logical connections, not physical metal routes
  - P/G net names defined in UPF
    - Use `-net` option with `<port_pin_list>` for non-UPF designs
  - This also connects tie-high/low pins to P/G nets

```tcl
connect_pg_net
check_mv_design
```

![2023-05-16-connect-pg-pins.png]({{site.baseurl}}/assets/images/2023-05-16-connect-pg-pins.png){: .align-center}  

Synthesis netlist has no PG supply nets or connection, we must explicitly connect PG pins to supply nets through `connect_pg_net`. Any cells that are subsequently added by optimization or CTS will be automatically connected to PG nets. After "manually" adding cells (e.g. after filler cell insertion or ECOs), you need to explicitly rerun `connect_pg_net`.  
합성 넷리스트에는 PG(Power Ground) 공급 넷이나 연결이 없으므로 `connect_pg_net`을 통해 명시적으로 PG 핀을 공급 넷에 연결해야 합니다. 최적화 또는 CTS(클럭 트리 신호화)에 의해 추가되는 모든 셀은 자동으로 PG 넷에 연결됩니다. "수동으로" 셀을 추가한 후(예: 필러 셀 삽입 또는 ECO 후)에는 명시적으로 `connect_pg_net`를 다시 실행해야 합니다.  
{: .notice--warning}  

## Enable Use of Tie-High/Low Cells

- If tie high/low inputs need to be connected to tie-high/low cells, enable this as shown here
- Tie-high/low cells will be inserted during placement optimization

![2023-05-16-tie-high-low-cells.png]({{site.baseurl}}/assets/images/2023-05-16-tie-high-low-cells.png){: .align-center}  

```tcl
set_dont_touch [get_lib_cells */TIE*] false
set_lib_cell_purpose -include optimization [get_lib_cells */TIE*]
```

- The name of the reference library is determined by the library file name with the .ndm suffix removed
  - Eg. `std_cell.ndm` is named `std_cell`: `get_lib_cells std_cell/AND3*`

If tie high/low inputs need to be connected to tie-high/low cells, enable this as shown in the slide Note that: •This only works for the specific extensions .ndm, .nlib and .ndb. •The library name of a library with any other extension will be the same as its file name, for example: The library with a file name of hello.mylib will have a library name of "hello.mylib"! Tie-high/low cells will be inserted during placement optimization  
만약 tie high/low 입력을 tie-high/low 셀에 연결해야 한다면, 슬라이드에 표시된대로 이를 활성화합니다. 주의할 점은 다음과 같습니다: •이 기능은 .ndm, .nlib 및 .ndb 확장자를 가진 특정 확장자에만 작동합니다. •다른 확장자를 가진 라이브러리의 경우 라이브러리 이름은 파일 이름과 동일하게 됩니다. 예를 들어, 파일 이름이 hello.mylib인 라이브러리의 라이브러리 이름은 "hello.mylib"가 됩니다! Tie-high/low 셀은 배치 최적화 중에 삽입됩니다.
{: .notice--warning}  

## TIP: Design Library Packages

![2023-05-16-design-lib-package.png]({{site.baseurl}}/assets/images/2023-05-16-design-lib-package.png){: .align-center}  

You can pack a library at any stage of the design flow by using `write_lib_package` •Design data and reference libraries are stored into a single file, including all application options and Tcl variables When the design is unpacked by `read_lib_package`, the design is in the same state as when it was packaged.  
디자인 플로우의 어떤 단계에서든 `write_lib_package`를 사용하여 라이브러리를 패킹할 수 있습니다. •디자인 데이터와 참조 라이브러리가 모든 앱 옵션과 Tcl 변수를 포함한 단일 파일로 저장됩니다. 디자인이 `read_lib_package`에 의해 언패킹되면, 디자인은 패키징된 상태와 동일한 상태로 복원됩니다.  
{: .notice--warning}  

## Usage Details - pack / unpack

- Pack a design library including reference libraries into a single file

```tcl
# icc_shell >
open_lib TOP.dlib
open_block abc
...
write_lib_package TOP.packed
# TOP.packed: Binary archive. Include all blocks in the library, by default
```

- Unpack a design library

```tcl
# icc2_shell >
read_lib_package TOP.packed
# Information: Loading library file '/.../.../TOP.dlib' (FILE-007)
# Information: Loading library file '/.../.../lib1.ndm' (FILE-007)
# ...
# Opening block 'TOP.dlib:abc.design' in edit mode

ls TOP.dlib/
# lib.ndm reflibs/ abc/
# Packing copies all reference libs to here and updates all reference library paths to ./reflibs
```

When packing: •All reference libraries are packed unless you specify `-exclude_reflibs` •You must open a design library before packing it •The current design library is packed by default. Use -library to pack a different library •All blocks are packed by default. Use -blocks to pack a specific list of blocks •Use -verbose for more details about the packing •Collections are not packed, you can rebuild them from the unpacked design •All variables and application options are saved as well  
When unpacking: •opens the library and sets the current block •All the reference libraries are located in the reflibs directory under the library •Use -destination to specify the destination directory in which to unpack the design •Use -overwrite to overwrite an existing library at the destination location  
패킹할 때: •`-exclude_reflibs`를 지정하지 않으면 모든 참조 라이브러리가 패킹됩니다. •패킹하기 전에 디자인 라이브러리를 열어야 합니다. •기본적으로 현재 디자인 라이브러리가 패킹됩니다. 다른 라이브러리를 패킹하려면 -library를 사용합니다. •기본적으로 모든 블록이 패킹됩니다. 특정 블록의 목록을 패킹하려면 -blocks를 사용합니다. •패킹에 대한 자세한 정보는 -verbose를 사용합니다. •컬렉션은 패킹되지 않으며, 언패킹된 디자인에서 다시 빌드할 수 있습니다. •모든 변수와 앱 옵션이 저장됩니다.  
언패킹할 때: •라이브러리를 열고 현재 블록을 설정합니다. •모든 참조 라이브러리는 라이브러리의 reflibs 디렉토리에 위치합니다. •-destination을 사용하여 디자인을 언패킹할 대상 디렉토리를 지정합니다. •-overwrite를 사용하여 대상 위치에 이미 존재하는 라이브러리를 덮어씁니다.  
{: .notice--warning}  
