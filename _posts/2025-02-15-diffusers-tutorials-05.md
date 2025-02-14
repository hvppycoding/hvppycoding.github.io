---
title: "Diffusers - LoRA training"
excerpt: ""
date: 2025-02-15 02:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Diffusers
---

참고한 문서: [LoRA training](https://huggingface.co/docs/diffusers/training/lora)

[LoRA(Low-Rank Adaptation of Large Language Models)](https://hf.co/papers/2106.09685)은 가볍고 널리 활용되는 training technique로, trainable parameters의 수를 획기적으로 감소시킨다. 이는 적은 수의 새로운 weight를 모델에 추가하고 이들만 훈련시킨다. 이는 LoRA를 빠르고, 메모리-효율적이고, 작은 model weights(수 백 MB)만 필요해 공유하고 저장하기 쉽다. LoRA는 DreamBooth와 같은 트레이닝 테크닉과 결합하여 트레이닝 속도를 높일 수도 있다.

본 가이드에서는 [train_text_to_image_lora.py](https://github.com/huggingface/diffusers/blob/main/examples/text_to_image/train_text_to_image_lora.py) 스크립트를 살펴보고 이에 친숙해지도록 하여, 당신의 use-case에 적용하는 방법을 배울 수 있다.

시작하기 전에 라이브러리 설치를 확인하자.

```bash
git clone https://github.com/huggingface/diffusers
cd diffusers
pip install .
```

example 폴더로 이동하여 필요한 dependency를 설치하자.

```bash
cd examples/text_to_image
pip install -r requirements.txt
```

Accelerate는 여러 GPU/TPU나 mixed precision에서 훈련하는 것을 도와주는 라이브러리이다. 이 라이브러리는 사용자의 하드웨어와 환경에 따라 자동으로 트레이닝 환경을 설정한다. 자세한 내용은 Accelerate의 [Quicktour](https://huggingface.co/docs/accelerate/quicktour)를 참고하자.
{: .notice--info}

Accelerate 환경 초기화하기

```bash
accelerate config
```

혹은 configuration 설정 없이 기본 설정으로 초기화하기

```bash
accelerate config default
```

shell이 없을 경우 다음과 같이 설정할 수 있다.

```python
from accelerate.utils import write_basic_config

write_basic_config()
```

## Script parameters

training script는 많은 parameter를 제공하며, 이를 통해 트레이닝 세팅을 커스터마이징할 수 있다. 각 파라미터의 설명은 [`parse_args()`](https://github.com/huggingface/diffusers/blob/dd9a5caf61f04d11c0fa9f3947b69ab0010c9a0f/examples/text_to_image/train_text_to_image_lora.py#L85) 함수에서 찾을 수 있다. 기본으로 설정된 파라미터들을 바로 사용해도 상당히 잘 동작하지만, 각자 세팅할 수도 있다.

예를 들어, training epochs를 늘리고 싶다면,

```bash
accelerate launch train_text_to_image_lora.py \
  --num_train_epochs=150 \
```

많은 중요한 기본 파라미터들은 [Text-to-image](https://huggingface.co/docs/diffusers/training/text2image#script-parameters) training guided에 있으며, 이 가이드에서는 LoRA 관련된 파라미터에 집중한다.

- `--rank`: 훈련할 low-rank matrices의 inner dimension이다. 더 높은 rank는 더 많은 trainable parameters를 의미한다.
- `--learning_rate`: 기본 learning rate는 1e-4이나, LoRA 훈련에서는 더 높은 learning rate를 사용할 수 있다.

## Training script

dataset preprocessing code와 training loop는 [`main()`](https://github.com/huggingface/diffusers/blob/dd9a5caf61f04d11c0fa9f3947b69ab0010c9a0f/examples/text_to_image/train_text_to_image_lora.py#L371) 함수에서 찾을 수 있다. 또한 training script를 수정한다면 이 부분을 변경해야할 것이다.

훈련 스크립트에 대한 자세한 설명은 [텍스트-이미지 훈련 가이드](https://huggingface.co/docs/diffusers/training/text2image#training-script)에서 제공됩니다. 대신, 이 가이드에서는 스크립트 중 LoRA와 관련된 부분에 대해 살펴봅니다.

### UNet

Diffusers는 LoRA 파라미터(rank, alpha, LoRA weights를 넣을 모듈)를 설정하기 위해 `PEFT` 라이브러리의 `~peft.LoraConfig`를 사용합니다. 이 어댑터는 UNet에 추가되며, `lora_layers`에서 LoRA 레이어만 필터링되어 최적화됩니다.

```python
unet_lora_config = LoraConfig(
    r=args.rank,
    lora_alpha=args.rank,
    init_lora_weights="gaussian",
    target_modules=["to_k", "to_q", "to_v", "to_out.0"],
)

unet.add_adapter(unet_lora_config)
lora_layers = filter(lambda p: p.requires_grad, unet.parameters())
```

### text encoder

Diffusers는 필요한 경우 (예: Stable Diffusion XL (SDXL) 미세 조정) `PEFT` 라이브러리의 LoRA를 사용하여 텍스트 인코더 미세 조정도 지원합니다. `~peft.LoraConfig`는 LoRA 어댑터의 매개변수를 구성하는 데 사용되며, 이 어댑터는 텍스트 인코더에 추가되고, LoRA 레이어만 훈련을 위해 필터링됩니다.

```python
text_lora_config = LoraConfig(
    r=args.rank,
    lora_alpha=args.rank,
    init_lora_weights="gaussian",
    target_modules=["q_proj", "k_proj", "v_proj", "out_proj"],
)

text_encoder_one.add_adapter(text_lora_config)
text_encoder_two.add_adapter(text_lora_config)
text_lora_parameters_one = list(filter(lambda p: p.requires_grad, text_encoder_one.parameters()))
text_lora_parameters_two = list(filter(lambda p: p.requires_grad, text_encoder_two.parameters()))
```

[optimizer](https://github.com/huggingface/diffusers/blob/e4b8f173b97731686e290b2eb98e7f5df2b1b322/examples/text_to_image/train_text_to_image_lora.py#L529)는 `lora_layers`와 initialized 됩니다(optimized될 weights들은 LoRA weights만 포함되기 때문).

```python
optimizer = optimizer_cls(
    lora_layers,
    lr=args.learning_rate,
    betas=(args.adam_beta1, args.adam_beta2),
    weight_decay=args.adam_weight_decay,
    eps=args.adam_epsilon,
)
```

LoRA 레이어 세팅 부분을 제외하면, training script는 train_text_to_image.py와 거의 동일하다.

## Launch the script

모든 변경 사항을 적용했거나 기본 구성이 마음에 들면 학습 스크립트를 시작할 준비가 된 것입니다!

[Naruto BLIP 캡션](https://huggingface.co/datasets/lambdalabs/naruto-blip-captions) 데이터 세트에서 학습하여 자신만의 Naruto 캐릭터를 생성해 봅시다. 환경 변수 `MODEL_NAME`과 `DATASET_NAME`을 각각 모델과 데이터 세트로 설정하세요. 또한 `OUTPUT_DIR`에 모델을 저장할 위치를, `HUB_MODEL_ID`에 Hub에 저장할 모델 이름을 지정해야 합니다. 스크립트는 다음과 같은 파일들을 생성하여 저장소에 저장합니다.

- 저장된 모델 체크포인트
- `pytorch_lora_weights.safetensors` (훈련된 LoRA 가중치)

둘 이상의 GPU에서 훈련하는 경우 `accelerate launch` 명령어에 `--multi_gpu` 파라미터를 추가하세요.

트레이닝 과정은 2080 Ti GPU(11GB VRAM)에서 ~5시간이 걸립니다.
{: .notice--warning}

```bash
export MODEL_NAME="stable-diffusion-v1-5/stable-diffusion-v1-5"
export OUTPUT_DIR="/sddata/finetune/lora/naruto"
export HUB_MODEL_ID="naruto-lora"
export DATASET_NAME="lambdalabs/naruto-blip-captions"

accelerate launch --mixed_precision="fp16"  train_text_to_image_lora.py \
  --pretrained_model_name_or_path=$MODEL_NAME \
  --dataset_name=$DATASET_NAME \
  --dataloader_num_workers=8 \
  --resolution=512 \
  --center_crop \
  --random_flip \
  --train_batch_size=1 \
  --gradient_accumulation_steps=4 \
  --max_train_steps=15000 \
  --learning_rate=1e-04 \
  --max_grad_norm=1 \
  --lr_scheduler="cosine" \
  --lr_warmup_steps=0 \
  --output_dir=${OUTPUT_DIR} \
  --push_to_hub \
  --hub_model_id=${HUB_MODEL_ID} \
  --report_to=wandb \
  --checkpointing_steps=500 \
  --validation_prompt="A naruto with blue eyes." \
  --seed=1337
```

트레이닝이 완료되면 모델을 inference에 사용할 수 있습니다.

```python
from diffusers import AutoPipelineForText2Image
import torch

pipeline = AutoPipelineForText2Image.from_pretrained("stable-diffusion-v1-5/stable-diffusion-v1-5", torch_dtype=torch.float16).to("cuda")
pipeline.load_lora_weights("path/to/lora/model", weight_name="pytorch_lora_weights.safetensors")
image = pipeline("A naruto with blue eyes").images[0]
```