---
title: "Ch.13 Red-Black Trees"
excerpt: "Introduction to Algorithms"
date: 2023-05-29 13:00:00 +0900
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

Chapter 12에서 binary search tree의 기본적인 연산들을 살펴보았다. binary search tree의 높이가 $h$일 때 연산들의 시간복잡도는 $O(h)$이다. 그러므로 binary search tree의 높이가 작을수록 빠르다. Red-black tree는 많은 "balanced" search-tree shceme 중 하나이며 기본 연산이 worst case $O(\lg n)$이 보장된다.  

## 13.1 Properties of red-black trees

**red-black tree**는 노드마다 **color**라는 1 bit 정보를 추가적으로 갖고 있다. **color**는 `RED`나 `BLACK`이 될 수 있다. root로부터 leaf까지의 모든 simple path에 대해 노드 color를 제한함으로서 특정 path가 다른 path보다 2배 이상 길어지지 않음을 보장한다. 그래서 tree가 거의 balanced되도록 한다. 사실 우리는 $n$ 키를 가지는 red-black tree의 높이가 최대 $2 \lg (n+1)$이 됨을 확인할 것이다($O(\lg n)$).  

red-black tree는 다음의 **red-black property**들을 만족하는 binary search tree이다.

1. 모든 노드는 `RED`나 `BLACK`이다.
2. root는 `BLACK`이다.
3. 모든 leaf(`NIL`)는 `BLACK`이다.
4. `RED` 노드의 children은 모두 `BLACK`이다.
5. 각 노드에 대해 그 노드로부터 descendant leaves까지의 simple path는 모두 같은 수의 `BLACK` 노드를 포함하고 있다. 이 수를 **black-height**라고 한다.

boundary condition을 다룰 때 편의를 위해 `NIL`을 나타내는 하나의 sentinel을 사용한다. red-black tree $T$에 대해 sentinel $T.nil$은 보통의 tree 노드와 같은 필드를 가지고 있다. 이 노드의 color는 `BLACK`이고, 나머지 attribute들은 임의의 값을 가질 수 있다.  

![2023-05-29-rb-tree-sentinel.png]({{site.baseurl}}/assets/images/2023-05-29-rb-tree-sentinel.png){: .align-center}  

(b)는 `NIL`이 모두 $T.nil$을 가리키는 포인터로 대체됨을 보여준다. 왜 sentinel을 사용할까? sentinel은 `NIL` child를 일반적인 노드처럼 다룰 수 있게 만든다. 다른 design으로는 `NIL`을 각각의 sentinel 노드로 만들 수도 있고, 각 `NIL`의 parent가 제대로 정의되도록 만들 수도 있다. 하지만 이런 design은 불필요하게 공간을 낭비한다. 대신에 하나의 sentinel $T.nil$이 모든 `NIL`들을 나타내도록 할 수 있다(모든 leaf와 root의 parent). sentinel의 *p*, *left*, *right*, *key*는 중요하지 않다. sentinel을 사용해서 더 간단한 코드로 구현할 수 있다.  

노드 $x$로부터 내려가는 simple path($x$는 포함하지 않음) 내의 black node의 수를 **black-height**라고 부르며 $bh(x)$로 표기한다.  

### Lemma 13.1

$n$개의 내부 노드를 가지는 red-black tree의 height는 최대 $2 \lg (n+1)$이다.

### Proof

어떤 노드 $x$를 root로 가지는 subtree에 대해 적어도 $2^{bh(x)} - 1$개의 internal node가 존재하는 것을 먼저 보일 것이다. height $x$에 대해 induction을 적용한다. 만약 $x$의 height가 0이면 $x$는 leaf($T.nil$)이고 $bh(x) = 0$이고, $x$의 subtree는 $2^{bh(x)} - 1 = = 2^0 - 1 = 0$개의 internal node를 가진다. Inductive step을 보면 $x$가 positive height를 가진 internal node라고 하자. node $x$는 두 개의 child를 가지고, child는 leaf일 수도 있다. 만약 child가 black이면 $x$의 black-height에 1로 count되고, red라면 $x$나 자기 자신의 black-height에 count 되지 않는다. 그러므로 child의 black-height는 $bh(x) - 1$(black일 경우)이거나 $bh(x)$(red일 경우)이다. $x$의 child의 height는 $x$ 자신의 height보다 작으며, inductive hypothesis로부터 각 child는 적어도 $2^{bh(x)-1}$개 이상의 internal node를 가진다. 이로부터 $x$는 적어도 $2^{bh(x)-1} + 2^{bh(x)-1} + 1 = 2^{bh(x)} - 1$개의 internal node를 가지므로 induction이 성립한다.  
*h*가 tree의 height라고 하면 property 4로부터 적어도 root에서 leaf까지의 root를 제외한 simple path 노드의 절반은 black이다. 결과적으로 root의 black-height는 적어도 $h/2$ 이상이다. 이로부터 $n \ge 2^{h/2} - 1$를 얻는다. 1을 좌항으로 보내고 양쪽에 로그를 취하면 $\lg(n+1) \ge h/2$ 혹은 $h \le 2 \lg(n+1)$을 얻을 수 있다.  

## 13.2 Rotations

search-tree의 `TREE-INSERT`나 `TREE-DELETE` 연산을 $n$개의 키를 가진 red-black tree에서 수행 시 $O(\lg n)$ 시간이 걸린다. 이 연산들은 tree를 변경하므로 결과적으로 red-black property를 위반하게 될 수 있다. property를 복구하기 위해 노드의 color나 pointer들이 변경될 필요가 있다.  
pointer 구조는 **rotation**을 통해 변경한다. rotation은 binary-search-tree property를 유지하면서 tree의 구조를 변경하는 간단한 local operation이다. rotation은 두 가지 종류가 있다: left rotation과 right rotation.  

![2023-05-29-rb-tree-rotation.png]({{site.baseurl}}/assets/images/2023-05-29-rb-tree-rotation.png){: .align-center}  

노드 $x$에 대한 left rotation을 살펴보자. $x$는 right child $y$를 가지고 있므로 $y$는 $T.nil$이 아니어야 한다. left rotation은 $x$가 루트인 subtree에 대해 $x$와 $y$ 사이의 연결을 왼쪽으로 튼다. subtree의 새로운 root는 $y$가 되고 $x$는 $y$의 left child가 되고 원래 $y$의 left child 였던 부분은 $x$의 right child가 된다. 아래 `LEFT-ROTATE` pseudocode는 $x.right \ne T.nil$이고 root의 parent가 $T.nil$임을 가정한다.  

```text
LEFT-ROTATE(T, x)
 1  y = x.right
 2  x.right = y.left      // y의 left subtree를 x의 right subtree로 옮긴다
 3  if y.left != T.nil    // y의 left subtree가 존재한다면
 4    y.left.p = x        // x를 y의 left subtree의 parent로 설정한다
 5  y.p = x.p             // x의 기존 parent를 y의 parent로 설정한다
 6  if x.p == T.nil       // x가 기존 root인 경우
 7    T.root = y          // y를 root로 설정한다
 8  elseif x == x.p.left  // x가 기존에 left child인 경우
 9    x.p.left = y        // y가 새로운 left child가 된다
10  else x.p.right = y    // x가 기존에 right child인 경우
11  y.left = x            // x가 y의 left child가 된다
12  x.p = y
```

`RIGHT-ROTATE`는 `LEFT-ROTATE`와 비슷하지만 좌우반전으로 동작한다. rotate 연산은 $O(1)$ 시간이 걸린다.  

## 13.3 Insertion

`RB-INSERT`는 처음에는 일반적인 binary search tree와 같이 노드 $z$를 tree $T$에 추가한다. 그리고 $z$는 `RED` color로 설정된다. 그 후 red-black property를 유지하기 위해 보조 함수인 `RB-INSERT-FIXUP`을 호출한다.  

```text
RB-INSERT(T,z)
 1  x = T.root
 2  y = T.nil
 3  while x != T.nil
 4    y = x
 5    if z.key < x.key
 6      x = x.left
 7    else x = x.right
 8  z.p = y
 9  if y == T.nil
10    T.root = z
11  elseif z.key < y.key
12    y.left = z
13  else y.right = z
14  z.left = T.nil
15  z.right = T.nil
16  z.color = RED
17  RB-INSERT-FIXUP(T,z)
```

```text
RB-INSERT-FIXUP(T,z)
 1  while z.p.color == RED
 2    if z.p == z.p.p.left      // z의 parent가 왼쪽 자식인 경우
 3      y = z.p.p.right         // y는 z의 uncle
 4      if y.color == RED       // z의 parent와 uncle이 모두 red인가?
 5        z.p.color = BLACK     // Case 1
 6        y.color = BLACK
 7        z.p.p.color = RED
 8        z = z.p.p
 9      else 
10        if z == z.p.right
11          z = z.p             // Case 2
12          LEFT-ROTATE(T,z)
13        z.p.color = BLACK     // Case 3
14        z.p.p.color = RED
15        RIGHT-ROTATE(T,z.p.p)
16    else    // 3-15와 동일하나 right, left만 반대이다
17      y = z.p.p.left
18      if y.color == RED
19        z.p.color = BLACK
20        y.color = BLACK
21        z.p.p.color = RED
22        z = z.p.p
23      else
24        if z == z.p.left
25          z = z.p
26          RIGHT-ROTATE(T,z)
27        z.p.color = BLACK
28        z.p.p.color = RED
29        LEFT-ROTATE(T,z.p.p)
30  T.root.color = BLACK
```

![2023-05-29-rb-tree-insert.png]({{site.baseurl}}/assets/images/2023-05-29-rb-tree-insert.png){: .align-center}  

`RB-INSERT-FIXUP`의 동작. **(a)** $z$ 추가 후이다 $z$와 $z.p$ 모두 red이므로 property 4를 위반한다. $z$의 uncle $y$가 red이므로 Case 1에 해당된다. $z$의 grandparent는 black이어야만 하며, 이 black이 $z$의 parent와 uncle로 옮겨진다. 그 후 pointer $z$는 두 레벨 위로 올라가며 **(b)**와 같이 된다. 다시 $z$와 $z$의 parent가 모두 red이다. 하지만 이번에는 uncle은 black이다. $z$가 $z.p$의 right child에 해당되기 때문에 Case 2에 해당된다. left rotation을 수행하면 **(c)**가 된다. 이제 $z$는 left child가 되며 Case 3에 해당되게 된다. Recoloring과 right rotation을 수행하면 **(d)**가 되며, 이는 올바른 red-black tree가 된다.  

red-black tree를 설명할 때 node의 parent의 sibling을 참조해야할 때가 많다. 이를 **uncle**이라고 부르자. 코드의 절반은 대칭이므로 주로 라인 3-15 위주로 설명하겠다. **while** 루프 동안 반복 시작 시 세 가지 invariant를 유지한다.  

1. Node $z$는 red이다.
2. $z.p$가 루트이면 $z.p$는 black이다.
3. 만약 tree가 red-black property를 위반한다면 최대 1가지만 위반되며, 이는 property 2번 위반(root가 red)이거나 4번 위반($z$, $z.p$가 모두 red) 중 하나이다.

## 13.4 Deletion

노드의 삭제도 $O(\lg n)$의 시간이 걸린다. 노드 삭제는 삽입보다 더 복잡하다. 먼저 앞에서 사용한 `TRANSPLANT`와 같이 새로운 subroutine `RB-TRANSPLANT`를 사용한다. `NIL` 대신에 $T.nil$을 사용하며 $v$가 sentinel($T.nil$)인 경우에도 $v.p$를 assign할 수 있게 되었다.

```text
RB-TRANSPLANT(T,u,v)
1  if u.p == T.nil
2    T.root = v
3  elseif u == u.p.left
4    u.p.left = v
5  else u.p.right = v
6  v.p = u.p
```

```text
RB-DELETE(T,z)
 1  y = z
 2  y-original-color = y.color
 3  if z.left == T.nil
 4    x = z.right
 5    RB-TRANSPLANT(T,z,z.right)
 6  elseif z.right == T.nil
 7    x = z.left
 8    RB-TRANSPLANT(T,z,z.left)
 9  else y = TREE-MINIMUM(z.right)
10    y-original-color = y.color
11    x = y.right
12    if y != z.right
13      RB-TRANSPLANT(T,y,y.right)
14      y.right = z.right
15      y.right.p = y
16    else x.p = y
17    RB-TRANSPLANT(T,z,y)
18    y.left = z.left
19    y.left.p = y
20    y.color = z.color
21  if y-original-color == BLACK
22    RB-DELETE-FIXUP(T,x)
```

기존 노드 $y$가 black이었을 경우 red-black properties 위반이 발생할 수 있다. 만약 $y$가 red였을 경우 red-black properties가 유지된다.

1. black-height는 변경되지 않는다.
2. 인접해 있는 red node가 존재하지 않는다.
3. $y$가 red였으므로 root가 아니다.

기존 노드 $y$가 black이었을 경우 세 가지 문제가 발생할 수 있다. 첫 번째로 $y$가 root이고 $y$의 red child가 새로운 root가 될 수 있다(property 2). 두 번째로 $x$와 새로운 parent가 red일 수 있다(property 4). 세 번째로 $y$를 옮기는 것은 기존에 $y$를 포함했던 simple path의 black node를 1개 줄일 수 있다(property 5). 이 경우 property 5는 $y$의 모든 ancestor들이 위반하게 된다. 이 문제를 해결하기 위해 black node $y$가 제거되거나 옮겨질 때 $y$의 blackness가 $y$의 원래 자리로 옮겨지는 노드 $x$로 전달되어 $x$가 "추가적인" black을 갖도록 할 수도 있다. $x$를 포함하는 simple path의 black node count에 대해 +1을 더 해주는 것이다. 이 경우 property 5를 만족한다고 할 수 있다. 하지만 다른 문제가 발생한다. 노드 $x$는 red도 아니고 black도 아니므로 property 1을 위반한다. 대신에 노드 $x$는 "doubly black"이거나 "red-and-black"이 되며 각각 2, 1의 black count가 된다. $x$의 color attribute는 여전히 `RED`(red-and-black일 경우)나 `BLACK`(double black일 경우)이다.

`RB-DELETE-FIXUP`은 property 1, 2, 4를 복원한다. 아래 코드에서 **`while`** 루프의 목표는 extra black을 tree 위로 옮기는 것이다. 다음을 만족할 때까지

1. $x$가 red-and-black 노드를 가리킨다. 라인 44에서 $x$가 black이 되도록 한다.
2. $x$가 root를 가리킨다. extra black은 사라진다.
3. 적절한 rotation과 recoloring을 수행한다.

**`while`** 루프 안에서 $x$는 항상 nonroot double black node를 가리킨다. 라인 2에서 $x$가 parent $x.p$의 왼쪽 child인지 오른쪽 child인지 확인한다. $x$의 sibling은 항상 $w$로 표기한다. $x$가 항상 doubly black이므로 노드 $w$는 $T.nil$이 될 수 없다(그러면 $x.p$에서의 black height가 양쪽이 달라지기 때문). 

```text
RB-DELETE-FIXUP(T,x)
 1  while x != T.root and x.color == BLACK
 2    if x == x.p.left      // x가 left child인 경우
 3      w = x.p.right       // w는 x의 sibling
 4       if w.color == RED
 5        w.color = BLACK   // Case 1
 6        x.p.color = RED
 7        LEFT-ROTATE(T,x.p)
 8        w = x.p.right
 9      if w.left.color == BLACK and w.right.color == BLACK
10        w.color = RED     // Case 2
11        x = x.p
12      else
13        if w.right.color == BLACK
14          w.left.color = BLACK  // Case 3
15          w.color = RED
16          RIGHT-ROTATE(T,w)
17          w = x.p.right
18        w.color = x.p.color     // Case 4
19        x.p.color = BLACK
20        w.right.color = BLACK
21        LEFT-ROTATE(T,x.p)
22        x = T.root
23    else    // 라인 3-22와 좌우만 반전
24      w = x.p.left
25      if w.color == RED
26        w.color = BLACK
27        x.p.color = RED
28        RIGHT-ROTATE(T,x.p)
29        w = x.p.left
30      if w.right.color == BLACK and w.left.color == BLACK
31        w.color = RED
32        x = x.p
33      else
34        if w.left.color == BLACK
35          w.right.color = BLACK
36          w.color = RED
37          LEFT-ROTATE(T,w)
38          w = x.p.left
39        w.color = x.p.color
40        x.p.color = BLACK
41        w.left.color = BLACK
42        RIGHT-ROTATE(T,x.p)
43        x = T.root
44  x.color = BLACK
```

![2023-05-29-rb-tree-delete-fixup.png]({{site.baseurl}}/assets/images/2023-05-29-rb-tree-delete-fixup.png){: .align-center}  

### Case 1: $x$의 sibling $w$가 red일 때

Case 1(라인 5-8)은 노드 $x$의 sibling $w$가 red인 경우이다. $w$가 red이기 때문에 black인 children을 가지고 있다. 이 경우 $w$와 $x.p$의 색깔을 서로 바꾸고 $x.p$에 대해 left-rotation을 수행하면 red-black property를 위반하지 않게 된다. $x$의 새로운 sibling은 rotation 전의 $w$의 child이다. Case 1은 Case 2, 3, 4 중 하나로 변경된다. Case 2, 3, 4는 노드 $w$가 black일 경우이며, $w$의 chilren 색깔에 따라 구분된다.

### Case 2: $x$의 sibling $w$가 black이고 $w$ children이 모두 black일 때

Case 2(라인 10-11)은 $w$와 $w$의 두 child가 모두 black일 경우이다. $w$ 역시 black이기 때문에 이 경우 $x$를 일반 black으로 두고 $w$를 red로 만들면 $x$와 $w$에서 모두 하나의 black을 제거할 수 있다. $x$와 $w$에서 1 black을 잃는 것을 보상하기 위해 $x.p$가 extra black을 갖는다. 라인 11에서 $x$를 1 레벨 위로 옮기는 것을 확인할 수 있다. 이로 인해 `while` 루프는 $x.p$를 새로운 $x$로 하여 반복한다. 만약 Case 2가 Case 1 후 수행되었다면 새로운 $x$는 red-and-black 노드가 되고 color 속성이 `RED`이므로 루프가 종료된다.

### Case 3: $x$의 sibling $w$가 black이고 $w$의 left child가 red, right child가 black일 때

Case 3(라인 14-17)은 $w$가 black이고, $w$의 left child가 red, right child가 black인 경우이다. 이 경우 $w$의 color와 $w.left$의 color를 서로 바꾸고 $w$에 대해 rotation을 수행한다. 이는 red-black property를 위반하지 않는다. 결과적으로 새로운 $w$는 red right child를 가지는 black 노드가 되며, 이는 Case 4에서 처리된다.

### Case 4: $x$의 sibling $w$가 black이고 $w$의 right child가 red일 때

Case 4(라인 18-22)는 $w$가 black이고, $w$의 right child가 red인 경우이다. color change와 $x.p$에서 left rotation을 통해 $x$의 extra black이 사라지고 singly black으로 만들며 red-black property도 위반하지 않게 된다. 라인 22에서 $x$에 루트를 넣어주어 `while` 루프가 terminate되도록 만든다.  
