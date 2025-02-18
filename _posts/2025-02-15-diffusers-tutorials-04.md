---
title: "Diffusers - DreamBooth"
excerpt: ""
date: 2025-02-15 01:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Stable Diffusion
---

참고한 문서: [DreamBooth](https://huggingface.co/docs/diffusers/training/dreambooth)

DreamBooth는 entire diffusion model을 몇 개의 이미지로 훈련하는 테크닉이다.  

만약 제한적인 VRAM GPU로 훈련을 한다면 `gradient_checkpointing`과 `mixed_precision` 파라미터를 켜야한다. 또한 memory-efficient attention xFormers를 사용하여 메모리 사용량을 줄일 수 있다. JAX/Flax training은 TPU와 GPU에서의 효율적인 훈련을 지원한다. 하지만 이는 gradient checkpointing나 xFormers를 지원하지 않는다. Flax에서 빠르게 훈련하기 위해선 30GB memory 이상의 GPU가 필요하다.

이 가이드에서는 [train_dreambooth.py](https://github.com/huggingface/diffusers/blob/main/examples/dreambooth/train_dreambooth.py) 스크립트를 살펴보고 이에 친숙해지도록 하여, 당신의 use-case에 적용하는 방법을 배울 수 있다.

스크립트를 실행하기 전에 library가 잘 설치되었는지 확인하자.

```bash
git clone https://github.com/huggingface/diffusers
cd diffusers
pip install .
```

example 폴더로 이동하여 필요한 dependency를 설치하자.

```bash
cd examples/dreambooth
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

DreamBooth는 트레이닝 hyperparameters에 민감하며, overfit 되기 쉽다. [Blog post](https://huggingface.co/blog/dreambooth)를 읽어보고 주제에 따른 추천 세팅을 참고하자.
{: .notice--info}

training script는 많은 parameter를 제공하며, 이를 통해 트레이닝 세팅을 커스터마이징할 수 있다. 각 파라미터의 설명은 [`parse_arags()`](https://github.com/huggingface/diffusers/blob/072e00897a7cf4302c347a63ec917b4b8add16d4/examples/dreambooth/train_dreambooth.py#L228) 함수에서 찾을 수 있다. 기본으로 설정된 파라미터들을 바로 사용해도 상당히 잘 동작하지만, 각자 세팅할 수도 있다.

예를 들어, bf16 포맷으로 훈련한다면:

```bash
accelerate launch train_dreambooth.py \
    --mixed_precision="bf16"
```

몇몇 기본적이고 중요한 파라미터는 다음과 같다.

- `--pretrained_model_name_or_path`: Pretrained model의 local path 혹은 Hub의 모델명
- `--instance_data_dir`: training dataset을 포함하고 있는 folder 경로
- `--instance_prompt`: example image를 가리키는 special word를 포함한 text prompt
- `--train_text_encoder`: text encoder를 훈련할 지 여부
- `--output_dir`: 훈련된 모델을 저장할 directory
- `--push_to_hub`: 훈련된 모델을 Hub에 push할 지 여부
- `--checkpointing_steps`: 모델 훈련 시 checkpoint를 저장하는 빈도. 이는 어떤 이유로 training이 중단되었을 때, 체크포인트에서 training을 재개할 수 있다(`--resume_from_checkpoint` 옵션 사용)

### Min-SNR weighting

[Min-SNR](https://huggingface.co/papers/2303.09556) weighting 전략은 loss to achieve faster convergence를 리밸런싱하여 트레이닝을 돕는다. 트레이닝 스크립트는 `epsilon`(noise)이나 `v_prediction`을 중 하나를 예측하는 것을 지원하는데, Min-SNR은 두 예측 타입에 모두 적용할 수 있다. 이 weighting strategy는 PyTorch에서만 지원되며, Flax training script에서는 사용할 수 없다.

`--snr_gamma` 파라미터를 추가하면 되며, 추천 값은 5.0이다.

```bash
accelerate launch train_dreambooth.py \
    --snr_gamma=5.0
```

### Prior preservation loss

Prior preservation loss는 모델 스스로가 생성한 샘플들을 이용하여 더욱 다양한 이미지들을 생성하는 것을 도와주는 방법이다. 이 생성된 샘플 이미지들은 우리가 제공한 이미지들과 같은 클래스에 속하기 때문에, 모델이 클래스에 대해 학습한 내용을 유지하고, 이미 클래스에 대해 알고 있는 것을 사용하여 새로운 구성을 만드는 데 어떻게 활용할 수 있는지 도움을 줍니다.

- `--with_prior_preservation`: prior preservation loss를 사용할 지 여부
- `--prior_loss_weight`: prior preservation loss가 model에 주는 영향을 control
- `--class_data_dir`: generated class sample images를 포함하고 있는 folder 경로
- `--class_prompt`: 생성된 sample images를 설명하는 text prompt

```bash
accelerate launch train_dreambooth.py \
  --with_prior_preservation \
  --prior_loss_weight=1.0 \
  --class_data_dir="path/to/class/images" \
  --class_prompt="text prompt describing class"
```

### Train text encoder

생성된 아웃풋의 품질을 향상시키기 위해서 UNet에 더해 text encoder도 훈련할 수 있다. 이는 추가적인 메모리와 적어도 24GB VRAM GPU가 필요하다. 하드웨어가 있다면, text encoder를 훈련하는 게 더 나은 결과를 가져온다. 특히 얼굴 이미지들 생성할 때. 이 옵션을 다음으로 활성화 하자.

```bash
accelerate launch train_dreambooth.py \
  --train_text_encoder
```

## Training script

DreamBooth는 own data classes를 사용한다.

- [`DreamBoothDataset`](https://github.com/huggingface/diffusers/blob/072e00897a7cf4302c347a63ec917b4b8add16d4/examples/dreambooth/train_dreambooth.py#L604): 이미지와 클래스 이미지를 preprocess하며, prompt를 tokenize한다.
- [`PromptDataset`](https://github.com/huggingface/diffusers/blob/072e00897a7cf4302c347a63ec917b4b8add16d4/examples/dreambooth/train_dreambooth.py#L738): class image를 생성하기 위한 prompt embedding을 생성한다.

[prior preservation loss](https://github.com/huggingface/diffusers/blob/072e00897a7cf4302c347a63ec917b4b8add16d4/examples/dreambooth/train_dreambooth.py#L842)를 활성화한다면, class image들은 여기서 생성된다:

```python
sample_dataset = PromptDataset(args.class_prompt, num_new_images)
sample_dataloader = torch.utils.data.DataLoader(sample_dataset, batch_size=args.sample_batch_size)

sample_dataloader = accelerator.prepare(sample_dataloader)
pipeline.to(accelerator.device)

for example in tqdm(
    sample_dataloader, desc="Generating class images", disable=not accelerator.is_local_main_process
):
    images = pipeline(example["prompt"]).images
```

[`main()`](https://github.com/huggingface/diffusers/blob/072e00897a7cf4302c347a63ec917b4b8add16d4/examples/dreambooth/train_dreambooth.py#L799) 함수는 훈련을 위한 dataset 세팅을 관리하고, training loop 자체이다. 스크립트에서 [tokenizer](https://github.com/huggingface/diffusers/blob/072e00897a7cf4302c347a63ec917b4b8add16d4/examples/dreambooth/train_dreambooth.py#L898), [scheduler와 model](https://github.com/huggingface/diffusers/blob/072e00897a7cf4302c347a63ec917b4b8add16d4/examples/dreambooth/train_dreambooth.py#L912C1-L912C1)을 로드한다.

```python
# Load the tokenizer
if args.tokenizer_name:
    tokenizer = AutoTokenizer.from_pretrained(args.tokenizer_name, revision=args.revision, use_fast=False)
elif args.pretrained_model_name_or_path:
    tokenizer = AutoTokenizer.from_pretrained(
        args.pretrained_model_name_or_path,
        subfolder="tokenizer",
        revision=args.revision,
        use_fast=False,
    )

# Load scheduler and models
noise_scheduler = DDPMScheduler.from_pretrained(args.pretrained_model_name_or_path, subfolder="scheduler")
text_encoder = text_encoder_cls.from_pretrained(
    args.pretrained_model_name_or_path, subfolder="text_encoder", revision=args.revision
)

if model_has_vae(args):
    vae = AutoencoderKL.from_pretrained(
        args.pretrained_model_name_or_path, subfolder="vae", revision=args.revision
    )
else:
    vae = None

unet = UNet2DConditionModel.from_pretrained(
    args.pretrained_model_name_or_path, subfolder="unet", revision=args.revision
)
```

[training dataset 생성](https://github.com/huggingface/diffusers/blob/072e00897a7cf4302c347a63ec917b4b8add16d4/examples/dreambooth/train_dreambooth.py#L1073) 및 DataLoader from `DreamBoothDataset`을 생성한다.

```python
train_dataset = DreamBoothDataset(
    instance_data_root=args.instance_data_dir,
    instance_prompt=args.instance_prompt,
    class_data_root=args.class_data_dir if args.with_prior_preservation else None,
    class_prompt=args.class_prompt,
    class_num=args.num_class_images,
    tokenizer=tokenizer,
    size=args.resolution,
    center_crop=args.center_crop,
    encoder_hidden_states=pre_computed_encoder_hidden_states,
    class_prompt_encoder_hidden_states=pre_computed_class_prompt_encoder_hidden_states,
    tokenizer_max_length=args.tokenizer_max_length,
)

train_dataloader = torch.utils.data.DataLoader(
    train_dataset,
    batch_size=args.train_batch_size,
    shuffle=True,
    collate_fn=lambda examples: collate_fn(examples, args.with_prior_preservation),
    num_workers=args.dataloader_num_workers,
)
```

마지막으로 [training loop](https://github.com/huggingface/diffusers/blob/072e00897a7cf4302c347a63ec917b4b8add16d4/examples/dreambooth/train_dreambooth.py#L1151)는 이미지를 latent space로 변환하거나, input에 noise를 추가하거나, noise residual을 예측하거나, loss를 계산하는 것과 같이 남아있는 스텝들을 처리한다.

만약 training loop의 작동에 대해 더 알고 싶다면 [Understanding pipelines, models and schedulers](https://huggingface.co/docs/diffusers/using-diffusers/write_own_pipeline) tutorial을 참고하자.

## Launch the script

이제 훈련 스크립트를 실행할 준비가 되었다.

본 가이드에서는 [개 이미지](https://huggingface.co/datasets/diffusers/dog-example)를 다운받아 디렉터리에 저장한다. 각자 자신만의 데이터셋을 만들어도 된다([트레이닝 데이터셋 만들기](https://huggingface.co/docs/diffusers/training/create_dataset)).

```python
from huggingface_hub import snapshot_download

local_dir = "./dog"
snapshot_download(
    "diffusers/dog-example",
    local_dir=local_dir,
    repo_type="dataset",
    ignore_patterns=".gitattributes",
)
```

`MODEL_NAME` 환경 변수를 Hub의 model id나 local model로 설정하고, `INSTANCE_DIR`을 다운받은 개 이미지들이 있는 path로 지정한다. 그리고 `OUTPUT_DIR`를 모델을 저장하고 싶은 곳으로 한다. 여기에서는 `sks`를 training할 special word로 사용할 것이다.

training process를 체크하기 위해서 주기적으로 생성된 이미지를 저장할 수 있다. 트레이닝 커맨드에 다음을 추가하자.

```bash
--validation_prompt="a photo of a sks dog"
--num_validation_images=4
--validation_steps=100
```

## LoRA

LoRA는 획기적으로 trainable parameter의 수를 줄여주는 테크닉이다. 결과적으로 트레이닝은 빠르며, 결과 weight를 저장하기 쉽다(크기가 ~100MBs로 작다). LoRA training을 위해서는 [train_dreambooth_lora.py](https://github.com/huggingface/diffusers/blob/main/examples/dreambooth/train_dreambooth_lora.py)를 사용하자.

LoRA training script는 [LoRA training guide](https://huggingface.co/docs/diffusers/training/lora)에서 자세히 다룬다.