---
title: "Hugging Face - Dataset[Vision]"
excerpt: ""
date: 2025-02-15 12:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Stable Diffusion
---

## 이미지 데이터셋 만들기

캡션을 포함한 이미지 데이터셋을 만드는 방법을 알아보자.

필요한 라이브러리를 설치한다.

```bash
pip install datasets[vision]
```

아래와 같이 폴더 구성한다.

```text
dataset_load
├── dataset_load.py
└── data
    ├── metadata.csv
    ├── 001.png
    ├── 002.png
    ...
```

`metadata.csv`의 내용은 다음과 같다.

```csv
file_name,caption
001.png,white hair
002.png,black hair
003.png,ivory hair
004.png,purple hair
005.png,yellow hair
```

```python
from datasets import load_dataset

dataset = load_dataset("./data")
print(dataset)
# Generating train split: 5 examples [00:00, 694.77 examples/s]
# DatasetDict({
#     train: Dataset({
#         features: ['image', 'caption'],
#         num_rows: 5
#     })
# })
dataset["train"][0]["image"].show()
```
