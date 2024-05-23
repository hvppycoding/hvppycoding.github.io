---
title: "OpenROAD Internals"
excerpt: "OpenROAD Internals"
date: 2024-05-22 01:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
---

## 1. Flow

OpenROAD Stage별 설명 자료: [Stage Tutorial](https://openroad-flow-scripts.readthedocs.io/en/latest/tutorials/FlowTutorial.html#understanding-and-analyzing-openroad-flow-stages-and-results)

OpenROAD의 Flow는 `flow/Makefile` 파일에 정의되어 있다.

순서는 다음과 같다.

* `do-yosys`
  * `tools/install/yosys/bin/yosys`
  * output: `1_synth.v`, `1_synth.sdc`
* `do-floorplan`
  1. `floorplan.tcl`: STEP 1: Translate verilog to odb, `2_1_floorplan`
  2. `io_place_random.tcl`: STEP 2: IO Placement (random), `2_2_floorplan_io`
  3. `tdms_place.tcl`: STEP 3: Timing Driven Mixed Sized Placement, `2_3_floorplan_tdms`
  4. `macro_place.tcl`: STEP 4: Macro Placement, `2_4_floorplan_macro`
    * `floorplan_debug_macros.tcl`: `2_floorplan_debug_macros`
  5. `tapcell.tcl`: STEP 5: Tapcell and Welltie insertion, `2_5_floorplan_tapcell`
  6. `pdn.tcl`: STEP 6: PDN generation, `2_6_floorplan_pdn`
* `do-place`
  1. `global_place_skip_io.tcl`: STEP 1: Global placement without placed IOs, timing-driven, and routability-driven, `3_1_place_gp_skip_io`
  2. `io_placement.tcl`: STEP 2: IO placement (non-random), `3_2_place_iop`
  3. `global_place.tcl`: STEP 3: Global placement with placed IOs, timing-driven, and routability-driven, `3_3_place_gp`
  4. `resize`: STEP 4: Resizing & Buffering, `3_4_place_resize`
* `do-cts`
  1. `cts.tcl`: Run TritonCTS, `4_1_cts`
* `do-route`
  1. `global_route.tcl`: STEP 1: Run global route, `5_1_grt`
  2. `fillcell.tcl`: STEP 2: Filler cell insertion, `5_2_fillcell`
  3. `detail_route.tcl`: STEP 3: Run detailed route, `5_3_route`
* `do-finish`
  1. `density_fill.tcl`: `6_1_fill`
  2. `final_report.tcl`: `6_report`, `6_final.def`, `6_final.v`, `6_final.spef`


## `do-yosys`

### ABC

ABC: System for Sequential Logic Synthesis and Formal Verification

Here is a [fork](https://github.com/yongshiwo/abc.git) of ABC containing Agdmap, a novel technology mapper for LUT-based FPGAs. Agdmap is based on a technology mapping algorithm with adaptive gate decomposition.

* L. Fan and C. Wu, ["FPGA technology mapping with adaptive gate decomposition"](https://dl.acm.org/doi/10.1145/3543622.3573048), ACM/SIGDA FPGA International Symposium on FPGAs, 2023.

### yosys

[yosys](https://github.com/The-OpenROAD-Project/yosys) – Yosys Open SYnthesis Suite

This is a framework for RTL synthesis tools. It currently has extensive Verilog-2005 support and provides a basic set of synthesis algorithms for various application domains.

Yosys can be adapted to perform any synthesis job by combining the existing passes (algorithms) using synthesis scripts and adding additional passes as needed by extending the yosys C++ code base.

* C. Wolf, J. Glaser. Yosys - A Free Verilog Synthesis Suite. In Proceedings of Austrochip 2013. [download pdf](https://yosyshq.net/yosys/files/yosys-austrochip2013.pdf)

## `do-floorplan`

### `floorplan.tcl`

```tcl
# init_floorplan
source "helpers.tcl"
read_lef Nangate45/Nangate45.lef
read_liberty Nangate45/Nangate45_typ.lib
read_verilog reg1.v
link_design top
initialize_floorplan -die_area "0 0 1000 1000" \
  -core_area "100 100 900 900" \
  -site FreePDK45_38x28_10R_NP_162NW_34O

set def_file [make_result_file init_floorplan1.def]
write_def $def_file
diff_files init_floorplan1.defok $def_file
```

## `do-place`

### Global Placement

ePlace: Electrostatics based Placement using Fast Fourier Transform and Nesterov’s Method([paper](https://cseweb.ucsd.edu/~jlu/papers/eplace-todaes14/paper.pdf)

논문 "ePlace: Electrostatics based Placement using Fast Fourier Transform and Nesterov’s Method"의 요약입니다:

이 논문은 VLSI 설계에서 비선형 글로벌 배치 문제를 해결하기 위해 새로운 알고리즘인 ePlace를 제안합니다. 주요 내용은 다음과 같습니다:

1. **도입 및 배경**:
   - VLSI 설계의 글로벌 배치 문제는 배치된 셀들의 밀도를 균일하게 유지하면서 배선 길이를 최소화하는 것을 목표로 합니다.
   - 기존의 비선형 배치 알고리즘들은 효율성과 품질 면에서 한계가 있습니다.

2. **ePlace 알고리즘 개요**:
   - ePlace는 전기역학 시스템을 모델로 사용하여 배치 문제를 해결합니다. 여기서 각 셀은 전하로 모델링되고 전기장의 힘에 의해 이동됩니다.
   - 전기 포텐셜과 필드 분포는 푸아송 방정식을 통해 계산되며, 이를 빠르고 정확하게 해결하기 위해 고속 푸리에 변환(FFT)을 사용합니다.

3. **Nesterov의 방법 사용**:
   - 비선형 배치 문제를 해결하기 위해 Nesterov의 최적화 방법을 사용합니다. 이 방법은 빠른 수렴 속도를 가지고 있으며, Lipschitz 상수를 동적으로 예측하여 스텝 길이를 결정합니다.
   - 실험 결과, Nesterov의 방법은 기존의 공액 경사법(CG)보다 더 짧은 배선 길이와 빠른 실행 시간을 보여주었습니다.

4. **실험 및 검증**:
   - ISPD 2005 및 2006 벤치마크를 사용한 실험에서 ePlace는 짧은 배선 길이와 높은 효율성을 입증했습니다.
   - ePlace는 기존의 선형 배치 알고리즘보다 우수한 성능을 보였으며, 다양한 설계 목표에 대해 확장 가능성이 높습니다.

5. **결론 및 미래 연구**:
  - ePlace는 전기역학적 접근 방식을 통해 배치 문제를 해결하는 새로운 방법을 제시하였으며, 향후 병렬 컴퓨팅 플랫폼에서의 성능 향상 가능성을 탐색할 것입니다.

이 논문은 VLSI 설계의 글로벌 배치 문제를 해결하기 위해 전기역학적 접근 방식을 도입하여 기존 방법들보다 효율적이고 품질이 우수한 솔루션을 제공합니다.

논문 "ePlace: Electrostatics based Placement using Fast Fourier Transform and Nesterov’s Method"의 전체적인 수행 플로우와 스도코드입니다.

### 수행 플로우

1. **초기화 단계**:
   - 설계된 회로의 초기 배치 정보를 불러오고, 각 셀을 전하로 모델링하여 초기 전하 밀도를 설정합니다.

2. **밀도 함수 및 전기장 계산**:
   - 푸아송 방정식을 사용하여 전기 포텐셜과 전기장을 계산합니다.
   - 밀도 분포와 전기장의 상호작용을 통해 셀의 이동 방향과 힘을 결정합니다.

3. **Nesterov의 방법을 사용한 최적화**:
   - Nesterov의 방법을 사용하여 현재 배치 상태에서의 최적화 과정을 진행합니다.
   - Lipschitz 상수를 동적으로 예측하여 스텝 길이를 결정합니다.

4. **밀도 균등화 및 배치 갱신**:
   - 전기장에 의해 계산된 힘을 사용하여 셀을 이동시켜 밀도를 균등화합니다.
   - 새로운 배치를 기반으로 밀도 함수를 갱신하고 반복합니다.

5. **종료 조건 확인**:
   - 배치 최적화가 수렴하거나 최대 반복 횟수에 도달하면 종료합니다.

6. **최종 배치 출력**:
   - 최적화된 배치 결과를 출력하고, 설계 목적에 맞는 배선 길이와 밀도 분포를 검토합니다.

### 스도코드

```pseudo
Algorithm ePlace

Input: Initial placement P, design constraints C
Output: Optimized placement P*

1. Initialize:
   - Load initial placement P
   - Initialize charge density for each cell

2. while (not converged) and (iteration < max_iterations) do
     a. Compute electric potential and field:
        - Solve Poisson's equation using FFT to get potential ψ
        - Compute electric field ξ from ψ using gradient descent
     
     b. Nesterov's method optimization:
        - Compute gradient ∇f(v)
        - Predict Lipschitz constant L
        - Update step length α = 1 / L
        - Update placement using Nesterov's method

     c. Update density and placement:
        - Move cells based on electric field ξ
        - Update density function ρ

     d. Check convergence criteria:
        - Evaluate if the placement has converged or max iterations reached

3. Output optimized placement P*

End Algorithm
```

### Nesterov의 방법을 사용한 최적화 단계의 스도코드

```pseudo
Algorithm Nesterov-Solver

Input: Initial solution u0, initial reference v0
Output: Optimized solution u*

1. Initialize:
   - Set initial parameters a0 = 1, k = 0
   - Set initial step length α0 = 1 / Lipschitz constant

2. while (not converged) and (iteration < max_iterations) do
     a. Compute gradient ∇f(vk)
     b. Compute step length αk using Lipschitz constant prediction
     c. Update solution:
        - uk+1 = vk - αk * ∇f(vk)
     d. Update parameter:
        - ak+1 = (1 + sqrt(1 + 4 * ak^2)) / 2
     e. Update reference solution:
        - vk+1 = uk+1 + (ak - 1) / ak+1 * (uk+1 - uk)
     f. Increment iteration k

3. Return optimized solution u*

End Algorithm
```

이 스도코드는 ePlace 알고리즘의 주요 단계를 요약한 것으로, 전기역학적 접근 방식과 Nesterov의 방법을 사용한 비선형 최적화 과정을 포함하고 있습니다.

### 내 분석

#### `doInitialPlace`
1. `placeInstsCenter()`: 모든 셀을 중앙으로 이동시킨다.
2. `while iter < maxIter:`
  - `updatePinInfo()`: Net별 max, min x, y 핀을 찾는다.
  - `createSparseMatrix()`: 
    - `Triplet`: 희소 행렬에서의 값이 있는 위치를 저장하는 자료구조로 (i, j, value) 형태로 저장된다.
  - `cpuSparseSolve(int maxSolverIter, int iter, SMatrix& placeInstForceMatrixX, Eigen::VectorXf& fixedInstForceVecX, Eigen::VectorXf& instLocVecX, SMatrix& placeInstForceMatrixY, Eigen::VectorXf& fixedInstForceVecY, Eigen::VectorXf& instLocVecY, utl::Logger* logger)`
    - `BiCGSTAB` 이용: IterativeLinearSolvers_Module

#### `doNestrobPlace();`



### GUI에서 결과보기

`make gui_final`을 실행하면 결과가 제대로 로드된 GUI가 나타난다.

[관련 Discussion](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/discussions/696)
