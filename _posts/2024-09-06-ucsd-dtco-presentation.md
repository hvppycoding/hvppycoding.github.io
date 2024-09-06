---
title: "UCSD DTCO Presentation"
excerpt: ""
date: 2024-09-06 01:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

1. Hello everyone. My name is Minsoo Kim, PhD student at UC San Diego.   
   안녕하세요 여러분. 저는 UC 샌디에이고에서 박사과정을 하고 있는 김민수입니다.

2. Today, the title of my presentation is “A Novel Framework for DTCO: Fast and Automatic Routability Assessment with Machine Learning for Sub-3nm Technology Options”.   
   오늘 제가 발표할 제목은 "새로운 DTCO 프레임워크: 3nm 이하 기술 옵션을 위한 기계 학습 기반의 빠르고 자동화된 배선 가능성 평가"입니다.

3. This work is in collaboration with Qualcomm.   
   이 연구는 Qualcomm과 협력하여 진행되었습니다.

4. This is an outline of my presentation. First, I will start with the background of our work and then explain what PROBE2.0 framework is.   
   이것이 제 발표의 개요입니다. 먼저, 우리의 연구 배경에 대해 설명한 후, PROBE2.0 프레임워크가 무엇인지 설명하겠습니다.

5. Then, I will explain our study with sub-3nm technology configurations. And, I will conclude my talk.   
   그 후 3nm 이하 기술 구성에 대한 연구를 설명하고, 발표를 마치겠습니다.

6. Let me start with the background.   
   배경 설명부터 시작하겠습니다.

7. [Click] Design Technology Co-Optimization (also known as DTCO) is an essential process for technology development at advanced nodes.   
   [클릭] 설계 기술 최적화(DTCO)는 첨단 노드에서 기술 개발에 필수적인 과정입니다.

8. However, the process requires large turnaround time and engineering efforts for design feedback, as shown in the figure below.   
   그러나 이 과정은 아래 그림에서 보시다시피, 설계 피드백을 위해 긴 처리 시간과 많은 엔지니어링 노력이 필요합니다.

9. [Click] Also, scaling boosters, such as buried power rails, backside PDN, etc., are used at advanced technology nodes for the best PPA.   
   [클릭] 또한 매립 전원 레일, 후면 PDN 등과 같은 스케일링 부스터는 최상의 PPA를 위해 첨단 기술 노드에서 사용됩니다.

10. Due to the routability impact by standard-cell architectures with scaling boosters, small standard-cell area does not always bring small block-level area.   
    스케일링 부스터가 적용된 표준 셀 아키텍처로 인한 배선 가능성 문제로, 작은 표준 셀 영역이 항상 작은 블록 수준의 영역을 의미하지는 않습니다.

11. [Click] Therefore, block-level area needs to be estimated at an early stage of technology development.   
    [클릭] 따라서 기술 개발 초기 단계에서 블록 수준의 영역을 추정해야 합니다.

12. Now I will explain about PROBE2.0, which is the baseline framework for our study.   
    이제 PROBE2.0에 대해 설명하겠습니다. 이는 우리의 연구에 기본이 되는 프레임워크입니다.

13. [Click] As I mentioned earlier, we use PROBE2.0 framework to evaluate routability for sub-3nm technology configurations.   
    [클릭] 앞서 말씀드린 것처럼, 우리는 3nm 이하 기술 구성을 평가하기 위해 PROBE2.0 프레임워크를 사용합니다.

14. PROBE2.0 framework is a fast and systematic framework for routability assessment.   
    PROBE2.0 프레임워크는 배선 가능성 평가를 위한 빠르고 체계적인 프레임워크입니다.

15. [Click] This includes satisfiability modulo theories (SMT)-based automatic standard-cell layout generation.   
    [클릭] 여기에는 SMT(만족 가능성 모듈로 이론) 기반의 자동 표준 셀 레이아웃 생성이 포함됩니다.

16. [Click] PROBE2.0 routability evaluations are based on rank-ordering with K threshold. I will explain what Kth is in the later slide.   
    [클릭] PROBE2.0 배선 가능성 평가는 K 임계값에 따른 순위 정렬을 기반으로 합니다. Kth에 대한 설명은 나중에 설명드리겠습니다.

17. [Click] Also, we utilize machine learning to expedite the assessment. Here, with the trained models, we don’t need to generate all the design enablements and perform large numbers of place-and-route runs. I will also give more detailed information in the later slide.   
    [클릭] 또한 우리는 기계 학습을 활용하여 평가를 가속화합니다. 여기서, 학습된 모델을 통해 모든 설계 기능을 생성하고 다수의 배치 및 배선 실행을 수행할 필요가 없습니다. 자세한 내용은 나중에 설명드리겠습니다.

18. [Click] PROBE2.0 includes SMT-based “automatic” standard-cell layout generation.   
    [클릭] PROBE2.0는 SMT 기반의 "자동" 표준 셀 레이아웃 생성을 포함합니다.

19. [Click] Given technology and design configurations, the SMT-based standard cell layout generation generates complete and optimal solutions with simultaneous placement and routing in standard cells.   
    [클릭] 기술 및 설계 구성이 주어지면, SMT 기반 표준 셀 레이아웃 생성은 표준 셀 내에서 동시에 배치 및 배선을 수행하여 완전하고 최적의 솔루션을 생성합니다.

20. It supports conditional design rules which are fully grid-based rules.   
    이 시스템은 완전한 그리드 기반의 조건부 설계 규칙을 지원합니다.

21. [Click] In case that you are interested, our code has been open-sourced at the GitHub repository here.   
    [클릭] 관심이 있으시면, 저희 코드는 GitHub 저장소에 오픈 소스로 공개되어 있습니다.

22. [Click] Also, we build the LEF generator which generates LEF format of layouts. We convert the layouts from SMT-based generator to LEF format while we add power/ground pin shapes in LEF.   
    [클릭] 또한 우리는 LEF 레이아웃 포맷을 생성하는 LEF 생성기를 구축했습니다. SMT 기반 생성기에서 생성된 레이아웃을 LEF 형식으로 변환할 때 전원/접지 핀 모양을 추가합니다.

23. There are two previous works, PROBE1.0 and PROBE2.0, and both are based on the idea of placement tangling.   
    PROBE1.0과 PROBE2.0 두 가지 이전 연구가 있으며, 두 연구 모두 배치 얽힘(placement tangling)의 개념을 기반으로 하고 있습니다.

24. [Click] Placement tangling means repeatedly making neighbor-swaps.   
    [클릭] 배치 얽힘은 이웃 셀을 반복적으로 교환하는 것을 의미합니다.

25. [Click] The basic idea of PROBE is to randomly perform a neighbor-swap operation to increase the routing difficulty.   
    [클릭] PROBE의 기본 아이디어는 배선의 난이도를 높이기 위해 이웃 교환 작업을 무작위로 수행하는 것입니다.

26. [Click] Here we show an example of neighbor swaps for better understanding. For the cell to be swapped, we randomly select one of the neighboring cells.   
    [클릭] 더 나은 이해를 돕기 위해 이웃 교환의 예를 보여드리겠습니다. 교환할 셀은 이웃 셀 중 하나를 무작위로 선택합니다.

27. [Click] And we swap the cells, we have more complex connectivity after the swap.   
    [클릭] 셀을 교환하면, 교환 후 더 복잡한 연결 구조가 형성됩니다.

28. [Click] Here, we use ”K” and the number of swaps is K times the total number of instances in design.   
    [클릭] 여기서 우리는 "K"를 사용하며, 교환 횟수는 설계에서 총 인스턴스 수의 K배입니다.

29. K threshold is our metric for routability. K threshold is defined as the minimum K which leads to an unroutable placement.   
    K 임계값은 배선 가능성을 평가하는 우리의 기준입니다. K 임계값은 배치가 불가능해지는 최소 K값으로 정의됩니다.

30. [Click] Based on the definition of Kth, higher Kth means better intrinsic routability.   
    [클릭] Kth의 정의에 따라, 더 높은 Kth는 더 나은 고유 배선 가능성을 의미합니다.

31. [Click] So, PROBE2.0 routability evaluation is based on rank ordering with K threshold.   
    [클릭] 따라서 PROBE2.0의 배선 가능성 평가는 K 임계값에 따른 순위 정렬에 기반합니다.

32. This slide explains the machine learning-based Kth prediction.   
    이 슬라이드는 기계 학습 기반 Kth 예측에 대해 설명합니다.

33. [Click] PROBE2.0 had three disadvantages. To obtain Kth, we need to launch a number of P&R runs, and it requires large runtime, storage and number of licenses.   
    [클릭] PROBE2.0에는 세 가지 단점이 있었습니다. Kth를 얻기 위해 여러 번의 P&R 실행을 해야 하며, 이는 긴 실행 시간, 대용량 저장 공간 및 다수의 라이선스를 필요로 합니다.

34. [Click] This machine learning-based Kth prediction helps us to mitigate the drawbacks.   
    [클릭] 이 기계 학습 기반 Kth 예측은 이러한 단점을 완화하는 데 도움이 됩니다.

35. [Click] The solution is that, given technology and design parameters, we generate Kth for partial configurations for training and predict Kth for the remaining configurations. In this way, we can significantly reduce the number of the required P&R runs and design enablement.   
    [클릭] 해결책은 기술과 설계 파라미터가 주어졌을 때, 일부 구성에서 Kth를 생성하여 학습에 사용하고, 나머지 구성에 대해서는 Kth를 예측하는 것입니다. 이렇게 하면 필요한 P&R 실행 횟수와 설계 지원을 크게 줄일 수 있습니다.

36. [Click] For example, we run by default 30 P&R runs to obtain a single Kth, which takes a few hours of runtime. However, Kth prediction takes a few seconds.   
    [클릭] 예를 들어, 기본적으로 단일 Kth를 얻기 위해 30번의 P&R 실행을 수행하며, 이는 몇 시간의 실행 시간이 걸립니다. 그러나 Kth 예측은 몇 초 만에 완료됩니다.

37. [Click] AutoML is used as a platform of the ML work. The benefit of AutoML is that AutoML can recommend best ML models for our problem.   
    [클릭] AutoML은 기계 학습 작업의 플랫폼으로 사용됩니다. AutoML의 장점은 우리 문제에 가장 적합한 기계 학습 모델을 추천할 수 있다는 것입니다.

38. Now, I will explain our study for sub-3nm technology options.   
    이제, 3nm 이하 기술 옵션에 대한 우리의 연구를 설명하겠습니다.

39. [Click] As a part of DTCO process, we perform an early-stage routability assessment using PROBE2.0, with sub-3nm configurations.   
    [클릭] DTCO 프로세스의 일환으로, 우리는 PROBE2.0을 사용하여 3nm 이하 구성에서 초기 배선 가능성 평가를 수행합니다.

40. [Click] To remind, the PROBE2.0 framework is to evaluate routability and we can assess routability with extended configurations.   
    [클릭] 다시 말하지만, PROBE2.0 프레임워크는 배선 가능성을 평가하기 위한 것이며, 확장된 구성으로 배선 가능성을 평가할 수 있습니다.

41. [Click] In this work, we use the following configurations for our study, which reflect to sub-3nm technology options.   
    [클릭] 이번 연구에서는 3nm 이하 기술 옵션을 반영한 다음과 같은 구성을 사용했습니다.

42. [Click] We generate standard-cell architectures with 80 to 150nm cell height.   
    [클릭] 우리는 80nm에서 150nm까지의 셀 높이를 가진 표준 셀 아키텍처를 생성했습니다.

43. [Click] We also consider 39 to 57nm contacted poly pitch and [Click] 16 to 30nm metal pitch.   
    [클릭] 또한, 39nm에서 57nm까지의 컨택트 폴리 피치와 [클릭] 16nm에서 30nm까지의 메탈 피치를 고려했습니다.

44. [Click] For power delivery network, we have an option whether we use buried power rails or not.   
    [클릭] 전력 공급 네트워크에 대해서는 매립 전원 레일을 사용할지 여부에 대한 옵션이 있습니다.

45. [Click] To study sub-3nm technology configurations, we first generate nine standard-cell libraries based on the following aspects.  
    [클릭] 3nm 이하 기술 구성을 연구하기 위해, 우리는 다음과 같은 측면을 기반으로 9개의 표준 셀 라이브러리를 생성했습니다.

46. [Click] Available M2 routing tracks on standard cells, 4 and 5 tracks.  
    [클릭] 표준 셀에서 사용 가능한 M2 배선 트랙은 4개와 5개 트랙입니다.

47. [Click] Use/non-use of buried power rails and  
    [클릭] 매립 전원 레일 사용 여부와

48. [Click] Gear ratios for metal pitch to CPP.  
    [클릭] 메탈 피치 대 CPP(컨택트 폴리 피치)의 기어비를 고려합니다.

49. [Click] Also, different numbers of routing tracks with the same cell height. 120nm with 4 and 5 tracks.  
    [클릭] 또한, 같은 셀 높이를 가진 다양한 배선 트랙 수를 고려합니다. 예를 들어, 120nm에서 4개와 5개 트랙입니다.

50. [Click] This table shows nine sets of configurations in detail.  
    [클릭] 이 표는 9개의 구성 세트를 자세히 보여줍니다.

51. [Click] Here, Lib{1,2,5,6} have 4 available M2 routing tracks while Lib{3,4,7,8} have 5 tracks.  
    [클릭] 여기서 Lib{1,2,5,6}은 4개의 사용 가능한 M2 배선 트랙을 가지고 있고, Lib{3,4,7,8}은 5개의 트랙을 가지고 있습니다.

52. [Click] Lib{1,3,5,7,9} have BPR while Lib{2,4,6,8} have M1 power/ground pins.  
    [클릭] Lib{1,3,5,7,9}은 매립 전원 레일(BPR)을 사용하고, Lib{2,4,6,8}은 M1 전원/접지 핀을 사용합니다.

53. [Click] Also, Lib{1,2,3,4} have a 1:2 gear ratio while Lib{5,6,7,8} have a 2:3 ratio.  
    [클릭] 또한, Lib{1,2,3,4}는 1:2 기어비를 가지고 있고, Lib{5,6,7,8}은 2:3 기어비를 가지고 있습니다.

54. [Click] Last, Lib3 has 5 M2 routing tracks with 120nm cell height and Lib9 has 4 tracks with the same cell height.  
    [클릭] 마지막으로, Lib3은 120nm 셀 높이에서 5개의 M2 배선 트랙을 가지고 있으며, Lib9은 같은 셀 높이에서 4개의 트랙을 가지고 있습니다.

55. The following slides show how to evaluate routability using PROBE2.0.  
    다음 슬라이드는 PROBE2.0을 사용하여 배선 가능성을 평가하는 방법을 보여줍니다.

56. [Click] With the generated libraries in the previous slides, we perform routability assessments. PROBE2.0 supports two types of routability assessment, cell- and block-level assessment.  
    [클릭] 이전 슬라이드에서 생성된 라이브러리를 사용하여 배선 가능성 평가를 수행합니다. PROBE2.0은 셀 수준 및 블록 수준의 두 가지 배선 가능성 평가를 지원합니다.

57. In this cell-level assessment, we assess 15 cells in each library.  
    이 셀 수준 평가에서 우리는 각 라이브러리에서 15개의 셀을 평가합니다.

58. [Click] This graph shows the result of cell-level assessment. And, as a reminder, larger Kth means better routability.  
    [클릭] 이 그래프는 셀 수준 평가 결과를 보여줍니다. 다시 말씀드리지만, Kth 값이 클수록 배선 가능성이 좋다는 의미입니다.

59. [Click] If we focus on NAND cells, here’s a picture of how more tangling – increasing on the x-axis, leads to more routing DRCs, on the y-axis.  
    [클릭] NAND 셀에 집중하면, x축에서 얽힘이 증가할수록 y축에서 더 많은 배선 DRC(설계 규칙 위반)가 발생하는 모습을 볼 수 있습니다.

60. Here, [Click] the NAND3_X1 (dashed lines) has the worse inherent routability – it fails first, [Click] while the NAND2_X1 (solid lines) has the better inherent routability.  
    여기서, [클릭] NAND3_X1(점선)은 고유 배선 가능성이 가장 나빠서 먼저 실패하고, [클릭] NAND2_X1(실선)은 고유 배선 가능성이 더 좋습니다.

61. [Click] Also, if we focus on library, we can see that the cells with larger cell height (140nm) has better inherent routability.  
    [클릭] 또한 라이브러리에 집중해 보면, 셀 높이가 큰 셀(140nm)이 더 나은 고유 배선 가능성을 가지고 있음을 알 수 있습니다.

62. [Click] Also, we can assess library (cell set) with block-level routability assessment.  
    [클릭] 또한 우리는 블록 수준의 배선 가능성 평가를 통해 라이브러리(셀 세트)를 평가할 수 있습니다.

63. [Click] This graph shows Kth per library in four designs, AES, JPEG, VGA and LDPC. 39 standard cells are included in each library. Also, again, larger Kth means better routability.  
    [클릭] 이 그래프는 AES, JPEG, VGA, LDPC라는 네 가지 설계에서 각 라이브러리별 Kth를 보여줍니다. 각 라이브러리에는 39개의 표준 셀이 포함되어 있습니다. 역시, Kth가 클수록 배선 가능성이 좋습니다.

64. [Click] Similarly, when we look in detail for the AES case, we measure DRC counts as we increase routing difficulty (K).  
    [클릭] 유사하게, AES 케이스의 세부 사항을 보면, 우리는 배선 난이도(K)를 증가시키면서 DRC 카운트를 측정합니다.

65. [Click] Based on K threshold, the rank-ordering would be following the order. Lib8 is the best cell set in terms of routability [Click] while Lib{1,5} are the worst.  
    [클릭] K 임계값을 기준으로 하면, 순위는 다음과 같습니다. Lib8이 배선 가능성 측면에서 가장 우수하며 [클릭] Lib{1,5}가 가장 열악합니다.

66. [Click] From this slide, we show that PROBE2.0 is a useful framework for routability assessment in pathfinding and DTCO problems.  
    [클릭] 이 슬라이드에서 우리는 PROBE2.0이 경로 탐색 및 DTCO 문제에서 배선 가능성 평가에 유용한 프레임워크임을 보여줍니다.

67. [Click] In order to measure block-level area scaling, we define achievable utilization and compare it with K threshold. Achievable utilization is a metric that is used in the real world.   
    [클릭] 블록 수준의 면적 확장을 측정하기 위해 우리는 달성 가능한 활용도를 정의하고 이를 K 임계값과 비교합니다. 달성 가능한 활용도는 실제로 사용되는 지표입니다.

68. [Click] We perform P&R with various utilization and measure Kth for each case. Achievable utilization is defined as the largest utilization when the number of DRCs is smaller than the threshold.   
    [클릭] 우리는 다양한 활용도를 사용하여 P&R을 수행하고 각 경우에 대해 Kth를 측정합니다. 달성 가능한 활용도는 DRC 수가 임계값보다 작은 경우의 최대 활용도로 정의됩니다.

69. [Click] This scatter plot shows the relationship between Kth and achievable utilization.   
    [클릭] 이 산점도는 Kth와 달성 가능한 활용도 간의 관계를 보여줍니다.

70. [Click] As you see in the plot, achievable utilization increases by approximately the same rate as Kth changes.   
    [클릭] 그래프에서 보시다시피, 달성 가능한 활용도는 Kth 변화에 따라 대략 같은 비율로 증가합니다.

71. Based on cell height and achievable block-level area, we measure cell- and block-level area overhead in the following cases.   
    셀 높이와 달성 가능한 블록 수준의 면적을 기준으로, 우리는 다음과 같은 경우에서 셀 및 블록 수준의 면적 오버헤드를 측정합니다.

72. [Click] For case 1, we compare standard cells with different available M2 routing tracks.   
    [클릭] 첫 번째 경우로, 우리는 다른 M2 배선 트랙을 가진 표준 셀을 비교합니다.

73. [Click] For case 2, we compare standard cells with and without BPR.   
    [클릭] 두 번째 경우로, 우리는 매립 전원 레일(BPR)이 있는 셀과 없는 셀을 비교합니다.

74. [Click] For case 3, we use cells with different metal pitch to CPP ratios, and [Click] different routing track with the same cell height for case 4.   
    [클릭] 세 번째 경우로, 우리는 메탈 피치 대 CPP 비율이 다른 셀을 사용하고, [클릭] 네 번째 경우로, 같은 셀 높이에서 다른 배선 트랙을 사용합니다.

75. [Click] We report cell- and block-level area and [Click] in this case, we can see that block-level area decreases due to routability even though cell-level area is larger.   
    [클릭] 우리는 셀 및 블록 수준의 면적을 보고하며 [클릭] 이 경우 셀 수준의 면적이 더 커도 배선 가능성으로 인해 블록 수준의 면적이 감소하는 것을 알 수 있습니다.

76. This slide shows our ML-based Kth prediction results.   
    이 슬라이드는 기계 학습 기반의 Kth 예측 결과를 보여줍니다.

77. [Click] Here, we have a question. What data should we actually generate for training?   
    [클릭] 여기서 우리는 질문을 던집니다. 학습을 위해 실제로 어떤 데이터를 생성해야 할까요?

78. [Click] In this work, we propose to use Latin Hypercube Sampling for efficient training set selection.   
    [클릭] 이번 연구에서는 효율적인 학습 세트 선택을 위해 라틴 하이퍼큐브 샘플링을 사용할 것을 제안합니다.

79. [Click] For our experiment, we generate 448 standard-cell libraries with the configurations in this table.   
    [클릭] 실험을 위해 우리는 이 표에 있는 구성으로 448개의 표준 셀 라이브러리를 생성합니다.

80. And [Click], we select training sets by using Latin Hypercube Sampling. These figures show our prediction results with reasonably small average errors.   
    그리고 [클릭] 우리는 라틴 하이퍼큐브 샘플링을 사용하여 학습 세트를 선택합니다. 이 도표는 우리가 합리적으로 작은 평균 오차를 가진 예측 결과를 보여줍니다.

81. Now, I will conclude my talk.   
    이제 제 발표를 마무리하겠습니다.

82. [Click] In this work, we report block-level area scaling can be reversed from cell-level area scaling based on our experiment.   
    [클릭] 이번 연구에서는 실험을 통해 셀 수준의 면적 확장과 블록 수준의 면적 확장이 반대될 수 있음을 보고합니다.

83. [Click] We use the PROBE2.0 framework and show that it is a useful framework for an early-stage routability evaluation.   
    [클릭] 우리는 PROBE2.0 프레임워크를 사용하여 이것이 초기 단계 배선 가능성 평가에 유용한 프레임워크임을 보여줍니다.

84. [Click] We also show that machine learning can help the assessment.   
    [클릭] 또한 기계 학습이 평가에 도움이 될 수 있음을 보여줍니다.

85. [Click] We study more than 400 unique standard-cell libraries with sub-3nm technology configurations.   
    [클릭] 우리는 3nm 이하 기술 구성을 가진 400개 이상의 고유 표준 셀 라이브러리를 연구합니다.

86. [Click] As our future work, we will bring power and performance aspects into the framework.   
    [클릭] 향후 연구로 우리는 전력 및 성능 측면을 프레임워크에 포함할 예정입니다.

87. Thank you for listening to my presentation.   
    제 발표를 들어주셔서 감사합니다.