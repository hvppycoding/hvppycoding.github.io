---
title: "Ch 1. 허깅페이스 실습 코드 학습 - 개요"
excerpt: ""
date: 2025-02-11 01:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Hugging Face
---

허깅페이스는 다양한 머신러닝 모델을 공유하는 플랫폼이다.  

## GPT2 From Scratch

시간이 있으면 [이 블로그 글](https://jalammar.github.io/illustrated-gpt2)을 읽어보자.  
우리는 허깅 페이스를 사용하는 대신, Tensorflow로 GPT2를 구현해볼 것이다.  
구글 코랩에서 GPT2를 학습시킬 수는 없다. 만약 A100과 같은 GPU를 사용할 수 있다면 학습할 수 있다.  

```python
import tensorflow as tf
from tensorflow.keras.layers import Input, Embedding, Dense, LayerNormalization
from tensorflow.keras.layers import MultiHeadAttention, Dropout, LayerNormalization
from tensorflow.keras.models import Model


class GPT2(tf.keras.Model):
    def __init__(
        self,
        vocab_size: int,
        max_sequence_length: int,
        num_layers: int = 12,
        num_heads: int = 8,
        d_model: int = 512,
        dff: int = 1024,
        dropout_rate: float = 0.1,
    ):
        """초기화 메서드 - 모델의 구조와 매개 변수를 설정한다.

        Args:
            vocab_size (int): 단어 집합의 크기
            max_sequence_length (int): 시퀀스의 최대 길이
            num_layers (int, optional): 인코더 레이어의 개수 (기본값: 12)
            num_heads (int, optional): 어텐션 헤드의 개수 (기본값: 8)
            d_model (int, optional): 모델의 차원 크기 (기본값: 512)
            dff (int, optional): 피드포워드 신경망의 차원 크기 (기본값: 1024)
            dropout_rate (float, optional): 드롭아웃 비율 (기본값: 0.1)
        """
        super(GPT2, self).__init__()
        self.num_layers = num_layers
        self.d_model = d_model

        # Embedding 객체를 생성하여 단어를 벡터로 변환하는 역할을 수행한다.
        # 단어 집합의 크기는 vocab_size이고, 임베딩 벡터의 차원은 d_model이다.
        self.embedding = Embedding(vocab_size, d_model)

        # positional_encoding() 메서드를 호출하여 시퀀스 위치 정보를 인코딩하는 값을 저장한다.
        # 입력으로는 max_sequence_length와 d_model이 사용된다.
        self.positional_encoding = self.positional_encoding(
            max_sequence_length, d_model
        )

        # EncoderLayer 객체 리스트를 생성한다.
        self.encoder_layers = [
            EncoderLayer(d_model, num_heads, dff, dropout_rate)
            for _ in range(num_layers)
        ]
        self.dropout = Dropout(dropout_rate)

        # 예측 결과를 출력하는데 사용되는 Dense 객체를 생성한다.
        # 출력의 크기는 vocab_size이다.
        self.final_layer = Dense(vocab_size)

    def call(self, inputs):
        """Transformer 모델의 호출을 담당하는 메서드이다. 입력 데이터 inputs를 받아서
        출력을 반환한다.

        Args:
            inputs : _description_

        Returns:
            _type_: _description_
        """

        # 입력 시퀀스의 길이
        sequence_length = tf.shape(inputs)[1]
        # 입력 데이터를 기반으로 패딩 마스크 생성
        mask = self.create_padding_mask(inputs)

        # 입력 데이터를 임베딩
        x = self.embedding(inputs)
        # 임베딩 벡터에 d_model의 루트 값을 곱하여 벡터의 크기를 조정
        x *= tf.math.sqrt(tf.cast(self.d_model, tf.float32))
        # 포지셔널 인코딩을 임벧이 벡터에 추가
        x += self.positional_encoding[:, :sequence_length, :]
        # 드롭아웃을 적용하여 과적합 방지
        x = self.dropout(x)

        # 인코더 레이어를 통해 입력 데이터 처리
        for i in range(self.num_layers):
            x = self.encoder_layers[i](x, mask)

        # 최종 출력
        output = self.final_layer(x)
        return output

    def positional_encoding(self, sequence_length, d_model):
        position = tf.expand_dims(
            tf.range(0, sequence_length, dtype=tf.float32), axis=1
        )
        div_term = tf.pow(
            10000, 2 * tf.range(0, d_model, 2, dtype=tf.float32) / d_model
        )
        pe = tf.concat(
            [tf.sin(position / div_term), tf.cos(position / div_term)], axis=1
        )
        return tf.expand_dims(pe, axis=0)

    def create_padding_mask(self, sequence):
        mask = tf.cast(tf.math.equal(sequence, 0), tf.float32)
        return mask[:, tf.newaxis, tf.newaxis, :]


class EncoderLayer(tf.keras.layers.Layer):
    def __init__(self, d_model, num_heads, dff, dropout_rate):
        super(EncoderLayer, self).__init__()
        self.mha = MultiHeadAttention(d_model, num_heads)
        self.dropout1 = Dropout(dropout_rate)
        self.ln1 = LayerNormalization(epsilon=1e-6)

        self.ffn = self.point_wise_feed_forward_network(d_model, dff)
        self.dropout2 = Dropout(dropout_rate)
        self.ln2 = LayerNormalization(epsilon=1e-6)

    def call(self, inputs, mask):
        attention_output = self.mha(inputs, inputs, attention_mask=mask)
        attention_output = self.dropout1(attention_output)
        output1 = self.ln1(inputs + attention_output)

        ffn_output = self.ffn(output1)
        ffn_output = self.dropout2(ffn_output)
        output2 = self.ln2(output1 + ffn_output)

        return output2

    def point_wise_feed_forward_network(self, d_model, dff):
        """포인트와이즈 피드포워드 신경망을 생성한다.

        Args:
            d_model (int): 모델의 차원
            dff (int): 피드포워드 신경망의 차원

        Returns:
            tf.keras.Sequential: 입력 차원을 가진 Dense 레이어와 출력 차원을 가진
            Dense 레이어로 이루어진 Sequential 모델을 반환한다.
        """
        return tf.keras.Sequential([Dense(dff, activation="relu"), Dense(d_model)])


# Example usage (continued):

# Define hyperparameters
num_layers = 12
num_heads = 8
d_model = 512
dff = 1024
dropout_rate = 0.1

# Create an instance of the GPT-2 model
gpt2 = GPT2(
    vocab_size, max_sequence_length, num_layers, num_heads, d_model, dff, dropout_rate
)

# Compile the model
gpt2.compile(
    optimizer="adam",
    loss=tf.keras.losses.SparseCategoricalCrossentropy(from_logits=True),
)

# Train the model
gpt2.fit(train_dataset, epochs=10)

# Generate text using the trained model
input_sequence = "Once upon a time"
generated_text = generate_text(gpt2, input_sequence, max_length=100)

print(generated_text)
```

## GPT2 with HuggingFace transformers

`pip install transformers` 명령어로 transformers 라이브러리를 설치하여 사용할 수 있다. 

```python
from transformers import TFGPT2LMHeadModel, GPT2Tokenizer

# transformers 라이브러리에서 TFGPT2LMHeadModel과 GPT2Tokenizer를 가져온다.
model_name = 'gpt2'

# 미리 훈련된 GPT-2 모델과 토크나이저를 로드한다.
# model_name에는 사용할 모델의 이름을 지정한다.
model = TFGPT2LMHeadModel.from_pretrained(model_name)
tokenizer = GPT2Tokenizer.from_pretrained(model_name)

# 이 함수는 입력 텍스트와 최대 길이를 인자로 받아서 GPT-2 모델을 사용해 텍스트를 생성한다.
def generate_text(input_text, max_length=100):
    # input_text를 토크나이저로 인코딩하여 input_ids를 생성한다.
    print("input_text:", input_text)
    input_ids = tokenizer.encode(input_text, return_tensors='tf')
    print("input_ids:", input_ids)
    # 모델의 generate 메서드를 사용해 텍스트를 생성한다. 
    # 이 output도 정수 형식일 것이므로 토크나이저를 사용해 디코딩하여 텍스트로 변환한다.
    output = model.generate(input_ids, max_length=max_length, num_return_sequences=1)
    print("output:", output)
    generated_text = tokenizer.decode(output[0], skip_special_tokens=True)
    print("generated_text:", generated_text)
    return generated_text

# Example usage:
input_sequence = "Once upon a time"
generated_text = generate_text(input_sequence, max_length=100)
```

실행 결과  

```text
input_text: Once upon a time
input_ids: tf.Tensor([[7454 2402  257  640]], shape=(1, 4), dtype=int32)
output: tf.Tensor(
[[7454 2402  257  640   11  262  995  373  257 1295  286 1049 8737  290
  1049 3514   13  383  995  373  257 1295  286 1049 3514   11  290  262
   995  373  257 1295  286 1049 3514   13  383  995  373  257 1295  286
  1049 3514   11  290  262  995  373  257 1295  286 1049 3514   13  383
   995  373  257 1295  286 1049 3514   11  290  262  995  373  257 1295
   286 1049 3514   13  383  995  373  257 1295  286 1049 3514   11  290
   262  995  373  257 1295  286 1049 3514   13  383  995  373  257 1295
   286 1049]], shape=(1, 100), dtype=int32)
generated_text: Once upon a time, the world was a place of great beauty and great danger. The world was a place of great danger, and the world was a place of great danger. The world was a place of great danger, and the world was a place of great danger. The world was a place of great danger, and the world was a place of great danger. The world was a place of great danger, and the world was a place of great danger. The world was a place of great
```
