---
title: "Diffusers Study"
excerpt: ""
date: 2025-02-13 03:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Hugging Face
---

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
{:.notice--info}

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

![squirrel_picasso]({{site.baseurl}}/assets/images/2025-02-13-diffusers-study/squirrel_picasso.png){: .align-center}  

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

