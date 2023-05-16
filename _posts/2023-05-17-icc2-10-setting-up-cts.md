---
title: "IC Compiler II - 10 Setting Up CTS"
excerpt: "Block-level Implementation"
date: 2023-05-17 06:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - IC Compiler II
---

## Objectives

After completing this module, you should be able to

- Set up clock tree balancing constraints
- Handle pre-existing clock tree elements
- Specify non-default rules (NDRs) for clock tree routing
- Specify timing and DRC constraints

## ICC II PnR Flow

![2023-05-17-icc2-pnr-flow-clock-opt.png]({{site.baseurl}}/assets/images/2023-05-17-icc2-pnr-flow-clock-opt.png){: .align-center}  

## Clock Tree Balancing

Let's start off talking about Clock tree balancing as there are several sub topics associated with that.  Before we discuss specific settings and commands, let's first understand what Clock tree synthesis does by default, without specific guidance.  So, we'll cover the basics of where a Clock tree structure begins and where CTS ends when automatically building the Clock trees in our design.  We'll then add more detail by discussing how to change the default behavior in 4 common areas of user control… 1.Sink and Ignore Pins 2.Clock Tree Skew and Latency Targets 3.Controlling CTS Cell Selection Inter-Clock-Tree Balancing  
시작하기 전에, 구체적인 지침 없이 기본 설정으로 Clock 트리를 합성할 때 Clock 트리 합성이 수행하는 작업에 대해 이해해 보겠습니다. 디자인에서 Clock 트리 구조가 시작되는 위치와 CTS가 종료되는 위치에 대한 기본 개념을 다룰 것입니다. 그런 다음 사용자가 제어할 수 있는 네 가지 일반적인 영역에서 기본 동작을 변경하는 방법에 대해 자세히 알아볼 것입니다. 1. Sink and Ignore Pins: Sink 및 Ignore 핀, 2. Clock Tree Skew and Latency Targets: Clock 트리 스쿠 및 지연 목표, 3. Controlling CTS Cell Selection: CTS 셀 선택 제어, 4. Inter-Clock-Tree Balancing: 클럭 트리 간 균형 조정. 각각의 주제에 대해 자세히 알아보겠습니다.  
{: .notice--warning}  

## Where Does the Clock Tree Begin and End?

![2023-05-17-where-clock-tree-begin-end.png]({{site.baseurl}}/assets/images/2023-05-17-where-clock-tree-begin-end.png){: .align-center}  

The start point of a Clock tree is at its source and that is where the create_clock constraints are usually applied.  For example, Clock input ports.  If you have 10 clocks, you will have 10 clock sources and you would expect the process of CTS to automatically synthesize 10 clock trees.  The clock trees end, by default, on clock pins of registers or macros.  Clock tree synthesis generally builds a Clock tree between the source and those endpoints, as shown by the blue buffers. If we wish to change the default behavior, we provide specific CTS constraint guidance; commonly referred to as "clock tree exceptions".  
Clock 트리의 시작점은 그의 소스(source)이며, 일반적으로 create_clock 제약 조건이 적용되는 곳입니다. 예를 들어, 클럭 입력 포트들입니다. 만약 10개의 클럭이 있다면, 10개의 클럭 소스가 있고 CTS 과정에서 자동으로 10개의 클럭 트리가 합성될 것으로 예상됩니다. 기본적으로 클럭 트리 합성은 레지스터나 매크로의 클럭 핀에서 끝납니다. Clock 트리 합성은 일반적으로 소스와 해당 종점 사이에 Clock 트리를 빌드합니다. 이는 파란색 버퍼로 표시됩니다. 그러나 기본 동작을 변경하고자 한다면, 구체적인 CTS 제약 조건 가이드를 제공해야 합니다. 이는 일반적으로 "클럭 트리 예외"라고 알려져 있습니다.  
{: .notice--warning}  

## Balance Points: Sink and Ignore Pins

![2023-05-17-balance-points-sink-ignore.png]({{site.baseurl}}/assets/images/2023-05-17-balance-points-sink-ignore.png){: .align-center}  

- Sink Pins:
  - CTS optimizes for DRC and clock tree targets (skew, insertion delay)
  - Considers internal insertion delays
- Ignore Pins:
  - CTS ignores skew and insertion delay targets
  - CTS inserts a guide buffer
    - Only DRCs fixed after guide buffer

These endpoints, called sink pins, (that's SYN-C not SIN-K) are where CTS tries to balance delays throughout the clock tree structure such as to minimize clock skew effects whilst honoring the strict requirements for logical clock DRC; maximum transition and maximum capacitance rules.  By default, CTS considers the clock pins on sequential cells (such as flip-flops and latches) and macros to be sink pins. Conversely, anything not considered a sink pin is generally considered an ignore pin.  These, as their name suggests, should be ignored during clock tree balancing. Some example ignore pins are shown here; non-clock pins on sequential cells (e.g. data, set, reset etc.) and clock output ports.  The notes contain further examples. However, CTS still works on the nets leading to ignore pins.  It does this to meet the clock tree logical DRC rules, max_transition and max_fanout, on these paths. It inserts what are termed guide buffers to separate ignore pins from the rest of the tree structure and to ensure the DRC rules are met. When CTS determines these default sink and ignore pins in the clock tree fanout, this is an implicit decision.  So, we refer to implicit sink pins as well as implicit ignore pins.  We will discuss their explicit counterparts soon… CTS can also consider internal insertion delays of other design macros.  Let's look at the IP example shown here.  Imagine this is a large IP or macro block that already has its own Clock tree built within; represented here by the dotted nets and buffers. That sub-tree will connect to the clock tree in our design and so we will need to consider what happens to the clock sinks in our design, relative to the sinks in the IP block.  Do we want CTS to balance the latency seen inside the IP block with the clock tree branches of the design or do we wish it to ignore those internal delays? By default, the IP's clock pins are seen as sink pins and CTS will attempt to balance them.  
이러한 종점인 sink 핀들은 CTS가 클럭 트리 구조 전체에서 지연을 균형있게 조정하려고 하는 지점입니다. 클럭 스큐 효과를 최소화하면서 논리적인 클럭 DRC(Design Rule Check) 요구 사항인 최대 전이와 최대 용량 규칙을 준수하기 위해 노력합니다. 기본적으로 CTS는 순차 셀(플립플롭이나 래치 등)과 매크로의 클럭 핀을 sink 핀으로 간주합니다. 반대로, sink 핀으로 간주되지 않는 것은 일반적으로 ignore 핀으로 간주됩니다. 이름에서 알 수 있듯이, 이러한 ignore 핀은 클럭 트리 균형 조정 중 무시되어야 합니다. 여기에는 순차 셀의 클럭 핀 이외의 핀(데이터, 설정, 리셋 등) 및 클럭 출력 포트와 같은 예제 ignore 핀이 표시되어 있습니다. 추가적인 예제는 참고 자료에 있습니다. 그러나 CTS는 여전히 ignore 핀으로 연결되는 넷에서 작동합니다. 이는 클럭 트리의 논리적인 DRC 규칙(최대 전이 및 최대 팬아웃)을 충족하기 위해 이러한 경로에서 가이드 버퍼를 삽입하는 것입니다. CTS가 클럭 트리의 기본 sink 및 ignore 핀을 결정할 때, 이는 암묵적인 결정입니다. 따라서 우리는 암묵적인 sink 핀과 ignore 핀이라고 말합니다. 곧 명시적인 대응 핀에 대해 자세히 논의할 것입니다. 또한 CTS는 다른 디자인 매크로의 내부 삽입 지연도 고려할 수 있습니다. 여기에서 보여지는 IP 예시를 살펴보겠습니다. 이것은 이미 자체적인 클럭 트리가 내장된 대규모 IP 또는 매크로 블록이라고 가정합니다. 점선으로 표시된 넷과 버퍼로 표현되는 것입니다. 그 하위 트리는 우리 디자인의 클럭 트리와 연결되므로, 우리는 우리 디자인의 클럭 종점이 IP 블록 내의 종점과 어떻게 동작하는지 고려해야 합니다. CTS가 IP 블록 내부의 지연을 디자인의 클럭 트리 가지와 균형있게 조정하도록 할 것인지, 아니면 해당 내부 지연을 무시하도록 할 것인지를 결정해야 합니다. 기본적으로 IP의 클럭 핀은 sink 핀으로 간주되며, CTS는 이들을 균형 조정하려고 시도합니다.  
{: .notice--warning}  

## Generated and Gated Clocks

![2023-05-17-generated-and-gated-clocks.png]({{site.baseurl}}/assets/images/2023-05-17-generated-and-gated-clocks.png){: .align-center}  

Clock gating is extensively deployed in modern clock tree design to save power and so integrated clock gating cells (or ICGs) are usually part of the overall clock tree structure. In addition, we can define generated clocks when we write our clock timing constraints, typically to establish clock divider circuitry that runs at a different frequency relative to its master clock. In this example we see one clock that fans out to 5 sequential flops through different branches. These branches contain both an ICG and clock divider example. For the ICG cell (near the top), despite it having a clock input pin, CTS, by default, does not consider it a sink pin and continues to build and balance the clock tree structure beyond the ICG.  You could imagine it as an AND gate or any combinatorial logic.  So ICG cells are transparent to the tree building process but automatically identified during CTS.  They are neither sink or ignore pins. On the bottom the clock tree structure traverses through a divide-by-two flip flop driving 2 leaf level registers.  Normally the clock pin of a flip flop would be considered a sink but due to the generated clock definition on the clock divider flops output Q pin, it understands that flops input clock pin is not a sink pin.  Therefore, CTS will traverse through the divider flop and balance the sinks beyond (clock pins of flops FF4/FF5) What we see here in general is that master clocks and generated clocks are part of the same clock domain from a clock tree synthesis perspective. All these endpoints, FF1 through FF3, and FF4 and FF5, even though at different frequencies, are all considered part of the same Clock domain and so CTS will attempt to balance the delays to all five endpoints to minimize clock skew. That's called global skew balancing. Here the picture annotates the clock tree latency arcs from source to sinks after CTS, indicating that it realistically achieved a latency range of 0.63 to 0.65 library units.  These would be considered the minimum and maximum clock tree latencies and therefore a skew of 0.02 was achieved.  

현대적인 클럭 트리 설계에서는 전력 절약을 위해 클록 게이팅이 널리 사용되며, 통합 클록 게이팅 셀(ICG)은 일반적으로 전체 클럭 트리 구조의 일부입니다. 또한, 클록 타이밍 제약을 작성할 때 생성된 클록을 정의할 수 있습니다. 이는 일반적으로 마스터 클록과 다른 주파수에서 동작하는 클록 분배 회로를 설정하는 데 사용됩니다. 이 예에서는 하나의 클록이 서로 다른 가지를 통해 5개의 순차 플립플롭에 팬아웃되는 것을 볼 수 있습니다. 이 가지에는 ICG와 클록 분배 예시가 포함되어 있습니다. 맨 위의 ICG 셀은 클록 입력 핀을 가지고 있지만, 기본적으로 CTS는 이를 sink 핀으로 간주하지 않고 ICG를 넘어서 클럭 트리 구조를 계속 구축하고 균형을 조정합니다. 이는 AND 게이트나 기타 조합 논리로 생각할 수 있습니다. 따라서 ICG 셀은 트리 구축 과정에서는 투명하게 동작하지만, CTS 중에 자동으로 식별됩니다. 이들은 sink 핀이나 ignore 핀이 아닙니다. 아래에서는 클럭 트리 구조가 2개의 리프 레벨 레지스터를 구동하는 2로 나누는 플립플롭을 통해 이동합니다. 일반적으로 플립플롭의 클록 핀은 sink로 간주될 것입니다. 그러나 클록 분배 플립플롭의 출력인 Q 핀에 대한 생성된 클록 정의로 인해, CTS는 플립플롭의 입력 클록 핀이 sink 핀이 아니라는 사실을 이해합니다. 따라서 CTS는 분배 플립플롭을 통과하고 이후의 sink(FF4/FF5의 클록 핀)를 균형있게 처리할 것입니다. 일반적으로 여기서 볼 수 있는 것은 마스터 클록과 생성된 클록이 클럭 트리 합성 관점에서 동일한 클럭 도메인의 일부라는 것입니다. FF1부터 FF3, FF4 및 FF5까지의 이러한 종점들은 서로 다른 주파수에서 동작하더라도 동일한 클록 도메인의 일부로 간주되므로, CTS는 클럭 스큐를 최소화하기 위해 모든 다섯 종점으로의 지연을 균형있게 조정하려고 할 것입니다. 이를 전역 스큐 균형화라고 합니다. 여기에서 사진은 CTS 이후에 소스에서 sink로 향하는 클럭 트리 지연 아크를 주석으로 표시하고 있으며, 실제로 0.63에서 0.65 라이브러리 단위의 지연 범위를 달성했다고 나타냅니다. 이는 최소 및 최대 클럭 트리 지연이며, 따라서 0.02의 스큐를 달성했습니다.  
{: .notice--warning}  

## User-defined or Explicit Sink Pins

![2023-05-17-user-defined-sink-pins.png]({{site.baseurl}}/assets/images/2023-05-17-user-defined-sink-pins.png){: .align-center}  

- **Scenario:** If a macro cell clock pin is correctly defined, CTS will treat that pin as an implicit sink pin. In this example the clock pin is not correctly defined. What will CTS do?
- The macro’s clock pin is marked as an implicit ignore pin - no skew optimization!

To change the default behavior of the clock tree building process that we've discussed so far, the user can consider applying clock tree exceptions.  In this example we see an IP block connected to the clock tree structure.  This IP was intended to have a clock input pin but something went wrong in the macro definition; maybe a model was handwritten by the IP creator. For a clock pin to be recognized as a clock pin, it must have two attributes •a triggering edge direction (positive edge or negative edge) •at least one timing arc associated with it (a setup or a hold timing arc) If either of those two things are missing or incorrectly applied, then this is not seen as a clock pin and CTS will treat it as an implicit ignore pin (as previously discussed). But what if you want to tell CTS that the intention is to balance to this IP clock pin, despite the clock pin definition issue?  What can you do to override the implicit ignore behavior? The ideal solution would be to request the IP owner correctly generate the library model to ensure it's seen as a proper clock pin.  But that might not always be practical or timely.  
지금까지 논의한 클럭 트리 구축 과정의 기본 동작을 변경하려면 사용자는 클럭 트리 예외를 적용할 수 있습니다. 이 예에서는 IP 블록이 클럭 트리 구조에 연결된 것을 볼 수 있습니다. 이 IP는 클록 입력 핀을 가지고 있어야 했지만 매크로 정의 과정에서 무언가 잘못되었을 수 있습니다. 아마도 IP 생성자가 직접 모델을 작성했을 수도 있습니다. 클록 핀이 클록 핀으로 인식되기 위해서는 두 가지 속성이 있어야 합니다.  1. 트리거링 엣지 방향 (양의 엣지 또는 음의 엣지), 2. 연결된 최소한 하나의 타이밍 아크 (셋업 또는 홀드 타이밍 아크). 이 두 가지 중 하나라도 누락되거나 잘못 적용된 경우, 이는 클록 핀으로 인식되지 않으며 CTS는 암묵적인 ignore 핀으로 처리합니다 (이전에 설명한 대로). 그러나 클록 핀 정의 문제에도 불구하고 CTS에게 이 IP 클록 핀을 균형 조정할 의도가 있다고 알리고 싶다면 어떻게 해야 할까요? 암묵적인 ignore 동작을 무시하도록 재정의하려면 어떻게 해야 할까요? 이상적인 해결책은 IP 소유자에게 올바르게 라이브러리 모델을 생성하도록 요청하여 올바른 클록 핀으로 인식되도록 하는 것입니다. 그러나 그것이 항상 현실적이거나 적시적으로 이루어질 수 있는 것은 아닐 수 있습니다.  
{: .notice--warning}  

## Defining an Explicit Sink Pin

![2023-05-17-define-explicit-sink.png]({{site.baseurl}}/assets/images/2023-05-17-define-explicit-sink.png){: .align-center}  

- Defining an explicit sink pin allows CTS to optimize for skew and insertion delay targets.
- CTS has no knowledge of the IP-internal clock delay - it can only “see” up to the sink pin!

```tcl
set_clock_balance_points -balance_points [get_pins IP/IP_CLK]
```

You can override the default behavior by explicitly applying a clock balance point exception on the IP's clock pin. With that defined, clock tree synthesis will now see the IP's clock pin as a valid sink and balance to it. Remember, we only require this in the situation where the IP's clock pin is not being properly modelled as a clock pin. If it were, we'd simply have an implicit sink pin and no requirement for this additional balance point constraint. When you subsequently receive an updated (and corrected) IP with the correct clock pin definition you can then swap it out. You don't have to rebuild your Clock tree because it's already balanced. As the IP's incorrectly modelled clock pin has now been made a sink through user control, we refer to this as an explicit sink pin. The explicit sink pin definition is created using the set_clock_balance_points command. Note, however, that whilst we have balanced to the IP's clock pin, CTS will not see the IP's internal clock tree latency. We'll cover how to accommodate that situation next…  
IP의 클록 핀에 명시적으로 클록 균형 지점 예외를 적용하여 기본 동작을 재정의할 수 있습니다. 이렇게 정의하면 클록 트리 합성은 이제 IP의 클록 핀을 유효한 sink로 인식하고 균형을 맞출 것입니다. 기억해야 할 점은 이를 IP의 클록 핀이 올바르게 모델로 구현되지 않는 경우에만 필요합니다. 그렇지 않다면, 암묵적인 sink 핀으로 간주되며 이러한 추가적인 균형 지점 제약이 필요하지 않습니다. 이후 올바른 클록 핀 정의가 있는 업데이트(및 수정)된 IP를 받게 되면 해당 IP로 교체할 수 있습니다. 클럭 트리를 다시 빌드할 필요가 없으며 이미 균형을 맞춘 상태입니다. IP의 잘못된 클록 핀이 사용자의 제어를 통해 sink로 만들어졌으므로 이를 명시적인 sink 핀이라고 합니다. 명시적인 sink 핀 정의는 set_clock_balance_points 명령을 사용하여 생성됩니다. 그러나 IP의 클록 핀에 균형을 맞추었지만, CTS는 IP의 내부 클록 트리 지연을 인식하지 않습니다. 다음으로 이러한 상황을 수용하는 방법에 대해 다룰 것입니다.  
{: .notice--warning}  

## Defining an Explicit Sink Pin With Delay

![2023-05-17-define-explicit-sink-with-delay.png]({{site.baseurl}}/assets/images/2023-05-17-define-explicit-sink-with-delay.png){: .align-center}  

- Defining an explicit sink pin with delay allows CTS to adjust the insertion delays based on the specification.

```tcl
set_clock_balance_points    \
    -corner ss125c          \
    -delay 0.15 -late       \
    -balance_points IP/IP_CLK
```

In addition to creating a clock balance point on the IP clock pin, you can also model the internal clock tree latency within the IP.   You can specify both the minimum (early) and maximum (late) clock tree latencies values. In the example shown and with the IP now modelled with a maximum internal latency of 0.15, CTS can determine the appropriate latency of the connecting branch in order to balance the clock skew globally.  Here it shows the insertion delay of 0.28 being evaluated by CTS (0.28+0.15 of IP, equals 0.43 total latency on the IP branch) . Notice that when you specify a positive number for your clock balance point delay, that means the latency of the clock path to the IP clock pin is reduced.  We are preponing the clock arrival time.  If you recall from previous units we talked about balance points and that during CCD place_opt is going to determine where to postpone or prepone clocks. It calculates latency offsets which are translated into clock balance points that include a delay specification.  A positive number doesn't mean the clock should arrive later.  It means it should arrive earlier.  Negative delays mean you are postponing. This, by the way, is also a trick you can deploy to manually postpone or prepone clock tree branches in order to meet specific design goals. For example, macros or RAMs commonly have timing closure issues so the user can intervene and manually postpone or prepone the clock arrival times on RAMs to help meet the input side timing or the output side timing; depending on where the timing challenges lie.  
IP의 클록 핀에 클록 균형 지점을 생성하는 것 외에도 IP 내부의 클록 트리 지연을 모델링할 수도 있습니다. 최소(일찍)와 최대(늦게) 클록 트리 지연 값을 지정할 수 있습니다. 위 예시에서는 IP를 최대 내부 지연이 0.15인 모델로 변경한 경우, CTS는 클록 스큐를 전역적으로 균형잡기 위해 연결되는 가지의 적절한 지연을 결정할 수 있습니다. 여기에서는 0.28의 삽입 지연이 CTS에 의해 평가되고 있으며 (0.28+0.15의 IP), IP 가지의 총 지연은 0.43입니다. 클록 균형 지점 지연에 양수 값을 지정할 때, IP 클록 핀까지의 클록 경로의 지연이 감소한다는 것을 알아두세요. 클록 도착 시간을 조기화하고 있습니다. 이전 단원에서 균형 지점에 대해 이야기했을 때, CCD place_opt는 클록을 연기하거나 조기화할 위치를 결정한다고 말했습니다. 이는 지연 오프셋을 계산하고 이를 클록 균형 지점에 포함하는 것으로 전환되는 것입니다. 양수 값은 클록이 더 늦게 도착해야 함을 의미하는 것이 아닙니다. 클록이 더 일찍 도착해야 함을 의미합니다. 음수 지연은 연기함을 의미합니다. 이것은 또한 특정 설계 목표를 충족시키기 위해 수동으로 클럭 트리 가지를 연기하거나 조기화하는 데 사용할 수 있는 방법입니다. 예를 들어, 매크로나 RAM과 같은 경우 일반적으로 타이밍 클로저 문제가 발생할 수 있으므로 사용자는 RAM의 클록 도착 시간을 수동으로 연기하거나 조기화하여 입력측 타이밍이나 출력측 타이밍을 충족시키는 데 도움을 줄 수 있습니다.  
{: .notice--warning}  

## Defining an Explicit Ignore(or Exclude) Pin

![2023-05-17-define-explicit-ignore-pin.png]({{site.baseurl}}/assets/images/2023-05-17-define-explicit-ignore-pin.png){: .align-center}  

- Explicit ignore pins cause CTS to ignore this pin and down-stream sinks for skew balancing.

```tcl
set_clock_balance_points          \
    -clock CLOCK                  \
    -consider_for_balancing false \
    -balance_points [get_pins AN2/A]
```

→ CTS inserts a guide buffer; CTS engine fixes DRCs only along the clock path past ignore pins.  

There are times when we wish to turn off the automatic balancing behavior of the clock tree building so that we can guide CTS towards the branches that would benefit most from being balanced. To achieve that we can define an explicit ignore pin; a point at which CTS stops balancing. In the example, the clock splits into 2 main branches; the upper driving clock pins of FF1 and FF2 and a lower branch that traverses to other sinks after going through a clock gate. As we previously discussed, CTS would automatically traverse through the clock gate and onwards to it sinks, such that FF1/FF2 and all the other sinks are globally balanced. But that was not our intention here.  Maybe for some reason we know that the top two registers do not communicate or interact with the bottom logic.  Or maybe we simply do not care about skew balancing in that lower branch as perhaps that's not timing critical logic. If we only want skew balancing to occur on FF1 and FF2 and nothing else, since we have some gating logic, the approach taken here is to define the A-pin of the And-gate (gator) as a pin where balancing is not considered.  We use the same set_clock_balance_point command but this time the -consider_for_balancing option is set to false; i.e. we tell CTS to ignore this pin and its associated fanout from clock tree balancing. Consequently, and for the reasons previously discussed, CTS inserts a guide buffer on that branch prior to the and gate to ensure that clock net DRCs are honored.  
클럭 트리 구축의 자동 균형 조정 동작을 끄고, CTS를 가장 균형을 맞추어야 하는 가지로 안내할 수 있는 경우가 있습니다. 이를 위해 명시적인 ignore 핀을 정의하여 CTS가 균형을 맞추지 않도록 할 수 있습니다. 예를 들어, 클록이 2개의 주요 가지로 분기되는 경우가 있습니다. 상위 가지는 FF1과 FF2의 클록 핀을 구동하고, 하위 가지는 클럭 게이트를 통과한 후 다른 종단지로 이동합니다. 이전에 논의한 대로, CTS는 클럭 게이트를 자동으로 통과하여 FF1/FF2와 다른 종단지가 전역적으로 균형있게 구축됩니다. 그러나 이 경우에는 우리의 의도와는 다릅니다. 어떤 이유에서든지 상위 두 레지스터가 하단 로직과 통신이나 상호작용하지 않는다는 것을 알고 있는지도 모르겠고, 아니면 하위 가지에서의 스큐 균형 조정이 타이밍에 중요하지 않을 수도 있습니다. 우리는 FF1과 FF2에서만 스큐 균형 조정이 발생하도록 원하며, 게이팅 로직이 있기 때문에 여기에서 취한 접근 방식은 And 게이트(gator)의 A 핀을 균형 조정이 고려되지 않는 핀으로 정의하는 것입니다. 동일한 set_clock_balance_point 명령을 사용하지만, 이번에는 -consider_for_balancing 옵션을 false로 설정합니다. 즉, CTS에게 이 핀과 해당하는 클럭 트리 fanout을 균형 조정에서 무시하도록 지시합니다. 이로 인해 이전에 논의한 이유로 CTS는 클럭 넷 DRC를 준수하기 위해 And 게이트 이전에 가이드 버퍼를 삽입합니다.  
{: .notice--warning}  

## Defining a Clock Skew Group

![2023-05-17--define-clock-skew-group.png]({{site.baseurl}}/assets/images/2023-05-17--define-clock-skew-group.png){: .align-center}  

- Define a clock skew group if you want to balance a group of clock pins independently from the rest of the clock tree.
- The target skew will be the same as the master clock of this skew group. You cannot set the skew independently.

```tcl
create_clock_skew_group -mode TEST -objects {FF1/CLK FF2/CLK}
```

## 생략

## Summary

That now concludes this training module on CTS setup.  We discussed the following main topic areas that should now enable you to… •Set up clock balancing constraints. •Handle any pre-existing clock tree elements in your design. •Specify Non-Default routing Rules to ensure clock tree routing avoids Signal Integrity issues such as Crosstalk and Electromigration. •Specify clock specific timing and DRC constraints. Please refer to the Lab notes for Lab 3, Setting Up CTS, which helps put these concepts into practice!  
이제 CTS 설정에 대한 교육 모듈을 마무리합니다. 다음과 같은 주요 주제를 논의했으며, 이를 통해 다음과 같은 작업을 수행할 수 있게 되었습니다. •클록 균형 조정 제약 설정하기 •설계 내 기존 클록 트리 요소 처리하기 •신호 무결성 문제인 크로스토크 및 전기 이동과 같은 문제를 피하기 위해 클록 트리 라우팅에 대한 비표준 라우팅 규칙 지정하기 •클록 특정 타이밍 및 DRC(Design Rule Check) 제약 사항 지정하기 이러한 개념을 실제로 적용하는 데 도움이 되는 Lab 3, "CTS 설정"의 실험 자료를 참조해 주세요!  
{: .notice--warning}  
