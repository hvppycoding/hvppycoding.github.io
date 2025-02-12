---
title: "Ch 2. 허깅페이스 실습 - Autoregressive Models"
excerpt: ""
date: 2025-02-12 01:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Hugging Face
---

## GPT2

GPT2 모델을 이해하기 위해서는 [GPT2 논문]({{site.baseurl}}/assets/images/2025-02-12-hugging-face-02/GPT2_paper.pdf)을 꼭 읽어보자.

우리는 Hugging Face 모델을 사용하여 GPT2를 사용할 것이다.  

```bash
pip install -q git+https://github.com/huggingface/transformers.git
pip install -q tensorflow
```

```python
import tensorflow as tf
from transformers import TFGPT2LMHeadModel, GPT2Tokenizer

tokenizer = GPT2Tokenizer.from_pretrained("gpt2")

# add the EOS token as PAD token to avoid warnings
model = TFGPT2LMHeadModel.from_pretrained("gpt2", pad_token_id=tokenizer.eos_token_id)

print(tokenizer.eos_token_id)

# encode context the generation is conditioned on
input_ids = tokenizer.encode('My name is Hasnain and I am a software engineer.', return_tensors='tf')

# generate text until the output length (which includes the context length) reaches 50
greedy_output = model.generate(input_ids, max_length=50)

print("Output:\n")
print(tokenizer.decode(greedy_output[0]))
# Output:
# My name is Hasnain and I am a software engineer. I am a member of the Open Source 
# Software Foundation. I am a member of the Open Source Software Foundation. I am a
# member of the Open Source Software Foundation. I am a member

# set no_repeat_ngram_size to 2
beam_output = model.generate(
    input_ids, 
    max_length=50, 
    num_beams=5, 
    no_repeat_ngram_size=2, 
    early_stopping=True
)

print("Output:\n")
print(tokenizer.decode(beam_output[0], skip_special_tokens=True))
# Output:
# My name is Hasnain and I am a software engineer. I have been working in the IT 
# industry for over 20 years. 
# I have a passion for software development and have worked on many projects over 
# the past few years, including:
```

greedy 방식은 한 단어씩 가장 확률이 높은 단어를 선택하는 방식이다.  
반면에 beam 방식은 여러 단어 조합을 본 후, 가장 확률이 높은 문장을 선택하는 방식이다.  

## Transformer-XL

Transformer-XL은 구글에서 출시한 매우 유명한 트랜스포머 중 하나이다.

논문 링크: [Transformer-XL 논문]({{site.baseurl}}/assets/images/2025-02-12-hugging-face-02/Transformer-XL_paper.pdf)

기존 GPT2, BERT의 문제점은 고정된 길이의 Context(프롬프트)가 있다는 한계가 있다.  
Transformer-XL은 이 한계를 극복하며, 훨씬 더 긴 Context를 다룰 수 있다.

이 챕터에서는 Transformer-XL을 사용하여 레이블 분류를 수행해볼 것이다.  
Classification에는 Multi Label과 Single Label이 있다.

첫 예시는 Single Label Classification이고, 두 번째 예시는 Multi Label Classification이다.

```bash
pip install torch
pip install sacremoses
```

```python
import torch
from transformers import AutoTokenizer, TransfoXLForSequenceClassification

tokenizer = AutoTokenizer.from_pretrained("transfo-xl-wt103")
model = TransfoXLForSequenceClassification.from_pretrained("transfo-xl-wt103")

input_str = "My name is Hasnain"

inputs = tokenizer(input_str, return_tensors="pt")
print("inputs:", inputs)
# inputs: {'input_ids': tensor([[1162,  237,   23,   24]])}

with torch.no_grad():
    logits = model(**inputs).logits

predicted_class_id = logits.argmax().item()
print("predicted_class_id:", predicted_class_id)
# predicted_class_id: 0
print("model.config.id2label:", model.config.id2label)
# model.config.id2label: {0: 'LABEL_0', 1: 'LABEL_1'}

# To train a model on `num_labels` classes, you can pass `num_labels=num_labels` to `.from_pretrained(...)`
num_labels = len(model.config.id2label)
model = TransfoXLForSequenceClassification.from_pretrained("transfo-xl-wt103", num_labels=num_labels)

labels = torch.tensor([1])
loss = model(**inputs, labels=labels).loss

input_str = "I was going with my dog"

inputs = tokenizer(input_str, return_tensors="pt")
with torch.no_grad():
    logits = model(**inputs).logits

predicted_class_ids = torch.arange(0, logits.shape[-1])[torch.sigmoid(logits).squeeze(dim=0) > 0.5]
print("predicted_class_ids:", predicted_class_ids)

input_str = "I was going with my dog"

inputs = tokenizer(input_str, return_tensors="pt")
num_labels = len(model.config.id2label)
model = TransfoXLForSequenceClassification.from_pretrained("transfo-xl-wt103", num_labels=num_labels, problem_type="multi_label_classification")

labels = torch.sum(torch.nn.functional.one_hot(predicted_class_ids[None, :].clone(), num_classes=num_labels), dim=1).to(torch.float)
loss = model(**inputs, labels=labels).loss
```