---
title: "[Review] Leakage-Power-Aware Clock Period Minimization"
excerpt: "DATE 2014"
date: 2023-06-04 18:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Review
mathjax: "true"
---

Hua-Hsin Yeh†, Shih-Hsu Huang†, and Yow-Tyng Nieh‡  

† Department of Electronic Engineering Chung Yuan Christian University Chung Li, Taiwan, R.O.C.  

‡ Information and Communications Research Labs Industrial Technology Research Institute Hsin Chu, Taiwan, R.O.C.  

## Abstract

In the design of nonzero clock skew circuits, an increase of the path delay may improve circuit speed and reduce leakage power. However, the impact of increasing path delay on the trade-off between circuit speed and leakage power has not been well studied. In this paper, we propose a two-step approach for leakage-power-aware clock period minimization. Compared with previous works, our approach has the following two significant contributions. First, our approach is the first leakage-power-aware clock skew scheduling that can guarantee working with the lower bound of the clock period. Second, our approach is also the first work that demonstrates the problem of minimizing the number of extra buffers is a polynomial-time problem. Benchmark data show that our approach achieves the best results in terms of the clock period and the leakage power.  

비제로 클럭 스쿠 회로의 설계에서는 경로 지연의 증가가 회로 속도를 향상시키고 누설 전력을 감소시킬 수 있습니다. 그러나 경로 지연의 증가가 회로 속도와 누설 전력 간의 트레이드오프에 미치는 영향은 잘 연구되지 않았습니다. 본 논문에서는 누설 전력을 고려한 클럭 주기 최소화를 위한 두 단계 접근 방식을 제안합니다. 이전의 연구와 비교했을 때, 우리의 접근 방식은 다음 두 가지 중요한 기여를 가지고 있습니다. 첫째, 우리의 접근 방식은 클럭 주기의 하한과 함께 작동을 보장할 수 있는 최초의 누설 전력을 고려한 클럭 스쿠 스케줄링입니다. 둘째, 우리의 접근 방식은 또한 추가 버퍼의 수를 최소화하는 문제가 다항 시간 문제임을 보여주는 최초의 연구입니다. 벤치마크 데이터는 우리의 접근 방식이 클럭 주기와 누설 전력 측면에서 최상의 결과를 달성한다고 보여줍니다.  

## Keywords

Clock Period Minimization, Leakage Power, Clock Skew Scheduling, Sequential Timing Optimization  

## I. INTRODUCTION

By properly scheduling the clock arrival times of registers, the clock period of a nonzero clock skew circuit can be shorter than the longest path delay [1,2]. However, hold constraints often limit the smallest feasible clock period that clock skew scheduling can achieve. In fact, hold violations can be resolved by increasing the path delay [3]. Therefore, in recent years, several research efforts [4-8] have been devoted to study the combination of clock skew scheduling and delay insertion for further clock period reduction. Especially, in [5-8], the lower bound of the clock period can be achieved. In addition to clock period minimization, low power is also a very important subject. Especially, due to technology scaling, leakage power has become a significant source of power consumption. It is known that gate downsizing (including higher threshold voltage assignment) is one of the most useful techniques for leakage power reduction. Thus, in [9-11], they use gate downsizing to reduce leakage power by slowing down non-critical paths. Moreover, in recent years, several leakage-power-aware clock skew scheduling approaches [12-14], which make use of clock skew scheduling to extend the timing slack of each logic gate for gate downsizing, have been proposed for further leakage power reduction.  

레지스터의 클럭 도착 시간을 적절하게 스케줄링함으로써 비제로 클럭 스쿠 회로의 클럭 주기를 가장 긴 경로 지연보다 짧게 할 수 있습니다 [1,2]. 그러나 홀드 제약 조건은 클럭 스케줄링이 달성할 수 있는 최소 클럭 주기를 제한하는 경우가 많습니다. 실제로 홀드 위반이 경로 지연을 증가시킴으로써 해결될 수 있습니다 [3]. 따라서 최근 몇 년간은 클럭 스케줄링과 지연 삽입의 조합을 연구하여 클럭 주기를 더욱 줄이기 위한 연구 노력 [4-8]이 기울여졌습니다. 특히 [5-8]에서는 클럭 주기의 하한을 달성할 수 있었습니다. 클럭 주기 최소화에 추가로 낮은 전력 소비도 매우 중요한 주제입니다. 특히 기술 축소로 인해 누설 전력은 중요한 전력 소비원이 되었습니다. 누설 전력 감소를 위해 게이트 축소 (고 임계전압 할당 포함)는 가장 유용한 기술 중 하나로 알려져 있습니다. 따라서 [9-11]에서는 비중요 경로를 늦추어 누설 전력을 감소시키기 위해 게이트 축소를 사용합니다. 또한, 최근 몇 년간 누설 전력을 감소시키기 위해 각 논리 게이트의 타이밍 여유를 확장하기 위해 클럭 스케줄링을 활용하는 누설 전력 고려 클럭 스케줄링 접근 방식 [12-14]도 제안되었습니다.  

From the above discussions, we know that, in the design of nonzero clock skew circuits, an increase of the path delay may improve circuit speed and reduce leakage power. However, to the best of our knowledge, the impact of increasing the path delay on the trade-off between circuit speed and leakage power has not been studied. With an analysis to previous works, we find that previous works can be classified into the following 
two independent groups.  

위의 논의로부터, 비제로 클럭 스쿠 회로의 설계에서는 경로 지연의 증가가 회로 속도를 향상시키고 누설 전력을 감소시킬 수 있음을 알 수 있습니다. 그러나 우리의 지식으로는 경로 지연의 증가가 회로 속도와 누설 전력 간의 트레이드오프에 미치는 영향이 연구되지 않았습니다. 이전 연구들을 분석해 본 결과, 이전 연구들은 다음 두 개의 독립적인 그룹으로 분류될 수 있습니다.  

(1) One group (i.e., previous works [5-8]) uses buffer insertion to implement delay insertion for resolving hold violations. This group can achieve the lower bound of the clock period that the combination of clock skew scheduling and delay insertion can achieve. However, in fact, many hold violations can be resolved by gate downsizing. Since this group only uses buffer insertion to resolve hold violation, this group often suffers from a large overhead on leakage power due to the insertion of extra buffers.  

(1) 한 그룹 (즉, 이전 연구 [5-8])은 홀드 위반 해결을 위해 딜레이 삽입을 구현하기 위해 버퍼 삽입을 사용합니다. 이 그룹은 클럭 스케줄링과 딜레이 삽입의 조합이 달성할 수 있는 클럭 주기의 하한을 달성할 수 있습니다. 그러나 실제로 많은 홀드 위반은 게이트 축소에 의해 해결될 수 있습니다. 이 그룹은 홀드 위반 해결을 위해 버퍼 삽입만을 사용하기 때문에 추가 버퍼의 삽입으로 인해 누설 전력에 대한 큰 오버헤드를 겪을 수 있습니다.  

(2) The other group (i.e., previous works [12-14]) applies gate downsizing (including higher threshold voltage assignment) to each logic gate as possible for leakage power reduction. However, since not all hold violations can be resolved by gate downsizing, this group often does not work with the lower bound of the clock period.  

(2) 다른 그룹 (즉, 이전 연구 [12-14])은 누설 전력 감소를 위해 가능한 한 모든 논리 게이트에 게이트 축소 (고 임계전압 할당 포함)를 적용합니다. 그러나 게이트 축소로는 모든 홀드 위반을 해결할 수 없기 때문에 이 그룹은 종종 클럭 주기의 하한과 함께 작동하지 않을 수 있습니다.  

Based on those observations, in this paper, we are motivated to study the combination of buffer insertion and gate downsizing during clock skew scheduling for achieving the lower bound of the clock period with the minimum leakage power. Our basic idea is to make use of gate downsizing as possible under the lower bound of the clock period. In other words, only for those hold violations that cannot be resolved by gate downsizing, we apply buffer insertion to them for achieving the lower bound of the clock period. As a result, we can derive a nonzero clock skew circuit that works with the lower bound of the clock period with the minimum leakage power.  

위의 관찰을 바탕으로 본 논문에서는 클럭 스케줄링 중 버퍼 삽입과 게이트 축소의 조합을 연구하여 최소 누설 전력으로 클럭 주기의 하한을 달성하기 위해 동기를 부여받았습니다. 우리의 기본 아이디어는 클럭 주기의 하한 아래에서 가능한 한 게이트 축소를 활용하는 것입니다. 다시 말해, 게이트 축소로 해결할 수 없는 홀드 위반에 대해서만 버퍼 삽입을 적용하여 클럭 주기의 하한을 달성합니다. 이 결과로, 최소 누설 전력으로 클럭 주기의 하한과 함께 작동하는 비제로 클럭 스쿠 회로를 도출할 수 있습니다.  

Our design methodology includes two steps. At the first step, we use a linear programming (LP) approach to minimize the number of required inserted buffers for achieving the lower bound of the clock period with giving a higher priority to gate downsizing. Next, at the second step, we use a LP approach [12] to apply gate downsizing to all logic gates in the circuit (including those extra buffers inserted at the first step) as possible for minimizing the leakage power. Compared with previous works, experimental results consistently show that our two-step design methodology achieves the best results in terms of circuit speed, leakage power, and total power consumption.  

우리의 설계 방법론은 두 단계로 구성됩니다. 첫 번째 단계에서는 선형 프로그래밍 (LP) 접근 방식을 사용하여 게이트 축소에 더 높은 우선순위를 부여하면서 클럭 주기의 하한을 달성하기 위해 필요한 추가 버퍼의 수를 최소화합니다. 다음으로, 두 번째 단계에서는 회로의 모든 논리 게이트에 가능한 한 게이트 축소를 적용하여 (첫 번째 단계에서 삽입된 추가 버퍼를 포함하여) 누설 전력을 최소화하기 위해 LP 접근 방식 [12]을 사용합니다. 실험 결과는 이전의 연구와 비교했을 때 우리의 두 단계 설계 방법론이 회로 속도, 누설 전력 및 총 전력 소비 측면에서 일관되게 최상의 결과를 달성한다고 보여줍니다.  

Our approach has the following two significant contributions:  

(1) To the best of our knowledge, our approach is the only leakage-power-aware clock skew scheduling that can guarantee working with the lower bound of the clock period.  

(2) Different from previous work [8] that uses a mixed integer linear programming (MILP) approach (i.e., NP-hard approach) to minimize the number of required inserted buffers, we use a LP approach (i.e., polynomial-time approach) to minimize the number of required inserted buffers. To the best of our knowledge, our work provides the first proof of showing that the minimization of the number of required inserted buffers can be solved in polynomial-time complexity. Moreover, it should be mentioned that previous work [8] does not consider gate downsizing. Therefore, in fact, we solve a more complex problem with a smaller time complexity.  

우리의 방법론은 다음과 같은 두 가지 중요한 기여를 가지고 있습니다:  

(1) 우리의 방법론은 우리의 지식으로는 클럭 주기의 하한과 함께 작동을 보장할 수 있는 유일한 누설 전력을 고려한 클럭 스케줄링입니다.  

(2) 이전 연구 [8]와는 달리, 필요한 추가 버퍼의 수를 최소화하기 위해 혼합 정수 선형 프로그래밍 (MILP) 접근 방식 (즉, NP-hard 접근 방식)을 사용하는 대신 선형 프로그래밍 (LP) 접근 방식 (즉, 다항 시간 접근 방식)을 사용합니다. 우리의 연구는 필요한 추가 버퍼의 수를 다항 시간 복잡도로 최소화하는 방법을 처음으로 증명한 것으로 알려져 있습니다. 또한, 이전 연구 [8]는 게이트 축소를 고려하지 않았음을 언급해야 합니다. 따라서 사실상 우리는 보다 복잡한 문제를 더 작은 시간 복잡도로 해결합니다.  

요약하면, 우리의 방법론은 누설 전력을 최소화하면서 클럭 주기의 하한을 보장할 수 있을 뿐만 아니라 필요한 추가 버퍼의 수를 최소화하기 위한 더 효율적인 방법을 도입합니다. 이러한 기여들은 누설 전력을 고려한 클럭 스케줄링 분야에서 중요한 진전으로 평가됩니다.  

## II. PRELIMINARIES

### A. Clocking Constraints

An edge-triggered circuit consists of registers and logic gates, with wires connecting them. A timing arc is defined as the signal propagation from a wire to the other wire through one logic gate. A data path $R_i → R_j$ is defined as the combinational logic from register $R_i$ to register $R_j$. A timing path is defined as the signal propagation from a register (called begin port) to another register (called end port). Note that a data path may consist of several timing paths.  

에지 트리거 회로는 레지스터와 논리 게이트로 구성되어 있으며, 이들을 연결하는 와이어가 있습니다. 타이밍 아크는 와이어에서 다른 와이어로 신호 전달을 의미하며, 이는 하나의 논리 게이트를 통해 이루어집니다. 데이터 경로 $R_i → R_j$는 레지스터 $R_i$에서 레지스터 $R_j$로의 조합 논리를 의미합니다. 타이밍 경로는 레지스터(시작 포트라고도 함)에서 다른 레지스터(종료 포트라고도 함)로의 신호 전파를 의미합니다. 데이터 경로는 여러 개의 타이밍 경로로 구성될 수 있음에 유의해야 합니다.  

Let the notation $T_{R_i}$ denote the designated clock arrival time of register $R_i$. Let the notation $D_{R_i→R_j(max)}$ and the notation $D_{R_i → R_j(min)}$ denote the maximum delay and the minimum delay of data path $R_i → R_j$, respectively. For each data path $R_i → R_j$, there are two types of clocking constraints: the setup constraint corresponds to $T_{R_i}-T_{R_j} ≤ P - D_{R_i→R_j(max)}$, where $P$ is the clock period, and the hold constraint corresponds to $T_{R_j} - T_{R_i} ≤ D_{R_i→R_j(min)}$. The minimum-period clock skew scheduling problem [1,2] is to find the smallest feasible clock period and the corresponding clock skew schedule (i.e., the clock arrival times of registers) that satisfies all the clocking constraints. Several graph-based algorithms [2] use the constraint graph $G(V,E)$ to solve the minimum-period clock skew scheduling problem in polynomial-time complexity. In a constraint graph, each vertex $R_i ∈ V$ represents a register. A special vertex, called the host, is used for the synchronization of primary inputs and primary outputs. Each directed edge $e ∈ E$ represents a clocking constraint. For each data path $R_i → R_j$, its setup constraint is modeled as an S-edge $e_s(R_j, R_i)$, which is from register $R_j$ to register $R_i$ and associated with a weight $P - D_{R_i→R_j(max)}$, and its hold constraint is modeled as an H-edge $e_h(R_i,R_j)$, which is from register $R_i$ to register $R_j$ and associated with a weight $D_{R_i→R_j(min)}$. A constraint graph works with clock period P if and only if it the summation of weights in each cycle is not negative when the clock period is $P$.  

$T_{R_i}$ 표기법은 레지스터 $R_i$의 지정된 클럭 도착 시간을 나타냅니다. $D_{R_i→R_j(max)}$와 $D_{R_i → R_j(min)}$ 표기법은 데이터 경로 $R_i → R_j$의 최대 지연과 최소 지연을 각각 나타냅니다. 각 데이터 경로 $R_i → R_j$에는 두 가지 유형의 클럭 제약 조건이 있습니다. 설정 제약 조건은 $T_{R_i}-T_{R_j} ≤ P - D_{R_i→R_j(max)}$에 해당하며, 여기서 $P$는 클럭 주기를 의미합니다. 유지 제약 조건은 $T_{R_j} - T_{R_i} ≤ D_{R_i→R_j(min)}$에 해당합니다. 최소 주기 클럭 스큐 스케줄링 문제 [1,2]는 모든 클럭 제약 조건을 만족하는 가장 작은 실행 가능한 클럭 주기와 해당 클럭 스케줄 (즉, 레지스터의 클럭 도착 시간)을 찾는 것입니다. 몇 가지 그래프 기반 알고리즘 [2]은 다항 시간 복잡도로 최소 주기 클럭 스큐 스케줄링 문제를 해결하기 위해 제약 그래프 $G(V,E)$를 사용합니다. 제약 그래프에서 각 정점 $R_i ∈ V$는 레지스터를 나타냅니다. 호스트라는 특수한 정점은 기본 입력과 기본 출력의 동기화에 사용됩니다. 각 유향 간선 $e ∈ E$은 클럭 제약 조건을 나타냅니다. 각 데이터 경로 $R_i → R_j$에 대해 설정 제약 조건은 S-간선 $e_s(R_j, R_i)$로 모델링되며, 이는 레지스터 $R_j$에서 $R_i$로 향하는 간선으로, 가중치는 $P - D_{R_i→R_j(max)}$입니다. 또한 유지 제약 조건은 H-간선 $e_h(R_i,R_j)$로 모델링되며, 이는 레지스터 $R_i$에서 $R_j$로 향하는 간선으로, 가중치는 $D_{R_i→R_j(min)}$입니다. 제약 그래프는 클럭 주기 $P$에서만 작동합니다. 즉, 클럭 주기가 $P$일 때 사이클 내 가중치의 합이 음수가 아니어야 합니다.  

Take circuit ex1 shown in Figure 1 as an example. In Figure 1, each logic gate is associated with a delay value. For example, the delay of logic gate A is 3. The delay of each register is assumed to be 0. Figure 2 gives the corresponding constraint graph. By applying the minimum-period clock skew scheduling, the smallest feasible clock period is 6 and a clock skew schedule in which $T_{Host}=0$, $T_{R1}=1$, $T_{R2}=2$, and $T_{R3}=3$ is obtained. Figure 3 gives the corresponding constraint graph when the clock period is 6. For the convenience of presentation, in Figure 3, we label $T_{Ri}$ for each vertex $R_i$.  

도표 1에 표시된 회로 ex1을 예로 들겠습니다. 도표 1에서 각 논리 게이트는 지연 값을 가지고 있습니다. 예를 들어, 논리 게이트 A의 지연은 3입니다. 각 레지스터의 지연은 0으로 가정합니다. 도표 2는 해당하는 제약 그래프를 보여줍니다. 최소 주기 클럭 스큐 스케줄링을 적용하여 가장 작은 실행 가능한 클럭 주기는 6이며, $T_{Host}=0$, $T_{R1}=1$, $T_{R2}=2$, $T_{R3}=3$인 클럭 스큐 스케줄이 얻어집니다. 도표 3은 클럭 주기가 6일 때 해당하는 제약 그래프를 보여줍니다. 표시의 편의를 위해, 도표 3에서는 각 정점 $R_i$에 대해 $T_{Ri}$를 레이블로 표기합니다.  

![2023-06-04-fig1.png]({{site.baseurl}}/assets/images/2023-06-04-fig1.png){: .align-center}  

![2023-06-04-fig23.png]({{site.baseurl}}/assets/images/2023-06-04-fig23.png){: .align-center}  

### B. The Lower Bound of The Clock Period

Setup constraints give a lower bound on the clock period obtained by clock skew scheduling [2]. We use the notation $P_{set}$ to denote this lower bound. Take circuit ex1 shown in Figure 1 as an example. With an analysis to Figure 2, we find that the cycle consisting of $e_s(Host,R_3)$, $e_s(R_3,R_2)$, $e_s(R_2,R_1)$, and $e_s(R_1,Host)$ determines this lower bound, and thus we have $P_{set}(ex1) = 5$. Due to the limitation of hold constraints, the minimum-period clock skew scheduling often does not achieve this lower bound. Using circuit ex1 as an example, the minimum-period clock skew scheduling is 6 instead of 5. With an analysis to Figure 3, we find that $e_s(R_3,R_2)$, $e_s(R_2,R_1)$, and $e_h(R_1,R_3)$ form a critical cycle. Thus, if we can insert delay to the data path $R_1 → R_3$, the clock period may be further reduced.  

설정 제약 조건은 클럭 스큐 스케줄링에 의해 얻어진 클럭 주기의 하한 값을 제공합니다 [2]. 이 하한 값을 표기하기 위해 $P_{set}$ 표기법을 사용합니다. 도표 1에 표시된 회로 ex1을 예로 들겠습니다. 도표 2를 분석하면 $e_s(Host,R_3)$, $e_s(R_3,R_2)$, $e_s(R_2,R_1)$ 및 $e_s(R_1,Host)$로 구성된 사이클이 이 하한 값을 결정한다는 것을 알 수 있습니다. 따라서 $P_{set}(ex1) = 5$입니다. 하지만 유지 제약 조건의 제한으로 인해 최소 주기 클럭 스큐 스케줄링은 종종 이 하한 값을 달성하지 못합니다. 도표 1의 회로 ex1을 예로 들면, 최소 주기 클럭 스큐 스케줄링은 5가 아닌 6입니다. 도표 3을 분석하면 $e_s(R_3,R_2)$, $e_s(R_2,R_1)$ 및 $e_h(R_1,R_3)$이 치명적인 사이클을 형성한다는 것을 알 수 있습니다. 따라서 데이터 경로 $R_1 → R_3$에 지연을 삽입한다면 클럭 주기를 더욱 줄일 수 있습니다.  

We say that the path delay difference of timing path p is $D(p)- d(p)$, where $D(p)$ and $d(p)$ are the maximum delay and the minimum delay of timing path p, respectively. From [5-7], if a clock skew schedule has satisfied all the setup constraints, the largest path delay difference among all the timing paths gives a lower bound of the clock period for inserting delays to resolve all the hold violations without affecting the circuit speed. We use the notation $P_{ins}$ to denote this lower bound. In circuit ex1, we have $P_{ins}(ex1) = 0$.  

타이밍 경로 p의 경로 지연 차이를 $D(p)-d(p)$라고 정의합니다. 여기서 $D(p)$와 $d(p)$는 각각 타이밍 경로 p의 최대 지연과 최소 지연을 의미합니다. [5-7]에 따르면, 클럭 스큐 스케줄이 모든 설정 제약 조건을 만족한다면, 모든 홀드 위반을 해결하기 위해 지연을 삽입하는 동시에 회로 속도에 영향을 미치지 않고 클럭 주기의 하한값을 결정하는 모든 타이밍 경로 중에서 가장 큰 경로 지연 차이가 제공됩니다. 이 하한 값을 표기하기 위해 $P_{ins}$ 표기법을 사용합니다. 회로 ex1에서는 $P_{ins}(ex1) = 0$입니다.  

Finally, we study the lower bound of the clock period for the combination of clock skew scheduling and delay insertion. We use the notation $P_{LB}$ to denote this lower bound. From [5-8], we know $P_{LB} = maximum(P_{set},P_{ins})$. In circuit ex1, we have $P_{LB}(ex1) = 5$.  

마지막으로, 클럭 스큐 스케줄링과 지연 삽입의 조합에 대한 클럭 주기의 하한 값을 연구합니다. 이 하한 값을 표기하기 위해 $P_{LB}$ 표기법을 사용합니다. [5-8]에서 알 수 있듯이, $P_{LB} = \max(P_{set}, P_{ins})$입니다. 회로 ex1에서는 $P_{LB}(ex1) = 5$입니다.  

### C. Leakage Power Minimization

The basic idea of leakage-power-aware clock skew scheduling [12-14] is to leverage on borrowed time for leakage power reduction. Thus, in [12-14], the problem of gate downsizing is regarded as distributing available timing slacks (for delay insertion) to individual logic gates. After that, the assigned timing slack (i.e., the inserted delay) is converted to power saving on each logic gate by identifying the most efficient gate size (including threshold voltage assignment) for that gate. We use the notation $C_A$ to denote the power-delay sensitivity of logic gate A (i.e., the power saving for each unit of the assigned timing slack of logic gate A). If the assigned timing slack of logic gate A is $δ_A$, then the power saving on logic gate A is $C_A×δ_A$. Thus, the objective of leakage-power-aware clock skew scheduling is to maximize $∑_{∀A ∈ GT} C_A×δ_A$, where the notation $GT$ denotes the set that includes all logic gates in the circuit. In [12], Ni et al. have used a LP approach to formally formulate the problem of leakage power minimization under clock skew scheduling.  

Leakage-power-aware clock skew scheduling [12-14]의 기본 아이디어는 누설 전력 감소를 위해 대여 시간을 활용하는 것입니다. 따라서 [12-14]에서 게이트 축소 문제는 가능한 지연 삽입을 위한 사용 가능한 타이밍 여유를 개별 논리 게이트에 분배하는 것으로 간주됩니다. 그 후, 할당된 타이밍 여유 (즉, 삽입된 지연)는 해당 게이트에 대해 가장 효율적인 게이트 크기 (스레시홀드 전압 할당 포함)를 식별함으로써 각 논리 게이트에서 전력 절약으로 변환됩니다. 논리 게이트 A의 전력-지연 감도 (power-delay sensitivity)를 나타내기 위해 $C_A$ 표기법을 사용합니다 (즉, 논리 게이트 A의 할당된 타이밍 여유에 대한 각 단위의 전력 절약). 논리 게이트 A의 할당된 타이밍 여유가 $δ_A$인 경우, 논리 게이트 A의 전력 절약은 $C_A×δ_A$입니다. 따라서 누설 전력-주의 클럭 스큐 스케줄링의 목표는 $∑_{∀A ∈ GT} C_A×δ_A$를 최대화하는 것입니다. 여기서 $GT$ 표기법은 회로에 있는 모든 논리 게이트를 포함하는 집합을 나타냅니다. [12]에서 Ni et al.은 LP 방법론을 사용하여 클럭 스큐 스케줄링에서의 누설 전력 최소화 문제를 형식적으로 정립하였습니다.  

## III. OBSERVATION AND MOTIVATION

### A. Drawback of Buffer Insertion

For working with the lower bound of the clock period, previous works [5-8] use buffer insertion to resolve hold violations. However, extra buffers cause a large overhead on leakage power. In fact, most hold violations can be resolved by gate downsizing. If we can resolve hold violations by gate downsizing, the leakage power even can be reduced.  

이전 연구들 [5-8]에서는 클럭 주기의 하한 값과 함께 작업하기 위해 홀드 위반을 해결하기 위해 버퍼 삽입을 사용했습니다. 그러나 추가적인 버퍼는 누설 전력에 큰 부담을 주게 됩니다. 사실, 대부분의 홀드 위반은 게이트 축소를 통해 해결될 수 있습니다. 게이트 축소를 통해 홀드 위반을 해결할 수 있다면, 누설 전력을 심지어 줄일 수도 있습니다.  

Let’s use circuit ex1 to demonstrate our observation. With an analysis to Figure 2, the summation of the weights in the cycle consisting of $e_s(R_3,R_2)$, $e_s(R_2,R_1)$, and $e_h(R_1,R_3)$ is $2P-12$. Thus, if the clock period is 5 (i.e., the lower bound of the clock period), to ensure the summation of weights in this cycle is not negative, the increase of the minimum delay of data path $R_1 → R_3$ should be at least 2 (in other words, $D_{R_1→R_3(min)}$ should be increased from 2 to at least 4).  

우리의 관찰을 설명하기 위해 회로 ex1을 사용해 보겠습니다. 도표 2를 분석하면 $e_s(R_3,R_2)$, $e_s(R_2,R_1)$ 및 $e_h(R_1,R_3)$으로 구성된 사이클 내 가중치의 합은 $2P-12$입니다. 따라서 클럭 주기가 5 (즉, 클럭 주기의 하한 값)인 경우 이 사이클 내 가중치의 합이 음수가 되지 않도록 하려면 데이터 경로 $R_1 → R_3$의 최소 지연 증가는 최소 2 (즉, $D_{R_1→R_3(min)}$을 2에서 최소 4로 증가)되어야 합니다.  

Previous works [5-7] focus on the minimization of required inserted delay, but they pay no attention to the minimization of the number of inserted buffers. Therefore, they [5-7] may obtain circuit ex1\* as shown in Figure 4 for working with the lower bound of the clock period. We use the notation $<R_1,C>$ to denote the wire from register $R_1$ to logic gate C. In circuit ex1\*, two extra buffers, whose delay values are 1, are inserted into the wire $<R_1,C>$ and the wire $<C,E>$, respectively. Note that these two extra buffers cause an overhead on leakage power.  

이전 연구들 [5-7]은 삽입된 지연의 최소화에 초점을 맞추었지만, 삽입된 버퍼의 수의 최소화에는 관심을 두지 않았습니다. 따라서 이전 연구들 [5-7]은 클럭 주기의 하한 값과 함께 작업하기 위해 도표 4에 표시된 회로 ex1\*을 얻을 수 있습니다. $<R_1,C>$를 사용하여 레지스터 $R_1$에서 논리 게이트 C로 향하는 와이어를 표기합니다. 회로 ex1\*에서는 지연 값이 1인 두 개의 추가 버퍼가 각각 $<R_1,C>$와 $<C,E>$의 와이어에 삽입됩니다. 이 두 개의 추가 버퍼는 누설 전력에 부담을 주는 것에 유의해야 합니다.  

![2023-06-04-fig4.png]({{site.baseurl}}/assets/images/2023-06-04-fig4.png){: .align-center}  


Previous work [8] uses a MILP approach to minimize the number of inserted buffers for working with the lower bound of the clock period. Thus, previous work [8] can obtain circuit ex1’ as shown in Figure 5, in which one extra buffer (whose delay value is 2) is inserted into the wire $<C,E>$. Although the number of required inserted buffer is minimized, this extra buffer still causes an overhead on leakage power.  

이전 연구 [8]는 클럭 주기의 하한 값과 함께 작업하기 위해 삽입된 버퍼의 수를 최소화하기 위해 MILP (Mixed Integer Linear Programming) 방법론을 사용합니다. 따라서 이전 연구 [8]는 도표 5에 표시된 회로 ex1'을 얻을 수 있습니다. 회로 ex1'에서는 지연 값이 2인 하나의 추가 버퍼가 $<C,E>$의 와이어에 삽입됩니다. 삽입된 버퍼의 수는 최소화되었지만, 이 추가 버퍼는 여전히 누설 전력에 부담을 주게 됩니다.  

![2023-06-04-fig5.png]({{site.baseurl}}/assets/images/2023-06-04-fig5.png){: .align-center}  

In fact, in this example, we need not to insert any extra buffer. We can use gate downsizing to increase the delay of logic gate C from 1 to 3. As a result, we not only resolve the hold violation but also reduce the leakage power.  

사실, 이 예제에서는 추가 버퍼를 삽입할 필요가 없습니다. 우리는 게이트 축소를 사용하여 논리 게이트 C의 지연을 1에서 3으로 증가시킬 수 있습니다. 결과적으로, 홀드 위반을 해결할 뿐만 아니라 누설 전력을 줄일 수도 있습니다.  

For further reducing the leakage power, the increase of the delay of logic gate C can be larger. Actually, even the delay of logic gate C is increased to be 8, all the clocking constraints are still satisfied. Figure 6 gives circuit ex1” that works with the lower bound of the clock period under the clock skew schedule in which $T_{Host}=0$, $T_{R1}=-2$, $T_{R2}=0$, and $T_{R3}=2$. In next section, we will present an approach to obtain circuit ex1”.  

![2023-06-04-fig6.png]({{site.baseurl}}/assets/images/2023-06-04-fig6.png){: .align-center}  

### B. Limitation of Gate Downsizing

It should be mentioned that not all hold violations can be resolved by gate downsizing. Since previous leakage-power-aware clock skew scheduling approaches [12-14] only use gate downsizing, they often does not work with the lower bound of the clock period. We use circuit ex2 shown in Figure 7 as an example to demonstrate our observation.  

각주 추가사항: 게이트 축소로는 모든 홀드 위반을 해결할 수 있는 것은 아닙니다. 이전의 누설 전력 감안 클럭 스큐 스케줄링 접근법 [12-14]은 게이트 축소만을 사용하기 때문에, 종종 클럭 주기의 하한 값과 함께 작동하지 않을 수 있습니다. 도표 7에 표시된 회로 ex2를 예로 들어 우리의 관찰을 설명하겠습니다.  

Figure 8 gives the corresponding constraint graph of circuit ex2. From Figure 8, we have $P_{set}(ex2) = 5$. Since $P_{ins}(ex2) = 0$, we have $P_{LB}(ex2) = P_{set}(ex2) = 5$. However, by applying the minimum-period clock skew scheduling, the smallest feasible clock period is 7 instead of 5. Figure 9 gives the corresponding constraint graph when the clock period is 7. With an analysis, we find that $e_h(R_1,R_3)$ and $e_s(R_3,R_1)$ form a critical cycle. To achieve the lower bound of the clock period, we should increase the minimum delay of data path $R_1 → R_3$.  

도표 8은 회로 ex2의 해당하는 제약 그래프를 보여줍니다. 도표 8에서는 $P_{set}(ex2) = 5$임을 알 수 있습니다. $P_{ins}(ex2) = 0$이므로 $P_{LB}(ex2) = P_{set}(ex2) = 5$입니다. 그러나 최소 주기 클럭 스큐 스케줄링을 적용하면 가장 작은 실행 가능한 클럭 주기는 5가 아닌 7이 됩니다. 도표 9는 클럭 주기가 7일 때 해당하는 제약 그래프를 보여줍니다. 분석을 통해, $e_h(R_1,R_3)$과 $e_s(R_3,R_1)$이 치명적인 사이클을 형성한다는 것을 알 수 있습니다. 클럭 주기의 하한 값을 달성하기 위해서는 데이터 경로 $R_1 → R_3$의 최소 지연을 증가시켜야 합니다.  

![2023-06-04-fig789.png]({{site.baseurl}}/assets/images/2023-06-04-fig789.png){: .align-center}  

In circuit ex2, the timing path $R_1 → C → R_3$ determines the minimum delay of data path $R_1 → R_3$. Note that this timing path has only one logic gate (i.e., logic gate C). With an analysis, we find that we cannot increase the delay of logic gate C for achieving the lower bound of the clock period. The reason is below. Logic gate C is also in the timing path $R_1 → B → C → R_3$ that determines the maximum delay of data path $R_1 → R_3$. Since data path $R_1 → R_3$ is critical to the setup constraint, we cannot increase the delay of logic gate C. Therefore, in this example, since previous leakage-power-aware clock skew scheduling approaches [12-14] only use gate downsizing, they cannot work with the lower bound of the clock period.  

회로 ex2에서 타이밍 경로 $R_1 → C → R_3$는 데이터 경로 $R_1 → R_3$의 최소 지연을 결정합니다. 이 타이밍 경로에는 논리 게이트 C 하나만이 포함되어 있음에 유의하세요. 분석을 통해, 우리는 클럭 주기의 하한 값을 달성하기 위해 논리 게이트 C의 지연을 증가시킬 수 없다는 것을 알 수 있습니다. 그 이유는 다음과 같습니다. 논리 게이트 C는 또한 데이터 경로 $R_1 → R_3$의 최대 지연을 결정하는 타이밍 경로 $R_1 → B → C → R_3$에도 있습니다. 데이터 경로 $R_1 → R_3$가 설정 제약 조건에 중요하므로, 논리 게이트 C의 지연을 증가시킬 수 없습니다. 따라서 이 예제에서는 이전의 누설 전력 감안 클럭 스큐 스케줄링 접근법 [12-14]이 게이트 축소만 사용하기 때문에 클럭 주기의 하한 값과 함께 작동할 수 없습니다.  

However, in this example, previous work [8] can obtain circuit ex2’ as shown in Figure 10 to work with the lower bound of the clock period. In ex2’, one extra buffer (whose delay value is 2) is inserted into the wire $<R1,C>$.  

그러나 이 예제에서 이전 연구 [8]은 회로 ex2'를 얻어 클럭 주기의 하한 값과 함께 작동할 수 있습니다. 도표 10에 표시된 대로 ex2'에서는 와이어 $<R1,C>$에 지연 값이 2인 하나의 추가 버퍼가 삽입됩니다.  

![2023-06-04-fig1011.png]({{site.baseurl}}/assets/images/2023-06-04-fig1011.png){: .align-center}  

For further reducing the leakage power, the delay of this extra buffer can be increased from 2 to 7 and the delay of logic gate A can be increased from 3 to 5. Figure 11 gives circuit ex2” that works with the lower bound of the clock period under the clock skew schedule in which $T_{Host}=0$, $T_{R1}=0$, $T_{R2}=0$, and $T_{R3}=3$. Suppose that the power-delay sensitivity of each logic gate is 1. Then, compared with circuit ex2’, the power saving of circuit ex2” achieves 7, i.e., $(7-2)×1+(5-3)×1=7$. In next section, we will present an approach to obtain circuit ex2”.  

### C. Our Motivation

From the discussions on circuit ex1 and circuit ex2, we have the following two observations.  

(1) Many hold violations can be resolved by gate downsizing. Since previous works [5-8] only use buffer insertion to resolve hold violations, they suffer from a large overhead on leakage power.  

(2)  Not all hold violations can be resolved by gate downsizing. Since previous works [12-14] only use gate downsizing, they often does not work with the lower bound of the clock period.  

Based on those observations, we are motivated to study the combination of buffer insertion and gate downsizing during clock skew scheduling.  

회로 ex1과 회로 ex2에 대한 논의를 통해 다음 두 가지 관찰 결과를 얻을 수 있습니다.  

(1) 게이트 축소를 통해 많은 홀드 위반을 해결할 수 있습니다. 이전 연구들 [5-8]은 홀드 위반 해결을 위해 버퍼 삽입만을 사용하기 때문에, 누설 전력에 큰 부담을 받습니다.  

(2) 모든 홀드 위반을 게이트 축소로 해결할 수 있는 것은 아닙니다. 이전 연구들 [12-14]은 게이트 축소만을 사용하기 때문에, 종종 클럭 주기의 하한 값과 함께 작동하지 않을 수 있습니다.  

이러한 관찰을 바탕으로, 우리는 클럭 스큐 스케줄링 중 버퍼 삽입과 게이트 축소의 조합을 연구하고자 하는 동기를 가지게 되었습니다.  

## IV. THE PROPOSED APPROACH

This section proposes a two-step methodology for leakage-power-aware clock period minimization. At the first step, we use a LP approach to minimize the number of required 
inserted buffers for achieving the lower bound of the clock period. Then, at the second step, we use a LP approach [12] to apply gate downsizing to all logic gates for leakage power minimization. Section IV-A addresses the first step. Section IV-B addresses the second step. Section IV-C gives examples.  

이 섹션에서는 누설 전력을 고려한 클럭 주기 최소화를 위한 두 단계 방법론을 제안합니다. 첫 번째 단계에서는 LP 방법론을 사용하여 클럭 주기의 하한 값 달성을 위해 필요한 삽입된 버퍼의 수를 최소화합니다. 그런 다음, 두 번째 단계에서는 누설 전력을 최소화하기 위해 모든 논리 게이트에 대해 게이트 축소를 적용하기 위해 LP 방법론 [12]을 사용합니다. IV-A 절에서는 첫 번째 단계를 다룹니다. IV-B 절에서는 두 번째 단계를 다룹니다. IV-C 절에서는 예시를 제시합니다.  

### A. Minimization of the Number of Buffers

Our basic idea is below: only for those hold violations that cannot be resolved by gate downsizing, we apply buffer insertion for achieving the lower bound of the clock period. Compared with previous work [8] that also attempts to reduce the number of required inserted buffers, our approach has the following two advantages. First, our approach considers gate downsizing, while previous work [8] does not. Therefore, the number of our required inserted buffers is much less than previous work [8]. Second, our approach is a LP approach, while previous work [8] is a MILP approach. Thus, our approach has a smaller time complexity.  

우리의 기본 아이디어는 다음과 같습니다: 게이트 축소로 해결할 수 없는 홀드 위반에 대해서만 클럭 주기의 하한 값 달성을 위해 버퍼 삽입을 적용합니다. 이전 연구 [8]과 마찬가지로 삽입된 버퍼의 수를 줄이기 위해 노력하지만, 우리의 접근 방식은 다음의 두 가지 이점을 가지고 있습니다. 첫째, 우리의 접근 방식은 게이트 축소를 고려합니다. 이에 반해 이전 연구 [8]은 고려하지 않습니다. 따라서 우리가 필요로 하는 삽입된 버퍼의 수는 이전 연구 [8]보다 훨씬 적습니다. 둘째, 우리의 접근 방식은 LP 방법론을 사용합니다. 반면 이전 연구 [8]은 MILP 방법론을 사용합니다. 따라서 우리의 접근 방식은 더 작은 시간 복잡도를 가지게 됩니다.  

We introduce the notations used in our LP approach. The real-value variable $T_{Ri}$ denotes the clock arrival time of register $R_i$. The real-value variables $E_A$ and $L_A$ denote the data earliest arrival time and the data latest arrival time, respectively. Since register $R_i$ may serve as both the begin port and the end port, in order to distinguish them, when register $R_i$ serves as the begin port, we use the notations $E_{Ri}$ and $L_{Ri}$; when register $R_i$ serves as the end port, we use the notations $E_{Ri’}$ and $L_{Ri’}$. The real-value variable $δ_A$ denotes the amount of increased delay of logic gate A (due to gate downsizing). The real-value variable $X_{A,B}$ denotes the amount of the delay inserted into the wire $<A,B>$ due to buffer insertion. The constant $D_A$ denotes the original delay of logic gate A. The constant $P_{LB}$ denotes the lower bound of the clock period. The set $W$ denotes a set that includes all the wires in the circuit. The constant INF denotes a very large constant value that approximates infinity, i.e., $INF → ∞$. Actually, in theory, with an analysis to our LP approach, we can derive the lower bound on the constant INF (for guaranteeing that the circuit obtained by our LP approach can work with the lower bound of the clock period) below: the multiplication of the constant INF and the constant $ρ$ should be greater than the number of wires in the circuit, where the constant $ρ$ should be less than or equal to the smallest required inserted delay among all delay insertions in the solutions of previous works [5-7]. Due to the limit on the number of pages, here we omit the formal proof.  

우리의 LP 방법론에서 사용하는 표기법을 소개합니다. 실수값 변수 $T_{Ri}$는 레지스터 $R_i$의 클럭 도착 시간을 나타냅니다. 실수값 변수 $E_A$와 $L_A$는 각각 데이터의 가장 이른 도착 시간과 가장 늦은 도착 시간을 나타냅니다. 레지스터 $R_i$가 시작 포트와 끝 포트 모두로 작동할 수 있으므로, 이를 구분하기 위해 레지스터 $R_i$가 시작 포트로 작동할 때는 $E_{Ri}$와 $L_{Ri}$로 표기하고, 레지스터 $R_i$가 끝 포트로 작동할 때는 $E_{Ri'}$와 $L_{Ri'}$로 표기합니다. 실수값 변수 $δ_A$는 게이트 축소로 인해 논리 게이트 A의 지연이 증가하는 양을 나타냅니다. 실수값 변수 $X_{A,B}$는 버퍼 삽입으로 인해 와이어 $<A,B>$에 삽입된 지연 양을 나타냅니다. 상수 $D_A$는 논리 게이트 A의 원래 지연을 나타냅니다. 상수 $P_{LB}$는 클럭 주기의 하한 값을 나타냅니다. 집합 $W$는 회로에 있는 모든 와이어를 포함하는 집합입니다. 상수 INF는 무한을 근사한 매우 큰 상수 값을 나타냅니다. 실제로 이론적으로, 우리의 LP 방법론을 분석하여 상수 INF의 하한 값을 유도할 수 있습니다 (우리의 LP 방법론으로 얻은 회로가 클럭 주기의 하한 값과 함께 작동할 수 있도록 보장하기 위한 것): 상수 INF와 상수 $ρ$의 곱은 회로 내의 와이어 수보다 커야 합니다. 여기서 상수 $ρ$는 이전 연구 [5-7]의 솔루션에서 필요한 지연 삽입 중 가장 작은 값보다 작거나 같아야 합니다. 페이지 수의 제한으로 인해, 여기서는 형식적인 증명을 생략합니다.  

To minimize the number of required inserted buffers, our basic idea is to let the cost of each buffer insertion be very tremendous. We define the real-value variable $Y_{A,B}$ to reflect the cost: if no buffer is inserted into the wire $<A,B>$, we let the value of variable $Y_{A,B}$ be only 1; on the other word, if a buffer is inserted into the wire $<A,B>$, we let the value of variable $Y_{A,B}$ be the multiplication of $X_{A,B}$ and $INF$ (note that, from the definition of the constant $ρ$, if the value of variable $X_{A,B}$ is nonzero, the value of variable $X_{A,B}$ is not less than $ρ$; thus, the value of variable $Y_{A,B}$, i.e., the multiplication of $X_{A,B}$ and $INF$, is greater than the number of wires in the circuit). Then, our objective function can become to minimize $\sum_{<A,B>∈W}Y_{A,B}$.  

삽입되어야 하는 버퍼의 수를 최소화하기 위해, 우리의 기본 아이디어는 각 버퍼 삽입의 비용을 매우 크게 설정하는 것입니다. 비용을 반영하기 위해 실수값 변수 $Y_{A,B}$를 정의합니다. 와이어 $<A,B>$에 버퍼가 삽입되지 않는 경우, 변수 $Y_{A,B}$의 값을 1로 설정합니다. 다시 말해, 와이어 $<A,B>$에 버퍼가 삽입되는 경우, 변수 $Y_{A,B}$의 값을 $X_{A,B}$와 $INF$의 곱으로 설정합니다 (상수 $ρ$의 정의에서 변수 $X_{A,B}$의 값이 0이 아니면 변수 $X_{A,B}$는 $ρ$ 이상임을 알 수 있습니다. 따라서 변수 $Y_{A,B}$, 즉 $X_{A,B}$와 $INF$의 곱은 회로 내의 와이어 수보다 큽니다). 이렇게 하면, 우리의 목적 함수는 $\sum_{<A,B>∈W}Y_{A,B}$를 최소화하는 것이 됩니다.  

The constraints in our LP approach are elaborated below. Since the amount of delay increase should be non-negative, for each wire $<A,B>$, we have the constraint: $X_{A,B} ≥ 0$, and for each logic gate A, we have the constraint: $δ_A ≥ 0$.  

우리의 LP 방법론에서의 제약 조건은 다음과 같이 설명됩니다. 지연 증가량은 양수이어야 하므로, 각 와이어 $<A,B>$에 대해 다음 제약 조건을 가지게 됩니다: $X_{A,B} ≥ 0$. 또한, 각 논리 게이트 A에 대해 다음 제약 조건을 가지게 됩니다: $δ_A ≥ 0$.  

For each timing arc from logic gate A to logic gate B, since the latest data arrival time of logic gate B should not be less than the summation of the latest data arrival time of logic gate A, the original delay of logic gate A, the amount of the increased delay of logic gate A, and the amount of the delay inserted into the wire $<A,B>$, we have the constraint: $L_B ≥ L_A + D_A + X_{A,B} + δ_A$.  

논리 게이트 A에서 논리 게이트 B로의 각 타이밍 아크에 대해, 논리 게이트 B의 최신 데이터 도착 시간은 논리 게이트 A의 최신 데이터 도착 시간, 논리 게이트 A의 원래 지연, 논리 게이트 A의 지연 증가량, 그리고 와이어 $<A,B>$에 삽입된 지연량의 합보다 작지 않아야 합니다. 따라서 다음 제약 조건을 가지게 됩니다: $L_B ≥ L_A + D_A + X_{A,B} + δ_A$.  

since the earliest data arrival time of logic gate B should not be greater than the summation of the earliest data arrival time of logic gate A, the original delay of logic gate A, the amount of the increased delay of logic gate A, and the amount of the delay inserted into the wire $<A,B>$, we have the constraint: $E_B ≤ E_A + D_A + X_{A,B} + δ_A$.  

논리 게이트 A에서 논리 게이트 B로의 각 타이밍 아크에 대해, 논리 게이트 B의 가장 이른 데이터 도착 시간은 논리 게이트 A의 가장 이른 데이터 도착 시간, 논리 게이트 A의 원래 지연, 논리 게이트 A의 지연 증가량, 그리고 와이어 $<A,B>$에 삽입된 지연량의 합보다 크지 않아야 합니다. 따라서 다음 제약 조건을 가지게 됩니다: $E_B ≤ E_A + D_A + X_{A,B} + δ_A$.  

For each register $R_i$ that serves as the begin port of any timing path, the data arrival time of register $R_i$ should be the clock arrival time of register $R_i$; thus, we have the constraints: $L_{Ri} = T_{Ri}$ and $E_{Ri} = T_{Ri}$.  

어떤 타이밍 경로의 시작 포트로 작동하는 각 레지스터 $R_i$에 대해, 레지스터 $R_i$의 데이터 도착 시간은 레지스터 $R_i$의 클럭 도착 시간과 같아야 합니다. 따라서 다음 제약 조건을 가지게 됩니다: $L_{Ri} = T_{Ri}$ 및 $E_{Ri} = T_{Ri}$.  

For each register $R_i$ that serves as the end port of any timing path, since the latest data arrival time of register $R_i$ should not be later than the next clock arrival time of register $R_i$, we have the constraint: $L_{Ri’} ≤ P_{LB} + T_{Ri}$. since the earliest data arrival time of register Ri should not be earlier than the clock arrival time of register $R_i$, we have the constraint: $E_{Ri’} ≥ T_{Ri}$.  

어떤 타이밍 경로의 끝 포트로 작동하는 각 레지스터 $R_i$에 대해, 레지스터 $R_i$의 최신 데이터 도착 시간은 다음 클럭 도착 시간보다 나중이어서는 안 됩니다. 따라서 다음 제약 조건을 가지게 됩니다: $L_{Ri’} ≤ P_{LB} + T_{Ri}$. 또한, 레지스터 $Ri$의 가장 이른 데이터 도착 시간은 레지스터 $R_i$의 클럭 도착 시간보다 빠르지 않아야 합니다. 따라서 다음 제약 조건을 가지게 됩니다: $E_{Ri’} ≥ T_{Ri}$.  

According to the definition of variable $Y_{A,B}$, we have the constraints: $Y_{A,B} ≥ 1$ and $Y_{A,B} ≥  X_{A,B} × INF$.  

변수 $Y_{A,B}$의 정의에 따라 다음 제약 조건을 가지게 됩니다: $Y_{A,B} ≥ 1$ 및 $Y_{A,B} ≥ X_{A,B} × INF$.  

### B. Leakage Power Minimization

At the second step, we use a LP approach to increase the delays of logic gates as possible for leakage power minimization. In fact, the framework of our LP approach is the same as that of previous work [12]. The only difference is that we use the circuit obtained at the first step as the input circuit (i.e., those extra buffers inserted at the first step are included). We use the constant CA to denote the power-delay sensitivity of logic gate A. Then, our objective function is to maximize $\sum_{∀A∈GT} C_A × δ_A$, and the constraints in our LP approach are the same as previous work [12].  

두 번째 단계에서는 누설 전력을 최소화하기 위해 가능한 한 논리 게이트의 지연을 증가시키기 위해 LP 방법론을 사용합니다. 사실, 우리의 LP 방법론의 구조는 이전 연구 [12]와 동일합니다. 유일한 차이점은 첫 번째 단계에서 얻은 회로를 입력 회로로 사용한다는 것입니다 (즉, 첫 번째 단계에서 삽입된 추가 버퍼가 포함됨). 논리 게이트 A의 전력-지연 민감도를 나타내기 위해 상수 CA를 사용합니다. 그런 다음, 목적 함수는 $\sum_{∀A∈GT} C_A × δ_A$를 최대화하는 것이고, 우리의 LP 방법론에서의 제약 조건은 이전 연구 [12]와 동일합니다.  

### C. Examples

