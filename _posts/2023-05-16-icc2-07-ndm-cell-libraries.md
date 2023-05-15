---
title: "IC Compiler II - 07 NDM Cell Libraries"
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

After completing this module, you should be able to  

- List the elements that a CLIB (NDM Cell Library) contains
- Describe how the library configuration flow works
- Describe a use case for the library configuration flow

After completing this unit, you should be able to list the various elements that are inside of a CLIB or cell library. You should be able to describe how library configuration flow works, and then you should be able to describe a case for the library configuration flow.  
이 유닛을 완료한 후에는 CLIB 또는 셀 라이브러리 내부에 있는 다양한 요소들을 나열할 수 있어야 합니다. 라이브러리 구성 플로우가 어떻게 작동하는지 설명할 수 있어야 하며, 라이브러리 구성 플로우에 대한 사례를 설명할 수 있어야 합니다.  
{: .notice--warning}  

## NDM Cell Libraries (CLIBs)

- ICC II uses standard and macro cell libraries in NDM format, called CLIBs
  - Does not use `.db`, Milkyway, GDS or LEF
- Each cell in a CLIB contains complete physical and logic/timing/power definitions required for placement, routing and optimization
  - Logic/timing/power data is stored in timing view
    - Originates from multiple `.db` files
  - Physical data is stored in frame view
    - Originates from GDS or LEF (not Milkyway)
  - CLIBs can optionally also contain design and layout views (see notes)

![2023-05-16-ndm-cell-lib.png]({{site.baseurl}}/assets/images/2023-05-16-ndm-cell-lib.png){: .align-center}  

- CLIBs are associated with a specific technology (from the `.tf` file)

IC compiler II uses standard cell and macro cell libraries in a format called NDM, which stands for New Data Model and we refer to these as CLIBs or cell libraries. If you are familiar with the design compiler you might be familiar with the `.db`, and Milkyway library formats which are used by the design compiler or IC compiler. But IC compiler II does not use these formats. It uses the new format called NDM. You may be familiar with GDS or lef formats and other physical library formats, it does not use those either. So, each cell in a cell library contains complete physical as well as logic, timing, and power definitions,  that are required for placement, routing, and optimization. So, the logic, timing, and power data, which by the way comes originally from the `.db` files in Liberty format. So, all of that information is stored in what we call a timing view of the cell, that's shown here in green. So, every cell and gate, or gate, etc has 2 views. It has a timing view where the logic definition, the timing, and the power information are stored. And then there's also the physical information, that's stored as a frame view and the original physical data comes from GDS or lef. It doesn't come from Milky Way. Now there are two optional additional views, a Design View and a layout view that can also be available in an NDM library, by default they're not, but they could be made available. The frame view, by the way, contains the minimum physical information that's needed for each cell. For example, for an and gate, it will contain just the input and output metal pins, connections, VDD and ground, any routing blockages over the cell, but no base layer information, like N well, P well, the diffusion layer, polysilicon, gates contact information. None of that is needed for purposes of placement and routing. So, the frame view will not have the base layers. The layout view contains all the layers. The design view and the layout view, it has all the polygons, but it has pin connectivity, and pin naming information in there. So, we don't use these views for place and route, but when you tape out, we do need all the layers. These cell libraries are associated with a specific technology. So, when a cell library is created, it also tries to associate with a `.tf` file, just like your design library needs to be associated with a particular `.tf` file.  
IC Compiler II는 NDM(New Data Model)이라는 형식으로 표준 셀 및 매크로 셀 라이브러리(CLILB 또는 셀 라이브러리라고도 함)를 사용합니다. Design Compiler에서 사용하는 `.db` 및 Milkyway 라이브러리 형식에 익숙하신 분이라면 아실 수 있습니다. 하지만 IC Compiler II는 이러한 형식을 사용하지 않습니다. 대신 NDM이라는 새로운 형식을 사용합니다. GDS나 LEF와 같은 물리 라이브러리 형식에 익숙하실 수도 있습니다. IC Compiler II는 이러한 형식도 사용하지 않습니다. 각 셀 라이브러리에는 배치, 라우팅 및 최적화에 필요한 완전한 물리, 논리, 타이밍 및 전력 정의가 포함되어 있습니다. 셀의 논리 정의, 타이밍 및 전력 정보는 타이밍 뷰에 저장됩니다. 물리적 정보는 프레임 뷰로 저장됩니다. 프레임 뷰에는 셀의 입력 및 출력 메탈 핀, 연결, VDD 및 그라운드, 셀 위에 있는 라우팅 블록이 포함되지만, N 웰, P 웰, 확산층, 폴리실리콘, 게이트 접촉 정보와 같은 베이스 레이어 정보는 필요하지 않습니다. 레이아웃 뷰에는 모든 레이어 정보가 포함됩니다. 디자인 뷰와 레이아웃 뷰는 폴리곤과 핀 연결 및 핀 이름 정보를 포함합니다. 이러한 뷰들은 배치와 라우팅에는 사용되지 않지만, 실제 생산에 사용될 때 모든 레이어 정보가 필요합니다. 이러한 셀 라이브러리는 특정 기술과 관련되어 있습니다. 따라서 셀 라이브러리가 생성될 때는 디자인 라이브러리가 특정 `.tf` 파일과 연관되도록 설정됩니다.
{: .notice--warning}  

## Library and Technology Data Sources

![2023-05-16-lib-and-tech-data-sources.png]({{site.baseurl}}/assets/images/2023-05-16-lib-and-tech-data-sources.png){: .align-center}  

- Library Compiler creates:
  - `.db` from Liberty `.lib`
  - frame from GDS or LEF
- Library Manager (LM) creates CLIBs from:
  - Logic/Timing `.dbs`
  - Physical NDM `.frame`
  - Tech-File
- IC Compiler II uses CLIBs
  - Invokes LM under-the-hood And/OR
- IC Compiler II uses Fusion Libraries
  - `lc_sh` command allows running Library Compiler directly in `icc2_shell`

The IC Compiler II uses fusion libraries and/or cell libraries. Fusion libraries can be created by using the `lc_sh` command in IC Compiler II. It allows running Library Compiler directly in `icc2_shell`. Using `lc_sh` to run Library Compiler can replace the Library Manager Library configuration flow. This module is focused on the library configuration flow using cell libraries (CLIBs). ICC II uses cell libraries in NDM format. ICC II needs parasitic model information that's either an NXTGRD or TLU plus format. ICC II also needs technology information that comes from a tech file. These are the three pieces of information that ICC II uses: cell libraries, technology file, and RC model file. The NDM libraries are created by a different tool called Library Manager. This Library Manager can be run standalone to create the NDMs. The dashed line here, also implies that the Library Manager can be invoked under the hood within the IC compiler II to create these NDM libraries. This is the library configuration flow. The inputs of the library manager are, `.db` files which contain the logic, timing, and power information. A `.frame` file that contains the physical information. Library Manager merges the logical information with the physical information. The `.db` file is generated by Library Compiler. The Library Compiler takes an ASCII Liberty format file. This is a standard, old Synopsys format. It converts this ASCII file into a binary encrypted `.db` file. A Library compiler also takes GDS or LEF plus the tech file and converts that into a frame-only NDM library. And this frame only NDM then will be combined with the `.db`, which gives the full CLIB.  
IC Compiler II는 퓨전 라이브러리와/또는 셀 라이브러리를 사용합니다. 퓨전 라이브러리는 IC Compiler II의 `lc_sh` 명령을 사용하여 생성할 수 있습니다. `lc_sh`를 사용하여 Library Compiler를 직접 `icc2_shell`에서 실행할 수 있습니다. `lc_sh`를 사용하여 Library Compiler를 실행하면 Library Manager 라이브러리 구성 플로우를 대체할 수 있습니다. 이 모듈은 셀 라이브러리(CLIB)를 사용한 라이브러리 구성 플로우에 중점을 둡니다. ICC II는 NDM 형식의 셀 라이브러리를 사용합니다. ICC II는 NXTGRD 또는 TLU 플러스 형식의 파라사이트 모델 정보를 필요로 합니다. 또한, 테크 파일에서 가져온 기술 정보가 필요합니다. ICC II가 사용하는 세 가지 정보는 셀 라이브러리, 테크 파일 및 RC 모델 파일입니다. NDM 라이브러리는 Library Manager라는 다른 도구에 의해 생성됩니다. 이 Library Manager는 NDM을 생성하기 위해 독립적으로 실행될 수 있습니다. 여기에 점선이 그어져 있는 것은 Library Manager가 IC Compiler II 내부에서 NDM 라이브러리를 생성하기 위해 내부적으로 호출될 수 있다는 것을 나타냅니다. 이것이 라이브러리 구성 플로우입니다. Library Manager의 입력은 논리, 타이밍 및 전력 정보를 포함하는 `.db` 파일입니다. 물리적 정보를 포함하는 `.frame` 파일도 입력으로 사용됩니다. Library Manager는 논리적 정보와 물리적 정보를 병합합니다. `.db` 파일은 Library Compiler에 의해 생성됩니다. Library Compiler는 ASCII Liberty 형식 파일을 사용합니다. 이는 표준 Synopsys 형식인 ASCII 파일을 이진 암호화된 `.db` 파일로 변환합니다. Library Compiler는 GDS 또는 LEF 플러스와 테크 파일도 사용하며, 이를 프레임 전용 NDM 라이브러리로 변환합니다. 그리고 이 프레임 전용 NDM은 `.db`와 결합되어 완전한 CLIB를 형성합니다.  
{: .notice--warning}  

## Standard Cell Definition in `.lib`

![2023-05-16-std-cell-def-in-lib.png]({{site.baseurl}}/assets/images/2023-05-16-std-cell-def-in-lib.png){: .align-center}  

This is the snippet that shows the information contained in the liberty file used to create the cell library or NDMs. As you can see, the file started with the name of the cell OR2_4X. Concerning the tool, this name is not having any meaning since the library can give any name to the cell. Next, you can see the area of the cell, here it is 8. The later part of the file will have information related to each pin A, B, and Y. For example, information related to pin Y is shown here; its direction, type of timing arc, transition table, etc. In the next segment of the file, information related to functionality, Design rule constraints like max and min capacitance values is present. Similarly, we get the detailed information regarding the remaining pins pin A and pin B.  
이 스니펫은 셀 라이브러리 또는 NDM을 생성하는 데 사용되는 Liberty 파일에 포함된 정보를 보여줍니다. 볼 수 있듯이, 파일은 셀 이름인 OR2_4X로 시작합니다. 도구 관점에서 이 이름은 셀에 어떤 의미도 가지지 않으며, 라이브러리는 셀에 어떤 이름이든 부여할 수 있습니다. 다음으로, 셀의 면적이 표시됩니다. 여기서는 8입니다. 파일의 이후 부분에는 각 핀 A, B 및 Y와 관련된 정보가 있습니다. 예를 들어, 핀 Y와 관련된 정보는 여기에 표시되어 있습니다. 방향, 타이밍 아크 유형, 전환 테이블 등입니다. 파일의 다음 세그먼트에는 기능과 관련된 정보, 최대 및 최소 커패시턴스 값과 같은 설계 규칙 제약 조건이 포함되어 있습니다. 마찬가지로, 나머지 핀인 핀 A와 핀 B에 대한 자세한 정보를 얻을 수 있습니다.  
{: .notice--warning}  

## Technology File

![2023-05-16-technology-file.png]({{site.baseurl}}/assets/images/2023-05-16-technology-file.png){: .align-center}  

- The technology file is unique to each technology
- Defines parameters for all process layers:
  - Layer name
  - GDSII layer number
  - Colors and patterns for display
  - Design rules (width, spacing, area, pitch, etc.)
  - Via contact definitions (lower/upper metal layers, metal enclosure, etc.)
  - Default via array rules
  - Site definitions

Here you can see the content of the technology file. The tech file is unique to each technology. Refer to the example shown here for metal layer 1. It contains the following information: Metal layer name, GDS2 layer number, color, and pattern for each metal layer for display purpose, foundry design rules like min-width, space, pitch, via definitions, via rules, site definition, etc.  
여기에서는 테크 파일의 내용을 볼 수 있습니다. 테크 파일은 각각의 공정에 고유합니다. 이곳에서는 메탈 레이어 1의 예제를 참조하십시오. 다음과 같은 정보를 포함합니다: 메탈 레이어 이름, GDS2 레이어 번호, 디스플레이용으로 각 메탈 레이어의 색상과 패턴, 최소 너비, 간격, 피치와 같은 파운드리 설계 규칙, 비아 정의, 비아 규칙, 사이트 정의 등이 포함되어 있습니다.  
{: .notice--warning}  

## Layout vs. Frame Views

- `.frame` contains the minimal data required for Place & Route
  - In general, provided by the vendor

![2023-05-16-layout-vs-framce-views.png]({{site.baseurl}}/assets/images/2023-05-16-layout-vs-framce-views.png){: .align-center}  

So as mentioned earlier, the frame view contains the minimum data, that's required for place and route. It is generally provided by your vendor. It only has the input and output metal PIN connections, VDD and ground, power, and ground metal pins, simply we call frame view is the outline view of the cell. But the full layout view would have all of the layers, including the N well, or P wells diffusions, polysilicon gates, Contacts, etc, which none of that is needed for placing route purposes. When we input GDS or lef into the library compiler, and we use the `create_frame` command, that will strip out all of this information and create this frame view. Then it will be used as input to the library manager to merge with the `.db` information and create a CLIB or NDM cell library.  
앞서 언급한 대로, 프레임 뷰에는 플레이스와 라우트에 필요한 최소한의 데이터가 포함되어 있습니다. 이는 일반적으로 공급 업체에서 제공됩니다. 프레임 뷰에는 입력 및 출력 메탈 핀 연결, VDD 및 그라운드, 전원 및 그라운드 메탈 핀 등만 포함되어 있습니다. 간단히 말해서 프레임 뷰는 셀의 윤곽 뷰라고 할 수 있습니다. 하지만 전체 레이아웃 뷰에는 N 웰이나 P 웰, 디퓨전, 폴리실리콘 게이트, 컨택트 등을 포함한 모든 레이어가 포함됩니다. 이는 플레이스와 라우트 목적으로는 필요하지 않습니다. GDS나 LEF를 라이브러리 컴파일러에 입력하고 `create_frame` 명령을 사용하면 이러한 정보가 제거되고 프레임 뷰가 생성됩니다. 그런 다음 이를 라이브러리 매니저에 입력하여 `.db` 정보와 병합하여 CLIB 또는 NDM 셀 라이브러리를 생성합니다.  
{: .notice--warning}  

## Cell Library Data Representation

![2023-05-16-cell-lib-data-representation.png]({{site.baseurl}}/assets/images/2023-05-16-cell-lib-data-representation.png){: .align-center}  

- CLIBs contain the logic and physical models
- Each `.db` characterized for a PVT corner is represented as a “pane”
- After CLIB creation, source `.db`/`.frame` files are no longer required

Let us discuss the CLIB creation in more detail. As we discussed, IC compiler 2 uses only CLIBs generated by the library manager, which contains all physical data and logic, timing, and power data required for placement and route. You need to provide a .frame and .db file generated from the library compiler as well as a technology file as inputs to the library manager. As you can see, .db files are defined for each PVT corner, here in the example we have 4 corners so there are 4 .db files. Once you provide these inputs to the library manager, it will generate single, unified optimized CLIBs for the ICC2 tool. As I mentioned earlier, this CLIB contains physical data as well as logic, timing, and power data required for placement and routing. In the CLIB, each .db characterized for a PVT corner is represented as a "pane". Since we have 4 .db files in the example we have 4 panes; pane0, 1, 2, and 3. After the creation of CLIBs, ICCII will not use the source files anymore, it only refers to the CLIBs.  
CLIB 생성에 대해 좀 더 자세히 알아보겠습니다. 이전에 설명했듯이, IC Compiler II는 라이브러리 매니저에서 생성된 CLIB만 사용하며, 이 CLIB에는 플레이스와 라우트에 필요한 모든 물리 데이터 및 논리, 타이밍 및 전력 데이터가 포함됩니다. 라이브러리 컴파일러에서 생성된 .frame 및 .db 파일과 테크놀로지 파일을 라이브러리 매니저에 입력해야 합니다. 예시에서 볼 수 있듯이, 각 PVT 코너에 대해 .db 파일이 정의되어 있으며, 예시에서는 4개의 코너가 있으므로 4개의 .db 파일이 있습니다. 이러한 입력을 라이브러리 매니저에 제공하면, 라이브러리 매니저는 ICC2 도구에 대해 단일한 최적화된 CLIB를 생성합니다. 앞서 언급했듯이, 이 CLIB에는 플레이스와 라우트에 필요한 물리 데이터 및 논리, 타이밍 및 전력 데이터가 포함됩니다. CLIB에서 각 PVT 코너에 대해 정의된 .db는 "pane"으로 표시됩니다. 예시에서는 4개의 .db 파일이 있으므로 pane0, 1, 2 및 3이 있습니다. CLIB 생성 후에는 ICCII는 더 이상 원본 파일을 사용하지 않으며, CLIB를 참조합니다.  
{: .notice--warning}  

## Library Queries Can Still use Original `.db` Names

- The commands `get_libs`, `get_lib_cells`, `get_lib_pins`, `get_lib_timing_arcs` accept `.db` library names and NDM library names
- When `.db` library names are used, NDM library names are returned

![2023-05-16-lib-query-can-use-db-names.png]({{site.baseurl}}/assets/images/2023-05-16-lib-query-can-use-db-names.png){: .align-center}  

Even though the tool will not use the source file after the creation of CLIBs, the user can use the source db file names and NDM library names interchangeably during the query commands like `get_libs`, `get_lib_cells`, `get_lib_pins`, etc. When you use the `.db` library names in the query command, NDMs/CLIBs names will be returned as query output. For example, when you reported the library `saed32hvt_ff_c`, which is the name of the NDM library, the report will have different corner information, process, voltage, temperature, original db name, and original DB filename as shown here. When you give the library DB name in the get_libs command, it will return the NDM cell library. Similarly, the next two examples are evidence for we can use original DB file names and NDM cell library names interchangeably.  
CLIB 생성 후에는 도구가 원본 파일을 사용하지 않지만, 사용자는 쿼리 명령(`get_libs`, `get_lib_cells`, `get_lib_pins` 등) 중에 원본 db 파일 이름과 NDM 라이브러리 이름을 상호 교환하여 사용할 수 있습니다. 쿼리 명령에서 `.db` 라이브러리 이름을 사용할 때는 NDM/CLIB 이름이 쿼리 출력으로 반환됩니다. 예를 들어, NDM 라이브러리의 이름인 `saed32hvt_ff_c` 라이브러리를 보고할 때, 리포트에는 다른 코너 정보, 프로세스, 전압, 온도, 원래 db 이름 및 원래 DB 파일 이름이 표시됩니다. get_libs 명령에 라이브러리 DB 이름을 제공하면 NDM 셀 라이브러리가 반환됩니다. 마찬가지로 다음 두 가지 예시는 원본 DB 파일 이름과 NDM 셀 라이브러리 이름을 서로 교환하여 사용할 수 있음을 보여줍니다.  
{: .notice--warning}  

## Read `.db` and `.frame` directly

![2023-05-16-read-db-frame-directly.png]({{site.baseurl}}/assets/images/2023-05-16-read-db-frame-directly.png){: .align-center}  

- Instead of using pre-assembled cell libraries, it is also possible to read liberty(.db) and frame-only-views (.frame) directly
  - Library Manager is called under-the-hood to create cell libraries (CLIBs)
  - The generated CLIBs are automatically linked to the design
  - This is referred to as library configuration
- Use the `search_path` and `link_library` variables (similar to Design Compiler)
- Generated CLIBs are automatically re-generated if their logical or physical source files change:
  - Added or removed logic libraries (`.db` files)
  - Updated source files, e.g. technology file, `.frame` views

Instead of using the pre-assembled cell libraries, you can read liberty and frame only views directly. During such cases, the library manager will be called under the hood to create cell libraries. These generated libraries are automatically linked to the design, this process is referred to as library configuration. Like the design compiler, the user can use `search_path` and `link_library` application variables to specify the path and linking of these CLIBs. One important feature of the tool is, that these generated CLIBs are automatically re-generated if their logical or physical source files change.  
미리 구성된 셀 라이브러리 대신, 직접 리버티와 프레임 전용 뷰를 읽어들일 수도 있습니다. 이러한 경우 라이브러리 매니저가 뒷단에서 호출되어 셀 라이브러리를 생성합니다. 생성된 라이브러리는 자동으로 디자인과 연결되며, 이 과정을 라이브러리 구성이라고 합니다. 디자인 컴파일러와 마찬가지로 사용자는 `search_path와` `link_library` 애플리케이션 변수를 사용하여 이러한 CLIB의 경로와 링크를 지정할 수 있습니다. 도구의 중요한 기능 중 하나는 논리 또는 물리 소스 파일이 변경되면 이러한 생성된 CLIB가 자동으로 재생성된다는 점입니다.  
{: .notice--warning}  

## Library Configuration(Automatic CLIB Creation) Example

```tcl
# Specify where to find .db and .frame
lappend search_path lib_data/DB lib_data/FRAME

# Specify the .db(s) - same as in Design Compiler
set_app_var link_library "abc14_ff_1p16v_n40c.db ... abc14_ss_0p95v_125c.db"

# Specify the .frame(s) in the create_lib call
create_lib ORCA.dlib \
    -technology abc14_9m.tf \
    -ref_libs {abc14_svt.frame ... abc14_sram.frame}
```

- The assembled CLIB(s) are stored at `./CLIBs` by default for later reuse

This is an example of how library configuration will work. As mentioned in the previous slide, you can use the `search_path` application variable to specify the directory paths of DB files and frame-only-view files. Like Design Compiler specify the different .db files by using the `link_library` application variable. These app variables settings are not stored with the design and need to be re-applied after each invocation of IC Compiler II. Specify the required `.frame` files and tech files in the `create_lib` command as shown in the example. By default, the assembled cell libraries are stored in the created CLIBs directory in the current working directory. The default location of the generated CLIBs can be configured by using the application option `lib.configuration.local_output_dir`.  
이것은 라이브러리 구성이 작동하는 예시입니다. 이전 슬라이드에서 언급한 것처럼, `search_path` 애플리케이션 변수를 사용하여 DB 파일과 프레임 전용 뷰 파일의 디렉토리 경로를 지정할 수 있습니다. Design Compiler와 마찬가지로 `link_library` 애플리케이션 변수를 사용하여 다른 .db 파일을 지정할 수 있습니다. 이러한 애플리케이션 변수 설정은 디자인과 함께 저장되지 않으며, IC Compiler II를 실행할 때마다 다시 적용해야 합니다. 예시에서와 같이 `create_lib` 명령에서 필요한 `.frame` 파일과 tech 파일을 지정하세요. 기본적으로, 조립된 셀 라이브러리는 현재 작업 디렉토리의 created CLIBs 디렉토리에 저장됩니다. 생성된 CLIB의 기본 위치는 `lib.configuration.local_output_dir` 애플리케이션 옵션을 사용하여 구성할 수 있습니다.  
{: .notice--warning}  

## Managing Auto-Assembled CLIBs (library cache)

- Library configuration allows a central and a local library cache
  - You specify the `link_library` variable, the .frame(s), and a central cell library location
  - If ICC Il finds a corresponding cell library in the central location, nothing will be assembled
  - If ICC II does not find the cell library, it assembles a new cell library in a local location and pulls the remaining cell libraries from the central location automatically

![2023-05-16-managing-auto-assembled-clibs.png]({{site.baseurl}}/assets/images/2023-05-16-managing-auto-assembled-clibs.png){: .align-center}  

The library configuration allows central and local cache memory for CLIBs. Users can specify the location for the central library and .frames by using the `link_library` variable. As you can see in the diagram, there are 2 users user1 and user2. Assume that the NDM cell library required by user1 is already available in the central location, during such cases tool will find the corresponding cell library in the central location, and new NDM will not be assembled. If not, it assembles a new cell library in a local location and pulls the remaining cell libraries from the central location automatically.  
라이브러리 구성은 CLIB의 중앙 및 로컬 캐시 메모리를 지원합니다. 사용자는 `link_library` 변수를 사용하여 중앙 라이브러리와 .frame 파일의 위치를 지정할 수 있습니다. 다이어그램에서는 user1과 user2라는 두 명의 사용자가 있습니다. user1이 필요로하는 NDM 셀 라이브러리가 중앙 위치에 이미 있는 경우, 도구는 중앙 위치에서 해당 셀 라이브러리를 찾고 새로운 NDM은 조립되지 않습니다. 그렇지 않으면 로컬 위치에 새로운 셀 라이브러리를 조립하고 나머지 셀 라이브러리를 자동으로 중앙 위치에서 가져옵니다.  
{: .notice--warning}  

## Library Configuration Example - Advanced

```tcl
# Specify the search path for finding .db and .frame
set_app_var search_path ". lib_data/DB lib_data/FRAME"
# Run CLIB creation with multiple cores on same machine
set_app_options -name lib.configuration.cdpl_host -value "-hosts :4"
# Optional - specify where to store the assembled clib(s) (default is ./CLIBs) and configure the central cache
set_app_options -list {
    lib.configuration.local_output_dir ". /MY_CLIBs"
    lib.configuration.central_output_dir "/.../TEAM/CLIBs" }
# Add process labels (→ discussed in the Timing Setup unit)
set_app_options -name lib.configuration.process_label_mapping {
{p_slow {BB.db AA.db FF.db} } {p_typ {DD.db EE.db} } {p_fast CC.db} }
# Specify the .db(s)
set_app_var link_library "xx_ff_1pl6v_n40c.db ... xx_ss_0p95v_125c.db"
# .frame is specified in the create_lib call
# This will launch Library Manager under the hood to create the clib(s)
create_lib ORCA.dlib \
    -technology abc14.tf \
    -ref_libs {xx_abc14_hvt.frame ... RAM_abc14.frame}
```

This is an example of how library configuration will work. Let us start from the first line, using the search_path variable specifies the directory path for DB and Frame only view files. Run the library creation with multiple cores on the same machine using app option lib.configuration.cdpl_host to improve the run time. Optionally the user can specify the local directory location where the newly assembled CLIBs are being stored, and configure the central cache location by using the app options lib.configuration.local_output_dir and lib.configuration.central_output_dir  respectively. If you are not specifying the local output directory for newly assembled NDMs, by default they will be stored in the CLIBs directory of the current working directory. Add the process labels for different coroners as shown here. We will discuss it in the timing setup unit. Specify the .db files by using link_library and finally use the command create_lib to create the design library.  
다음은 라이브러리 구성이 작동하는 예시입니다. 첫 번째 줄에서는 search_path 변수를 사용하여 DB 및 프레임 전용 뷰 파일의 디렉토리 경로를 지정합니다. 런타임을 개선하기 위해 lib.configuration.cdpl_host 앱 옵션을 사용하여 동일한 기계에서 다중 코어로 라이브러리 생성을 실행할 수 있습니다. 새로 조립된 CLIB가 저장되는 로컬 디렉토리 위치를 선택적으로 지정하고, lib.configuration.local_output_dir 및 lib.configuration.central_output_dir 앱 옵션을 사용하여 중앙 캐시 위치를 구성할 수 있습니다. 새로 조립된 NDM의 로컬 출력 디렉토리를 지정하지 않은 경우, 기본적으로 현재 작업 디렉토리의 CLIBs 디렉토리에 저장됩니다. 다음으로 다른 코너에 대한 프로세스 레이블을 추가합니다. 이에 대해서는 타이밍 설정 유닛에서 자세히 설명하겠습니다. link_library를 사용하여 .db 파일을 지정하고, 마지막으로 create_lib 명령을 사용하여 디자인 라이브러리를 생성합니다.  
{: .notice--warning}  

## Combination of Pre-Assembled and Generated CLIBS

- Besides generating all CLIBs (`.db` + `.frame`), it is possible to generate CLIBS for some, and use pre-assembled CLIBs for others
  - Example: `.ndm` for standard cells and `.db` + `.frame` for macros & IP (see notes)
  - `.ndm`(s) take precedence: Specified `.db` files that are used by `.ndm`(s) are ignored (see notes)
- If pre-assembled CLIBs were originally used, and updated `.db` files and/or `.frame` library(ies) become available, CLIB(s) can easily be generated for the
updated data (example shown next)
  - Need to specify `.db`(s) and `.frame`(s) for CLIB generation to occur

----------

Example using .ndm and .frame+.db:  

```tcl
set_app_var search_path ". lib_data/DB lib_data/FRAME"
set_app_var link_library "* srams_ff_1p16v_n40c.db ... srams_ss_0p95v_125c.db"
create_lib ORCA.dlib \
    -technology abc14.tf \
    -ref_libs {std_lvt.ndm std_hvt.ndm SRAM_abc14.frame}
```

This will invoke Library Manager to generate the SRAM cell library.  

Now assume that `link_library` was specified as:

```tcl
set_app_var link_library "* srams_ff_1p16v_n40c.db ...
std_lvt_ss_0p95_125c.db"
```

If the `std_lvt*db` was used originally to create the `std_lvt.ndm` cell library, it will be ignored. No
second, redundant, cell library will be generated.

Now, besides generating all CLIBs, it's possible to mix and match. So, you can generate CLIBs for some of your libraries and then use pre-assembled cell libs for others. So, for instance, you could use the fully preassembled NDM cell libraries for your standard cells. But for your macros and IP, you would assemble them from the .dbs and .frame. Suppose, if you have both the NDM as well as the .db and frame views available for the same cell, then NDMs will take precedence. If ICCII sees the NDMs, it's not going to assemble them. Now if pre-assembled CLIBs were originally used, but we have updated db files and/or updated frame files, then, I'm going to show how you can easily regenerate these CLIBs to update the cell libraries. Let us look at that example next.  
이제 모든 CLIB을 생성하는 것 외에도 혼합 및 조합이 가능합니다. 따라서 일부 라이브러리에 대해서는 CLIB를 생성하고, 나머지에 대해서는 사전에 조립된 셀 라이브러리를 사용할 수 있습니다. 예를 들어, 표준 셀에 대해서는 완전히 사전 조립된 NDM 셀 라이브러리를 사용할 수 있습니다. 그러나 매크로 및 IP에 대해서는 .db 및 .frame 파일에서 조립하여 사용할 수 있습니다. 만약 동일한 셀에 NDM과 .db 및 frame 뷰가 모두 있는 경우, NDM이 우선시됩니다. ICCII가 NDM을 감지하면 조립하지 않습니다. 이제 사전에 조립된 CLIB가 원래 사용되었지만 업데이트된 db 파일 및/또는 업데이트된 frame 파일이 있는 경우, 이러한 CLIB를 쉽게 다시 생성하여 셀 라이브러리를 업데이트하는 방법을 보여드리겠습니다. 다음에 해당 예시를 살펴보겠습니다.  
{: .notice--warning}  

## Example: Using Updated `.db` Files

- Original set-up of design library using pre-assembled CLIBs:

```tcl
lappend search_path /x/y/clibs
create_lib ORCA.dlib -technology abc14_9m.tf \
    -ref_libs { abc14_std.ndm abc14_ls.ndm abc14_sram.ndm }
analyze ...
```

- Configuration to use newer SRAM `.db` files:

```tcl
lappend search path /x/y/sram_db /x/y/sram_frame

# Specify the new .dbs
set link_library "abc14_sram_125C.db abc14_sram_20C.db abc14_sram_m40C.db"
# Replace pre-assembled SRAM .ndm with its original .frame
set_ref_libs { abc14_std.ndm abc14_ls.ndm sram.frame }
...library building process occurs...

report_ref_libs
*+  abc14_std   abc14_std.ndm
*+  abc14_ls    abc14_ls.ndm
*+  abc14_sram  CLIBs/abc14_sram.ndm
```

- It is necessary to replace the entry for the existing pre-assembled CLIB with its .frame(.ndm has priority over .db)

Here is an example, where, the original setup use the fully pre-assembled NDM/CLIBs. As for reference libraries, the complete NDM or CLIBs are given here. But then, let's say the SRAM .db is updated. So, for the SRAM NDM, it is now outdated, because the .db has been updated. So, how can it be easily updated, and a new cell library is created? so here again, append the search path, the location of this new updated .db file, and also point to the frame file associated with the SRAM library. Now with the link_library, specify the new .db files only for the SRAMs.  So, these are the updated SRAMs for the different PVT corners. And then, use the set_ref_libs command, which points to the existing unchanged standard cell library, along with the sram.frame which is not the fully pre-assembled SRAM. This will now kick off the library configuration and invoke the library manager to grab this frame and combine it with the updated .dbs, to create a new assembled CLIB, and store it in the local CLIBs directory.  
다음은 예시입니다. 원래 설정에서는 완전히 사전 조립된 NDM/CLIB를 사용합니다. 참조 라이브러리의 경우, 여기에 완전한 NDM 또는 CLIB가 제공됩니다. 그러나, SRAM .db가 업데이트되었다고 가정해보겠습니다. 따라서 SRAM NDM은 이제 .db가 업데이트되었기 때문에 오래되었습니다. 그렇다면 어떻게 쉽게 업데이트하고 새로운 셀 라이브러리를 생성할 수 있을까요? 이제 .db 파일이 업데이트된 위치에 대한 검색 경로를 추가하고, SRAM 라이브러리와 관련된 프레임 파일을 가리키도록 지정합니다. 그런 다음, link_library를 사용하여 SRAM에 대한 새로운 .db 파일만 지정합니다. 이렇게 하면 다양한 PVT 코너에 대한 업데이트된 SRAM이 됩니다. 그런 다음, set_ref_libs 명령을 사용하여 변경되지 않은 표준 셀 라이브러리와 완전히 사전 조립되지 않은 SRAM인 sram.frame을 가리킵니다. 이제 이를 통해 라이브러리 구성이 시작되고 라이브러리 매니저가이 프레임을 가져와 업데이트된 .db와 결합하여 새로운 조립된 CLIB를 생성하고 로컬 CLIBs 디렉토리에 저장합니다.  
{: .notice--warning}  

## Summary

You should now be able to

- List the elements a CLIB contains
- Describe how the library configuration flow works
- Describe a use case for the library configuration flow
