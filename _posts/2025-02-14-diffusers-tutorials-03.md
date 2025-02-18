---
title: "Diffusers - Load LoRAs for inference"
excerpt: ""
date: 2025-02-14 08:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Stable Diffusion
---

참고한 문서: [Load LoRAs for inference](https://huggingface.co/docs/diffusers/tutorials/using_peft_for_inference)

다양한 스타일을 만들기 위해 훈련된 많은 adapter 타입이 있습니다([LoRA](https://huggingface.co/docs/peft/conceptual_guides/adapter#low-rank-adaptation-lora)가 유명). 새롭고 유니크한 이미지를 만들기 위해 여러 어댑터를 조합할 수 있습니다.

이 튜토리얼에서는 Diffusers에서 어댑터들을 로드하고 관리하기 위한 [PEFT](https://huggingface.co/docs/peft/index)를 사용하는 방법을 살펴보겠습니다.

필요 라이브러리 설치

```bash
pip install -q transformers accelerate peft diffusers
```

[Stable Diffusion XL (SDXL)](https://huggingface.co/docs/diffusers/api/pipelines/stable_diffusion/stable_diffusion_xl) checkpoint 파이프라인 로드:

```python
from diffusers import DiffusionPipeline
import torch

pipe_id = "stabilityai/stable-diffusion-xl-base-1.0"
pipe = DiffusionPipeline.from_pretrained(pipe_id, torch_dtype=torch.float16).to("cuda")
```

그 후, [load_lora_weights()](https://huggingface.co/docs/diffusers/v0.32.2/en/api/loaders/lora#diffusers.loaders.StableDiffusionXLLoraLoaderMixin.load_lora_weights) method를 사용하여 [CiroN2022/toy-face](https://huggingface.co/CiroN2022/toy-face) adapter를 로드합니다. 🤗 PEFT integration에서는 특정한 `adapter_name`을 할당할 수도 있습니다. 이는 LoRA checkpoints 간에 쉽게 변경할 수 있도록 합니다. 이 어탭터를 adapter "toy"라고 부릅시다.

```python
pipe.load_lora_weights("CiroN2022/toy-face", weight_name="toy_face_sdxl.safetensors", adapter_name="toy")
```

토큰 `toy_face` 프롬프트에 포함하고 inference를 수행합니다.

```python
prompt = "toy_face of a hacker with a hoodie"

lora_scale = 0.9
image = pipe(
    prompt, num_inference_steps=30, cross_attention_kwargs={"scale": lora_scale}, generator=torch.manual_seed(0)
).images[0]
image.show()
```

![toy_face_hacker]({{site.baseurl}}/assets/images/2025-02-14-diffusers-tutorials-03/toy_face_hacker.png){: .align-center}  

`adapter_name`을 사용하여 쉽게 다른 adapter를 inference에 사용할 수 있습니다. pixel art image를 생성하도록  fine-tune된 어댑터인 [nerijs/pixel-art-xl](https://huggingface.co/nerijs/pixel-art-xl) adapter를 로드하고 `"pixel"`이라는 이름을 할당합니다.

```python
pipe.load_lora_weights("nerijs/pixel-art-xl", weight_name="pixel-art-xl.safetensors", adapter_name="pixel")
pipe.set_adapters("pixel")
```

토큰 `pixel art`를 프롬프트에 포함시킵니다.

```python
prompt = "a hacker with a hoodie, pixel art"
image = pipe(
    prompt, num_inference_steps=30, cross_attention_kwargs={"scale": lora_scale}, generator=torch.manual_seed(0)
).images[0]
image.show()
```

![pixel_art_hacker]({{site.baseurl}}/assets/images/2025-02-14-diffusers-tutorials-03/pixel_art_hacker.png){: .align-center}  

## Merge adapters

여러 adapter checkpoint의 style을 blend하기 위해 합칠 수도 있습니다.

다시 한번 `~PeftAdapterMixin.set_adapters` 메서드를 사용하여 `pixel`과 `toy` 어댑터를 활성화하고 weights를 명시합니다.

```python
pipe.set_adapters(["pixel", "toy"], adapter_weights=[0.5, 1.0])
```

Diffusion community에서 LoRA 체크포인트는 거의 항상 [DreamBooth](https://huggingface.co/docs/diffusers/main/en/training/dreambooth)로 얻어집니다. DreamBooth 훈련 모델은 입력 텍스트 프롬프트에 "trigger" 단어를 사용하는 경우가 많습니다. 여러 LoRA 체크포인트를 결합할 때는 해당 LoRA 체크포인트의 트리거 단어가 입력 텍스트 프롬프트에 포함되어 있는지 확인하는 것이 중요합니다.
{: .notice--info}

프롬프트에 두 모델의 트리거 단어를 포함시킵니다.

```python
prompt = "toy_face of a hacker with a hoodie, pixel art"
image = pipe(
    prompt, num_inference_steps=30, cross_attention_kwargs={"scale": 1.0}, generator=torch.manual_seed(0)
).images[0]
image.show()
```

![20250214-091648]({{site.baseurl}}/assets/images/2025-02-14-diffusers-tutorials-03/blend_hacker.png){: .align-center}  

모든 어댑터를 비활성화하려면 `~PeftAdapterMixin.disable_lora` 메서드를 사용합니다.

```python
pipe.disable_lora()

prompt = "toy_face of a hacker with a hoodie"
image = pipe(prompt, num_inference_steps=30, generator=torch.manual_seed(0)).images[0]
image.show()
```

## Customize adapter strength

각 어댑터가 얼마나 강하게 동작하게 할 것인지 컨트롤 할 수 있습니다. control sterngth(scale)을 딕셔너리에 포합시켜 `~PeftAdapterMixin.set_adapters`를 전달할 수 있습니다.

예를 들어, adapter를 `down` 파트에서는 어댑터를 켜고, `mid`와 `up` 파트에서는 끌 수 있습니다.

```python
pipe.enable_lora()  # enable lora again, after we disabled it above
prompt = "toy_face of a hacker with a hoodie, pixel art"
adapter_weight_scales = { "unet": { "down": 1, "mid": 0, "up": 0} }
pipe.set_adapters("pixel", adapter_weight_scales)
image = pipe(prompt, num_inference_steps=30, generator=torch.manual_seed(0)).images[0]
image
```

## Adapter 관리하기

```python
active_adapters = pipe.get_active_adapters()
active_adapters
# ["toy", "pixel"]
```

```python
list_adapters_component_wise = pipe.get_list_adapters()
list_adapters_component_wise
# {"text_encoder": ["toy", "pixel"], "unet": ["toy", "pixel"], "text_encoder_2": ["toy", "pixel"]}
```
