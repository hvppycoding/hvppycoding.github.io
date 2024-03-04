---
title: "Qualifying Examination - Programming"
excerpt: "Programming"
date: 2024-03-04 10:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - SNU
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

