---
title: "Ch.12 Binary Search Trees"
excerpt: "Introduction to Algorithms"
date: 2023-05-29 10:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Algorithm
mathjax: "true"
---

Introduction to Algorithms(4th Edition) 공부하면서 정리한 내용입니다.
{: .notice--info}

- Search tree: `SEARCH`, `MINIMUM`, `MAXIMUM`, `PREDECESSOR`, `SUCCESSOR`, `INSERT`, `DELETE` 연산을 제공하는 자료 구조
- Binary Search Tree의 기본적인 연산들은 Tree의 높이에 비례한 시간이 걸린다.
- 노드가 $n$개인 Complete Binary Tree의 경우, 연산에 $\Theta(\lg n)$ 시간이 걸린다. 하지만 Tree가 $n$개의 linear chain으로 구성될 경우 같은 동작이 $\Theta(n)$ 시간이 걸린다. 챕터 13에서는 binary tree의 변형인 red-black tree를 다룰 것이며, 이는 $\Theta(\lg n)$ height를 보장한다.

## 12.1 What is a binary search tree?

- Binary tree는 linked data structure로 표현할 수 있다.
- *key*와 satellite data 외에 각 노드는 *left*(왼쪽 child), *right*(오른쪽 child), *p*(parent)를 가진다. child나 parent가 없다면 해당 field는 `NIL`을 가진다.
- Tree는 루트 노드를 가리키는 `T.root` field를 가진다. `T.root`는 tree `T`에서 parent가 `NIL`인 유일한 노드이다.
- **Binary search tree property**: binary search tree의 어떤 노드 $x$에 대해, $x$의 왼쪽 subtree에 있는 어떤 노드 $y$에 대해 $y.key \le x.key$이고, $y$가 $x$의 오른쪽 subtree에 있다면 $y.key \ge x.key$이다.
- Binary search tree property로 인해 inorder tree walk를 사용하면 정렬된 순서로 노드를 프린트할 수 있다.

```text
INORDER-TREE-WALK(x)
1  if x ≠ NIL
2    INORDER-TREE-WALK(x.left)
3    print x.key
4    INORDER-TREE-WALK(x.right)
```

## 12.2 Querying a binary search tree

- `SEARCH`, `MINIMUM`, `MAXIMUM`, `PREDECESSOR`, `SUCCESSOR` 연산은 모두 $\Theta(h)$ 시간이 걸린다. 여기서 $h$는 tree의 height이다.

### 12.2.1 Searching

- `SEARCH` 연산은 subtree의 root 포인터 *x*와 key *k*가 주어졌을 때 `TREE-SEARCH(x, k)`는 subtree에서 key 값이 *k*인 노드의 포인터를 반환한다. 만약 key가 *k*인 노드가 존재하지 않으면 `NIL`을 리턴한다. key *k*를 전체 binary search tree *T*에서 찾으려면 `TREE-SEARCH(T.root, k)`를 호출하면 된다.

```text
TREE-SEARCH(x, k)
1  if x == NIL or k == x.key
2    return x
3  if k < x.key
4    return TREE-SEARCH(x.left, k)
5  else return TREE-SEARCH(x.right, k)
```

```text
ITERATIVE-TREE-SEARCH(x, k)
1  while x != NIL and k != x.key
2    if k < x.key
3      x = x.left
4    else x = x.right
5  return x
```

`TREE-SEARCH`는 루트에서 검색을 시작하고 simple path를 따라서 트리 아래로 내려간다. 마추치는 노드에 대해 키값 $k$와 $x.key$를 비교한다. 만약 두 키가 같으면 검색이 종료된다. 만약 $k$가 $x.key$보다 작아면 $x$의 왼쪽 subtree에 대해 검색을 계속한다. 왜냐하면 binary-search-tree property에 의해 $k$는 오른쪽 subtree에 존재할 수 없기 때문이다. 이와 유사하게 $k$가 $x.key$보다 크면 $x$의 오른쪽 subtree에 대해 검색을 계속한다. recursion 동안에 만나는 노드들은 root에서 아래로 내려가는 simple path가 된다. 따라서 `TREE-SEARCH`는 $\Theta(h)$ 시간이 걸린다. `ITERATIVE-TREE-SEARCH`는 `TREE-SEARCH`와 동일한 동작을 하지만 recursion을 사용하지 않고 **while** 루프로 "unroll"하였다. 대부분의 컴퓨터에서 `ITERATIVE-TREE-SEARCH`가 `TREE-SEARCH`보다 더 효율적이다.

### 12.2.2 Minimum and maximum

binary search tree의 가장 작은 element를 찾기 위해서는 그냥 `NIL`을 만날 때까지 *left* child를 따라가면 된다. `TREE-MINIMUM`은 주어진 노드 $x$를 루트로 하는 subtree의 minimum element에 대한 포인터를 리턴한다.

```text
TREE-MINIMUM(x)
1  while x.left != NIL
2    x = x.left
3  return x
```

```text
TREE-MAXIMUM(x)
1  while x.right != NIL
2    x = x.right
3  return x
```

### 12.2.3 Successor and predecessor

모든 key가 distinct하면 노드 $x$의 successor는 $x.key$보다 큰 키 중 가장 작은 키를 가진 노드이다. key의 distinct 여부와 관계 없는 정의로 indorder tree walk에서 $x$ 다음에 방문하는 노드로 정의한다.  
`TREE-SUCCESSOR` 코드에는 두 가지 경우가 존재한다. 노드 x의 right subtree가 존재할 경우에는 x의 successor는 x의 right subtree의 가장 왼쪽의 노드이다. 반면에 x의 오른쪽 subtree가 비어있고 successor $y$가 존재한다면, $y$의 left child로 $x$의 조상을 가지고 있는 조상 중 가장 가까운 조상이다.

```text
TREE-SUCCESSOR(x)
1  if x.right != NIL
2    return TREE-MINIMUM(x.right)
3  else
4    y = x.p
5    while y != NIL and x == y.right
6      x = y
7      y = y.p
8    return y
```

## 12.3 Insertion and deletion

### 12.3.1 Insertion

![2023-05-29-bst-insert.png]({{site.baseurl}}/assets/images/2023-05-29-bst-insert.png){: .align-center}  

새로운 키 13을 추가하는 그림  
{: .custom-caption }

`TREE-INSERT`는 새로운 노드를 바이너리 서치 트리에 추가한다. binary search tree $T$와 노드 $z$를 입력으로 받는다. $z.key$는 이미 채워진 상태이며 `z.left = NIL`, `z.right = NIL`로 세팅되어 있어야 한다. 이 함수는 $T$와 $z$의 일부 속성을 변경하여 $z$를 트리의 알맞는 위치에 추가한다.

```text
TREE-INSERT(T, z)
 1  x = T.root     // z와 비교할 노드
 2  y = NIL        // z의 parent가 될 노드
 3  while x != NIL 
 4    y = x
 5    if z.key < x.key
 6      x = x.left
 7    else x = x.right
 8  z.p = y        // 위치 발견 - z의 parent를 y로 설정
 9  if y == NIL    // tree가 비어있는 경우
10    T.root = z
11  else if z.key < y.key
12    y.left = z
13  else y.right = z
```

위 함수에서 **trailing pointer** $y$를 $x$의 부모 노드로 유지한다. 라인 3-7의 **while** 루프는 두 개의 포인터를 사용해서 트리를 내려한다. 이를 $x$가 `NIL`이 될 때까지 반복하고, 이 `NIL`이 $z$가 삽입될 위치이다. 정확히 말하자면 `NIL`은 $z$의 부모가 될 노드의 *left*나 *right* attribute이다. 아니면 $T$가 비어있다면 $T.root$이다. 그래서 이 함수는 trailing pointer $y$가 필요하다. 왜냐하면 $z$가 들어가야할 `NIL`을 찾았을 때 변경해줘야할 노드를 이미 지나치기 때문이다. 라인 8-13에서 $z$ 포인터를 설정한다. 다른 연산들처럼 `TREE-INSERT`는 $\Theta(h)$ 시간이 걸린다.  

### 12.3.2 Deletion

`TREE-DELETE`는 트리에서 노드 $z$를 제거한다. $z$의 자식 노드의 수에 따라 세 가지 경우가 존재한다.

- $z$가 자식 노드를 가지지 않는 경우. $z$를 그냥 제거하면 된다.
- $z$가 자식 노드를 하나만 가지는 경우. 해당 자식 노드를 $z$의 위치로 올리면 된다. (즉, $z$를 제거하고 $z$의 부모 노드와 $z$의 자식 노드를 연결한다.)
- $z$가 두 개의 자식 노드를 가지는 경우. $z$의 successor $y$를 찾는다($y$는 $z$의 오른쪽 subtree에 속한다). 그리고 $y$를 $z$의 위치로 옮긴다. $z$의 나머지 오른쪽 subtree는 $y$의 오른쪽 subtree가 된다. $z$의 왼쪽 subtree는 $y$의 왼쪽 subtree가 된다. $y$는 $z$의 successor이기 때문에 $y$는 왼쪽 child를 갖고 있지 않으며 $y$의 오른쪽 child를 $y$의 원래 위치로 옮긴다.

![2023-05-29-bst-delete.png]({{site.baseurl}}/assets/images/2023-05-29-bst-delete.png){: .align-center}  

주어진 노드 $z$를 binary search tree $T$에서 제거하는 함수는 $T$와 $z$에 대한 포인터를 argument로 받는다. 그리고 위에서 설명한 세 가지를 조금 다르게 구현한다.  

- $z$에게 left child가 없는 경우(a). $z$의 right child를 $z$의 위치로 옮긴다. right child의 `NIL` 여부는 상관하지 않는다.  만약 right child가 `NIL`인 경우에 $z$의 child가 존재하지 않는 경우이며 결과가 같기 때문이다.
- 아니면 $z$에게 left child 하나만 있는 경우(b). $z$를 left child로 대체한다.
- 아니면 $z$가 왼쪽과 오른쪽 child를 모두 가진 경우. $z$의 successor $y$를 찾는다. $y$는 $z$의 오른쪽 subtree에서 가장 작은 key를 가지고 있기 때문에 $y$의 left child가 없다. $y$ 떼어내어 $z$의 위치로 옮긴다. 옮기는 방법은 $y$가 $z$의 right child이냐 아니냐에 따라 달라진다.
  - $y$가 $z$의 right child인 경우(c). $z$를 $y$로 교체한다. $y$의 right child는 그대로 둔다.
  - $y$가 $z$의 right child가 아닌 경우(d). 먼저 $y$의 right child를 $y$의 위치로 옮긴다. $z$를 $y$로 교체한다.

노드를 삭제하는 과정에서 subtree가 binary search tree 내에서 이동해야 한다. `TRANSPLANT` 함수는 $u$와 $v$를 argument로 받는다. $v$의 parent가 $u$의 parent로 변경되며, $u$의 parent가 $v$를 child로 갖게되면서 함수가 종료한다. $v$는 모두 `NIL`일 수 있다.

```text
TRANSPLANT(T,u,v)
1  if u.p == NIL
2    T.root = v
3  else if u == u.p.left
4    u.p.left = v
5  else u.p.right = v
6  if v != NIL
7    v.p = u.p
```

라인 1-2는 $u$가 $T$의 root인 경우를 처리한다. 라인 3-5는 $u$가 $u.p$의 left child인지 right child인지에 따라 $u.p$의 left child나 right child를 $v$로 변경한다. 라인 6-7은 $v$가 `NIL`이 아닌 경우 $v$의 parent를 $u$의 parent로 변경한다.  

다음에 소개할 `TREE-DELETE` 함수는 binary search tree $T$에서 노드 $z$를 제거할 때 `TRANSPLANT` 함수를 사용한다. 4가지 케이스를 구분해서 처리한다. 라인 1-2는 노드 $z$의 left child가 없는 경우를 처리한다. 라인 3-4는 노드 $z$의 left child만 있는 경우를 처리한다. 라인 5-12는 나머지 두 경우를 다룬다. 라인 5에서 $z$의 successor 노드 $y$를 찾는다. $z$는 nonempty right subtree를 갖기 때문에 $z$의 successor는 right subtree의 가장 작은 키이다. 그러므로 `TREE-MINIMUM(z.right)`를 호출한다. 앞에서 얘기했듯이 $y$는 left child를 가지고 있지 않다. 이 함수에서 $y$를 현재 위치에서 떼어내어 $z$를 교체해야 한다. 만약 $y$가 $z$의 오른쪽 child이면 라인 10-12는 $z$를 $y$로 대체한다. 노드 $y$는 자신의 right child를 유지한다. $y$가 $z$의 오른쪽 child가 아니면 두 개의 노드를 움직여야 한다. 라인 7-9는 $y$를 $y$의 right child로 교체하고 $z$의 right child를 $y$의 right child로 만든다. 마지막으로 라인 10-12는 $z$를 $y$로 대체하고 $y$의 left child를 $z$의 left child로 만든다.  

```text
TREE-DELETE(T,z)
 1  if z.left == NIL
 2    TRANSPLANT(T,z,z.right)       // z를 rigth child로 대체
 3  else if z.right == NIL
 4    TRANSPLANT(T,z,z.left)        // z를 left child로 대체
 5  else y = TREE-MINIMUM(z.right)  // y는 z의 successor
 6    if y != z.right               // y가 z의 right child가 아니면
 7      TRANSPLANT(T,y,y.right)     // y를 y의 right child로 대체
 8      y.right = z.right           // z의 right child를 y의 right child로 만듬
 9      y.right.p = y
10    TRANSPLANT(T,z,y)             // z를 y로 대체
11    y.left = z.left               // z의 left child를 y의 left child로 만듬
12    y.left.p = y                  // (y는 기존 left child를 갖고 있지 않음)
```
