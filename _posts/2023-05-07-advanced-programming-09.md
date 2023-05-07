---
title: "Strongly Connected Components"
excerpt: "Advanced Programming Methodologies 09"
date: 2023-05-07 23:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

## Strongly Connected Components

- Directed Graph에서 동작하는 많은 알고리즘은 이러한 Decomposition을 기반으로 한다.
- Graph를 Strongly Connected Components로 Decompose한 후, 알고리즘을 각각에 대해 적용한 후 솔루션을 결합할 수 있다.

- Directed Graph G의 Strongly Connected Components(SCC)는 다음을 만족하는 vertex들의 maximal set C 이다. C ⊆ V such that ∀ u, v ∈ C, u와 v는 서로 reachable하다.
- SCC를 찾기 위해 DFS를 두 번 수행한다: $G$ 한 번, $G^T$ 한 번
  - Transpose of G: $G^T=(V, E^T)$, where $E^T=\lbrace (u, v): (v, u) \in E \rbrace$
  - Time Complexity: Θ(\|V\| + \|E\|)
- $G$와 $G^T$는 완전히 동일한 SCC를 가지고 있다.
  - G에서 u와 v가 서로 reachable하다면, $G^T$에서도 reachable하다.

```text
STRONGLY-CONNECTED-COMPONENTS(G)
1. call DFS(G) to compute finishing times u.f for each vertex u
2. comput G^T
3. call DFS(G^T), but in the main loop of DFS, consider the vertices 
   in order of decreasing u.f (as computed in line 1)
4. output the vertices of each tree in the depth-first forest formed 
   in line 3 as a separate strongly connected component
```

![2023-05-07-strongly-connected-components.png]({{site.baseurl}}/assets/images/2023-05-07-strongly-connected-components.png)

- Component graph $G^{SCC} = (V^{SCC}, E^{SCC})$ (DAG)
  - G가 strongly connected components $C_1$, $C_2$,..., $C_k$를 갖고 있다고 하자.
  - $V^{SCC} = \lbrace v_1, v_2,..., v_k \rbrace$이고 각각의 $C_i$마다 $v_i$가 존재한다.
  - $(v_i, v_j) \in E^{SCC}$ Edge가 존재하면 G에는 edge (x, y)가 존재한다(x ∈ $C_i$, y ∈ $C_j$).

### Lemma 22.13

- C와 C'가 G=(V, E)의 서로 다른 Strongly Connected Component이고, u,v ∈ C, u', v' ∈ C'라고 하자. 그리고 u ⤳ u'인 path가 존재한다고 하자.
- 그렇다면 G는 v' ⤳ v인 path를 포함하면 안된다.

- 증명:
  - G가 v' ⤳ v인 path를 포함한다고 가정하자. 그렇다면 u ⤳ u' ⤳ v'와 v' ⤳ v ⤳ u 경로도 존재한다.
  - 그러므로 u와 v'가 서로 reachable하며, C와 C'가 서로 다른 Strongly Connected Component라는 가정에 모순된다.

### Lemma 22.14

- $d(C) = \min_{u \in C} \lbrace u.d \rbrace$
- $f(C) = \max_{u \in C} \lbrace u.f \rbrace$
- C와 C'가 Directed Graph G=(V, E)의 서로 다른 Strongly Connected Component이다.
- u ∈ C, v ∈ C'인 vertex에 대해 (u, v) ∈ E라고 하자.
- 그렇다면 $f(C) > f(C')$이다.

- 증명 Case: d(C) < d(C')
  - x가 C에서 가장 처음 발견된 vertex라고 하자.
  - x.d 시점에서 C와 C'의 vertex들은 모두 white이다.
  - x.d 시점에서 G에는 C에 속한 모든 vertex들에 대한 white vertice로 구성된 path를 포함하고 있다.
  - 그리고 (u, v) ∈ E이므로, 어떤 vertex w ∈ C'에 대해 white vertex들로 구성된 x ⤳ w path가 존재한다.(x ⤳ u → v ⤳ w)
  - white-path theorem에 의해 C와 C'의 모든 vertex들은 x의 descendant가 된다.
  - Corollary 22.8에 의해 x는 descendant들보다 큰 finishing time을 가지므로, x.f = f(C) > f(C')이다.

> **Theorem 22.9 (White-path Theorem)**  
> (directed 혹은 undirected) Graph G=(V,E)의 depth-first forest에서 vertex v가 vertex u의 descendant이다. ⇔(iff) u.d 발견 시점에서 u에서 White vertex로 구성된 path를 통해 v로 도달할 수 있다.

> **Corollary 22.8**  
> Depth-first forest에서 Vertex v가 u의 proper descendant이다. ⇔(iff) u.d < v.d < v.f < u.f 이다.

- 증명 Case: d(C) > d(C')
  - y가 C'에서 가장 처음 발견된 vertex라고 하자.
  - y.d 시점에서 C'의 vertex들은 모두 white이고, G에는 C'에 속한 모든 vertex들에 대한 white vertice로 구성된 y로부터의 path를 포함하고 있다.
  - white-path theorem에 의해 C'의 모든 vertex들은 y의 descendant가 된다.
  - Corollary 22.8에 의해 y는 descendant들보다 큰 finishing time을 가지므로, y.f = f(C')이다.
  - y.d 시점에서 C의 모든 vertex들은 white이다.
  - (u,v) Edge가 C에서 C'로 향하는 edge가 존재하므로, Lemma 22.13으로부터 C'에서 C로 향하는 path는 존재할 수 없다.
  - 그러므로 C의 vertex들은 y에서 도달할 수 없다.
  - y.f 시점에서 C의 vertex들은 모두 white이다.
  - 그러므로 C에 속하는 어떤 vertex w에 대해 w.f > y.f이며, f(C) > f(C')이다.

### Corollary 22.15

- C와 C'가 Directed Graph G=(V, E)의 서로 다른 Strongly Connected Component이다.
- Edge $(u, v) \in E^T$가 존재한다고 하자. u ∈ C, v ∈ C'
- 그러면 f(C) < f(C') 이다.

- 증명:
  - (u, v) ∈ $E^T$ ⇔ (v, u) ∈ E
  - Lemma 22.14에 의해 f(C) < f(C')이다.

### Theorem 22.16

- `STRONGLY-CONNECTED-COMPONENTS`는 주어진 입력 Directed Graph G로부터 SCC를 제대로 찾는다.
- 증명:
  - $G^T$에 대한 depth-first search로부터 찾아진 depth-first tree 갯수에 대해 induction을 이용한다. 각 tree의 vertex들은 strongly connected component를 형성한다.
  - Induction hypothesis: 첫 k tree는 SCC를 형성한다.
  - Basis case: k=0일 때 trivial true
  - Inductive step:
    - 첫 k개의 depth-first tree는 SCC를 형성한다고 가정하자. (k+1)번째 tree에 대해 생각해보자.
    - (k+1)번째 tree의 root vertex u라고 하고, u가 strongly connected component C에 속해있다고 가정하자.
    - 우리는 root를 finishing time의 내림차순으로 고르므로, u.f = f(C) > f(C')이다. (C'는 아직 방문하지 않은 SCC)
    - Inductive hypothesis에 의해 u를 방문할 때 C의 다른 vertex들은 모두 white이다.
    - white-path theorem에 의해 C의 다른 모든 vertex들은 u의 descendant이다.
    - 또한 inductive hypothesis와 Corollary 22.15로부터 $G^T$에서 C에서 빠져나가는 edge들은 모두 이미 방문한 상태이다.
    - 그러므로 C 외에 다른 SCC에 속하는 vertex들은 u의 descendant가 될 수 없다.
    - 그러므로 $G^T$에서 u를 root로 하는 depth-first tree는 C에 속한 vertex들로만 구성된다. 이는 inductive step을 만족하며 증명이 끝난다.

