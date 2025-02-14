---
title: "Diffusers Quicktour"
excerpt: ""
date: 2025-02-13 03:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Diffusers
---

참고한 문서: [Diffusers Quicktour](https://huggingface.co/docs/diffusers/quicktour)

## Diffusers 개요

diffusers는 이미지, 오디오 및 분자의 3D 구조까지도 생성할 수 있는 최첨단 사전 훈련 모델을 위한 라이브러리입니다. 간단한 추론 솔루션, 자신만의 디퓨젼 모델 훈련도 지원합니다.

### 라이브러리의 세 가지 구성 요소

- Pipelines: 몇 줄의 코드로 inference 수행 가능한 State-of-the-art diffusion pipelines
- 생성 속도와 품질 간의 trade-off를 조정할 수 있게 해주는 noise schedulers
- Pretrained models는 scheduler와 결합되어 자신만의 end-to-end diffusion system을 만들 수 있음

## Diffuser Quicktour

Diffusion 모델은 이미지나 오디오 등의 관심 샘플<sup>a sample of interest</sup>을 생성하기 위해 랜덤 가우시안 노이즈를 step-by-step으로 denois하도록 훈련된 모델이다.  

이 Quicktour에서는 추론을 위해 DiffusionPipeline을 사용하는 방법을 보여줄 것이다. 그리고 model과 scheduler를 결합하며 DiffusionPipeline 내부에서 동작하는 방식을 흉내낼 것이다.  

본 Quicktour는 Diffuser [노트북](https://colab.research.google.com/github/huggingface/notebooks/blob/main/diffusers/diffusers_intro.ipynb)의 단순화 버젼으로, 더 자세히 알아보고 싶다면 해당 노트북을 참고하자.
{: .notice--info}

시작하기 전에 필요한 라이브러리들이 설치되어 있음을 확인하자.

```bash
pip install --upgrade diffusers accelerate transformers
```

- Accelerate는 inference와 training 모델 로딩을 빠르게 해주는 라이브러리
- Transformers는 여러 인기있는 diffusion 모델들을 돌리기 위해 필요한 라이브러리

### DiffusionPipeline

DiffusionPipeline은 inference를 위한 pretrained diffusion 모델을 쉽게 사용할 수 있게 해준다. 이는 model과 scheduler를 포함하는 end-to-end system이다. 많은 작업에 대해 DiffusionPipeline을 사용할 수 있다. 예를 들어 다음과 같은 Task들이 있다.

- Unconditional Image Generation: 가우시안 노이즈로부터 이미지 생성
- Text-Guided Image  Generation: 텍스트 프롬프트가 주어진 이미지 생성
- Text-Guided Image-to-Image Translation: 텍스트 프롬프트가 주어진 이미지 변환
- Text-Guided Image-Inpainting: 주어진 이미지에서 마스키된 부분을 프롬프트에 따라 채우기
- Text-Guided Depth-to-Image Translation: 깊이 추정의 구조를 유지하면서 이미지 변환

```python
from diffusers import DiffusionPipeline

pipeline = DiffusionPipeline.from_pretrained("stable-diffusion-v1-5/stable-diffusion-v1-5", use_safetensors=True)
```

DiffusionPipeline은 모든 modeling, tokenization, scheduling 컴퍼넌트들을 다운로드하고 캐싱한다. Stable Diffusion pipeline은 UNet2DConditionModel과 PNDMScheduler로 구성된 것을 확인할 수 있다.

```python
print(pipeline)
# StableDiffusionPipeline {
#   "_class_name": "StableDiffusionPipeline",
#   "_diffusers_version": "0.32.2",
#   "_name_or_path": "stable-diffusion-v1-5/stable-diffusion-v1-5",
#   "feature_extractor": [
#     "transformers",
#     "CLIPImageProcessor"
#   ],
#   "image_encoder": [
#     null,
#     null
#   ],
#   "requires_safety_checker": true,
#   "safety_checker": [
#     "stable_diffusion",
#     "StableDiffusionSafetyChecker"
#   ],
#   "scheduler": [
#     "diffusers",
#     "PNDMScheduler"
#   ],
#   "text_encoder": [
#     "transformers",
#     "CLIPTextModel"
#   ],
#   "tokenizer": [
#     "transformers",
#     "CLIPTokenizer"
#   ],
#   "unet": [
#     "diffusers",
#     "UNet2DConditionModel"
#   ],
#   "vae": [
#     "diffusers",
#     "AutoencoderKL"
#   ]
# }
```

모델은 약 14억 개의 파라미터로 구성되어 있기 때문에 GPU에서 파이프라인을 실행하는 것이 좋다. 

```python
pipeline.to("cuda")
```

이제 텍스트 프롬프트를 `pipeline`에 전달하여 이미지를 생성할 수 있다. 기본적으로 출력 이미지는 `PIL.Image` 객체로 래핑된다.

```python
image = pipeline("An image of a squirrel in Picasso style").images[0]
image.show()
```

![squirrel_picasso]({{site.baseurl}}/assets/images/2025-02-13-diffusers-quicktour/squirrel_picasso.png){: .align-center}  

`save` 메서드를 사용하여 이미지를 저장할 수 있다.

```python
image.save("squirrel_picasso.jpg")
```

#### Local pipeline

로컬의 파이프라인도 가능할 수 있다. 먼저 사용할 가중치를 다운받아야 한다.

```bash
git clone https://huggingface.co/stable-diffusion-v1-5/stable-diffusion-v1-5
```

저장된 weights를 로드

```python
pipeline = DiffusionPipeline.from_pretrained("./stable-diffusion-v1-5", use_safetensors=True)
```

#### Swapping schedulers

여러 스케줄러는 denosing 속도와 quality 사이 trade-off가 존재한다. 어떤 스케줄러가 좋을지는 시도해서 알아낸 것이 좋다. Diffusers는 쉽게 scheduler를 바꿀 수 있게 해준다. 예를 들어, 기본 `PNDMScheduler`를 `EulerDiscreteScheduler`로 바꾸려면 `from_config()` 메서드를 이용하여 로드할 수 있다.

```python
from diffusers import EulerDiscreteScheduler

pipeline = DiffusionPipeline.from_pretrained("stable-diffusion-v1-5/stable-diffusion-v1-5", use_safetensors=True)
pipeline.scheduler = EulerDiscreteScheduler.from_config(pipeline.scheduler.config)
```

새 스케줄러로 이미지 생성하고 차이가 있는지 확인하자.

다음 섹션에서는 model과 scheduler 컴퍼넌트를 자세히 살펴볼 것이다. DiffusionPipeline을 구성하여 고양이 이미지를 생성해보자.

### Models

대부분의 모델들은 noisy sample을 받아, 각 timestep마다 *noise residual*(noisy image와 input image 간의 차이)을 예측한다.  

퀵투어에서는 고양이 이미지에 대해 훈련된 UNet2DModel 모델을 로드할 것이다.

```python
from diffusers import UNet2DModel

repo_id = "google/ddpm-cat-256"
model = UNet2DModel.from_pretrained(repo_id, use_safetensors=True)

print(model.config)
# FrozenDict({'sample_size': 256, 'in_channels': 3, 'out_channels': 3, 'center_input_sample': False, 'time_embedding_type': 'positional', 'time_embedding_dim': None, 'freq_shift': 1, 'flip_sin_to_cos': False, 'down_block_types': ['DownBlock2D', 'DownBlock2D', 'DownBlock2D', 'DownBlock2D', 'AttnDownBlock2D', 'DownBlock2D'], 'up_block_types': ['UpBlock2D', 'AttnUpBlock2D', 'UpBlock2D', 'UpBlock2D', 'UpBlock2D', 'UpBlock2D'], 'block_out_channels': [128, 128, 256, 256, 512, 512], 'layers_per_block': 2, 'mid_block_scale_factor': 1, 'downsample_padding': 0, 'downsample_type': 'conv', 'upsample_type': 'conv', 'dropout': 0.0, 'act_fn': 'silu', 'attention_head_dim': None, 'norm_num_groups': 32, 'attn_norm_num_groups': None, 'norm_eps': 1e-06, 'resnet_time_scale_shift': 'default', 'add_attention': True, 'class_embed_type': None, 'num_class_embeds': None, 'num_train_timesteps': None, '_use_default_values': ['attn_norm_num_groups', 'downsample_type', 'num_train_timesteps', 'resnet_time_scale_shift', 'upsample_type', 'dropout', 'time_embedding_dim', 'add_attention', 'num_class_embeds', 'class_embed_type'], '_class_name': 'UNet2DModel', '_diffusers_version': '0.0.4', '_name_or_path': 'google/ddpm-cat-256'})
```

모델 구성은 고정된 딕셔너리로, 모델이 생성된 후에는 변수들을 변경할 수 없다.  

중요한 파라미터들로는  

- `sample_size`: 이미지의 크기
- `in_channels`: 입력 샘플의 입력 채널 수
- `down_block_types`, `up_block_types`: UNet 아키텍처를 생성하는데 사용되는 다운 및 업샘플링 블록의 유형
- `block_out_channels`: 다운샘플링 블록의 출력 채널 수. 업샘플링 블록의 입력 채널 수에 역순으로 사용되기도 한다.
- `layer_per_block`: 각 UNet 블록에 존재하는 ResNet 블록의 수

추론에 모델을 사용하려면 랜덤 가우시안 노이즈로 이미지 모양을 만듭니다. 모델이 여러 개의 무작위 노이즈를 수신할 수 있으므로 `batch` 축, 입력 채널 수에 해당하는 `channel` 축, 이미지의 높이와 너비를 나타내는 `sample_size` 축이 있어야 합니다:

```python
import torch

torch.manual_seed(0)

noisy_sample = torch.randn(1, model.config.in_channels, model.config.sample_size, model.config.sample_size)
noisy_sample.shape
```

추론을 위해 모델에 노이즈가 있는 이미지와 timestep을 전달합니다. ‘timestep’은 입력 이미지의 노이즈 정도를 나타내며, 시작 부분에 더 많은 노이즈가 있고 끝 부분에 더 적은 노이즈가 있습니다. 이를 통해 모델이 diffusion 과정에서 시작 또는 끝에 더 가까운 위치를 결정할 수 있습니다. sample 메서드를 사용하여 모델 출력을 얻습니다:

```python
with torch.no_grad():
    noisy_residual = model(sample=noisy_sample, timestep=2).sample
```

하지만 실제 예시를 생성하려면 노이즈 제거 프로세스를 안내할 스케줄러가 필요합니다. 다음 섹션에서는 모델을 스케줄러와 결합하는 방법에 대해 알아봅니다.  

### Schedulers

스케줄러는 모델 출력이 주어졌을 때 노이즈가 많은 샘플에서 노이즈가 적은 샘플로 변환하는 것을 관리합니다 - 이 경우 `noisy_residual`.

Diffusers는 Diffusion 시스템을 구축하기 위한 툴박스입니다. DiffusionPipeline을 사용하면 미리 만들어진 Diffusion 시스템을 편리하게 시작할 수 있지만, 모델과 스케줄러 구성 요소를 개별적으로 선택하여 사용자 지정 Diffusion 시스템을 구축할 수도 있습니다.
{: .notice--info}

여기서는 `from_config()` 메서드를 사용하여 `DDPMScheduler`를 인스턴스화합니다.

```python
from diffusers import DDPMScheduler

scheduler = DDPMScheduler.from_config(repo_id)
print(scheduler)
# DDPMScheduler {
#   "_class_name": "DDPMScheduler",
#   "_diffusers_version": "0.13.1",
#   "beta_end": 0.02,
#   "beta_schedule": "linear",
#   "beta_start": 0.0001,
#   "clip_sample": true,
#   "clip_sample_range": 1.0,
#   "num_train_timesteps": 1000,
#   "prediction_type": "epsilon",
#   "trained_betas": null,
#   "variance_type": "fixed_small"
# }
```

모델과 달리 스케줄러에는 학습 가능한 가중치가 없으며 매개변수도 없습니다!
{: .notice--info}

가장 중요한 매개변수는 다음과 같습니다:

- `num_train_timesteps`: 노이즈 제거 프로세스의 길이, 즉 랜덤 가우스 노이즈를 데이터 샘플로 처리하는 데 필요한 타임스텝 수입니다.
- `beta_schedule`: 추론 및 학습에 사용할 노이즈 스케줄 유형입니다.
- `beta_start` 및 `beta_end`: 노이즈 스케줄의 시작 및 종료 노이즈 값입니다.

노이즈가 약간 적은 이미지를 예측하려면 스케줄러의 `step()` 메서드에 모델 출력, `timestep`, 현재 `sample`을 전달하세요.  

```python
less_noisy_sample = scheduler.step(model_output=noisy_residual, timestep=2, sample=noisy_sample).prev_sample
print(less_noisy_sample.shape)
```

`less_noisy_sample`을 다음 `timestep`으로 넘기면 노이즈가 더 줄어듭니다! 이제 이 모든 것을 한데 모아 전체 노이즈 제거 과정을 시각화해 보겠습니다.

먼저 노이즈 제거된 이미지를 후처리하여 `PIL.Image`로 표시하는 함수를 만듭니다:

```python
import PIL.Image
import numpy as np


def display_sample(sample, i):
    image_processed = sample.cpu().permute(0, 2, 3, 1)
    image_processed = (image_processed + 1.0) * 127.5
    image_processed = image_processed.numpy().astype(np.uint8)

    image_pil = PIL.Image.fromarray(image_processed[0])
    display(f"Image at step {i}")
    display(image_pil)
```

노이즈 제거 프로세스의 속도를 높이려면 입력과 모델을 GPU로 옮기세요:

```python
model.to("cuda")
noisy_sample = noisy_sample.to("cuda")
```

이제 노이즈가 적은 샘플의 잔차를 예측하고 스케줄러로 노이즈가 적은 샘플을 계산하는 노이즈 제거 루프를 생성합니다:

```python
import tqdm

sample = noisy_sample

for i, t in enumerate(tqdm.tqdm(scheduler.timesteps)):
    # 1. predict noise residual
    with torch.no_grad():
        residual = model(sample, t).sample

    # 2. compute less noisy image and set x_t -> x_t-1
    sample = scheduler.step(residual, t, sample).prev_sample

    # 3. optionally look at image
    if (i + 1) % 50 == 0:
        display_sample(sample, i + 1)
```

![cat_steps]({{site.baseurl}}/assets/images/2025-02-13-diffusers-quicktour/cat_steps.png){: .align-center}  

### 사용한 전체 코드

```python
from diffusers import UNet2DModel

repo_id = "google/ddpm-cat-256"
model = UNet2DModel.from_pretrained(repo_id, use_safetensors=True)

print(model.config)

from diffusers import DDPMScheduler

scheduler = DDPMScheduler.from_config(repo_id)
print(scheduler)

import PIL.Image
import numpy as np

import torch

torch.manual_seed(0)

noisy_sample = torch.randn(1, model.config.in_channels, model.config.sample_size, model.config.sample_size)
noisy_sample.shape

def display_sample(sample, i):
    image_processed = sample.cpu().permute(0, 2, 3, 1)
    image_processed = (image_processed + 1.0) * 127.5
    image_processed = image_processed.numpy().astype(np.uint8)

    image_pil = PIL.Image.fromarray(image_processed[0])
    image_pil.show()
    
import tqdm

sample = noisy_sample
model.to("cuda")

for i, t in enumerate(tqdm.tqdm(scheduler.timesteps)):
    t = t.to("cuda")
    sample = sample.to("cuda")
    
    # 1. predict noise residual
    with torch.no_grad():
        residual = model(sample, t).sample

    # 2. compute less noisy image and set x_t -> x_t-1
    sample = scheduler.step(residual, t, sample).prev_sample

    # 3. optionally look at image
    if (i + 1) % 50 == 0:
        display_sample(sample, i + 1)
```
