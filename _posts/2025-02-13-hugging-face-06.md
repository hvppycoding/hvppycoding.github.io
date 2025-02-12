---
title: "Ch 6. 허깅페이스 실습 - 컴퓨터 비전"
excerpt: ""
date: 2025-02-13 01:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Hugging Face
---


## Image2Image Stable Diffusion Image Variations

[Stable Diffusion 논문]({{site.baseurl}}/assets/images/2025-02-13-hugging-face-06/Stable_Diffusion.pdf)

허깅페이스의 [Pipeline 문서](https://huggingface.co/docs/diffusers/api/pipelines/stable_diffusion/img2img)를 참고하자.

class diffusers.StableDiffusionImg2ImgPipeline

- vae (AutoencoderKL) — Variational Auto-Encoder (VAE) model to encode and decode images to and from latent representations.
- text_encoder (CLIPTextModel) — Frozen text-encoder (clip-vit-large-patch14).
tokenizer (CLIPTokenizer) — A CLIPTokenizer to tokenize text.
unet (UNet2DConditionModel) — A UNet2DConditionModel to denoise the encoded image latents.
- scheduler (SchedulerMixin) — A scheduler to be used in combination with unet to denoise the encoded image latents. Can be one of DDIMScheduler, LMSDiscreteScheduler, or PNDMScheduler.
- safety_checker (StableDiffusionSafetyChecker) — Classification module that estimates whether generated images could be considered offensive or harmful. Please refer to the model card for more details about a model’s potential harms.
- feature_extractor (CLIPImageProcessor) — A CLIPImageProcessor to extract features from generated images; used as inputs to the safety_checker.

### 실습

```bash
pip install torch torchvision
pip install transformers
pip install datasets
pip install diffusers accelerate safetensors
```

```python
import PIL
import requests
import torch
from diffusers import StableDiffusionInstructPix2PixPipeline, EulerAncestralDiscreteScheduler

model_id = "timbrooks/instruct-pix2pix"
pipe = StableDiffusionInstructPix2PixPipeline.from_pretrained(model_id, torch_dtype=torch.float16, safety_checker=None)
pipe.to("cuda")
pipe.scheduler = EulerAncestralDiscreteScheduler.from_config(pipe.scheduler.config)

url = "https://raw.githubusercontent.com/timothybrooks/instruct-pix2pix/main/imgs/example.jpg"
def download_image(url):
    image = PIL.Image.open(requests.get(url, stream=True).raw)
    image = PIL.ImageOps.exif_transpose(image)
    image = image.convert("RGB")
    return image
image = download_image(url)

prompt = "turn him into cyborg"
images = pipe(prompt, image=image, num_inference_steps=10, image_guidance_scale=1).images
images[0].show()
```

#### 원래 이미지

![original]({{site.baseurl}}/assets/images/2025-02-13-hugging-face-06/original.png){: .align-center}  

#### 변환된 이미지

"turn him into cyborg" 프롬프트에 따라 이미지가 변환되었다.

![cyborg]({{site.baseurl}}/assets/images/2025-02-13-hugging-face-06/cyborg.png){: .align-center}  

### Fine-tuning 하기

Pipeline 전체를 fine-tuning 하는 것은 힘들다. 예시에서는 U-net만 fine-tuning 한다.

```python
from datasets import load_dataset
import torch
from diffusers import StableDiffusionInstructPix2PixPipeline, EulerAncestralDiscreteScheduler
from torchvision import transforms
import numpy as np
from PIL import Image

transform = transforms.Compose([
    transforms.ToTensor(),
])

# Load and preprocess the dataset
dataset = load_dataset('cifar10')
dataset = dataset['train'].select(range(500))  # selecting a subset

def preprocess(data):
    image = data['img']
    image = transform(image)  # Converts PIL Image to PyTorch Tensor
    instruction = 'Make the image look like class ' + str(data['label'])
    return {'image': image, 'instruction': instruction}

dataset = dataset.map(preprocess, remove_columns=['img', 'label'])
dataset.set_format(type='torch', columns=['image', 'instruction'])

# Load the model
model_id = "timbrooks/instruct-pix2pix"
pipe = StableDiffusionInstructPix2PixPipeline.from_pretrained(model_id, torch_dtype=torch.float16, safety_checker=None)
pipe.to("cuda")
pipe.scheduler = EulerAncestralDiscreteScheduler.from_config(pipe.scheduler.config)

print(pipe)
# StableDiffusionInstructPix2PixPipeline {
#   "_class_name": "StableDiffusionInstructPix2PixPipeline",
#   "_diffusers_version": "0.32.2",
#   "_name_or_path": "timbrooks/instruct-pix2pix",
#   "feature_extractor": [
#     "transformers",
#     "CLIPImageProcessor"
#   ],
#   "image_encoder": [
#     null,
#     null
#   ],
#   "requires_safety_checker": false,
#   "safety_checker": [
#     null,
#     null
#   ],
#   "scheduler": [
#     "diffusers",
#     "EulerAncestralDiscreteScheduler"
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

model_unet = pipe.unet
optimizer = torch.optim.Adam(model_unet.parameters(), lr=0.001)

from torchvision.transforms.functional import to_tensor

def loss_function(output, target):
    output_tensor = to_tensor(output).to(target.device).requires_grad_()
    return ((output_tensor - target) ** 2).mean()

# Move the model to GPU if available
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
pipe = pipe.to(device)

# Training loop
for epoch in range(3):  # for 3 epochs
    for data in dataset:
        optimizer.zero_grad()

        # Move data to the device
        image = data['image'].to(device)
        instruction = data['instruction']

        # Forward pass
        outputs = pipe(instruction, image=image, num_inference_steps=10, image_guidance_scale=1)

        # Compute loss
        loss = loss_function(outputs.images[-1], image)  # Now both are tensors
        # Backward pass and optimization
        loss.backward()
        optimizer.step()

print("Training is done.")
```

