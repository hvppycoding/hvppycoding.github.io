---
title: "[Review] Sensitivity-guided metaheuristics for accurate discrete gate sizing"
excerpt: "ICCAD 2012"
date: 2023-06-05 20:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Review
mathjax: "true"
---

Jin Hu†, Andrew B. Kahng‡ + , SeokHyeong Kang‡ , Myung-Chul Kim† and Igor L. Markov†  

† University of Michigan, 2260 Hayward St., Ann Arbor, MI 48109, {jinhu, mckima, imarkov}@eecs.umich.edu UC San Diego,  

‡ ECE and + CSE Depts., La Jolla, CA 92093, {abk, shk046}@ucsd.edu  

## Abstract

The well-studied gate-sizing optimization is a major contributor to IC power-performance tradeoffs. Viable optimizers must accurately model circuit timing, satisfy a variety of constraints, scale to large circuits, and effectively utilize a large (but finite) number of possible gate configurations, including Vt and Lg . Within the research-oriented infrastructure used in the ISPD 2012 Gate Sizing Contest, we develop a metaheuristic approach to gate sizing that integrates timing and power optimization, and handles several types of constraints. Our solutions are evaluated using a rigorous protocol that computes circuit delay with Synopsys PrimeTime. Our implementation Trident outperforms the best-reported results on all but one of the ISPD 2012 benchmarks. Compared to the 2012 contest winner, we further reduce leakage power by an average of 43%.  

잘 연구된 게이트 크기 최적화는 집적회로의 전력-성능 트레이드오프에 주요한 요소로 알려져 있다. 유효한 최적화 도구는 회로의 타이밍을 정확하게 모델링하고 다양한 제약 조건을 충족시키며 대규모 회로에 적용할 수 있어야 하며, Vt와 Lg를 포함한 다양한 게이트 구성의 가능성을 효과적으로 활용해야 한다. ISPD 2012 게이트 크기 조정 대회에서 사용된 연구 지향적 인프라에서는 타이밍 및 전력 최적화를 통합하고 여러 유형의 제약 조건을 처리하는 게이트 크기 조정에 대한 메타휴리스틱 접근 방식을 개발한다. 우리의 솔루션은 Synopsys PrimeTime을 사용하여 회로 지연을 계산하는 엄밀한 프로토콜을 통해 평가된다. 저희의 구현체인 Trident는 ISPD 2012 벤치마크 중 한 가지를 제외하고 모든 벤치마크에서 최고의 결과를 보여준다. 2012 대회 우승자와 비교하여, 저희는 누설 전력을 평균 43% 더 줄였다.  

## 1. Introduction

The sizing problem in VLSI design seeks to determine design parameters (e.g., gate width and threshold voltage) for each gate, so as to optimize timing, area and power of a circuit, subject to constraints. The problem has been extensively studied, and a number of heuristics have been proposed. However, there have been no definitive comparisons (empirical or mathematical) of different techniques. Moreover, many published techniques make unrealistic assumptions about the underlying circuits, such as the possibility of continuous gate sizing and Vt assignment and the convexity of delay functions. Some publications neglect the effect of rounding when using a discrete gate library, or do not account for realistic capacitance and slew constraints. Scalability to circuits with hundreds of thousands of gates is also an important issue, whereas many previous publications use much smaller benchmarks mapped into outdated technologies. To address these shortcomings in published literature, Intel researchers have recently prepared and released an extensive infrastructure for research on large-scale gate sizing [23]. This includes (i) a set of benchmarks ranging from small to large, mapped into a modern discrete gate library, and (ii) a set of evaluation protocols that includes checking timing constraints using industry-standard software (Synopsys PrimeTime [33]) and measuring total leakage power for a particular sizing solution. This infrastructure has been used in the ISPD 2012 GateSizing Contest which provided a definitive baseline for further research on this topic.  

VLSI(Very Large Scale Integration) 설계에서의 크기 조정 문제는 각 게이트에 대해 설계 매개변수(예: 게이트 폭과 임계 전압)를 결정하여 회로의 타이밍, 면적 및 전력을 최적화하고 제약 조건을 만족시키는 것을 목표로 합니다. 이 문제는 광범위하게 연구되었으며, 여러 가지 휴리스틱 기법이 제안되었습니다. 그러나 다양한 기법 간의 결정적인 비교(경험적 또는 수학적)는 없었습니다. 게다가, 많은 게재된 기법들은 게이트 크기 조정 및 Vt 할당의 연속성 가능성 및 지연 함수의 볼록성과 같은 기본 회로에 대한 비현실적인 가정을 하였습니다. 일부 출판물은 이산적인 게이트 라이브러리를 사용할 때 반올림의 효과를 간과하거나 현실적인 커패시턴스 및 슬루 제약 조건을 고려하지 않습니다. 수십만 개의 게이트를 가진 회로로의 확장성은 또한 중요한 문제이며, 이전의 많은 출판물은 오래된 기술로 매핑된 훨씬 작은 벤치마크를 사용합니다. 출판된 문헌의 이러한 단점을 해결하기 위해 인텔 연구원들은 최근 대규모 게이트 크기 조정 연구를 위한 포괄적인 인프라를 준비하고 공개했습니다 [23]. 이는 (i) 작은 것부터 큰 것까지 다양한 벤치마크를 현대적인 이산 게이트 라이브러리에 매핑한 것과 (ii) 특정 크기 조정 솔루션의 타이밍 제약 조건을 업계 표준 소프트웨어(Synopsys PrimeTime [33])를 사용하여 확인하고 총 누설 전력을 측정하는 평가 프로토콜 세트가 포함되어 있습니다. 이 인프라는 이후 연구를 위한 명확한 기준선을 제공한 ISPD 2012 Gate Sizing Contest에서 사용되었습니다.  

Our research reported in this paper focuses on large-scale optimization of gate sizes under realistic circumstances. Our techniques accurately track circuit timing throughout the optimization process, ensure the satisfaction of several types of constraints, and identify the gates with the greatest impact on power-performance tradeoffs. Rather than approximate gate sizing by continuous convex optimization, as is done in many prior publications, we fully account for the discrete nature of the problem and the nonconvexities of circuit delay functions. Our optimizations try not to overlook available opportunities to improve power-performance tradeoffs, but are also fast and can quickly traverse large regions of the solution space. They are highly modular, and are organized in a hierarchy where high-level metaheuristics configure and drive heuristics, which are assembled from lower-level optimization blocks. The lower-level optimizations are in turn based on high-performance timing and power analysis, constraint repair, search, gain calculations, sorting and prioritization, as well as roll-back. Some of these ideas have been known before, but every hierarchical level in our techniques contains some novel elements. Moreover, in a highly studied and competitive field such as gate sizing, reliable individual components form only a small fraction of overall success — a larger fraction of success is in the selection, ordering and composition of these components, as well as the overall ﬂow. Our most important decisions rely on new insights into large-scale gate sizing and its interaction with design constraints. The main contributions of our work are summarized below.  

본 논문에서 보고된 연구는 현실적인 상황에서의 대규모 게이트 크기 최적화에 초점을 맞추고 있습니다. 우리의 기술은 최적화 과정 전반에 걸쳐 회로의 타이밍을 정확하게 추적하며 여러 종류의 제약 조건을 만족시키고 전력-성능 트레이드오프에 가장 큰 영향을 미치는 게이트를 식별합니다. 많은 이전 출판물에서 수행되는 것처럼 연속적인 볼록 최적화에 의한 게이트 크기 조정을 근사하지 않고, 문제의 이산적인 성격과 회로 지연 함수의 비볼록성을 완전히 고려합니다. 우리의 최적화는 전력-성능 트레이드오프를 개선할 수 있는 가능성을 놓치지 않으려고 하지만 동시에 빠르며 솔루션 공간의 큰 영역을 신속하게 탐색할 수 있습니다. 이러한 기술은 매우 모듈화되어 있으며, 고수준 메타휴리스틱이 휴리스틱을 구성하고 주도하는 계층 구조로 구성되어 있습니다. 낮은 수준의 최적화는 고성능 타이밍 및 전력 분석, 제약 조건 복구, 탐색, 이득 계산, 정렬 및 우선 순위 지정, 롤백 등을 기반으로 합니다. 이러한 아이디어 중 일부는 이전에 알려져 있었지만, 우리 기술의 각 계층은 새로운 요소를 포함하고 있습니다. 또한, 게이트 크기 조정과 디자인 제약 조건의 상호작용에 대한 대규모 게이트 크기 조정에 대한 새로운 통찰력을 기반으로 가장 중요한 결정을 내립니다. 우리의 연구의 주요 기여는 아래에 요약되어 있습니다.  

- We use a sensitivity-guided metaheuristic approach based on sequential importance sampling [1] that integrates power and timing optimization, and handles several types of constraints.
- We propose to use Total Negative Slack (TNS) rather than just the worst slack within sensitivity functions, but observe that the impact of individual gate sizing changes on TNS takes too long to compute. Therefore, we apply a fast technique to estimate TNS impact of gate sizing.
- We define a parameterized space of sensitivity functions for gate sizing and traverse this space using a multistart technique that naturally lends itself to efficient parallelization on multi-core and shared memory CPU architectures, and distributed systems.
- Empirical results on the ISPD 2012 contest benchmarks show that our heuristics further improve the best-found solutions in the contest on 13 out of 14 benchmarks by 11% (Table 3). Given that different contestants excelled on different benchmarks, our sizer Trident outperforms any one contestant by a wide margin.

The rest of this paper is organized as follows. Section 2 covers background and reviews prior work. Section 3 introduces our gate sizing heuristics and their details. Section 4 provides experimental results and analysis. Section 5 concludes the paper.

- 우리는 전력 및 타이밍 최적화를 통합하고 여러 종류의 제약 조건을 처리하는 감도 지도 메타휴리스틱 접근 방식을 사용합니다. 이는 순차적 중요성 샘플링을 기반으로 한다 [1].
- 우리는 감도 함수 내에서 최악의 여유 시간(slack)뿐만 아니라 Total Negative Slack (TNS)을 사용하는 것을 제안하지만, 개별 게이트 크기 변경이 TNS에 미치는 영향을 계산하는 데에는 시간이 너무 오래 걸린다는 것을 관찰하였습니다. 따라서, 게이트 크기 조정의 TNS 영향을 빠르게 추정하기 위해 빠른 기술을 적용합니다.
- 우리는 게이트 크기 조정을 위한 매개변수화된 감도 함수 공간을 정의하고, 이 공간을 다중 시작 기법을 사용하여 탐색합니다. 이는 멀티코어 및 공유 메모리 CPU 아키텍처 및 분산 시스템에서 효율적인 병렬화에 적합합니다.
- ISPD 2012 대회 벤치마크에 대한 경험적인 결과는 우리의 휴리스틱이 대회에서 찾은 최고의 솔루션을 14개 벤치마크 중 13개에서 11% 개선함을 보여줍니다 (표 3). 다른 참가자들이 다른 벤치마크에서 우수한 성과를 거둔 것을 감안할 때, 우리의 크기 조정 기법인 Trident는 어떤 참가자보다 우수한 결과를 크게 보여줍니다.

이 논문의 나머지는 다음과 같이 구성되어 있습니다. 섹션 2에서는 배경과 이전 연구를 다룹니다. 섹션 3에서는 게이트 크기 조정 휴리스틱과 그 세부 사항을 소개합니다. 섹션 4에서는 실험 결과와 분석을 제공합니다. 섹션 5에서 논문을 마무리합니다.  

## 2. BACKGROUND AND PRIOR WORK

The problem of sizing standard cells in digital circuits has been extensively studied due to its importance, and many optimization techniques have been developed. Before discussing specific ideas, we introduce the optimization objectives and relevant constraints.  

디지털 회로에서의 표준 셀 크기 조정 문제는 중요성 때문에 광범위하게 연구되었으며, 많은 최적화 기법이 개발되었습니다. 구체적인 아이디어를 논의하기 전에 최적화 목표와 관련된 제약 조건을 소개하겠습니다.  

### 2.1 Gate Sizing Formulation

Consider a netlist $N = (C, I, P)$, where $C$ is the set of cells, $I$ is the set of interconnects between cells, and $P$ is the set of pins on $C$ . Also given is the input technology library $L$ , where each cell $c ∈ C$ has a set of valid (manufacturable) sizes. Given $N$ and $L$, the objective of (discrete) gate sizing is to map $C$ to $L$ while minimizing the total power consumption subject to design constraints. We consider (i) slack constraints, (ii) capacitance constraints, and (iii) slew constraints.  

넷리스트 $N = (C, I, P)$가 주어진다. 여기서 $C$는 셀의 집합, $I$는 셀 간의 인터커넥션의 집합, $P$는 셀 $C$의 핀의 집합이다. 또한, 입력 기술 라이브러리 $L$이 주어진다. 각 셀 $c ∈ C$는 유효한(제조 가능한) 크기의 집합을 가진다. $N$과 $L$이 주어진 경우, (이산적인) 게이트 크기 조정의 목표는 설계 제약 조건을 만족하면서 전체 전력 소비를 최소화하는 동안 $C$를 $L$에 매핑하는 것이다. 우리는 다음과 같은 제약 조건을 고려한다: (i) 여유 시간(slack) 제약 조건, (ii) 커패시턴스(capacitance) 제약 조건, 그리고 (iii) 슬루(slew) 제약 조건.  

### 2.2 Preview Work on Gate Sizing

The vast majority of gate sizing research has focused on finding continuous solutions, where parameters can take values within certain contiguous ranges. Some techniques optimize transistor parameters (e.g., threshold voltage), and others focus on entire standard cells and deal with drive strength and input-pin capacitances. However, as modern digital methodologies use only discrete cell types, continuous-valued parameters must be efficiently rounded to allowed discrete values.  

게이트 크기 조정 연구의 대부분은 매개변수가 특정 연속 범위 내에서 값을 가질 수 있는 연속적인 솔루션을 찾는 데 초점을 맞추었습니다. 일부 기술은 트랜지스터 매개변수(예: 임계 전압)를 최적화하고, 다른 기술은 전체 표준 셀에 초점을 맞추며 드라이브 강도와 입력 핀 커패시턴스 등을 다룹니다. 그러나 현대의 디지털 방법론에서는 이산 셀 유형만을 사용하므로, 연속값 매개변수를 허용된 이산값으로 효율적으로 반올림해야 합니다.  

**Continuous methods.** As the gate-sizing problem is easily formulated as an optimization problem, many different implementations have shown success, including:  

- Linear programming[2, 5, 17] and network ﬂow[25]
- Convex nonlinear optimization[6, 26, 27, 31], including Lagrangian relaxation[4, 6, 19, 30, 31]
- Slew budgeting[14]

Due to the complexity of modern designs, the two most scalable approaches are those based on Lagrangian relaxation and sensitivity. Instead of satisfying every imposed constraint, Lagrangian relaxation changes the original constrained problem into an unconstrained problem such that the solutions to the latter can be mapped back to the former. However, Lagrangian relaxation is typically formulated for continuous-variable functions and does not naturally handle discrete gate sizes. Numerical optimization techniques used with Lagrangian relaxation sometimes also assume convexity, which does not hold for practical circuit-delay models.  

**연속적인 방법.** 게이트 크기 조정 문제는 최적화 문제로 쉽게 정식화될 수 있기 때문에 다양한 구현 방법이 성공을 거두었습니다. 이에는 다음이 포함됩니다:  

- 선형 프로그래밍 [2, 5, 17] 및 네트워크 플로우 [25]
- 볼록 비선형 최적화 [6, 26, 27, 31], 라그랑지안 완화(Lagrangian relaxation) [4, 6, 19, 30, 31] 포함
- Slew budgeting [14]

현대적인 설계의 복잡성으로 인해, 라그랑지안 완화와 감도(Sensitivity)를 기반으로 한 두 가지 가장 확장 가능한 접근 방식이 있습니다. 라그랑지안 완화는 강제된 모든 제약 조건을 충족시키는 대신, 원래의 제약이 있는 문제를 제약이 없는 문제로 변환하여 후자의 해결책을 전자에 매핑합니다. 그러나 라그랑지안 완화는 일반적으로 연속 변수 함수를 위해 정식화되며, 이산적인 게이트 크기를 자연스럽게 처리하지 못합니다. 라그랑지안 완화와 함께 사용되는 수치 최적화 기법은 때로는 볼록성을 가정하는데, 이는 실제 회로 지연 모델에는 성립하지 않습니다.  

**Discrete methods.** In contrast to continuous methods, discrete methods assume a fixed cell (and technology) library with a set of discrete cell sizes. While discrete methods avoid the difficult “rounding” problem that continuous methods face, they must directly solve an NP-hard problem [18]. Branch-and-bound [24] and dynamic programming [15, 19, 22] based approaches can be categorized as discrete.  

Another discrete method, the sensitivity-based (iterative) approach, modifies individual components one at a time, such as by changing (i) transistor width, (ii) threshold voltage (in dual-Vt designs), (iii) source voltage (in dual-VDD designs), and (iv) gate and/or wire resistances and capacitances. To further improve performance, the authors of [7] developed a multi-directional search, and the authors of [3] suggested using multistarts.  

Recently, the authors of [22] developed a Lagrangian-relaxation graph-based approach to efficiently assign cell types in very large netlists. Their optimizations have significantly improved power- performance tradeoffs in ICs designed at Intel. This work has also spurred the ISPD 2012 (Discrete) Gate Sizing Contest[23].  

**이산적인 방법.** 연속적인 방법과 달리, 이산적인 방법은 이산적인 셀 크기의 고정된 셀(및 기술) 라이브러리를 가정합니다. 이산적인 방법은 연속적인 방법이 직면하는 어려운 "반올림" 문제를 피하면서도 NP-난해한 문제를 직접적으로 해결해야 합니다 [18]. 가지치기(branch-and-bound) [24] 및 동적 프로그래밍 [15, 19, 22] 기반의 접근 방식은 이산적인 방법으로 분류될 수 있습니다.  

또 다른 이산적인 방법인 감도 기반 (반복적인) 접근 방식은 (i) 트랜지스터 폭, (ii) 임계 전압 (이중-Vt 설계에서), (iii) 소스 전압 (이중-VDD 설계에서), 그리고 (iv) 게이트 및/또는 와이어 저항과 커패시턴스와 같은 개별 구성 요소를 한 번에 하나씩 수정합니다. 성능을 더욱 개선하기 위해 [7]의 저자들은 다방향 검색을 개발하였으며, [3]의 저자들은 다중 시작(multistarts)을 사용하는 것을 제안했습니다.  

최근에는 [22]의 저자들이 그래프 기반의 라그랑지안 완화 접근 방식을 개발하여 매우 큰 넷리스트에서 셀 유형을 효율적으로 할당했습니다. 그들의 최적화는 Intel에서 설계된 IC에서 전력-성능 트레이드오프를 크게 개선시켰습니다. 이 작업은 또한 ISPD 2012 (이산적인) 게이트 크기 조정 대회[23]를 촉발시키기도 했습니다.  

### 2.3 The ISPD 2012 Gate Size Contest

To more accurately model the discrete gate-sizing problem for modern technologies, researchers from Intel organized the ISPD 2012 Gate Sizing Contest [23]. Here, contestants are given (i) a set of benchmarks and (ii) a (simplified) modern standard-cell library. The benchmarks are derived from the IWLS 2005 benchmarks [16] and have between 25K and 995K cells. The contest employs standard industry formats for benchmarks, including Verilog netlists, interconnect parasitics in IEEE SPEF format, and timing constraints in Synopsys Design Constraints (SDC) format. The library implements 12 different logic functions, and 30 different cell types (options) for each logic function — three different threshold voltages (Vt ) and ten different sizes for each Vt . The cell library also has lookup tables for (a) delay and transition time (slew), and (b) max load capacitance (in the Synopsys Liberty format) for each cell type. An industry timing engine – Synopsys PrimeTime – is used as the reference timer. For simplicity, the contest does not capture false paths, placement-dependent distributed interconnect parasitic models, or multiple clock domains. The contest compares leakage power of violation-free solutions. Specific constraints include (i) zero negative slack on all pins, (ii) a 300 ps transition time upper bound, and (iii) maximum load capacitance per cell type.  

인텔 연구원들은 현대 기술을 위해 이산적인 게이트 크기 조정 문제를 보다 정확하게 모델링하기 위해 ISPD 2012 Gate Sizing Contest [23]를 개최했습니다. 여기에서 참가자들은 (i) 벤치마크 세트와 (ii) (단순화된) 현대 표준 셀 라이브러리를 제공받습니다. 벤치마크는 IWLS 2005 벤치마크 [16]를 기반으로 하며, 25K에서 995K 개의 셀을 포함하고 있습니다. 이 대회에서는 벤치마크에 대해 Verilog 넷리스트, IEEE SPEF 형식의 인터커넥트 파라미터, Synopsys Design Constraints (SDC) 형식의 타이밍 제약 조건과 같은 표준 업계 형식을 사용합니다. 라이브러리는 12개의 다른 논리 함수와 각 논리 함수마다 30개의 다른 셀 유형(옵션)을 구현합니다 - 세 가지 다른 임계 전압(Vt) 및 각 Vt에 대해 열 가지 다른 크기. 각 셀 유형에 대해 (a) 지연과 전이 시간(슬루)을 위한 룩업 테이블 및 (b) 최대 부하 커패시턴스 (Synopsys Liberty 형식)가 셀 라이브러리에 포함되어 있습니다. 업계에서 사용되는 타이밍 엔진인 Synopsys PrimeTime이 참조 타이머로 사용됩니다. 간편함을 위해, 이 대회에서는 거짓 경로, 배치에 의존하는 분산 인터커넥트 파라미터 모델 또는 다중 클럭 도메인을 고려하지 않습니다. 대회는 위반 없는 솔루션의 누설 전력을 비교합니다. 구체적인 제약 조건에는 (i) 모든 핀에 대한 음수 여유 시간 제로, (ii) 300 ps의 전이 시간 상한값, (iii) 셀 유형당 최대 부하 커패시턴스가 포함됩니다.  

### 2.4 Stochastic Combinatorial Optimization

High-dimensional, hard combinatorial optimization problems do not admit such mathematically elegant solutions as Lagrangian relaxation, and are often solved using simulated annealing or other metaheuristics that combine local search with a global strategy [9] (e.g., stochastic hill-climbing, tabu search, or genetic crossover). These techniques are difficult to implement well (e.g., require heavy parameter tuning), are generally not reproducible, and require sophisticated mathematics to analyze their asymptotic performance.  

The go-with-the-winners (GWTW) metaheuristic [1] repeatedly invokes greedy heuristics within randomized multistarts to (i) explore a large search space by maintaining a small set of best-seen solutions, and (ii) find a global optimum with high probability, as proven in [1]. Local minima are avoided when better intermediate solutions are found (e.g., in parallel search threads) and yield multiple derived solutions. GWTW is asymptotically more efficient than a single round of independent randomized restarts, and is on par with simulated annealing but significantly easier to analyze [1].  

The maintenance of the best-seen solutions in GWTW is like the “survival of the fittest” invariant in genetic algorithms [9]. However, GWTW does not employ genetic crossover, does not exclude repeat solutions, and allows the number of kept solutions to vary. GWTW can be viewed from the statistics perspective as sequential importance sampling and traces its roots to statistical physics [10].  

고차원의 어려운 조합 최적화 문제는 라그랑지안 완화와 같은 수학적으로 우아한 해결책을 받아들일 수 없으며, 종종 시뮬레이티드 앤닐링(simulated annealing) 또는 로컬 탐색과 전역 전략을 결합한 기타 메타휴리스틱 [9] (예: 확률적 힐 클라이밍(stochastic hill-climbing), 타부 탐색(tabu search) 또는 유전 교차(genetic crossover))을 사용하여 해결됩니다. 이러한 기술들은 잘 구현하기 어렵고 (예: 매개변수 튜닝이 많이 필요), 일반적으로 재현성이 없으며, 그들의 점근적인 성능을 분석하기 위해서는 정교한 수학이 필요합니다.  

GWTW(Go-with-the-Winners) 메타휴리스틱 [1]은 랜덤화된 다중 시작 내에서 탐욕 휴리스틱을 반복적으로 호출하여 (i) 최상의 해를 유지하면서 큰 탐색 공간을 탐색하고, (ii) [1]에서 증명된 대로 높은 확률로 전역 최적해를 찾습니다. 더 나은 중간 솔루션이 발견되면 (예: 병렬 탐색 스레드에서), 지역 최솟값은 피하고 여러 파생 솔루션을 얻을 수 있습니다. GWTW는 독립된 랜덤 재시작 한 번에 비해 점근적으로 더 효율적이며, 시뮬레이티드 앤닐링과 유사한 성능을 가지지만 분석하기 훨씬 쉽습니다 [1].  

GWTW에서 최상의 해를 유지하는 것은 유전 알고리즘의 "생존자 유지" 불변식과 유사합니다 [9]. 그러나 GWTW는 유전 교차를 사용하지 않으며, 중복된 솔루션을 배제하지 않으며, 유지되는 솔루션의 수를 변동시킬 수 있습니다. GWTW는 통계적인 관점에서 순차적 중요성 샘플링과 연결되며, 통계 물리학에 근본을 두고 있습니다 [10].  

## 3. OUR METAHEURISTICS

Our proposed heuristic has two stages – Global Timing Recovery (GTR) and Power Reduction with Feasible Timing (PRFT). GTR first seeks violation-free (feasible) solutions, and then PRFT iteratively reduces total leakage power of sizing solutions by local search, as illustrated in Figure 1. At each stage of our optimization ﬂow, we parameterize the space of sensitivity functions, and traverse this space to find the best configurations of sensitivity by independent multistarts (Figure 2). After each multistart, we compare all obtained solutions and retain the best/non-dominated solutions. This is accomplished by adapting the go-with-the-winners (GWTW) metaheuristic (Section 2.4). However, our optimization is purely deterministic in that our multistart procedure begins with the small set of the best-seen solutions, whereas GWTW is typically randomized. Solutions after each stage are ensured to be feasible, which enables pruning of dominated solutions by GWTW.  

우리가 제안한 휴리스틱은 두 단계로 구성됩니다 - 전역 타이밍 복구(Global Timing Recovery, GTR) 및 실행 가능한 타이밍으로의 전력 감소(Power Reduction with Feasible Timing, PRFT). GTR은 먼저 위반 없는 (실행 가능한) 솔루션을 찾으며, 그런 다음 PRFT는 지역 탐색을 통해 크기 조정 솔루션의 전체 누설 전력을 반복적으로 감소시킵니다. 이는 그림 1에 설명된 대로 진행됩니다. 최적화 플로우의 각 단계에서 우리는 감도 함수 공간을 매개변수화하고, 이 공간을 독립적인 다중 시작을 통해 탐색하여 최적의 감도 구성을 찾습니다(그림 2). 각 다중 시작 후에는 모든 얻은 솔루션을 비교하고 가장 좋거나 비지배적인 솔루션을 유지합니다. 이는 go-with-the-winners (GWTW) 메타휴리스틱을 적용하여 수행됩니다 (2.4절). 그러나 우리의 최적화는 순전히 결정론적인데, 다중 시작 절차는 최상의 해를 유지하는 작은 집합으로 시작하며, GWTW는 일반적으로 랜덤화됩니다. 각 단계 이후의 솔루션은 실행 가능성이 보장되므로, GWTW에 의해 지배되는 솔루션을 제거할 수 있습니다.  

![2023-06-05-fig1.png]({{site.baseurl}}/assets/images/2023-06-05-fig1.png){: .align-center}  

Figure 1: During GTR(Section 3.1), our algorithm maintains the n best-seen timing-valid solutions by performing one stage of coarse-grain (global) search followed by two stages of fine-grain (local) search. Then, during PRFT (Section 3.2), we recover power while respecting timing constraints.  

Figure 1: GTR(Section 3.1) 중에 알고리즘은 두 단계의 세밀한 (지역적인) 탐색을 따르는 한 단계의 거친 (전역적인) 탐색을 수행하여 n개의 최상의 타이밍 유효한 솔루션을 유지합니다. 그런 다음, PRFT(Section 3.2) 중에는 타이밍 제약 조건을 준수하면서 전력을 회복합니다.  

### 3.1 Global Timing Recovery

This stage starts with an arbitrary cell configuration that is incrementally refined by increasing/decreasing gate sizes or downscaling/upscaling threshold voltages. The interaction of these steps in our implementation has been optimized for the case where the initial solution generally underestimates optimal power dissipation of individual gates and likely violates timing constraints. Therefore, we start with timing recovery by upsizing gates or downscaling threshold voltages. We observe that best-seen solutions for several ISPD 2012 benchmarks configure most cells at or close to their minimum-leakage configurations (Table 3), making minimum-leakage configurations for all cells an appropriate initial setting. For benchmarks with tighter timing constraints, alternative initial settings may save runtime. However, our optimization techniques are sufficiently fast and robust to start with minimum-leakage configurations. Empirically, finding a timing-feasible solution in the “coarse-search” stage of our optimization accounts only for a single-percent fraction of the total required runtime (Section 4.1).  

이 단계는 처음에 임의의 셀 구성으로 시작하여 게이트 크기를 증가/감소시키거나 임계 전압을 다운스케일링/업스케일링하여 점진적으로 개선합니다. 우리의 구현에서 이러한 단계들의 상호작용은 초기 솔루션이 일반적으로 개별 게이트의 최적 전력 소모를 과소평가하고 타이밍 제약 조건을 위반할 가능성이 높은 경우에 최적화되었습니다. 따라서, 우리는 게이트 크기를 증가시키거나 임계 전압을 다운스케일링하여 타이밍을 복구하기 시작합니다. 우리는 ISPD 2012 벤치마크의 대부분에 대해 최상의 해를 보여주는 솔루션들이 대부분 최소 누설 구성에 가깝거나 최소 누설 구성에서 설정되는 것을 관찰했습니다 (표 3). 따라서, 모든 셀에 대한 최소 누설 구성을 초기 설정으로 사용하는 것이 적절한 초기 설정입니다. 더 엄격한 타이밍 제약 조건을 가진 벤치마크의 경우, 대체 초기 설정은 실행 시간을 절약할 수 있습니다. 그러나 우리의 최적화 기술은 충분히 빠르고 견고하여 최소 누설 구성에서 시작할 수 있습니다. 경험적으로, 우리의 최적화의 "거친 탐색" 단계에서 타이밍 가능한 솔루션을 찾는 데 필요한 전체 실행 시간의 1% 정도만 차지합니다 (4.1절).  

![2023-06-05-fig2.png]({{site.baseurl}}/assets/images/2023-06-05-fig2.png){: .align-center}  

Figure 2: Search-range changes during the GTR search procedure (Algorithm 1). During coarse-grain search, the algorithm sweeps parameters α and γ. Then, fine-grain search local search focuses on ranges around the best-seen (α,γ) from coarse-grain search. Compared to each previous search stage, the ranges (as well as the step sizes) of both α and γ are reduced. After all search stages, GTR produces a set of the best-found solutions based on leakage power.  

Figure 2: GTR 검색 절차 (알고리즘 1) 중에 검색 범위 변경. 거친 탐색에서 알고리즘은 매개변수 α와 γ를 스윕합니다. 그런 다음 세밀한 탐색 로컬 탐색은 거친 탐색에서 최상의(α,γ) 주변 범위에 초점을 맞춥니다. 이전 검색 단계와 비교하여 α와 γ의 범위 (및 단계 크기)가 감소합니다. 모든 검색 단계 이후에 GTR은 누설 전력을 기반으로 최상의 발견된 솔루션 집합을 생성합니다.  

Starting with an underpowered configuration, we seek to generate feasible solutions by monotonically (i) increasing cell sizes or (ii) lowering cell threshold voltages (Vt ). Both cell upsizing and Vt downscaling are performed in smallest possible steps; the ordering of these actions is determined by their sensitivities, which are calculated by impacts on TNS and leakage power.  

$$\text{sensitivity}_{GTR} = \frac{\Delta TNS}{\Delta \text{leakage_power}^{\text{power_exponent}}}$$  

저전력 구성으로 시작하여, 우리는 (i) 셀 크기를 단조롭게 증가시키거나 (ii) 셀 임계 전압 (Vt)을 낮추어 실행 가능한 솔루션을 생성하려고 합니다. 셀 업사이징과 Vt 다운스케일링은 가능한 최소한의 단계로 수행됩니다. 이러한 동작의 순서는 TNS와 누설 전력에 대한 영향을 계산하여 결정되는 감도에 의해 결정됩니다.  

$$\text{sensitivity}_{GTR} = \frac{\Delta TNS}{\Delta \text{leakage_power}^{\text{power_exponent}}}$$  

When estimating the impact of a single cell modification, invoking STA can be computationally prohibitive. Therefore, we approximate the impact on TNS of a single cell modification($m_i^k$) on cell $c_i$ using $N Paths_i$, the number of negative-slack paths that pass through $c_i$. We define the nearest-neighbor set of $c_i$ to be the set of cells that have a driver (fanin) in common with $c_i$, and $N_i$ to be the union of $c_i$’s nearest-neighbor set and $c_i$ itself.  

$$\Delta TNS(m_i^k) \approx \sum_{c_j \in N_i} -\Delta delay_j^k \cdot \sqrt{N Paths_j}$$

단일 셀 수정의 영향을 추정할 때, STA를 호출하는 것은 계산적으로 제한적일 수 있습니다. 따라서, 셀 $c_i$에 대한 단일 셀 수정($m_i^k$)의 TNS에 대한 영향을 $c_i$를 통과하는 음수 여유 경로의 수인 $N Paths_i$를 사용하여 근사적으로 계산합니다. 우리는 $c_i$의 최근접 이웃 세트를 $c_i$와 동일한 드라이버(fanin)를 가지는 셀의 집합으로 정의하고, $N_i$를 $c_i$의 최근접 이웃 세트와 $c_i$ 자체의 합집합으로 정의합니다.  

왜 fanin을 공유하는 Cell들로 정의할까? $c_i$가 바뀌면 그에 따라 input pin capacitance가 변경되고, 이는 앞단 driver의 delay에 영향을 주기 때문인 것 같다.  
{: .notice--info}

$$\Delta TNS(m_i^k) \approx \sum_{c_j \in N_i} -\Delta delay_j^k \cdot \sqrt{N Paths_j}$$

Here, $\Delta delay_j^k$ is an estimated delay change on $c_j$ due to $m_i^k$. This approximation is based on the fact that any perturbation to cell $c_i$ will change the delay of $c_i$ , but also can impact the slacks of other cells that share path(s) with $c_i$, e.g., changing the $V_t$ of $c_i$ can change required arrival times (RATs) for its upstream cells, and change actual arrival times (AATs) for its downstream cells. To account for this, we introduce the factor $\sqrt{N Paths_j}$, which reflects the number of fanin and fanout cells of $c_j$ that are affected by the delay change of $c_j$. If this effect is not accounted for, particularly for cells that are on critical paths, the impact on TNS will be underestimated. To more accurately estimate the impact on TNS when we resize $c_i$, we must consider all cells in $N_i$, as changing the size will affect the capacitive load on the common driver, which affects their arrival (and transition) times. Following the empirical observation in [22], we assume that the propagation of increased transition time can be safely bounded to only the nearest neighbors.  

여기에서 $\Delta delay_j^k$는 $m_i^k$로 인한 $c_j$의 예상된 지연 변경입니다. 이 근사는 셀 $c_i$에 대한 어떠한 변동이 $c_i$의 지연을 변경할 뿐만 아니라 $c_i$와 경로를 공유하는 다른 셀들의 여유시간에도 영향을 줄 수 있다는 사실에 기반합니다. 예를 들어, $c_i$의 $V_t$를 변경하면 상위 셀들의 필요 도착 시간(RAT) 및 하위 셀들의 실제 도착 시간(AAT)이 변경될 수 있습니다. 이를 고려하기 위해, 우리는 $\sqrt{N Paths_j}$라는 요인을 도입합니다. 이 요인은 $c_j$의 fanin 및 fanout 셀 중 $c_j$의 지연 변경에 영향을 받는 셀의 수를 반영합니다. 이러한 영향을 고려하지 않을 경우, 특히 중요한 경로에 있는 셀들의 경우, TNS에 대한 영향이 과소평가될 수 있습니다. $c_i$의 크기를 변경할 때 TNS에 대한 영향을 더 정확하게 추정하기 위해, 우리는 $N_i$의 모든 셀을 고려해야 합니다. 크기를 변경함으로써 공통 드라이버의 용량 부하가 변경되어 도착 (및 전이) 시간에 영향을 줄 것입니다. [22]의 경험적인 관찰을 따르면, 증가한 전이 시간의 전파는 가장 가까운 이웃들에게만 안전하게 제한될 수 있다고 가정합니다.  

We sort the changes by non-increasing sensitivity values and commit them in order (Algorithm 1). Given that each single-cell modification is evaluated assuming other cells are fixed, the inaccuracies of sensitivity accumulate as multiple cells are changed. Therefore, we only commit the first γ% of the modifications between two consecutive STA invocations. The variables power_exponent $0 \le \alpha \le 3.0$ and commit_ratio $0 \lt \gamma \le 60%$ determine specific multistart configurations. To effectively reduce the size of the search space, we perform multilevel search (Figure 2). Fine-grain search is performed on non-dominated configurations from coarser search, but with finer steps and over smaller ranges.  

변화를 감소하는 감도 값의 비감소하는 순서로 정렬한 후 이들을 순서대로 적용하여 변화를 수행합니다 (알고리즘 1). 각 단일 셀 수정은 다른 셀이 고정되어 있다고 가정하고 평가되므로, 감도의 부정확성은 여러 셀이 변경될 때 누적됩니다.
따라서, 두 개의 연속적인 STA 호출 사이에서는 변경 사항의 처음 γ%만을 적용합니다. 변수 power_exponent에서 $0 \le \alpha \le 3.0$이고 commit_ratio에서 $0 \lt \gamma \le 60%$는 특정한 다중 시작 구성을 결정합니다. 탐색 공간의 크기를 효과적으로 줄이기 위해 다중 수준 탐색을 수행합니다 (그림 2). 세밀한 탐색은 더 거친 탐색에서 비지배적인 구성에 대해 수행되지만 더 작은 단계와 범위에서 이루어집니다.  

![2023-06-05-algo1.png]({{site.baseurl}}/assets/images/2023-06-05-algo1.png){: .align-center}  

### 3.2 Power Reduction with Feasible Timing

From the Global Timing Recovery (GTR) stage, we obtain a feasible sizing solution with no timing, slew or max-capacitance violations. However, the solution can improve further since some cells are oversized during the timing recovery stage. We reduce leakage power while maintaining timing feasibility by alternating (i) sensitivity-guided greedy sizing (SGGS), (ii) slack legalization, and (iii) speeding up bottleneck cells.  

전역 타이밍 복구(GTR) 단계에서는 타이밍, 슬루, 최대 용량 위반 없이 실행 가능한 크기 조정 솔루션을 얻습니다. 그러나 일부 셀은 타이밍 복구 단계에서 크기가 너무 커질 수 있으므로 솔루션이 더 개선될 수 있습니다. 우리는 (i) 감도에 따른 탐욕식 크기 조정(SGGS), (ii) 슬랙 합법화, (iii) 병목 셀 가속화를 번갈아 가며 타이밍 실행 가능성을 유지하면서 누설 전력을 감소시킵니다.  

**Sensitivity-guided greedy sizing (SGGS).** SGGS downsizes cells according to sensitivity while avoiding timing violations. Algorithm 2 presents pseudocode of our SGGS procedure. In this algorithm, $ISTA(c_i)$ is an incremental STA operation after cell $c_i$ is changed. In $ISTA(c_i)$, we start timing and slack updates at the fanin nodes of the changed cell. From the fanin nodes, transition times and AATs are propagated in the forward direction, and RATs are propagated in the backward direction.  

**감도에 따른 탐욕식 크기 조정(SGGS).** SGGS는 감도에 따라 셀을 다운사이징하면서 타이밍 위반을 피합니다. 알고리즘 2는 SGGS 절차의 의사코드를 제시합니다. 이 알고리즘에서 $ISTA(c_i)$는 셀 $c_i$가 변경된 후의 증분 STA 작업입니다. $ISTA(c_i)$에서는 변경된 셀의 팬인 노드에서 타이밍 및 슬랙 업데이트를 시작합니다. 팬인 노드에서 전이 시간과 AAT는 앞방향으로 전파되고, RAT는 역방향으로 전파됩니다.  

The SGGS algorithm starts with STA and initializes all timing nodes. Sensitivities are computed for all downsizable cells in Lines 3–14. We consider both gate downsizing and Vt upscaling for the sensitivity calculation. We define five sensitivities(Table 1).  

| Acronyms | Sensitivity functions |
|:--------:|:---------------------:|
| SF1      | $-\Delta \text{leakage_power} / \Delta delay$ |
| SF2      | $-\Delta \text{leakage_power} \times slack$ |
| SF3      | $-\Delta \text{leakage_power} / (\Delta delay \times \text{#}paths)$ |
| SF4      | $-\Delta \text{leakage_power} \times slack / \text{#}paths $ |
| SF5      | $-\Delta \text{leakage_power} \times slack / (\Delta delay \times \text{#}paths) $ |

Table 1: Sensitivity functions for SGGS. SF 4 and SF 5 appear most successful (Table 4), and our metaheuristic produces better results when using multiple functions. The slack and ∆delay values are expected to be positive.  

표 1: SGGS를 위한 감도 함수. SF 4와 SF 5가 가장 성공적으로 나타나며 (표 4), 여러 함수를 사용할 때 우리의 메타휴리스틱은 더 나은 결과를 도출합니다. 슬랙과 ∆delay 값은 양수일 것으로 예상됩니다.  

![2023-06-05-algo2.png]({{site.baseurl}}/assets/images/2023-06-05-algo2.png){: .align-center}  

∆leakage_power and ∆delay represent leakage power and cell delay changes after the downsizing of cell $c_i$ . The variable slack represents the slack at the output pin, and #paths is the number of timing paths that pass through the cell $c_i$. The slack value is positive since the downsizing is applied to cells with positive slack. #paths is calculated similarly to N Paths in Section 3.1, but including positive-slack paths. SF 1 and SF 2 have been used in [8] [12] and [11], respectively. We have added #paths into SF 3 and SF 4 to favor cells with smaller impact on the slacks of other cells. SF 5 is a hybrid of SF 1 and SF 2; similar logic is used in [29], but without considering #paths. In Lines 15–22, we select a cell $c_i$ with maximum sensitivity, and downsize ci or upscale its Vt . We perform incremental timing analysis and check for violations. If the sizing step creates a timing, slew or max-capacitance violation, it is undone. The loop continues until M becomes empty.  

∆leakage_power와 ∆delay는 셀 $c_i$의 다운사이징 이후의 누설 전력과 셀 딜레이 변경을 나타냅니다. 변수 slack은 출력 핀의 슬랙을 나타내고, #paths는 셀 $c_i$를 통과하는 타이밍 경로의 수입니다. 슬랙 값은 양수이며, 양수 슬랙을 가진 셀에 다운사이징을 적용하기 때문입니다. #paths는 3.1절의 N Paths와 유사하게 계산되지만, 양수 슬랙 경로를 포함합니다. SF 1과 SF 2는 각각 [8], [12] 및 [11]에서 사용되었습니다. SF 3과 SF 4에는 #paths가 추가되어 다른 셀의 슬랙에 미치는 영향이 작은 셀을 선호합니다. SF 5는 SF 1과 SF 2의 혼합입니다. 유사한 논리가 [29]에서 사용되었지만 #paths는 고려되지 않았습니다. 라인 15-22에서는 감도가 최대인 셀 $c_i$를 선택하고, $c_i$를 다운사이징하거나 Vt를 업스케일링합니다. 증분 타이밍 분석을 수행하고 위반을 확인합니다. 크기 조정 단계가 타이밍, 슬루 또는 최대 용량 위반을 생성하는 경우 취소됩니다. 루프는 M이 비어질 때까지 계속됩니다.  

**Slack legalization.** ISTA achieves a significant speedup over full-netlist STA through propagation of timing related to updated instances. We achieve further speedup by blocking the propagation when changes to timing are below a propagation_threshold. Due to this limited accuracy, SGGS can overlook a small number of timing violations. Instead of using GTR, we use a slack legalization procedure to rectify small timing violations at a small leakage power cost.  

**슬랙 합법화**. ISTA는 업데이트된 인스턴스와 관련된 타이밍 전파를 통해 전체 넷리스트 STA에 비해 상당한 속도 향상을 이룹니다. 우리는 propagation_threshold 이하의 타이밍 변경에 대한 전파를 차단하여 더 큰 속도 향상을 달성합니다. 이 제한된 정확도로 인해 SGGS는 일부 타이밍 위반을 무시할 수 있습니다. GTR 대신 슬랙 합법화 절차를 사용하여 작은 타이밍 위반을 작은 누설 전력 비용으로 수정합니다.  

In slack legalization, we first collect cells which are in critical (negative-slack) paths. These cells are sorted in decreasing order of $\vert \Delta delay \vert$ (delay improvement due to upsizing and Vt downscaling) and are modified in this order. Unlike GTR, slack legalization tracks slack changes after each cell modification, and ensures no timing degradation. Let ∆slack(c) be the slack change on output pin of cell c, and $C_{fi}(c)$ be set of fanin cells of cell c. After the modification of cell $c_i$, slack legalization restores the change if (i) $∆slack(c_i) ≤ 0$, or (ii) $∆slack(c_i) + \sum_{c_j \in C_{fi}(c_i)} \Delta slack(c_j) ≤ 0$. Slack legalization repeats the sizing until all timing violations are fixed.  

슬랙 합법화에서는 먼저 임계(음의 슬랙) 경로에 있는 셀들을 수집합니다. 이러한 셀들은 $\vert \Delta delay \vert$ (업사이징과 Vt 다운스케일링으로 인한 지연 개선)의 감소 순서대로 정렬되고, 이 순서로 수정됩니다. GTR과 달리 슬랙 합법화는 셀 수정 후 슬랙 변경을 추적하고 타이밍 저하가 없도록 보장합니다. 여기서 ∆slack(c)는 셀 c의 출력 핀의 슬랙 변화를 나타내고, $C_{fi}(c)$는 셀 c의 팬인 셀 집합을 나타냅니다. 셀 $c_i$의 수정 후, 슬랙 합법화는 (i) $∆slack(c_i) ≤ 0$, 또는 (ii) $∆slack(c_i) + \sum_{c_j \in C_{fi}(c_i)} \Delta slack(c_j) ≤ 0$일 경우 변경 사항을 되돌립니다. 슬랙 합법화는 모든 타이밍 위반을 수정할 때까지 크기 조정을 반복합니다.  

**Speeding up bottleneck cells.** During greedy sizing, we size gates monotonically downward with lower-size or higher-Vt library cells, but the resulting solution can be a local optimum. We observe that a key obstacle is that we have no timing slack available to allow further gate downsizing. Therefore, to recover timing slack with the least impact to power, we speed up bottleneck cells, i.e., cells that participate in many timing-critical paths. We identify hese cells by a bottleneck analysis similar to that provided in EDA tools [33]. We then perturb the converged solution by assigning larger sizes or lower Vt cells, and then repeat the downsizing procedure. To identify bottleneck cells, we estimate total slack changes by $\Delta delay \cdot \text{#} paths$ due to hypothetical cell upsizing (or Vt down-scaling). We commit the first γ% of such modifications with largest ∆total_slack, then optimize leakage power with SGGS. Timing violations created by speeding up bottleneck cells or SGGS are removed by slack legalization. To define specific multistart configurations, we sweep γ from 1% to 5% with a step size of 1%. The iterations (speeding up bottleneck cells + SGGS + slack legalization) terminate when the solutions stop improving. Figure 3 illustrates the progression of PRF T . Starting with a feasible solution from GTR, PRF T iteratively reduces leakage power while maintaining timing feasibility.  

**병목 셀 가속화**. 탐욕식 크기 조정 도중에는 낮은 크기나 높은 Vt 라이브러리 셀을 사용하여 게이트 크기를 단조롭게 줄입니다. 그러나 결과 솔루션이 국소 최적해가 될 수 있습니다. 우리는 더 이상의 게이트 다운사이징을 허용하기 위해 사용 가능한 타이밍 슬랙이 없다는 것이 주요 장애물인 것을 관찰하였습니다. 따라서 누설 전력에 최소한의 영향을 주면서 타이밍 슬랙을 복구하기 위해 병목 셀을 가속화합니다. 병목 셀은 많은 타이밍 중요 경로에 참여하는 셀로, EDA 도구 [33]에서 제공되는 병목 분석과 유사한 방법으로 식별합니다. 그런 다음 수렴한 솔루션을 크기가 더 큰 셀 또는 낮은 Vt 셀로 할당하여 변화시키고, 다운사이징 절차를 반복합니다. 병목 셀을 식별하기 위해 가상의 셀 업사이징 (또는 Vt 다운스케일링)으로 인한 $\Delta delay \cdot \text{#} paths$에 의한 총 슬랙 변화를 추정합니다. 가장 큰 ∆total_slack를 가진 γ%의 수정을 적용하고, 그 후 SGGS로 누설 전력을 최적화합니다. 병목 셀 가속화나 SGGS로 인해 발생하는 타이밍 위반은 슬랙 합법화에 의해 제거됩니다. 구체적인 다중시작 구성을 정의하기 위해 γ를 1%에서 5%까지 1% 단위로 변경합니다. PRFT의 반복(병목 셀 가속화 + SGGS + 슬랙 합법화)는 솔루션 개선이 멈출 때까지 진행됩니다. Figure 3은 PRFT의 진행 상황을 보여줍니다. GTR에서 실행 가능한 솔루션부터 시작하여 PRFT는 타이밍 실행 가능성을 유지하면서 반복적으로 누설 전력을 감소시킵니다.  

### 3.3 Handling Capacitance and Slew Violations

Each standard cell can drive a certain maximum capacitance load deﬁned in the library (e.g., based on the contest library, the maximum capacitance that can be driven by the smallest four-input NAND gate with high Vt is $3.2fF$). If a cell is overloaded, its output transition time slows down signiﬁcantly, resulting in overall degradation of slacks in its fanout cone. In our heuristic, we remove max-capacitance and slew violations at every iteration of GTR (Algorithm 1) by alternating backward- and forward-traversal repair as necessary. During backward traversal, we visit cells in a reverse topological order and continue to upsize driving cells until capacitance violations for the driving cells are removed or the driving cells reach their maximum sizes (whichever comes ﬁrst). This procedure resolves most of capacitance violations in the early iterations of GTR when cell sizes are relatively small. However, at later stages, cells on certain paths can be saturated at their maximum sizes,4 and we must downsize some of their fanout cells. Therefore, during forward traversal, we visit cells in a forward topological order and continue to downsize fanout cells until capacitance violations for the current cells are removed or all fanout cells shrink to their minimum sizes (whichever comes ﬁrst). Empirically, this requires one to two iterations of backward and forward traversals. As this happens, all output transitions become faster than the maximum slew allowed at the ISPD 2012 contest(300 ps).  

각 표준 셀은 라이브러리에서 정의된 최대용량을 구동할 수 있습니다(예: 콘테스트 라이브러리를 기준으로 가장 작은 입력 NAND 게이트의 최대용량은 고 Vt에서 3.2fF입니다). 셀이 과부하되면 출력 전이 시간이 크게 느려지고, 그 결과로 전체적으로 팬아웃 콘의 슬랙이 저하됩니다. 우리의 휴리스틱에서는 GTR의 각 반복에서 최대용량과 슬로우 위반을 교대로 역방향 및 순방향으로 수정하여 제거합니다(알고리즘 1). 역방향 탐색 중에는 역위상적인 순서로 셀을 방문하고, 구동 셀의 용량 위반을 제거하거나 구동 셀이 최대 크기에 도달할 때까지 구동 셀을 계속 업사이징합니다(먼저 발생하는 경우). 이 절차는 셀 크기가 비교적 작을 때 GTR 초기 반복에서 대부분의 용량 위반을 해결합니다. 그러나 후기 단계에서는 특정 경로의 셀이 최대 크기로 포화될 수 있으며, 이들의 팬아웃 셀을 다운사이징해야 합니다. 따라서 순방향 탐색 중에는 정방향 위상적인 순서로 셀을 방문하고, 현재 셀에 대한 용량 위반을 제거하거나 모든 팬아웃 셀이 최소 크기로 축소될 때까지 팬아웃 셀을 계속 다운사이징합니다(먼저 발생하는 경우). 실험적으로 이를 위해 역방향 및 순방향 탐색을 한 번 또는 두 번 진행해야 합니다. 이 과정에서 모든 출력 전이는 ISPD 2012 콘테스트에서 허용된 최대 슬로우(300 ps)보다 빠르게 발생합니다.  

## Appendix

추가로 알아본 내용들

### SPEF 포맷

SPEF는 "Standard Parasitic Exchange Format"의 약어로, 직역하면 "표준 잡음 교환 형식"입니다. SPEF는 반도체 설계와 레이아웃 도구 사이에서 사용되는 표준 데이터 형식입니다.  

반도체 설계에서 SPEF 파일은 전기적 특성을 설명하는 잡음 및 지연 정보를 포함합니다. 이 파일은 전기적 상호 연결을 나타내는 회로의 세부 정보와 해당 회로의 전압, 전류, 전송 시간 등과 같은 전기적 특성을 설명합니다. SPEF 파일은 주로 회로 시뮬레이션 및 타이밍 분석 도구에 사용되며, 이러한 도구는 회로의 성능과 신호 지연을 예측하고 최적화하는 데 사용됩니다.  

일반적으로 SPEF 파일은 반도체 설계 도구에서 자동으로 생성되며, 레이아웃 정보에서 파생됩니다. 이 파일은 대규모 반도체 설계에서 특히 중요하며, 회로의 전체적인 동작과 성능을 이해하고 분석하는 데 필수적입니다.  

### Liberty 파일의 `cell_rise`와 `rise_transition`

- `cell_rise` : Rise delay
- `rise_transition` : Slew

```text
cell_rise ("delay_outputslew_template_7X8") {
  index_1 ("0.00,3.00,6.00,12.00,24.00,48.00,96.00") ;  /* input slew */
  index_2 ("5.00,30.00,50.00,80.00,140.00,200.00,300.00,500.00") ;  /* output load */
  values (\ /* cell_rise values */
  "8.23, 14.43, 17.89, 21.74, 27.48, 32.10, 38.73, 49.94",\
  "13.44, 19.94, 24.75, 30.18, 38.07, 44.17, 52.51, 66.24",\
  "18.64, 25.14, 30.34, 37.12, 46.93, 54.40, 64.42, 80.31",\
  "29.06, 35.56, 40.76, 48.56, 61.69, 71.64, 84.77, 104.89",\
  "49.89, 56.39, 61.59, 69.39, 84.99, 99.07, 117.57, 145.40",\
  "91.56, 98.06, 103.26, 111.06, 126.66, 142.26, 167.92, 208.68",\
  "174.89, 181.39, 186.59, 194.39, 209.99, 225.59, 251.59, 303.59"\
  );
}
rise_transition ("delay_outputslew_template_7X8") {
  index_1 ("0.00,3.00,6.00,12.00,24.00,48.00,96.00") ;  /* input slew */
  index_2 ("5.00,30.00,50.00,80.00,140.00,200.00,300.00,500.00") ;  /* output load */
  values (\ /* rise_transition values */
  "8.31, 10.42, 13.02, 15.79, 19.39, 21.93, 25.44, 31.62",\
  "14.56, 15.47, 17.89, 21.94, 27.64, 31.62, 36.40, 43.73",\
  "20.81, 21.11, 22.84, 26.79, 34.17, 39.50, 45.99, 54.89",\
  "33.31, 33.31, 33.90, 36.56, 44.62, 52.13, 61.62, 74.67",\
  "58.31, 58.31, 58.31, 58.92, 63.99, 71.59, 85.38, 105.77",\
  "108.31, 108.31, 108.31, 108.31, 109.06, 112.95, 123.55, 151.24",\
  "208.31, 208.31, 208.31, 208.31, 208.31, 208.31, 211.23, 228.60"\
  );
}
```
