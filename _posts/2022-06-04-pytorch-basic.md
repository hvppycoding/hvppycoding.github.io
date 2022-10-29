---
title: "PyTorch Basic Operations"
excerpt: "PyTorch Basic Operations"
date: 2022-06-04 00:00:00 +0900
header:
  image: /assets/images/unsplash-emile-perron.jpg
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Python
---

# torch.repeat

```python
x = torch.tensor([1,2,3])
x.repeat(4, 2)

x = torch.tensor([1,2,3])
x.repeat(4, 2, 1)
```

<div class="no-highlight" markdown="1">

```
tensor([[1, 2, 3, 1, 2, 3],
        [1, 2, 3, 1, 2, 3],
        [1, 2, 3, 1, 2, 3],
        [1, 2, 3, 1, 2, 3]])

tensor([[[1, 2, 3],
         [1, 2, 3]],

        [[1, 2, 3],
         [1, 2, 3]],

        [[1, 2, 3],
         [1, 2, 3]],

        [[1, 2, 3],
         [1, 2, 3]]])
```

</div>

# torch.gather

`torch.gather(input, dim, index, *, sparse_grad=False, out=None) → Tensor`

```python
out[i][j][k] = input[index[i][j][k]][j][k]  # if dim == 0
out[i][j][k] = input[i][index[i][j][k]][k]  # if dim == 1
out[i][j][k] = input[i][j][index[i][j][k]]  # if dim == 2
```

```python
t = torch.tensor([[1, 2], [3, 4]])
torch.gather(t, 1, torch.tensor([[0, 0], [1, 0]]))
```

<div class="no-highlight" markdown="1">

```
tensor([[ 1,  1],
        [ 4,  3]])
```

</div>

# torch.unsqueeze

`torch.unsqueeze(input, dim) → Tensor`

`dim` 차원에 size 1이 추가된 텐서를 리턴한다.

```python
x = torch.tensor([1, 2, 3, 4])

torch.unsqueeze(x, 0)
# tensor([[ 1,  2,  3,  4]])

torch.unsqueeze(x, 1)
# tensor([[ 1],
#         [ 2],
#         [ 3],
#         [ 4]])
```

# torch.transpose

`torch.transpose(input, dim0, dim1) → Tensor`

`dim0`와 `dim1`이 swap된 텐서를 리턴한다.

```python
x = torch.randn(2, 3)
x
# tensor([[ 1.0028, -0.9893,  0.5809],
#         [-0.1669,  0.7299,  0.4942]])
torch.transpose(x, 0, 1)
# tensor([[ 1.0028, -0.1669],
#         [-0.9893,  0.7299],
#         [ 0.5809,  0.4942]])
```

# conv1d

`torch.nn.Conv1d(in_channels, out_channels, kernel_size, stride=1, padding=0, dilation=1, groups=1, bias=True, padding_mode='zeros', device=None, dtype=None)`

# torch.split

`torch.split(tensor, split_size_or_sections, dim=0)`

```python
a = torch.arange(10).reshape(5,2)
a
# tensor([[0, 1],
#         [2, 3],
#         [4, 5],
#         [6, 7],
#         [8, 9]])
torch.split(a, 2)
# (tensor([[0, 1],
#          [2, 3]]),
#  tensor([[4, 5],
#          [6, 7]]),
#  tensor([[8, 9]]))
torch.split(a, [1,4])
# (tensor([[0, 1]]),
#  tensor([[2, 3],
#          [4, 5],
#          [6, 7],
#          [8, 9]]))
```

# torch.multinomial

확률에 따라 Sampling 시도


```python
weights = torch.tensor([0, 10, 3, 0], dtype=torch.float) # create a tensor of weights
torch.multinomial(weights, 2)
#tensor([1, 2])
torch.multinomial(weights, 4) # ERROR!
#RuntimeError: invalid argument 2: invalid multinomial distribution (with replacement=False,
#not enough non-negative category to sample) at ../aten/src/TH/generic/THTensorRandom.cpp:320
torch.multinomial(weights, 4, replacement=True)
#tensor([ 2,  1,  1,  1])
```