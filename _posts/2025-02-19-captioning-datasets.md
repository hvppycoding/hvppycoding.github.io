---
title: "Captioning Datasets for Training Purposes"
excerpt: ""
date: 2025-02-19 01:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Stable Diffusion
---

This post is an archive of the [reddit's Captioning Datasets for Training Purposes](https://www.reddit.com/r/StableDiffusion/comments/118spz6/captioning_datasets_for_training_purposes/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button).
{: .notice--info}

다양한 SD 커뮤니티가 모델, 프로세스 등 모든 것을 공유하는 데 있어 얼마나 개방적인지 고려하여, 저는 충분히 주목받지 못한다고 생각하는 영역인 훈련 목적의 데이터 세트 캡션 작성에 대한 저의 지식과 경험을 바탕으로 글을 쓰려고 합니다.

In the spirit of how open the various SD communities are in sharing their models, processes, and everything else, I thought I would write something up based on my knowledge and experience so far in an area that I think doesn't get enough attention: captioning datasets for training purposes.

## 면책 조항 <sup>DISCLAIMER</sup>

저는 이 분야의 전문가가 아니며, 이는 단순히 저에게 효과가 있었던 것, 다른 사람들로부터 배운 것, 기본 프로세스에 대한 저의 이해, 그리고 다른 유형의 데이터 세트 작업에서 얻은 지식을 혼합한 것입니다.

I am not an expert in this field, this is simply a mix of what has worked for me, what I've learned from others, my understanding of the underlying processes, and the knowledge I've gained from working with other types of datasets.

자동 캡션과 비교하여 캡션의 결과 품질을 고려할 때 데이터 세트를 수동으로 캡션할 때 이 워크플로가 상당히 효율적이라는 것을 알았습니다. 하지만 수백 장의 사진에 캡션을 추가하려는 경우 여전히 시간이 걸립니다. 분명히 말씀드리자면, 이 방법은 수만 장의 이미지가 있는 정말 큰 데이터 세트에 캡션을 추가하는 데 적합하지 않습니다. 마조히스트가 아니라면요.

I have found this workflow to be reasonably efficient when manually captioning a dataset considering the resulting quality of the captions compared to automated captioning. But be warned, if you are looking to caption hundreds of photos, it's still gonna take some time. To be clear, that means I am saying this method is not good for captioning truly large datasets with tens of thousands of images. Unless you are a masochist.

때로는 "태그"라고 말하고 때로는 "캡션"이라고 말합니다. 모든 것을 수정하려고 했지만 캡션 작업이 있었기 때문에 나중에 통일할 수 있을 것 같습니다.

Sometimes I say "tag" and sometimes "caption". I was going to go through and fix it all, but I had captioning to do, so maybe I will make it uniform later.

저는 이 문서가 "완료"되었다고 생각하지 않습니다. 배울 것이 너무 많고 AI 분야는 너무 빠르게 움직이기 때문에 아마도 절대 완료되지 않을 것입니다. 그러나 필요에 따라 이 문서를 확장하고 변경하려고 노력할 것입니다.

I do not consider this document "finished". There is so much to learn, and the AI space is moving so fast, that it will likely never be finished. However, I will try to expand and alter this document as necessary.

저의 경험은 주로 LoRA 교육과 관련이 있지만 여기에 있는 몇 가지 측면은 모든 유형의 교육에 적용할 수 있습니다.

My experience has primarily been with LoRA training, but some of the aspects here are applicable to all types of training.

## 이 문서는 누구를 위한 것인가요?<sup>WHO IS THIS DOCUMENT FOR</sup>

이 문서가 자신의 데이터 세트를 사용하여 Stable Diffusion에서 자신의 모델을 훈련하는 데 다소 진지하게 관심이 있는 모든 사람에게 도움이 되기를 바랍니다. 당신의 목표가 모델에 얼굴을 빠르게 가르치는 것이라면 즉시 시작하고 실행할 수 있는 훨씬 더 나은 가이드가 있습니다. 그러나 목표가 좀 더 깊이 있고 교육을 더 자세히 살펴보는 것이라면 이 문서를 리소스에 추가할 수 있습니다.

I hope this document can be helpful to anyone who is somewhat seriously interested in training their own models in Stable Diffusion using their own datasets. If your goal is to quickly teach your face to a model, there are much better guides available which will have you up and running in a flash. But if your goal is to go a bit deeper, explore training in more depth, perhaps you can add this document to your resources.

## 데이터셋<sup>DATASET</sup>

좋은 데이터 세트를 얻는 것에 대해서는 다른 곳에서 광범위하게 다루므로 가장 중요한 부분만 포함했습니다.

- 고품질 입력은 고품질 출력을 의미합니다.
- 더 많은 양과 더 많은 다양성이 더 좋습니다.
- 품질과 양 중에서 선택해야 하는 경우 항상 품질이 우선입니다.
- 최후의 수단으로 업스케일하고 가능하면 피하십시오. 업스케일해야 할 때 Automatic1111을 통해 LDSR을 사용합니다.

Obtaining a good dataset is talked about extensively elsewhere, so I've only included the most important parts:

- high quality input means high quality output
- more quantity and more variety is better
- If you are forced to choose between quality and quantity, quality always wins.
- Upscale as a last resort, avoid it if possible. When I am forced to upscale, I use LDSR via Automatic1111.

## 준비<sup>PREPARATION</sup>

훈련 방법과 대상에 따라 사진을 특정 너비와 높이로 잘라야 할 수도 있습니다. 다른 유형의 훈련은 이미지를 다양한 크기로 분류하며 자르기가 필요하지 않습니다. 수행 중인 훈련 방법, 훈련 중인 모델 및 모델 훈련에 사용 중인 프로그램에 필요한 사항을 살펴보십시오.

Depending on how and what you are training, you may need to crop the photos to a specific width and height. Other types of training will bucket images into various sizes and do not require cropping. Look into what is required for the method of training you are doing, the model you are training on, and the program you are using to train your model with.

## 캡션 작성<sup>CAPTIONING</sup> - 일반 참고 사항<sup>GENERAL NOTES</sup>

다음 권장 사항은 제 실험, 다른 데이터 세트를 사용한 배경 작업, 주제별 논문 읽기 및 다른 성공적인 접근 방식에서 차용한 것을 기반으로 합니다.

The following recommendations are based on my experiments, my background work with other datasets, reading subject-matter papers, and borrowing from other successful approaches.

### 지금은 자동 캡션 작성을 피하십시오.<sup>Avoid automated captioning, for now</sup>

- BLIP과 deepbooru는 흥미롭지만 아직은 시기상조라고 생각합니다.
- 종종 실수가 많고 지나치게 반복적인 캡션을 발견하는데, 이를 정리하는 데 시간이 걸립니다.
- 컨텍스트와 상대적 중요성을 파악하는 데 어려움을 겪습니다.
- BLIP/deepbooru가 만든 실수를 수정하고 여전히 수동으로 캡션을 작성하는 것보다 수동으로 캡션을 작성하는 것이 더 빠르다고 생각합니다.

- BLIP and deepbooru are exciting, but I think it is a bit early for them yet.
- I often find mistakes and extremely repetitive captions, which take awhile to clean up.
- They struggle with context and with relative importance.
- I think it is faster to manually caption, rather than fix mistakes that BLIP/deepbooru made and still have to manually caption.

### 프롬프트하는 방식과 동일한 방식으로 캡션을 작성하세요<sup>Caption in the same manner you prompt</sup>

- 캡션 작성과 프롬프트는 관련이 있습니다.
- 일반적으로 어떻게 프롬프트하는지 인식하십시오. 장황한 문장? 짧은 설명? 모호한가요? 자세한가요?
- 프롬프트할 때 사용하는 것과 유사한 스타일과 장황함으로 캡션을 작성하십시오.

- Captioning and prompting are related.
- Recognize how you typically prompt. Verbose sentences? Short descriptions? Vague? Detailed?
- Caption in a similar style and verbosity as you tend to when prompting.

### 개념별로 설정된 구조를 따르십시오<sup>Follow a set structure per concept</sup>

- 구조를 따르면 프로세스가 더 쉬워지고, 객관적인 증거는 없지만 데이터 세트를 설명하는 데 일관된 구조를 사용하면 학습 프로세스에 도움이 될 것이라고 직감합니다.
- 사진에 사용하는 구조와 일러스트레이션에 사용하는 구조가 다를 수 있습니다. 하지만 단일 데이터 세트에 캡션을 추가할 때 구조를 혼합하고 일치시키는 것을 피하십시오.
- 아래에서 일반적으로 사용하는 구조를 설명했으며 예제로 사용할 수 있습니다.

- Following a structure makes the process easier on you, and although I have no objective evidence, my intuition says that using a consistent structure to describe your dataset will benefit the learning process.
- You might have a structure you use for photographs and another structure you use for illustrations. But try to avoid mixing and matching structures when captioning a single dataset.
- have explained the structure I generally use below, which can be used as an example.

### 캡션은 프롬프트에서 사용할 수 있는 변수와 같습니다<sup>Captions are like variables you can use in your prompts</sup>

- 캡션에서 설명하는 모든 것은 프롬프트에서 가지고 놀 수 있는 변수로 생각할 수 있습니다. 여기에는 두 가지 의미가 있습니다.

1. 암시적으로 가르치려는 개념이 아닌 모든 것에 대해 최대한 자세히 설명해야 합니다. **즉, 변수가 되기를 원하는 모든 것을 설명하십시오.** `예: 특정 얼굴을 가르치지만 머리 색깔을 변경할 수 있도록 하려면 각 이미지에서 머리 색깔을 설명하여 "머리 색깔"이 변수 중 하나가 되도록 해야 합니다.`
2. 암시적으로 가르치려는 것(클래스 수준 설명 이상)은 설명하지 않아야 합니다. **즉, 가르치려는 것은 변수가 되어서는 안 됩니다.** `예: 특정 얼굴을 가르치는 경우 큰 코가 있다고 설명해서는 안 됩니다. 코 크기가 변수가 되는 것을 원하지 않기 때문입니다. 하지만 원하는 경우 "얼굴"이라고 캡션할 수 있으며, 이는 훈련하는 모델에 컨텍스트를 제공합니다. 이는 다음 사항에서 설명하는 몇 가지 의미를 갖습니다.`

- Everything you describe in a caption can be thought of as a variable that you can play with in your prompt. This has two implications:

1. You want to describe as much detail as you can about anything that isn't the concept you are trying to implicitly teach. **In other words, describe everything that you want to become a variable**. `Example: If you are teaching a specific face but want to be able to change the hair color, you should describe the hair color in each image so that "hair color" becomes one of your variables.`
2. You don't want to describe anything (beyond a class level description) that you want to be implicitly taught. **In other words, the thing you are trying to teach shouldn't become a variable**. `Example: If you are teaching a specific face, you should not describe that it has a big nose. You don't want the nose size to be variable, because then it isn't that specific face anymore.However, you can still caption "face" if you want to, which provides context to the model you are training. This does have some implications described in the following point.`

### 클래스를 태그로 활용<sup>Leveraging classes as tags</sup>

- 여기에는 두 가지 개념이 있습니다.

1. 일반 클래스 태그를 사용하면 전체 클래스가 훈련 데이터 쪽으로 편향됩니다. 이는 목표에 따라 바람직할 수도 있고 그렇지 않을 수도 있습니다.
2. 일반 클래스 태그를 사용하면 학습 프로세스에 컨텍스트를 제공합니다. 개념적으로 모델이 이미 "얼굴"에 대한 합리적인 근사값을 가지고 있을 때 "얼굴"이 무엇인지 배우는 것이 더 쉽습니다.

- 모델의 전체 클래스를 훈련 이미지 쪽으로 편향시키려면 특정 태그 대신 광범위한 클래스 태그를 사용하십시오. 예: 모델에게 모든 남자가 브래드 피트처럼 보이도록 가르치려면 캡션에 "남자" 태그가 포함되어야 하지만 그보다 더 구체적이어서는 안 됩니다. 이렇게 하면 프롬프트에서 "남자"라는 단어를 사용할 때마다 모델이 브래드 피트처럼 보이는 남자를 생성하도록 영향을 미칩니다. 또한 이렇게 하면 모델이 훈련 중에 이미 알고 있는 "남자"라는 개념을 활용할 수 있습니다.
- 전체 클래스에 대한 훈련의 영향을 줄이려면 특정 태그를 포함하고 클래스 태그를 강조하지 마십시오. 예: 모델에게 "ohwxman"만 브래드 피트처럼 보이도록 가르치고 모든 "남자"가 브래드 피트처럼 보이도록 하지 않으려면 "남자"를 태그로 사용하지 않고 "ohwxman"으로만 태그를 지정합니다. 이렇게 하면 훈련 이미지가 "남자" 태그에 미치는 영향이 줄어들고 훈련 이미지가 "ohwxman"과 강력하게 연결됩니다. 모델은 "ohwxman"에 대해 알고 있는 것을 활용합니다. 이는 사실상 아무것도 아닙니다. \*NOTE 참고\*, 따라서 거의 전적으로 훈련 이미지에서 지식을 쌓아 매우 강력한 연관성을 만듭니다.

\*NOTE\* 이해를 돕기 위해 단순화되었습니다. 이것은 실제로 "ohwx"와 "man"이라는 두 개의 토큰으로 토큰화되지만 이러한 토큰은 훈련 목적으로 강력하게 상관관계가 있으므로 캡션에서 "man"을 토큰으로 사용하여 훈련할 때와 비교하여 "man"의 전체 클래스에 대한 영향을 줄여야 합니다. 모든 수학은 상당히 복잡하며 여기서는 다루지 않습니다.
{: .notice--info}

- There are two concepts here.

1. Using generic class tags will bias that entire class towards your training data. This may or may not be desired depending on what your goals are.
2. Using generic class tags provides context to the learning process. Conceptually, it is easier to learn what a "face" is when the model already has a reasonable approximation of "face".

- If you want to bias the entire class of your model towards your training images, use broad class tags rather than specific tags. `Example: If you want to teach your model that every man should look like Brad Pitt, your captions should contain the tag "man" but should not be more specific than that. This influences your model to produce a Brad Pitt looking man whenever you use the word "man" in your prompt. This also allows your model to draw on and leverage what it already knows about the concept of "man" while it is training.`
- If you want to reduce the impact of your training on the entire class, include specific tags and de-emphasize class tags. `Example: If you want to teach your model that only "ohwxman" should look like Brad Pitt, and you don't want every "man" to look like Brad Pitt you would not use "man" as a tag, only tagging it with "ohwxman". This reduces the impact of your training images on the tag "man", and strongly associates your training images with "ohwxman". Your model will draw on what it knows about "ohwxman", which is practically nothing \*see note\*, thus building up knowledge almost solely from your training images which creates a very strong association.`

\*NOTE\* This is simplified for the sake of understanding. This would actually be tokenized into two tokens, "ohwx" and "man", but these tokens would be strongly correlated for training purposes, which should still reduce the impact on the overall class of "man" when compared to training with "man" as a token in the caption. The math it all is quite complex and well beyond the scope here.
{: .notice--info}

### 관된 캡션 작성<sup>Consistent Captioning</sup>

- 모든 교육에서 일관된 캡션을 사용하십시오. 이렇게 하면 프롬프트할 때 개념을 더 잘 일관되게 호출하는 데 도움이 됩니다. 저는 항상 동일한 캡션을 사용하도록 이를 돕는 프로그램을 사용합니다.
- 데이터 세트에서 일관되지 않은 태그를 사용하면 SD가 개념과 해당 개념에 대한 다양한 표현을 모두 학습해야 하므로 가르치려는 개념을 파악하기가 더 어려워집니다. 단일 용어로 개념을 배우는 것이 훨씬 좋습니다.
- `예를 들어, 다리를 공중에 든 사람이라는 단일 개념을 가르치려는 경우 "공중에 든 다리"와 "든 다리"를 모두 사용하지 않으려고 합니다. 프롬프트에서 이 포즈를 일관되게 호출할 수 있도록 하려면 캡션을 작성하는 한 가지 방법을 선택하세요.`

- Use consistent captions across all of your training. This will help you better consistently invoke your concept when prompting. I use a program to aid me with this, ensuring that I always use the same captions.
- Using inconsistent tags across your dataset is going to make the concept you are trying to teach harder for SD to grasp as you are essentially forcing it to learn both the concept and the different phrasings for that concept. It's much better to have it just learn the concept under a single term.
- `For example, you probably don't want to have both "legs raised in air" and "raised legs" if you are trying to teach one single concept of a person with their legs up in the air. You want to be able to consistently invoke this pose in your prompt, so choose one way to caption it.`

### 반복 방지<sup>Avoid Repetition</sup>

가능하면 반복을 피하십시오. 프롬프트와 마찬가지로 단어를 반복하면 해당 단어의 가중치가 증가합니다.

예를 들어, 저는 종종 "배경"이라는 단어를 너무 많이 반복하는 것을 발견합니다. "배경"이라는 세 개의 태그가 있을 수 있습니다. (`예: 단순 배경, 흰색 배경, 배경의 램프`). 배경의 가중치를 낮추고 싶지만 의도하지 않게 가중치가 꽤 많이 증가했습니다. 이들을 결합하거나 바꿔 말하는 것이 좋습니다. (`예: 램프가 있는 단순한 흰색 배경`).

Try to avoid repetition wherever possible. Similar to prompting, repeating words increases the weighting of those words.

As an example, I often find myself repeating the word "background" too much. I might have three tags that say "background" (`Example: simple background, white background, lamp in background`). Even though I want the background to have low weight, I've unintentionally increased the weighting quite a bit. It would be better to combine these or reword them (`Example: simple white background with a lamp`).

### 순서에 주목하십시오<sup>Take Note of Ordering</sup>

- 다시 말하지만, 프롬프트와 마찬가지로 순서는 태그의 상대적 가중치에 중요합니다.
- 캡션에 일반적으로 사용하는 특정 구조/순서를 사용하면 데이터 세트의 이미지 간에 태그의 상대적 가중치를 유지하는 데 도움이 될 수 있으며, 이는 훈련 프로세스에 도움이 됩니다.
- 표준화된 순서를 사용하면 해당 구조에서 캡션 작성에 익숙해짐에 따라 전체 캡션 작성 프로세스가 더 빨라집니다.

- Again, just like with prompting, order matters for relative weighting of tags.
- Having a specific structure/order that you generally use for captions can help you maintain relative weightings of tags between images in your dataset, which should be beneficial to the training process.
- Having a standardized ordering makes the whole captioning process faster as you become familiar with captioning in that structure.

### 모델의 기존 지식을 활용하십시오<sup>Use Your Models Existing Knowledge to Your Advantage</sup>

- 모델은 이미 괜찮은 결과를 생성하고 프롬프트하는 내용을 합리적으로 이해합니다. 프롬프트에서 이미 잘 작동하는 단어로 캡션을 작성하여 이를 활용하십시오.
- 설명적인 단어를 사용하고 싶지만 너무 모호하거나 틈새 시장에 있는 단어를 사용하면 기존 지식을 많이 활용할 수 없습니다. 예: "sarcrastic"이라고 말할 수도 있고 "mordacious"라고 말할 수도 있습니다. SD는 "sarcastic"이 전달하는 내용에 대한 아이디어가 있지만 "mordacious"가 무엇인지 전혀 모릅니다.
- 반대의 관점에서도 볼 수 있습니다. "mordacious"라는 개념을 가르치려는 경우 "sarcastic"을 전달하는 이미지로 든 데이터 세트가 있을 수 있으며, "sarcastic"과 "mordacious"라는 태그를 나란히 캡션에 추가합니다(상대적 가중치가 가깝도록).

- Your model already produces decent results and reasonably understands what you are prompting. Take advantage of that by captioning with words that already work well in your prompts.
- You want to use descriptive words, but if you use words that are too obscure/niche, you likely can't leverage much of the existing knowledge. Example: you could say "sarcrastic" or you could say "mordacious". SD has some idea of what "sarcastic" conveys, but it likely has no clue what "mordacious" is.
- You can also look at this from the opposite perspective. If you were trying to teach the concept of "mordacious", you might have a dataset full of images that convey "sarcrastic" and caption them with both the tags "sarcastic" and "mordacious" side by side (so that they are close in relative weighting).

## 캡션 작성<sup>CAPTIONING</sup> - 구조<sup>STRUCTURE</sup>

저는 이것을 주로 사람/캐릭터에 사용하므로 판타지 풍경과 같은 것에는 적용되지 않을 수 있지만 약간의 영감을 줄 수 있습니다.

I use this mainly for people / characters, so it might not be quite as applicable to something like fantasy landscapes, but perhaps it can give some inspiration.

다시 한 번 강조하지만, 이것이 캡션을 작성하는 유일하거나 최선의 방법이라고 말하는 것이 아닙니다. 이것은 제 자신의 데이터 세트에서 제 자신의 캡션으로 성공을 거둔 방법일 뿐입니다. 제 목표는 단순히 제가 무엇을 하고 왜 하는지 공유하는 것이며, 여러분은 원하는 만큼 영감을 얻을 수 있습니다.

I want to emphasize again that I am not saying this is the only or best way to caption. This is just how I have found success with my own captions on my own datasets. My goal is simply to share what I do and why, and you are free to take as much or little inspiration from it as you want.

### 일반 형식<sup>General format</sup>

- `<전역> <유형/관점/"...의"> <동작 단어> <주제 설명> <주목할 만한 세부 사항> <배경/위치> <느슨한 연관성>`

- `<Globals> <Type/Perspective/"Of a..."> <Action Words> <Subject Descriptions> <Notable Details> <Background/Location> <Loose Associations>`

### 전역<sup>Globals</sup>

- 여기에서 훈련 중인 개념과 강하게 연결하고 싶은 희귀 토큰(예: "ohwx") 또는 훈련에 중요하고 데이터 세트 전체에서 균일한 모든 것을 고수합니다. `예: 남자, 여자, 애니메이션`

- This is where I would stick a rare token (e.g. "ohwx") that I want heavily associated with the concept I am training, or anything that is both important to the training and uniform across the dataset `Examples: man, woman, anime`

### 유형/관점/"...의<sup>"Type/Perspective/"of a..."</sup>

- 컨텍스트를 제공하기 위한 이미지에 대한 광범위한 설명입니다. 저는 일반적으로 이 작업을 "레이어"로 수행합니다.
- 무엇인가요? `예: 사진, 일러스트레이션, 그림, 초상화, 렌더링, 애니메이션.`
-...의 `예: 여자, 남자, 산, 나무, 숲, 판타지 장면, 도시 풍경`
- X는 어떤 유형인가요(x = 위의 선택)? `예: 전신, 클로즈업, 카우보이 샷, 자른, 필터링된, 흑백, 풍경, 80년대 스타일`
- X의 관점은 무엇인가요? `예: 위에서, 아래에서, 앞에서, 뒤에서, 측면에서, 강제 원근법, 틸트 시프트, 피사계 심도`

- Broad descriptions of the image to supply context. I usually do this in "layers".
- What is it? `Examples: photograph, illustration, drawing, portrait, render, anime.`
- Of a... `Examples: woman, man, mountain, trees, forest, fantasy scene, cityscape`
- What type of X is it (x = choice above)? `Examples: full body, close up, cowboy shot, cropped, filtered, black and white, landscape, 80s style`
- What perspective is X from? `Examples: from above, from below, from front, from behind, from side, forced perspective, tilt-shifted, depth of field`

### 동작 단어<sup>Action Words</sup>

- 주요 주제가 무엇을 하고 있는지 또는 주요 주제에 무슨 일이 일어나고 있는지에 대한 설명 또는 이미지의 개념에 적용할 수 있는 일반적인 동사입니다. 최대한 자세하게 설명하고 원하는 만큼 동사를 조합하여 설명하십시오.
- 목표는 발생하는 모든 동작, 포즈 등을 변수로 만드는 것입니다( "캡션 작성 - 일반"의 3번 지점에 설명됨). 그러면 SD가 특정 동작을 수행하는 주요 개념만 배우는 것이 아니라 일반적인 의미에서 주요 개념을 더 잘 배울 수 있기를 바랍니다.
- `사람을 예로 사용: 서 있는, 앉아 있는, 기대는, 머리 위로 팔을 든, 걷는, 뛰는, 점프하는, 한쪽 팔을 든, 한쪽 다리를 내민, 팔꿈치를 구부린, 포즈를 취하는, 무릎을 꿇는, 스트레칭하는, 앞으로 팔을 뻗은, 무릎을 구부린, 누워 있는, 멀리 보는, 위를 보는, 보는 사람을 보는`
- `꽃을 예로 사용: 시드는, 자라는, 피는, 썩는, 꽃이 피는`

- Descriptions of what the main subject is doing or what is happening to the main subject, or general verbs that are applicable to the concept in the image. Describe in as much detail as possible, with a combination of as many verbs as you want.
- The goal is to make all the actions, poses, and whatever else active that is happening into variables (as described in point 3 of "Captioning – General") so that, hopefully, SD is better able to learn the main concept in a general sense rather than only learning the main concept doing specific actions.
- `Using a person as an example: standing, sitting, leaning, arms above head, walking, running, jumping, one arm up, one leg out, elbows bent, posing, kneeling, stretching, arms in front, knee bent, lying down, looking away, looking up, looking at viewer`
- `Using a flower as an example: wilting, growing, blooming, decaying, blossoming`

### 주제 설명<sup>Subject Descriptions</sup>

- 가르치려는 주요 개념을 설명하지 않고 주제에 대한 최대한 많은 설명입니다. 다시 한 번, 이것을 프롬프트에서 변수가 되기를 원하는 모든 것을 선택하는 것으로 생각하십시오.
- `사람을 예로 사용: 흰색 모자, 파란색 셔츠, 은색 목걸이, 선글라스, 분홍색 신발, 금발 머리, 은색 팔찌, 녹색 재킷, 큰 배낭`
- `꽃을 예로 사용: 분홍색 꽃잎, 녹색 잎, 키가 큰, 곧은, 가시가 있는, 둥근 잎`

- As much description about the subject as possible, without describing the main concept you are trying to teach. Once again, think of this as picking out everything that you want to be a variable in your prompt.
- `Using a person as an example: white hat, blue shirt, silver necklace, sunglasses, pink shoes, blonde hair, silver bracelet, green jacket, large backpack`
- `Using a flower as an example: pink petals, green leaves, tall, straight, thorny, round leaves`

### 주목할 만한 세부 사항<sup>Notable Details</sup>

- 저는 이것을 "배경"이라고 생각하지 않는 모든 것(또는 배경이지만 강조하고 싶은 것)이지만 주요 주제도 아닌 것에 대한 일종의 만능으로 사용합니다.
- 일반적으로 이 지점에 들어가는 캡션 부분은 하나 또는 몇 개의 훈련 이미지에만 고유합니다.
- 저는 주로 Danbooru 스타일의 짧은 캡션을 사용하지만 더 복잡한 것을 설명해야 하는 경우 여기에 넣습니다.
- 예를 들어, 해변 사진에서 "노란색과 파란색 줄무늬 우산이 앞쪽에서 부분적으로 열려 있습니다"라고 넣을 수 있습니다.
- 예를 들어, 초상화에서 "그는 휴대폰을 귀에 대고 있습니다"라고 넣을 수 있습니다.

- I use this as a sort of catch-all for anything that I don't think is quite "background" (or something that is background but I want to emphasize) but also isn't the main subject.
- Normally the part of the caption going in this spot is unique to one or just a few training images.
- I predominately use short captions in Danbooru-style, but if I need to describe something more complex I put it here.
- `For example, in a photo at a beach I might put "yellow and blue striped umbrella partially open in foreground".`
- `For example, in a portrait I might put "he is holding a cellphone to his ear".`

### 배경<sup>Background</sup> / 위치<sup>Location</sup>

- 꽤 자명합니다. 이미지 배경에서 일어나고 있는 일에 대해 최대한 자세하게 설명하십시오. 저는 또한 몇 가지 "레이어"로 이 작업을 수행하여 세부 사항을 좁힙니다. 이는 여러 사진에 캡션을 추가할 때 도움이 됩니다.
- `예를 들어, 해변 사진의 경우 세 가지 "레이어"로 구분하여 다음과 같이 넣을 수 있습니다.`

1. 야외, 해변, 모래, 물, 해안, 일몰
2. 작은 파도, 바다에 있는 배, 모래성, 수건
3. 배는 빨간색과 흰색이고, 모래성 주변에는 해자가 있고, 수건은 노란색 줄무늬가 있는 빨간색입니다.

- Fairly self-explanatory. Be as descriptive as possible about what is happening in the images background. I tend to do this in a few "layers" as well, narrowing down to specifics, which helps when captioning several photos.
- `For example, for a beach photo I might put (separated by the three "layers"):`

1. Outdoors, beach, sand, water, shore, sunset
2. Small waves, ships out at sea, sandcastle, towels
3. the ships are red and white, the sandcastle has a moat around it, the towels are red with yellow stripes

### 느슨한 연관성<sup>Loose Associations</sup>

- 여기에 이미지와 관련된 최종적인 느슨한 연관성을 넣습니다.
- 이것은 제 머리에 떠오르는 모든 것일 수 있습니다. 일반적으로 이미지를 볼 때 느끼는 "감정" 또는 묘사된다고 생각하는 개념입니다. 이미지에 있는 한 실제로 무엇이든 여기에 들어갈 수 있습니다.
- 이것은 느슨한 연관성을 위한 것임을 명심하십시오. 이미지가 어떤 감정을 매우 명확하게 묘사하고 있는 경우 더 높은 가중치를 위해 캡션 시작 부분에 더 가까이 태그를 지정할 수 있습니다.
- `예: 행복한, 슬픈, 즐거운, 희망찬, 외로운, 어두운`

- This is where I put any final loose associations I have with the image.
- This could be anything that pops up in my head, usually "feelings" that I feel when looking at the image or concepts I feel are portrayed, really anything goes here as long as it exists in the image.
- Keep in mind this is for loose associations. If the image is very obviously portraying some feeling, you may want it tagged closer to the start of the caption for higher weighting.
- `For example: happy, sad, joyous, hopeful, lonely, sombre`

## Booru 데이터 세트 태그 관리자<sup>THE BOORU DATASET TAG MANAGER</sup>

데이터 세트가 있습니다. 구조를 결정했습니다. 캡션 작성을 시작할 준비가 되었습니다. 이제 워크플로의 마법 같은 부분인 [BooruDatasetTagManager](https://github.com/starik222/BooruDatasetTagManager)(BDTM)를 사용할 차례입니다. 이 편리한 소프트웨어는 워크플로의 속도를 크게 높여주는 두 가지 매우 중요한 작업을 수행합니다.

1. 태그는 \*\\tags\\list.tag에 미리 로드되며 편집할 수 있습니다. 이렇게 하면 일반적인 태그에 대한 자동 완성 기능이 제공되고 일반적인 태그를 두 번 클릭하여 입력할 필요가 없도록 하는 등의 이점이 있습니다.
2. 이미 사용된 캡션을 표시하여 입력하지 않고도 이미지에 쉽게 추가할 수 있도록 하여 캡션 작성을 쉽게 일관성 있게 유지할 수 있습니다.

추가 보너스로, 건망증이 있을 때 도움이 됩니다. 가끔 한쪽 발에 체중을 대부분 싣고 서 있는 것(하지만 두 발은 모두 땅에 붙이고)을 콘트라포스토라고 하는 것을 잊어버립니다. 하지만 태그로 저장해 두었고 보통 "contra"로 시작한다는 것을 기억합니다. 고맙게도 자동 완성 기능이 있어서 문제를 해결할 수 있습니다. 진지하게, 이러한 모든 태그를 손끝에서 사용할 수 있다는 것은 많은 태그를 기억하거나 다른 탭에서 booru 사이트를 열어 두는 것과는 큰 차이가 있습니다.

You've got a dataset. You've decided on a structure. You're ready to start captioning. Now it's time for the magic part of the workflow: [BooruDatasetTagManager](https://github.com/starik222/BooruDatasetTagManager)(BDTM). This handy piece of software will do two extremely important things for us which greatly speeds up the workflow:

1. Tags are preloaded in \*\\tags\\list.tag, which can be edited. This gives us auto-complete for common tags, allows us to double-click common tags so we don't need to type it out, etc.
2. It enables you to easily be consistent with your captioning by displaying already-used captions so that you can easily add it to an image without typing it out.

As an added bonus, it helps when you're forgetful. Sometimes I forget that standing with most of your weight on one foot (but with both feet on the ground) is called contrapposto. But I have it saved as a tag, and usually remember it starts as "contra". Thankfully auto-complete is there to save the day. Seriously, having all of these tags at your fingertips is a huge difference from trying to remember a bunch of tags or having booru sites open in other tabs.

## 프로세스<sup>THE PROCESS</sup>

1. 모든 이미지를 폴더에 넣은 다음 BDTM UI에서 해당 폴더로 이동하여 이미지가 있는 폴더를 선택합니다.
2. 상단에서 "보기"를 누른 다음 "미리보기 표시"를 눌러 선택한 이미지를 봅니다.
3. 전역적으로 적용할 수 있는 태그가 있는 경우 UI 오른쪽에 추가합니다. 이러한 전역 태그가 나타나는 위치(상단, 하단 또는 목록의 특정 위치)를 선택할 수 있습니다.
4. 왼쪽에서 이미지를 선택하고 태그를 추가하기 시작합니다. 가능한 한 구조를 따르는 것을 잊지 마십시오. 입력하면 태그에 list.tag 파일의 자동 완성 옵션이 표시되며 선택하거나 사용자 지정 태그를 입력할 수 있습니다.
5. 해당 데이터 세트의 어느 곳에서나 사용한 각 태그는 오른쪽( "모든 태그" 아래)에 표시됩니다. "모든 태그" 섹션에서 태그를 두 번 클릭하여 현재 선택한 이미지에 적용할 수 있으므로 많은 시간을 절약하고 데이터 세트 전체에서 태그 일관성을 보장할 수 있습니다.
6. 모든 이미지에 태그가 지정되면 처음으로 돌아가 다시 수행합니다. 이번에는 태그를 보고 원하는 가중치에 따라 적절하게 정렬되었는지 확인하고(필요한 경우 끌어서 순서를 변경할 수 있음), 구조를 따르는지 확인하고, 누락된 태그가 있는지 확인하는 등의 작업을 수행합니다.

그리고 그게 다입니다. 저는 모든 이미지를 주의 깊게 보고 적용 가능하다고 생각되는 태그를 추가하여 프롬프트 구조의 각 범주에 하나 또는 두 개의 태그가 있도록 합니다. 일반적으로 이미지당 8~20개의 태그를 지정하지만 때로는 더 많이 지정할 수도 있습니다.

시간이 지남에 따라 제공된 list.tag 파일을 편집하여 사용하지 않을 많은 태그를 제거하고 자주 사용하는 태그를 추가하여 전체 프로세스를 더 쉽게 만들었습니다.

1. Place all of your images in a folder and then navigate there in the BDTM UI, selecting the folder with your images.
2. At the top, press "View" and then "Show preview" to see the selected image.
3. If you have any globally applicable tags, add them on the right side of the UI. You can select where these global tags appear (top, bottom, or at a specific position in the list).
4. Select your image on the left and begin adding tags, remembering to follow you structure as best as possible. As you type, the tags will show auto-complete options from the list.tag file which you can select, or you can type in your own custom ones.
5. Each tag you have used anywhere in that dataset will show on the right side (under "All tags"). You can double-click a tag from the "All tags" section to apply it to the currently selected image, saving tons of time and ensuring tag consistency across your dataset
6. Once all of your images are tagged, go back to the start and do it again. This time look at your tags and make sure they are ordered appropriately according to the weighting you want (you can drag them to reorder if necessary), make sure they follow your structure, check for missing tags, etc.

And that's it. I patiently look at every image and add any tags I think are applicable, aiming to have at least one to two tags in each of the categories of my prompt structure. I usually have between 8 and 20 tags per image, though sometimes I might have even more.

Over time, I have edited the provided list.tag file removing many of the tags I'll never use and adding a bunch of tags that I use frequently, making the whole process even easier.

## 단일 이미지의 전체 예<sup>FULL EXAMPLE OF A SINGLE IMAGE</sup>

이것은 safebooru에서 선택한 단일 이미지에 캡션을 추가하는 방법의 예입니다. *이 이미지의 스타일을 훈련하고 "ohwxStyle" 태그와 연결하고 싶다고 가정하고 데이터 세트 내에 이 스타일의 이미지가 많다고 가정합니다.*

This is an example of how I would caption a single image I picked off of safebooru. *We will assume that I want to train the style of this image and associate it with the tag "ohwxStyle", and we will assume that I have many images in this style within my dataset.*

![sample_image]({{site.baseurl}}/assets/images/2025-02-19-captioning-datasets/sample_image.png){: .align-center}  

- 전역: ohwxStyle
- 유형/관점/...의: 애니메이션, 그림, 젊은 여성, 전신 샷, 측면에서
- 동작 단어: 앉아 있는, 보는 사람을 보는, 미소 짓는, 머리 기울이기, 휴대폰 들고 있는, 눈 감은
- 주제 설명: 짧은 갈색 머리, 어두운 가장자리가 있는 옅은 분홍색 드레스, 무릎에 봉제 동물, 갈색 슬리퍼
- 주목할 만한 세부 사항: 창문을 통한 햇빛이 조명 소스로 사용됨
- 배경/위치: 갈색 소파, 소파에 빨간색 무늬가 있는 천, 나무 바닥, 벽에 흰색 물 얼룩이 있는 페인트, 배경에 냉장고, 조리대에 커피 머신, 소파 앞에 테이블, 테이블에 바나나와 커피포트, 벽에 화이트보드, 벽에 시계, 바닥에 닭 봉제 동물
- 느슨한 연관성: 음울한 환경

전체: ohwxStyle, 애니메이션, 그림, 젊은 여성, 전신 샷, 측면에서, 앉아 있는, 보는 사람을 보는, 미소 짓는, 머리 기울이기, 휴대폰 들고 있는, 눈 감은, 짧은 갈색 머리, 어두운 가장자리가 있는 옅은 분홍색 드레스, 무릎에 봉제 동물, 갈색 슬리퍼, 창문을 통한 햇빛이 조명 소스로 사용됨, 갈색 소파, 소파에 빨간색 무늬가 있는 천, 나무 바닥, 벽에 흰색 물 얼룩이 있는 페인트, 배경에 냉장고, 조리대에 커피 머신, 소파 앞에 테이블, 테이블에 바나나와 커피포트, 벽에 화이트보드, 벽에 시계, 바닥에 닭 봉제 동물, 음울한 환경

- Globals: ohwxStyle
- Type/Perspective/Of a: anime, drawing, of a young woman, full body shot, from side
- Action words: sitting, looking at viewer, smiling, head tilt, holding a phone, eyes closed
- Subject description: short brown hair, pale pink dress with dark edges, stuffed animal in lap, brown slippers
- Notable details: sunlight through windows as lighting source
- Background/location: brown couch, red patterned fabric on couch, wooden floor, white water-stained paint on walls, refrigerator in background, coffee machine sitting on a countertop, table in front of couch, bananas and coffee pot on table, white board on wall, clock on wall, stuffed animal chicken on floor
- Loose associations: dreary environment

All together: ohwxStyle, anime, drawing, of a young woman, full body shot, from side, sitting, looking at viewer, smiling, head tilt, holding a phone, eyes closed, short brown hair, pale pink dress with dark edges, stuffed animal in lap, brown slippers, sunlight through windows as lighting source, brown couch, red patterned fabric on couch, wooden floor, white water-stained paint on walls, refrigerator in background, coffee machine sitting on a countertop, table in front of couch, bananas and coffee pot on table, white board on wall, clock on wall, stuffed animal chicken on floor, dreary environment

가장 좋은 점은 BDTM에서 이러한 모든 "전역" 항목을 모든 이미지에 적용되도록 설정할 수 있다는 것입니다. 또한 이러한 모든 태그를 두 번 클릭하여 사용할 수 있으므로 다음 이미지도 전신 샷, 측면에서, 앉아 있는 경우... 두 번 클릭하기만 하면 됩니다. 다시 입력하는 것보다 훨씬 쉽습니다.

The best part is, I can set all of those "global" ones in BDTM to apply to all of my images. I've now also got all of those tags ready just a double-click away, so if my next image is also a full body shot, from the side, sitting... I just double-click it. Much easier than typing it out again.

## 훈련<sup>TRAIN</sup>

훈련을 시작할 시간입니다! 여기에서 쓸 내용은 실험 외에는 많지 않습니다. 훈련과 관련하여 정해진 단계 수나 보장된 결과는 없습니다. 그래서 실험하는 것이 재미있습니다. 이제 매우 높은 품질의 데이터 세트가 있다는 것을 알고 실험할 수 있으므로 적절한 훈련 설정을 실제로 연마할 수 있습니다.

Time to start training! I don't have much to write here other than experiment. There is no golden number of steps or guaranteed results when it comes to training. That's why it's fun to experiment. And now you can experiment knowing that you have an extremely high quality dataset, allowing you to really hone-in on the appropriate training settings.

## 기타 생각과 참고 자료<sup>MISC THOUGHTS AND REFERENCES</sup>

- 저는 항상 우리가 학습 과정을 부드럽게 안내할 뿐 통제하지 않는다는 것을 상기시키려고 노력합니다. 캡션은 학습 과정을 올바른 방향으로 안내하는 데 도움이 되지만 캡션이 절대적인 것은 아닙니다. 캡션이 지정되지 않은 이미지의 항목에 대한 추론이 이루어지고 의도하지 않은 이미지 부분과 태그 간의 연관성이 생성되는 등의 작업이 수행됩니다. 안내는 하되 훈련과 이미지 품질도 신뢰하십시오.
- Danbooru/safebooru 태그는 훌륭합니다. 제 말은 의미가 없는 쓰레기 태그가 많지만 예를 들어 Danbooru 위키에서 태그 그룹 "자세"를 살펴보십시오. 다양한 팔 위치, 다리 위치 등에 대한 수십 가지의 특정 단어가 있습니다. danbooru 태그와 위키를 크롤링하여 스타일/포즈/조명/무엇이든 설명하는 데 필요한 특정 단어를 찾을 수 있습니다. 발가락이 아래로 향한 발레리나 스타일의 발로 포즈를 취한 사람을 항상 원했을 것입니다. 글쎄, 그것은 발바닥 굴곡이라고 합니다. danbooru 태그 덕분입니다.

- I always try to remind myself that we are just gently guiding the learning process, not controlling it. Your captions help point the learning process in the right direction, but the captions are not absolute. Inferences will be made on things in the image that weren't captioned, associations will be made between tags and parts of the image you didn't intend, etc. Try to guide, but trust in the training and the quality of your images as well.
- Danbooru/safebooru tags are great. I mean, there's a lot of trash ones that hold no meaning, but take a look at the Danbooru wiki for tag group "Posture" as an example. Dozens of specific words for different arm positions, leg positions, etc. You might just find that one specific word you've been searching for that describes the style/pose/lighting/whatever by crawling through the danbooru tags and wiki. Maybe you've always wanted someone posing with that ballerina style foot where the toes are pointed downwards. Well it's called plantar flexion; thanks danbooru tags.