---
title: "Optimization with Python 01 - Introduction"
excerpt: "Optimization with Python 01 - Introduction"
date: 2022-02-20 23:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Optimization
mathjax: "true"
---
**Notice:** 본 글은 Udemy의 Optimization with Python: Solve Operations Research Problems을 학습하며 정리한 글입니다.
{: .notice--info}

**Warning Notice:** 본 강의는 파이썬을 이용하여 최적화 문제를 푸는 것에 집중합니다. 파이썬 언어의 기초는 다루지 않으므로 주의하십시오.
{: .notice--warning}

# Introduction
## Outlines
- Python 설치
- Python Package 설치
- Linear Programming(LP)
- Mixed-Integer Linear Programming(MILP)
- Nonelinear Programming(NLP)
- Mixed-Integer Nonlinear Programming(MINLP)
- Heuristics(GA and Particle Swarm)
- Constraint Programming
- Practical and good examples

## What is Optimization
- 최적 결정을 위한 탐색
- Planning 관련 모든 문제(long term, medium term, short term, ...)
- 투자를 위한 결정, 연산, 경로 최적화, 비용 최적화 등
- Ex. 두 종류의 제품($x$와 $y$)를 팔아서 이익을 최대화하고 싶다. x와 y 모두 1개 판매 시 1달러를 번다고 하자.
  - 추가적으로 아래의 조건(Constraints)을 만족해야 한다.
  - $2y \leq x+8$ (time of production)
  - $2x + y \leq 14$ (raw material)
  - $2x \leq y+10$ (historical sales)
  - $x, y \leq 10$ (maximum daily production)

- 최적화 문제 풀이의 단계
  1. 문제 이해
  2. 문제 모델링: 문제를 수학적으로 모델링해야 한다.
  3. 문제 해결: 주로 컴퓨터를 이용하여 해결한다.
  4. 결과
  - 본 강의에서는 "3. 문제 해결" 단계에 집중한다. 가장 실용적인 부분이므로 이 단계를 먼저 학습한 후 모델링 연습을 하자.
  - 위의 문제는 x, y간 곱셈 항이 없으므로 Linear Programming을 통해 문제를 풀 수 있다.
  - 만약 $xy$ 항이 있다면 Non-linear이다.
  - 추가적으로 x와 y는 정수이다.

## Installing Python
- 파이썬 홈페이지를 통해 설치한다.

```sh
# pip 업데이트
python -m pip install --upgrade pip

# Package 설치
# pip install <package_name>

# matplotlib 설치
pip install matplotlib

# Spyder IDE 설치
pip install spyder

# Spyder IDE 실행
spyder

# jupyterlab 설치
pip install jupyterlab

# jupyterlab 실행
python -m jupyter lab
```

## Numpy
```python
import numpy as np

ages = np.array([10, 15, 20, 18])
marks = np.array([9, 8, 5, 7])

print(marks[ages%2==0])
```

<div class="no-highlight" markdown="1">

```
    [9 5 7]
```    

</div>

```python
print(np.min(ages))
print(np.max(ages))
print(np.mean(ages))
```

<div class="no-highlight" markdown="1">

```
    10
    20
    15.75
```

</div>


```python
print(np.mean(ages[marks>7]))
```

<div class="no-highlight" markdown="1">

```
    12.5
```

</div>

```python
np.cos(1)
```

<div class="no-highlight" markdown="1">

```
    0.5403023058681398
```

</div>


```python
x = np.inf

print(x)
```

<div class="no-highlight" markdown="1">

```
    inf
```

</div>
    

## Pandas

```python
import pandas as pd

df = pd.DataFrame({
    'name': ['rafael', 'gabriel', 'rafael', 'michael'],
    'mark': [10, 9, 8, 7],
})

df
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>mark</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>rafael</td>
      <td>10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>gabriel</td>
      <td>9</td>
    </tr>
    <tr>
      <th>2</th>
      <td>rafael</td>
      <td>8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>michael</td>
      <td>7</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.mark
```

<div class="no-highlight" markdown="1">

```
    0    10
    1     9
    2     8
    3     7
    Name: mark, dtype: int64
```

</div>



```python
df.mark[2]
```

<div class="no-highlight" markdown="1">

```
    8
```

</div>

```python
df['mark']
```

<div class="no-highlight" markdown="1">

```
    0    10
    1     9
    2     8
    3     7
    Name: mark, dtype: int64
```

</div>

```python
df.loc[2, 'mark']
```

<div class="no-highlight" markdown="1">

```
    8
```

</div>

```python
df.iloc[2, 1]
```

<div class="no-highlight" markdown="1">

```
    8
```

</div>

```python
df.T
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>name</th>
      <td>rafael</td>
      <td>gabriel</td>
      <td>rafael</td>
      <td>michael</td>
    </tr>
    <tr>
      <th>mark</th>
      <td>10</td>
      <td>9</td>
      <td>8</td>
      <td>7</td>
    </tr>
  </tbody>
</table>
</div>

```python
df.T.loc['mark']
```

<div class="no-highlight" markdown="1">

```
    0    10
    1     9
    2     8
    3     7
    Name: mark, dtype: object
```

</div>

```python
df.groupby('name').mean()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mark</th>
    </tr>
    <tr>
      <th>name</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>gabriel</th>
      <td>9.0</td>
    </tr>
    <tr>
      <th>michael</th>
      <td>7.0</td>
    </tr>
    <tr>
      <th>rafael</th>
      <td>9.0</td>
    </tr>
  </tbody>
</table>
</div>

```python
df.groupby('name').count()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mark</th>
    </tr>
    <tr>
      <th>name</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>gabriel</th>
      <td>1</td>
    </tr>
    <tr>
      <th>michael</th>
      <td>1</td>
    </tr>
    <tr>
      <th>rafael</th>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>

```python
df.mark[df.name=='rafael']
```

<div class="no-highlight" markdown="1">

```
    0    10
    2     8
    Name: mark, dtype: int64
```

</div>

```python
df.mark[df.name=='rafael'].mean()
```

<div class="no-highlight" markdown="1">

```
    9.0
```

</div>


```python
df.mark[df.name=='rafael'].max()
```

<div class="no-highlight" markdown="1">

```
    10
```

</div>

## Pandas: Reading Excel

```python
import pandas as pd

pd.read_excel('03_Pandas_reading_Excel.xlsx', sheet_name='people')
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>age</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>rafael</td>
      <td>15</td>
    </tr>
    <tr>
      <th>1</th>
      <td>gabriel</td>
      <td>10</td>
    </tr>
  </tbody>
</table>
</div>

```python
pd.read_excel('03_Pandas_reading_Excel.xlsx', sheet_name='marks')
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>test</th>
      <th>mark</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>rafael</td>
      <td>1</td>
      <td>7</td>
    </tr>
    <tr>
      <th>1</th>
      <td>gabriel</td>
      <td>1</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>rafael</td>
      <td>2</td>
      <td>8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>gabriel</td>
      <td>2</td>
      <td>9</td>
    </tr>
  </tbody>
</table>
</div>


```python
dataMarks = pd.read_excel('03_Pandas_reading_Excel.xlsx', sheet_name='marks')
dataPeople = pd.read_excel('03_Pandas_reading_Excel.xlsx', sheet_name='people')
dataAll = dataMarks.set_index('name').join(dataPeople.set_index('name'))
dataAll
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>test</th>
      <th>mark</th>
      <th>age</th>
    </tr>
    <tr>
      <th>name</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>gabriel</th>
      <td>1</td>
      <td>4</td>
      <td>10</td>
    </tr>
    <tr>
      <th>gabriel</th>
      <td>2</td>
      <td>9</td>
      <td>10</td>
    </tr>
    <tr>
      <th>rafael</th>
      <td>1</td>
      <td>7</td>
      <td>15</td>
    </tr>
    <tr>
      <th>rafael</th>
      <td>2</td>
      <td>8</td>
      <td>15</td>
    </tr>
  </tbody>
</table>
</div>

```python
marks = dataAll.groupby('name').mark.mean()
marks
```

<div class="no-highlight" markdown="1">

```
    name
    gabriel    6.5
    rafael     7.5
    Name: mark, dtype: float64
```
</div>

```python
marks.to_excel('03_Pandas_reading_Excel_output.xlsx')
```

## Matplotlib 

```python
import matplotlib.pyplot as plt

names = ['A', 'B', 'C', 'D']
salaries = [100, 150, 120, 170]

plt.plot(names, salaries, color='red')
```

<div class="no-highlight" markdown="1">

```
    [<matplotlib.lines.Line2D at 0x21eaf45c6c8>]
```

</div>


![plot]({{site.baseurl}}/assets/images/2022-02-21-optimization-with-python-01-01.png)
    



```python
plt.bar(names, salaries, color='green')
```

<div class="no-highlight" markdown="1">

```
    <BarContainer object of 4 artists>
```

</div>

![plot]({{site.baseurl}}/assets/images/2022-02-21-optimization-with-python-01-02.png)
    