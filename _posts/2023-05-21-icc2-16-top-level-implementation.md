---
title: "IC Compiler II - 16 Top Level Implementation(TO DO)"
excerpt: "Block-level Implementation"
date: 2023-05-21 06:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - IC Compiler II
---

## Objectives

- List the steps necessary for top-level implementation
- Describe what an abstract is and what it contains
- Describe how to assemble the top-level design

## 1. Top Level Implementation

- When implementing the top level, blocks can be design views, or they can be abstracted by using a block abstract or an ETM
  - Design views are OK for smaller blocks
  - Block abstracts can be generated in ICC2 and are the preferred choice
  - ETMs can provide memory and runtime benefits for very large designs
    - Also useful for IP that needs to be hidden
- The blocks can be at any design phase
  - Placed
  - Clocks completed
  - Completely routed and optimized

![2023-05-21-top-level-impl.png]({{site.baseurl}}/assets/images/2023-05-21-top-level-impl.png){: .align-center}  

Let us start with the Implementation of a Top-level Design. By "implementing the top level" we mean performing synthesis, placement, CTS, and routing along with all optimizations at the top level of a physically hierarchical design. The important aspect to consider while implementing a top-level design is to choose the right model for the sub designs or the sub blocks. The sub blocks can be directly used at the top-level as design views, or they can be abstracted using block abstraction or they could be represented as an Extracted timing model. For sub blocks with gate count of up to 100k, you can directly instantiate them at the top-level using design views. For sub blocks with higher gate count block abstraction can be performed in IC Compiler II and is the preferred choice as it provides good performance as well as enabling better debuggability. For very large sub blocks, using ETM's can provide some additional memory and runtime benefit. ETM's are a preferred choice for IP whose internals needs to be hidden The blocks at any design phase be it placed, clock implementation done or completely routed and optimized can be used for implementing the top-level. After the models have been instantiated, there's no difference between implementing a block and the top-level.  
우리는 최상위 설계의 구현으로 시작합시다. "최상위 구현"이란 물리적인 계층 구조 디자인의 최상위에서 합성, 배치, CTS(클록 트리 합성), 라우팅을 수행하면서 모든 최적화를 의미합니다. 최상위 설계를 구현하는 동안 중요한 측면은 서브 디자인 또는 서브 블록에 적합한 모델을 선택하는 것입니다. 서브 블록은 최상위에서 디자인 뷰로 직접 사용하거나 블록 추상화를 사용하여 추상화할 수 있으며, 추출된 타이밍 모델로 나타낼 수도 있습니다. 게이트 수가 최대 100,000개인 서브 블록의 경우 디자인 뷰를 사용하여 최상위에서 직접 인스턴스화할 수 있습니다. 게이트 수가 더 많은 서브 블록의 경우 IC Compiler II에서 블록 추상화를 수행할 수 있으며, 성능이 우수하며 디버깅을 용이하게 제공하기 때문에 선호됩니다. 매우 큰 서브 블록의 경우 ETM(추출된 타이밍 모델)을 사용하면 추가적인 메모리와 실행 시간 이점을 제공할 수 있습니다. ETM은 내부 정보를 숨겨야 하는 IP에 대한 우선적인 선택지입니다. 배치, 클록 구현 또는 완전히 라우팅 및 최적화된 어떤 디자인 단계에서든지 블록은 최상위 구현을 수행하는 데 사용될 수 있습니다. 모델이 인스턴스화된 후에는 블록과 최상위를 구현하는 데 차이가 없습니다.  
{: .notice--warning}  

## 2. Using Completely Implemented Blocks

![2023-05-21-using-completely-imp-blocks.png]({{site.baseurl}}/assets/images/2023-05-21-using-completely-imp-blocks.png){: .align-center}  

Coming to the flows available for top-level implementation, in a serial flow, you implement all the blocks entirely, then generate models from the fully routed and implemented blocks. Then the top-level implementation starts by using the models from the fully implemented blocks. But the disadvantage here is, designer need to wait until all blocks being implemented.  
최상위 구현에 사용 가능한 플로우에 대해 이야기하자면, 직렬 플로우에서는 모든 블록을 완전히 구현한 후, 완전히 라우팅 및 구현된 블록에서 모델을 생성합니다. 그런 다음 최상위 구현은 완전히 구현된 블록의 모델을 사용하여 시작됩니다. 그러나 이 경우의 단점은 디자이너가 모든 블록이 구현될 때까지 기다려야 한다는 점입니다.  
{: .notice--warning}  

## 3. Parallel Blocks and Top Implementation

![2023-05-21-parallel-blocks-and-top-implmentation.png]({{site.baseurl}}/assets/images/2023-05-21-parallel-blocks-and-top-implmentation.png){: .align-center}  

To eliminate the waiting period, we will go for parallel flow. In a parallel flow, you don't wait until all blocks have been fully implemented, instead you complete the first step (which is `place_opt`) for all blocks, then you pass the `place_opt` completed abstract models to perform top-level `place_opt`. Then you perform CTS on all blocks, and you pass on CTS-models for top-level CTS, so on and so forth. Of course, you can always use a model from a later block stage to perform an earlier stage at the top-level, for example use post-CTS models to perform `place_opt` at the top.  
대기 기간을 없애기 위해 병렬 플로우로 전환합니다. 병렬 플로우에서는 모든 블록이 완전히 구현될 때까지 기다리지 않고, 대신 모든 블록에 대해 첫 번째 단계(`place_opt`)를 완료한 후, `place_opt`가 완료된 추상 모델을 최상위 `place_opt`에 전달합니다. 그런 다음 모든 블록에 대해 CTS(클록 트리 합성)를 수행하고, CTS가 완료된 모델을 최상위 CTS에 전달하고, 이와 같이 진행됩니다. 물론 나중에 있는 블록 단계의 모델을 사용하여 최상위에서 이전 단계를 수행할 수도 있습니다. 예를 들어, 최상위에서 후 CTS 모델을 사용하여 `place_opt`를 수행할 수 있습니다.  
{: .notice--warning}  

## 4. Extracted Timing Model (ETM)

![2023-05-21-etm.png]({{site.baseurl}}/assets/images/2023-05-21-etm.png){: .align-center}  

- Timing model derived from block design data
- Timing view(s) generated using the new command `extract_model`
  - Invokes PrimeTime under the hood to perform model extraction
  - Invokes Library Manager to assemble the PrimeTime ETM.db and the frame view
- Can be used to model IP blocks, where internal details need to be hidden

Now let's go over the different block modelling options available in the IC Compiler II tool. The extracted timing model is an abstraction of the block interface timing behavior using sequential and combinational timing arcs. The tool extracts each timing arc and represents its delay as a function of input transitions and output loads. The ETM's and the ETM library can be generated within the IC compile II tool itself using the `extract_model` command. This command Invokes PrimeTime under the hood to perform model extraction. Invokes Library Manager to assemble the PrimeTime ETM.db and the frame view. As we saw earlier, the ETM's are useful to model IP blocks where internal details need to be hidden. For more information about using ETMs at the top level, refer the notes section.  
IC Compiler II 도구에서 제공되는 다양한 블록 모델링 옵션에 대해 알아봅시다. 추출된 타이밍 모델은 순차 및 조합적인 타이밍 아크를 사용하여 블록 인터페이스 타이밍 동작의 추상화입니다. 이 도구는 각 타이밍 아크를 추출하고 입력 전이와 출력 부하의 함수로서 딜레이를 나타냅니다. ETM(추출된 타이밍 모델)과 ETM 라이브러리는 IC Compiler II 도구 내에서 `extract_model` 명령을 사용하여 생성할 수 있습니다. 이 명령은 PrimeTime을 내부적으로 호출하여 모델 추출을 수행합니다. 또한 PrimeTime ETM.db와 프레임 뷰를 조립하기 위해 Library Manager도 호출합니다. 이전에 보았듯이, ETM은 내부 세부 정보를 숨겨야 하는 IP 블록을 모델링하는 데 유용합니다. 최상위에서 ETM을 사용하는 방법에 대한 자세한 정보는 노트 섹션을 참조하시기 바랍니다.  
{: .notice--warning}  

## 5. Abstract View

![2023-05-21-abstract-view.png]({{site.baseurl}}/assets/images/2023-05-21-abstract-view.png){: .align-center}  

- Partial netlist view derived from design view
- Much smaller than design view, therefore, leads to much better performance when implementing the top-level
- Can include the following additional information:
  - Clock exceptions
  - Crosstalk data
  - Power information (leakage and/or dynamic)
  - Signal EM data

Now let us see what an abstract is and what all information it contains. Abstract view is a partial netlist view derived from the actual block design view and hence a netlist-based model.Since it is much smaller than the design view, it leads to much better performance when we implement the top-level. Abstracts retains the critical interface logic across the active modes of the design. Abstracts can include additional information like clock exceptions, crosstalk data needed for SI analysis, power information and Signal Electronmigration data based on the settings active at the block-level during abstract creation.  
추상(Abstraction)은 실제 블록 디자인 뷰에서 유도된 부분적인 넷리스트 뷰이며, 따라서 넷리스트 기반 모델입니다. 디자인 뷰보다 훨씬 작기 때문에 최상위 구현을 수행할 때 훨씬 더 우수한 성능을 제공합니다. 추상은 디자인의 활성 모드 전체에서 중요한 인터페이스 로직을 보존합니다. 추상은 추상 생성 중 블록 수준에서 활성화된 설정을 기반으로 클록 예외, SI(신호 무결성) 분석에 필요한 크로스톡 데이터, 전력 정보 및 신호 전자 이동 데이터와 같은 추가 정보를 포함할 수 있습니다.  
{: .notice--warning}  

## 6. Creating an Abstract

- To create an abstract:
  - `create_abstract -read_only`
- Identifies and retains interface logic across all active modes
- Retains pins in the interface that have timing constraints
  - across all modes, active and inactive
- Retains RC parasitic of all corners (active and inactive)
- Stores UPF information relevant to the retained interface logic
  - Promoted to the top level during linking
- Ensure that all the scenarios used at **top level** are defined and active at the **block level** when creating an abstract

Coming to the abstract creation, by default, if you run `create_abstract`, the design view is locked, and the abstract view becomes editable. This is useful when performing hierarchical design planning because their changes can be made to the abstract. The changes made to the abstract will be automatically merged into the design view once that is opened. For top-level implementation, abstract editing does not occur, therefore you can keep the designs writable and lock the abstracts instead by using the `-read_only` option. The command `create_abstract`: •Identifies and retains interface logic across all active modes •Retains pins in the interface that have timing constraints •across all modes, active and inactive •Retains RC parasitics of all corners, active and inactive •Stores UPF information relevant to the retained interface logic. •This information gets Promoted to the top level during linking You must ensure that all the scenarios that are active at the top-level to be defined and active at the block-level when creating an abstract.  
추상 생성에 관하여, 기본적으로 `create_abstract`를 실행하면 디자인 뷰가 잠금 상태가 되고, 추상 뷰가 편집 가능해집니다. 이는 계층적 디자인 계획을 수행할 때 유용하며, 추상에 변경 사항을 반영할 수 있습니다. 추상에 가한 변경 사항은 디자인 뷰가 열릴 때 자동으로 디자인 뷰에 병합됩니다. 최상위 구현에서는 추상 편집이 발생하지 않으므로 디자인을 편집 가능하게 유지하고 `-read_only` 옵션을 사용하여 추상을 잠글 수 있습니다. `create_abstract` 명령은 다음과 같은 기능을 수행합니다: 모든 활성 모드에서 인터페이스 로직을 식별하고 보존합니다. 타이밍 제약이 있는 인터페이스 핀을 모든 활성 및 비활성 모드에서 보존합니다. 모든 코너(액티브 및 비활성)의 RC 파라미터를 보존합니다. 보존된 인터페이스 로직과 관련된 UPF 정보를 저장합니다. 이 정보는 링킹 과정에서 최상위로 확장됩니다. 추상을 생성할 때 최상위에서 활성 상태로 정의된 모든 시나리오가 블록 레벨에서 활성 상태로 정의되도록 해야 합니다.
{: .notice--warning}  

## 7. Frame View

![2023-05-21-frame-view.png]({{site.baseurl}}/assets/images/2023-05-21-frame-view.png){: .align-center}  

- The frame view is the physical abstraction required by the router, including the virtual router
  - You must create a frame view for each sub-block
  - `create_frame`
  - By default (for design blocks) complete routing blockages are only created for the layers used in the block (corresponds to `-block_all used_layers`)
  - To block all layers, use: `create_frame -block_all true`

Now we will look at one more view which is essential for implementing the top-level which is the frame view. The frame view is the physical abstraction required by the router, this is needed throughout the implementation flow and is used by virtual router, global and detail router. The command `create_frame` is used to create the frame view for each of the sub-blocks. By default, that is when `-block_all` option set to `used_layers` complete routing blockages are only created for the layers used in the block. To block all layers, you can set `-block_all` option to true.  
이제 최상위 구현을 위해 필수적인 또 다른 뷰인 프레임 뷰를 살펴보겠습니다. 프레임 뷰는 라우터에서 필요로 하는 물리적 추상화입니다. 이것은 구현 플로우 전체에서 필요하며, 가상 라우터, 글로벌 라우터, 상세 라우터에서 사용됩니다. `create_frame` 명령은 각 서브 블록에 대한 프레임 뷰를 생성하는 데 사용됩니다. 기본적으로 `-block_all` 옵션을 `used_layers` complete routing으로 설정하면 블록에 사용된 레이어만을 위한 블록킹이 생성됩니다. 모든 레이어를 블록킹하려면 `-block_all` 옵션을 true로 설정할 수 있습니다.  
{: .notice--warning}  

## 8. Generate Abstract and Frame: Complete Script

![2023-05-21-generate-abstract-and-frame.png]({{site.baseurl}}/assets/images/2023-05-21-generate-abstract-and-frame.png){: .align-center}  

This is the example script runs at the block level to create abtract view and frame view of the sub design or block. Here we are enabling few app options for the enablement of few features in the tool. Enable `time.si_enable_analysis` app option which controls the crosstalk analysis during timing analysis. It also sets up the extraction of the routed nets along with their coupling caps. Setup the required active scenarios for power analysis. Ensure that the power information is annotated to the created abstracts by enabling the `abstract.annotate_power` app option. User must set the `abstract.enable_signal_em_analysis` to true, to enable signal EM analysis on nets belonging to the block-abstract in the block-abstract netlist. The `extract.starrc_mode` application option controls how StarRC is used in extraction for routed blocks. By default, timing analysis uses the tool's native delay calculator. If `time.use_pt_delay` application option is turned on, timing analysis uses the PrimeTime delay calculator. Setting this option to true clears the existing delay data stored in the timer. Then you can create block's abstract view and frame view by using the command `create_abstract` and `create_frame` respectively.  
이는 블록 레벨에서 실행되는 예제 스크립트로, 서브 디자인 또는 블록의 추상 뷰와 프레임 뷰를 생성합니다. 여기에서는 몇 가지 기능을 사용하기 위해 몇 가지 앱 옵션을 활성화합니다. `time.si_enable_analysis` 앱 옵션을 활성화하여 타이밍 분석 중 크로스톡 분석을 제어합니다. 또한 라우트된 넷과 그에 따른 커플링 캡을 추출 설정합니다. 전력 분석에 필요한 활성 시나리오를 설정합니다. `abstract.annotate_power` 앱 옵션을 활성화하여 생성된 추상에 전력 정보를 주석 처리해야 합니다. 사용자는 `abstract.enable_signal_em_analysis` 옵션을 true로 설정하여 블록 추상 넷리스트에서 신호 EM(전자 이동) 분석을 활성화해야 합니다. `extract.starrc_mode` 앱 옵션은 라우트된 블록의 추출에 StarRC가 어떻게 사용되는지를 제어합니다. 기본적으로 타이밍 분석은 도구의 기본 지연 계산기를 사용합니다. `time.use_pt_delay` 앱 옵션을 켜면 타이밍 분석에 PrimeTime 지연 계산기가 사용됩니다. 이 옵션을 true로 설정하면 타이머에 저장된 기존 지연 데이터가 지워집니다. 그런 다음 각각 `create_abstract`와 `create_frame` 명령을 사용하여 블록의 추상 뷰와 프레임 뷰를 생성할 수 있습니다.  
{: .notice--warning}  

## 9. At the Top-Level: Specifying Block Reference Libraries

- After abstract and frame views have been generated for all blocks, include the sub-block design libraries containing the abstracts in the top-level library reference list

```tcl
create_lib TOP.dlib -tech mytech.tf \
  -ref_libs "stdcell.ndm macros.ndm ...
             MYBLOCKS/A.dlib MYBLOCKS/B.dlib ..."
```

Now let us see some of the steps needed at top-level for getting the design ready for implementation. First is the reference library setup, after abstract and frame views have been generated for all blocks, include the sub-block design libraries containing the abstracts in the top-level library reference list during the top-level library creation.  
최상위 구현을 위해 디자인을 준비하는 데 필요한 몇 가지 단계를 살펴보겠습니다. 첫 번째는 참조 라이브러리 설정입니다. 모든 블록에 대한 추상 뷰와 프레임 뷰가 생성된 후, 최상위 라이브러리를 생성하는 동안 추상이 포함된 서브 블록 디자인 라이브러리를 최상위 라이브러리의 라이브러리 참조 목록에 포함시킵니다.  
{: .notice--warning}  

## 10. Cascading Reference Library List

![2023-05-21-cascading-ref-lib.png]({{site.baseurl}}/assets/images/2023-05-21-cascading-ref-lib.png){: .align-center}  

By default, during linking at top-level, `link_block` command uses the reference library list of the top-block library to resolve all references at top design and block hierarchy. That is Block linking will depend on the top's reference library list. If you want to Use the original block's reference library list, then you can set the attribute `use_hier_ref_libs` to true on all the sub_block libraries.  
기본적으로 최상위에서 링킹(linking)을 수행할 때, `link_block` 명령은 최상위 블록 라이브러리의 참조 라이브러리 목록을 사용하여 최상위 디자인 및 블록 계층 구조의 모든 참조를 해결합니다. 즉, 블록 링킹은 최상위의 참조 라이브러리 목록에 따라 진행됩니다. 서브 블록의 원래 참조 라이브러리 목록을 사용하려면 모든 서브 블록 라이브러리에 `use_hier_ref_libs` 속성을 true로 설정할 수 있습니다. 이렇게 하면 최상위에서 블록을 링크할 때 해당 블록의 참조 라이브러리 목록을 사용합니다.  
{: .notice--warning}  

## 11. Reference Libraries in Multi-Level Hierarchical Designs

- For designs with multiple-levels of hierarchy, you should always set up your libraries to use cascaded reference libraries
  - TOP library only references std cells, macros and blocks instantiated at the top
  - MID only references std cells, macros and blocks instantiated at MID
  - Etc.

![2023-05-21-ref-libs-in-multi-level-hier-designs.png]({{site.baseurl}}/assets/images/2023-05-21-ref-libs-in-multi-level-hier-designs.png){: .align-center}  

For designs with multiple levels of hierarchy, it is better to use cascaded reference libraries. One advantage of using cascaded reflibs is that the TOP level reference library list does not have to be updated if any of the reference libraries of Level1 or Level2 blocks changes. That is to say, every sub-block library is responsible only for its direct descendants.  
다중 계층 구조를 갖는 디자인의 경우, 계단식(reference libraries를 사용하는 것이 좋습니다. 계단식 reflib을 사용하는 한 가지 장점은 Level1 또는 Level2 블록의 참조 라이브러리 중 하나가 변경되더라도 최상위(TOP) 레벨의 참조 라이브러리 목록을 업데이트할 필요가 없다는 것입니다. 즉, 각 서브 블록 라이브러리는 직계 하위 블록에 대해서만 책임을 집니다.  
{: .notice--warning}  

## 12. At the Top-Level: Linking the Sub-Blocks

- In a hierarchical flow, it may be required to load sub-blocks that are at different stages of implementation or come from different design teams with different naming standards
- Use labels when saving the various versions of the blocks

```tcl
save_block -as myBlock/placed (or) save_block -label placed
save_block -as myBlock/cts
save_block -as myBlock/route_opt
```

- By default, the tool only links blocks with same label as the top
- Use `set_label_switch_list` to override the default, and specify a list of labels to use when linking

Now let us look at linking of the sub-blocks at the top-level. In a hierarchical flow, it may be required to load sub-blocks, that are at different stages of implementation, or they might come from different design teams with different naming standards. Using labels while saving the blocks helps easier swapping of sub_block versions at the top-level. By default, IC compiler II tool only links sub blocks with the same label as the top-level. You can Use `set_label_switch_list` to override the default, and specify a list of labels to use when linking.  
최상위에서 서브 블록들을 링크하는 방법에 대해 알아보겠습니다. 계층적인 플로우에서는 구현 단계가 다른 서브 블록들을 로드해야 할 수도 있으며, 서로 다른 디자인 팀에서 서로 다른 네이밍 표준을 사용할 수도 있습니다. 블록을 저장할 때 레이블(label)을 사용하면 최상위에서 쉽게 서브 블록 버전을 교체할 수 있습니다. 기본적으로 IC Compiler II 도구는 최상위와 동일한 레이블을 가진 서브 블록들만 링크합니다. `set_label_switch_list`를 사용하여 기본 설정을 재정의하고 링킹할 때 사용할 레이블 목록을 지정할 수 있습니다.  
{: .notice--warning}  

## 13. Specify Label List for Linking

![2023-05-21-specify-label-list-for-linking.png]({{site.baseurl}}/assets/images/2023-05-21-specify-label-list-for-linking.png){: .align-center}  

In the `set_label_switch_list` command, you can set a prioritized list of labels for linking to sub-block abstracts, either globally, or for specified references. In the example, for all the sub-blocks the tool will first search for the `route_opt` labelled block followed by `clock_opt` and `place_opt`. If it cannot find these labelled blocks, then it falls back to default which is the same label as that of the top block. In the same example for the GPU block there is a reference specific setting given, hence the tool will first try to find the eco labelled block, if not available then fall back to the default behavior.  
`set_label_switch_list` 명령을 사용하여 서브 블록 추상에 대해 우선순위가 지정된 레이블 목록을 전역적으로 또는 특정한 레퍼런스에 대해 설정할 수 있습니다. 예를 들어, 모든 서브 블록들에 대해 `route_opt` 레이블이 지정된 블록을 먼저 찾고, 그 다음에 `clock_opt` 레이블이 지정된 블록을 찾고, 그리고 마지막으로 `place_opt` 레이블이 지정된 블록을 찾습니다. 만약 이러한 레이블이 지정된 블록을 찾을 수 없다면, 기본값으로 최상위 블록과 동일한 레이블을 사용하게 됩니다.  
{: .notice--warning}  

## 14. At the Top-Level: Reporting

![2023-05-21-top-level-reporting.png]({{site.baseurl}}/assets/images/2023-05-21-top-level-reporting.png){: .align-center}  

To know the information about the abstract like its editability that is whether it's `ready_only` or editable, compression which is a measure of netlist reduction, and type like compact timing, full interface timing, placement etc, you can use the `report_abstracts` command at the top-level.  
최상위에서는 `report_abstracts` 명령을 사용하여 추상에 대한 정보를 확인할 수 있습니다. 이 정보에는 추상의 편집 가능 여부 (읽기 전용 또는 편집 가능), 압축 (넷리스트 축소의 측정 값), 그리고 타입 (컴팩트 타이밍, 전체 인터페이스 타이밍, 플레이스먼트 등)과 같은 정보가 포함됩니다.  
{: .notice--warning}  

## 15. At the Top-Level: Promoting Clock Data

- Clock data, such as the following, can be promoted to the top-level
  - Ideal clock latency constraints derived by CCD and clock gate latency estimation
  - Balance point exceptions
  - Median latency of propagated clocks
  - Mesh annotations present at a lower-level physical hierarchy
- To promote the block-level clock data to the top level:
  - `promote_clock_data -balance_points`

Now let us look at the clock data promotion from the sub blocks to the top-level. During the pre-route stage, when we run `place_opt` command at the block-level, CCD and latency estimation engines would have set clock latency constraints on the clock pins of ICGs and registers. To promote clock latency constraints and in addition other data such as balance point exceptions applied at the block-level, and missing at the top-level, median latency annotation of the propagated clocks, and mesh annotations you can use the `promote_clock_data` command. In the example we are promoting the balance point exceptions that are applied at the block-level and are missing from the top-level.  
서브 블록에서 최상위로 클록 데이터를 프로모션하는 방법에 대해 살펴보겠습니다. 전처리 단계(pre-route stage)에서 블록 레벨에서 `place_opt` 명령을 실행할 때, CCD 및 지연 예측 엔진은 ICGs(Intermediate Clock Gating)와 레지스터의 클록 핀에 클록 지연 제약을 설정합니다. 클록 지연 제약과 더불어 블록 레벨에서 적용된 balance point 예외와 최상위에서 누락된 예외, 전파된 클록의 중앙 지연 주석, 그리고 메시 주석과 같은 다른 데이터를 프로모션하는 데에 `promote_clock_data` 명령을 사용할 수 있습니다. 예제에서는 블록 레벨에서 적용된 balance point 예외를 프로모션하여 최상위에서 누락된 예외를 보완합니다.  
{: .notice--warning}  

## 16. Queries and Attributes

![2023-05-21-queries-and-attributes.png]({{site.baseurl}}/assets/images/2023-05-21-queries-and-attributes.png){: .align-center}  

This table shows the attributes that exists for querying certain data that might be useful in a hierarchical design. For example, the attribute `is_soft_macro` can be used to identify the physical hierarchy cells in present in the top-level design.  
이 테이블은 계층적 디자인에서 유용한 일부 데이터를 조회하는 데 사용되는 속성들을 보여줍니다. 예를 들어, `is_soft_macro` 속성은 최상위 디자인에 있는 물리적 계층 셀들을 식별하는 데 사용될 수 있습니다.  
{: .notice--warning}  

## 17. Choosing Abstracts/Designs and Labels after Linking

- After linking, you can always swap individual instances or references to point to the design or abstract view, with or without labels
- Switch to the design view for a specific cell I_ADD_0
  - `change_abstract -view design -cells I_ADD_0`
- Switch to the abstract with label cts for a reference called ADD
  - `change_abstract -view abstract -label cts -references ADD`

At any point in the top-level implementation flow, if you want to swap individual sub_block instances or references to the design or abstract view, with or without label, you can use the `change_abstract` command. In the first example we are swapping a sub_block instrance to design view. One thing you must note here is that after swapping to design view, you need to apply the full chip constraints from the top-level to cover the constraints for the entire sub-block design view. In the second example we are swapping the sub_block instrances that belongs to reference ADD to the clock opt completed abstract.  
최상위 구현 플로우에서 언제든지 디자인 또는 추상 뷰로 개별 서브 블록 인스턴스 또는 레퍼런스를 레이블과 함께 또는 레이블 없이 교체하려면 `change_abstract` 명령을 사용할 수 있습니다. 첫 번째 예제에서는 서브 블록 인스턴스를 디자인 뷰로 교체하는 방법을 보여줍니다. 하나 주의할 점은 디자인 뷰로 교체한 후 최상위의 전체 칩(constraints) 제약 조건을 적용하여 전체 서브 블록 디자인 뷰에 대한 제약 조건을 포괄적으로 처리해야 한다는 것입니다. 두 번째 예제에서는 ADD 레퍼런스에 속하는 서브 블록 인스턴스를 clock opt가 완료된 추상으로 교체하는 방법을 보여줍니다.  
{: .notice--warning}  

## 18. At the Top-Level: Check the Design

![2023-05-21-check-the-design.png]({{site.baseurl}}/assets/images/2023-05-21-check-the-design.png){: .align-center}  

Now let us look at the checking mechanism available in the tool to ensure the top-level is ready for implementation. The command `check_hier_design` runs several checks based on the stage provided to make sure the design is ready for implementing that particular stage. Messages generated by this command are captured in an EMS database file and in the log file. In addition, implementation commands such as `place_opt`, `clock_opt` etc) will generate Errors if hierarchy issues are encountered.  
도구에서 최상위가 구현에 준비되었는지 확인하는 데 사용되는 검사 메커니즘에 대해 알아보겠습니다. `check_hier_design` 명령은 제공된 단계(stage)에 기반하여 여러 검사를 실행하여 해당 단계를 위해 디자인이 준비되었는지 확인합니다. 이 명령에 의해 생성된 메시지는 EMS 데이터베이스 파일과 로그 파일에 기록됩니다. 또한 `place_opt`, `clock_opt` 등의 구현 명령은 계층 구조 문제가 발견되면 오류를 생성합니다. 이를 통해 최상위가 정확한 계층 구조를 가지고 구현에 필요한 준비가 되었는지 확인하고, 문제가 발견되면 적절한 메시지를 통해 사용자에게 알려줍니다. 이는 최상위 구현이 성공적으로 진행될 수 있도록 유효성 검사를 수행하는 데 도움이 됩니다.  
{: .notice--warning}  

## 19. At the Top-Level: Handling Dirty Design Inputs

- Errors due to dirty design inputs reported by check hier design can be waived or fixed by choosing the appropriate policy from the allowed list of policies using:
  - `set_early_data_check_policy -checks <> -policy <> -references <>`
- There are several checks specific to hierarchical designs that can be continued as errors which is the default, waived as warnings or fixed by the tool
- To view the list of hierarchy readiness checks and available policies:
  - `report_hier_check_description`

Errors reported by `check_hier_design` command can be allowed to be waived or fixed by the tool by choosing the appropriate policy from the allowed list of policies using the `set_early_data_check_policy` command. To report the different checks that the `check_hier_design` performs, and learn about the available policies for those checks, you can use the command `report_hier_check_description`.  
`check_hier_design` 명령으로 보고된 오류는 `set_early_data_check_policy` 명령을 사용하여 도구에 의해 워닝 또는 에러로 처리되도록 허용할 수 있습니다. 허용된 정책 목록에서 적절한 정책을 선택하여 설정합니다. 또한 `check_hier_design` 명령이 수행하는 다양한 검사들과 해당 검사들에 대해 사용 가능한 정책들에 대한 정보를 알아보려면 `report_hier_check_description` 명령을 사용할 수 있습니다.  
{: .notice--warning}  

## 20. Timing Constraints and Application Options Consistency

![2023-05-21-timing-constraints-and-app-options.png]({{site.baseurl}}/assets/images/2023-05-21-timing-constraints-and-app-options.png){: .align-center}  

The contents of abstracts depend on the timing constraints loaded in the block. As a best practice for accuracy, it must be ensured that the timing constraints applied from the top is consistent with the timing constraints that were used while creating the abstracts. The `check_hier_design` command can check the consistency of the top-level and block-level constraints for abstracts when the application option `abstract.check_constraints_consistency` is enabled. Additionally, the application options related to timing analysis must be consistent between the block-level design which is represented as an abstract at the top-level and the top-level design to ensure good timing correlation and avoid unexpected stability issues. The `check_hier_design` command automatically compares the predetermined list of timer application option settings that could have a potential to affect timing correlation between the block-level and top-level design.  
추상의 내용은 블록에 로드된 타이밍 제약에 따라 달라집니다. 정확성을 보장하기 위한 최상의 방법은 최상위에서 적용되는 타이밍 제약이 추상을 생성할 때 사용된 타이밍 제약과 일치하는지 확인하는 것입니다. `check_hier_design` 명령은 `abstract.check_constraints_consistency` 애플리케이션 옵션이 활성화되었을 때, 추상의 최상위와 블록 레벨 제약 사이의 일관성을 확인할 수 있습니다. 또한, 타이밍 분석과 관련된 애플리케이션 옵션은 블록 레벨 디자인과 최상위 디자인 간의 일관성을 유지해야 하며, 타이밍 상관 관계를 향상시키고 예상치 못한 안정성 문제를 피하기 위해 조정되어야 합니다. `check_hier_design` 명령은 자동으로 블록 레벨과 최상위 디자인 간에 타이밍 상관관계에 영향을 줄 수 있는 미리 결정된 타이머 애플리케이션 옵션 설정 목록을 비교합니다. 이를 통해 최상위와 블록 레벨 디자인 사이의 제약 조건 일관성을 확인하고 문제를 예방할 수 있습니다. 따라서 추상의 정확성과 최상위 디자인의 타이밍 일치성을 보장하여 전체 디자인의 성능을 최적화할 수 있습니다.
{: .notice--warning}  

## 21. Implement the Top

![2023-05-21-implement-the-top.png]({{site.baseurl}}/assets/images/2023-05-21-implement-the-top.png){: .align-center}  

- The Reference Methodology (RM) provides reference methodology scripts for implementation at each level of the physical hierarchy
  - Supports synthesis, place, cts and route using abstracts and extracted timing models (ETMs)
- The RM scripts can be downloaded from the below link
  - https://solvnet.synopsys.com/rmgen/

Finally, to wrap up the IC Compiler II Reference Methodology provides the reference methodology scripts to perform synthesis, place and route for each level of the physical hierarchy. The RM scripts supports implementation using abstracts and ETM's  
IC Compiler II 참조 방법론은 물리적 계층의 각 수준에 대해 합성, 배치, 라우팅을 수행하기 위한 참조 방법론 스크립트를 제공합니다. 이 방법론 스크립트는 추상과 ETM(Extracted Timing Model)을 사용한 구현을 지원합니다.  
{: .notice--warning}  
