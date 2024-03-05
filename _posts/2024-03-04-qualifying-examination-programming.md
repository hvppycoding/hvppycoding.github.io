---
title: "Qualifying Examination - Programming"
excerpt: "Programming"
date: 2024-03-04 10:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - C++
classes: wide
mathjax: "true"
---

## 2020-1

1. (25) Which of the four statements below have the same semantics as the following statement? Choose all of them.  

```cpp
while (--counter >= 1)
  printf("%s\n", counter % 2 ? "even" : "odd");
```

(a)

```cpp
while (--counter >= 1)
  if (counter % 2)
    printf("even");
  else
    printf("odd");
```

(b)  

```cpp
while (counter >= 1)
  if (counter % 2)
    printf("even");
  else
    printf("odd");
  --counter;
```

(c)

```cpp
while (counter >= 1) {
  if (counter % 2)
    printf("even");
  else
    printf("odd");
  --counter;
}
```

(d)

```cpp
do {
  printf("%s\n", counter % 2 ? "odd" : "even");
  --counter;
} while (counter >= 2);
```

----------

2\. (25) What is the output of the following code?

```cpp
#include <stdio.h>

void Func(int k, int m, int e, int *s, double *a) {
  *s = k + m + e;
  *a = *s/3.0;
}

int main(void) {
  int k = 80;
  int m = 80;
  int e = 95;
  int s = 0;
  double a = 0;
  Func(k, m, e, &s, &a);
  printf("s=%d, a=%lf", s, a);
  return 0;
}
```

----------

3\. (30) Make a C program that prints all unique elements in a given array A. Use the following skeleton code. If you want, you can add another function other than the main function.  

```cpp
#include <stdio.h>

int main(void) {
  int A = {4, 2, 1, 5, 3, 4, 1, 1, 8, 6};
  ...
}
```

----------

4\. (20) Let A be a double type variable of a C program. Write the value of A for each of the following lines, and explain reasons for their results.  

(a) (10) A=7/2.0/2;  

(b) (10) A=7/2/2.0;  

----------

## 2020-2

1\. (20) Write the output of the following C program.  

```cpp
#include <stdio.h>

void foo(int x, int *y, int *z, int w)
{
  x++;
  *y = *y + 1;
  (*z) += 1;
  w--;
}

int main()
{
  int A[] = {0, 1, 2, 3};
  foo(A[0], &A[1], A+2, *(A+3));
  printf("%d, %d, %d, %d", A[0], A[1], A[2], A[3]);
}
```

----------

2\. (25) For a given integer array `A[]`, whose size is `M` and whose element values range between `0` and `N-1`, the following program computes `C[i]` (`0 <= i <= N-1`), the number of elements in `A[]` which are **less than or equal to** `i` (for our example `A[]`, the **printf** statement will print `C[0]=1 C[1]=2 C[2]=4 C[3]=6 ... C[8]=13 C[9]=15`; e.g., `C[2]=4` since there are four elements, `A[2]`, `A[3]`, `A[9]`, aand `A[11]`, which are <= 2)  

```cpp
#include <stdio.h>
#define M 15
#define N 10

int main() 
{
  int A[M] = {3, 5, 2, 1, 5, 6, 9, 3, 7, 0, 5, 2, 4, 9, 7};
  int C[N] = {0, 0, 0, 0, 0, 0, 0 ,0, 0, 0};
  int B[M], i;

  for ( i=0; i<M; i++) { (a) }  // After this, C[i] has num of elements in A[] equal to i
  for ( i=1; i<N; i++) { (b) }  // After this, C[i] has num of elements in A[] <= i
  for ( i=0; i<N; i++) { printf("C[%d] = %d\n", i, C[i])};
  /* HERE */
}
```

(1) Write a single C statement that should be in (a) and single C statement that should be in (b), for correct computation of `C[]` (Hint: check the comments; note that even if your answer to (a) is incorrect, you can answer (b) independently).  

(2) If we add the following statement at `/* HERE */` in the above, what will it do?

```cpp
for ( i=0; i<M; i++) { B[--C[A[i]]] = A[i]; }
```

----------

3\. Answer for the following C++ program.  

```cpp
#include <iostream>
using namespace std;

class A {
public:
  virtual void foo() { cout << "A_foo()" << endl; } // same as printf("A_foo()\n": in C
  virtual void boo() { cout << "A_boo()" << endl; }
}

class B: public A {
public:
  virtual void boo() { cout << "B_boo()" << endl; }
  virtual void zoo() { cout << "B_zoo()" << endl; }
}

int main()
{
  B b;
  b.foo(); b.zoo();
  A *a = &b;
  a->foo(); a1->boo(); /* HERE */
}
```

(a) Show the output of the above program (there are four lines).  

(b) For the output, which output line(s) correspond to case where **polymorphism**, **overriding**, and **virtual function call** occur **all together**? Explain briefly.  

(c) If we add a statement **`a1->zoo();`** at **`/* HERE */`** in the above, can it be compiled correctly? If so, can it run correctly? If so, what is its output?" Expalin briefly.  

----------

4\. (40) Answer the following questions in *two lines*(penalty if more than two lines)  

(a) **Pointers** are tricky to use. Why do we need pointers in programming? That is, what are pointers *mainly* used in C/C++ programming.  

(b) We can implement a **Set** using an array or linked list. What is the advantage of linked list compared to array? What is the disadvantage?  

----------
