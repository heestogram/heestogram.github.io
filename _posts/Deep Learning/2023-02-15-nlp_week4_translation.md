---
title: "[NLP] Attention으로 불어-영어 번역"
excerpt: "pytorch로 버나다우 attention을 구현해보고 불어를 입력하여 영어로 번역해보자"
toc: true
toc_label: "목차"
toc_sticky: true

tags: [Deep Learning, NLP, attention, pytorch]

published: true

categories:
  - DL

date: 2023-02-08 21:00:00
last_modified_at: 2023-02-08 21:00:00
---


<br>

<div class="notice--primary" markdown="1">
💡 교내 학회 NLP 분반에서 학습한 내용을 정리한 포스팅입니다.
</div>

<br>

## 1. 프랑스어-영어 데이터셋 가져오기




```python
import tensorflow as tf

import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
from sklearn.model_selection import train_test_split

import unicodedata
import re
import numpy as np
import os
import io
import time
import pandas as pd
```


```python
from google.colab import drive
drive.mount('/content/drive')
```

    Mounted at /content/drive
    


```python
path_to_file = '/content/drive/MyDrive/kubig/fra-eng/fra.txt'
```

<br>

## 2. 전처리, 토큰화



```python
# 유니코드 파일을 아스키 코드 파일로 변환합니다.
def unicode_to_ascii(s):
  return ''.join(c for c in unicodedata.normalize('NFD', s)
      if unicodedata.category(c) != 'Mn')


def preprocess_sentence(w):
  w = unicode_to_ascii(w.lower().strip())

  # 단어와 단어 뒤에 오는 구두점(.)사이에 공백을 생성합니다.
  # 예시: "he is a boy." => "he is a boy ."
  # 참고:- https://stackoverflow.com/questions/3645931/python-padding-punctuation-with-white-spaces-keeping-punctuation
  w = re.sub(r"([?.!,¿])", r" \1 ", w)
  w = re.sub(r'[" "]+', " ", w)

  # (a-z, A-Z, ".", "?", "!", ",")을 제외한 모든 것을 공백으로 대체합니다.
  w = re.sub(r"[^a-zA-Z?.!,¿]+", " ", w)

  w = w.strip()

  # 모델이 예측을 시작하거나 중단할 때를 알게 하기 위해서
  # 문장에 start와 end 토큰을 추가합니다.
  w = '<start> ' + w + ' <end>'
  return w
```


```python
en_sentence = u"May I borrow this book?"
fr_sentence = u"Puis-je emprunter ce livre"
print(preprocess_sentence(en_sentence))
print(preprocess_sentence(fr_sentence).encode('utf-8'))
```

    <start> may i borrow this book ? <end>
    b'<start> puis je emprunter ce livre <end>'
    

정의한 전처리 함수를 데이터셋에 적용한다. lines 객체에 데이터셋을 넣어주고 이를 반복문으로 순회하며 전처리를 한다. 그리고선 영어와 프랑스어어를 `zip`함수를 통해 한 쌍으로 만들어준다.

`num_examples` 파라미터는 데이터셋의 크기를 제한하려고 만든 파라미터이다.




```python
# 1. 문장에 있는 억양을 제거합니다.
# 2. 불필요한 문자를 제거하여 문장을 정리합니다.
# 3. 다음과 같은 형식으로 문장의 쌍을 반환합니다: [영어, 프랑스어]
def create_dataset(path, num_examples):
  lines = io.open(path, encoding='UTF-8').read().strip().split('\n')

  word_pairs = [[preprocess_sentence(w) for w in l.split('\t')]  for l in lines[:num_examples]]

  return zip(*word_pairs)
```

기존에 있는 코드를 쓰면 `too many values to unpack`이라는 오류가 뜨는데, txt 파일을 들여다보니 split을 시키면 총 3개의 return 값이 나와서 발생한 오류같다. (예제 코드에서 쓰인 파일은 약간 다른 형식이었나보다.) 그래서 함수의 return을 객체로 받아줄 때 en(영어), fr(프랑스어) 말고 임의로 c를 하나 넣어서 디버깅했다.


```python
en, fr, c = create_dataset(path_to_file, None) # c는 안쓰는 값이다
print(en[-1])
print(fr[-1])
```

    <start> it may be impossible to get a completely error free corpus due to the nature of this kind of collaborative effort . however , if we encourage members to contribute sentences in their own languages rather than experiment in languages they are learning , we might be able to minimize errors . <end>
    <start> il est peut etre impossible d obtenir un corpus completement denue de fautes , etant donnee la nature de ce type d entreprise collaborative . cependant , si nous encourageons les membres a produire des phrases dans leurs propres langues plutot que d experimenter dans les langues qu ils apprennent , nous pourrions etre en mesure de reduire les erreurs . <end>
    

fit_on_texts(lang) 함수로 lang이라는 문자열을 토큰화해준다. 토큰화된 각 단어는 정수에 매핑된다. texts_to_sequences(lang)을 사용하면 토큰화된 정수들을 시퀀스 형태로 변환해준다. pad_sequences()를 사용해서 길이가 다른 각 시퀀스들의 길이를 동일하게 맞추는 패딩 과정을 거친다.

아래는 어떻게 토큰화가 이루어지는지 예시를 보여준다.


```python
from tensorflow.keras.preprocessing.text import Tokenizer

sentences = [
  'I love my dog',
  'I love my cat',
  'You love my dog!'
]

tokenizer = Tokenizer(num_words = 100)
tokenizer.fit_on_texts(sentences)
word_index = tokenizer.word_index
print(word_index) # 각 단어가 토큰 단위로 숫자에 매핑된다.
```

    {'love': 1, 'my': 2, 'i': 3, 'dog': 4, 'cat': 5, 'you': 6}
    


```python
sequences = tokenizer.texts_to_sequences(sentences)

print(sequences) # 매핑된 정수가 시퀀스 형태로 나열된다.
```

    [[3, 1, 2, 4], [3, 1, 2, 5], [6, 1, 2, 4]]
    


```python
def tokenize(lang):
  lang_tokenizer = tf.keras.preprocessing.text.Tokenizer(
      filters='')
  lang_tokenizer.fit_on_texts(lang)

  tensor = lang_tokenizer.texts_to_sequences(lang)

  tensor = tf.keras.preprocessing.sequence.pad_sequences(tensor,
                                                         padding='post')

  return tensor, lang_tokenizer
```

위에서 정의한 전처리 함수 create_dataset과 토큰화 함수 tokenize를 load_dataset이라는 하나의 함수로 통합해준다. 여기서 input은 프랑스어, target은 영어이다.


```python
def load_dataset(path, num_examples=None):
  # 전처리된 타겟 문장과 입력 문장 쌍을 생성합니다.
  targ_lang, inp_lang, c = create_dataset(path, num_examples) # 전처리

  input_tensor, inp_lang_tokenizer = tokenize(inp_lang) # 전처리한 것을 토큰화 후 시퀀스 형태로 텐서 만들기
  target_tensor, targ_lang_tokenizer = tokenize(targ_lang)

  return input_tensor, target_tensor, inp_lang_tokenizer, targ_lang_tokenizer
```

훈련 데이터의 문장이 10만개를 넘어가면 런타임이 너무 오래 걸릴 수 있으므로 `num_examples`를 3만으로 지정하여 문장 수를 3만개로 줄인다.



```python
# 언어 데이터셋을 아래의 크기로 제한하여 훈련과 검증을 수행합니다.
num_examples = 30000
input_tensor, target_tensor, inp_lang, targ_lang = load_dataset(path_to_file, num_examples)

# 타겟 텐서와 입력 텐서의 최대 길이를 계산합니다.
max_length_targ, max_length_inp = target_tensor.shape[1], input_tensor.shape[1]
```


```python
# 타겟 텐서(영어)와 입력 텐서(프랑스어)의 최대 길이는 10, 17이다.
print(max_length_targ, max_length_inp)
```

    10 17
    


```python
# 훈련 집합과 검증 집합을 80대 20으로 분리합니다.
input_tensor_train, input_tensor_val, target_tensor_train, target_tensor_val = train_test_split(input_tensor, target_tensor, test_size=0.2)

# 훈련 집합과 검증 집합의 데이터 크기를 출력합니다.
print(len(input_tensor_train), len(target_tensor_train), len(input_tensor_val), len(target_tensor_val))
```

    24000 24000 6000 6000
    

아래 함수는 특정 문장이 어떤 정수 시퀀스로 구성되어있고, 그것이 어떤 단어와 매핑되어있는지를 보여주는 함수이다. 당연히 패딩으로 인해 0이 많을 것이므로 0은 스킵하고 양의 정수만 거르는 조건문을 넣었다.



```python
def convert(lang, tensor):
  for t in tensor:
    if t!=0:
      print ("%d ----> %s" % (t, lang.index_word[t]))
```


```python
print ("Input Language; index to word mapping")
convert(inp_lang, input_tensor_train[-1])
print ()
print ("Target Language; index to word mapping")
convert(targ_lang, target_tensor_train[-1])
```

    Input Language; index to word mapping
    1 ----> <start>
    26 ----> ce
    27 ----> n
    5 ----> est
    13 ----> pas
    477 ----> marrant
    3 ----> .
    2 ----> <end>
    
    Target Language; index to word mapping
    1 ----> <start>
    22 ----> this
    10 ----> is
    33 ----> not
    191 ----> funny
    3 ----> .
    2 ----> <end>
    

<br>

## 3. 데이터셋 입력 파이프라인 빌드
`tf.data`를 사용해 데이터셋 입력 파이프 라인 빌드를 할 수 있다.


```python
BUFFER_SIZE = len(input_tensor_train)
BATCH_SIZE = 64
steps_per_epoch = len(input_tensor_train)//BATCH_SIZE
embedding_dim = 256
units = 1024
vocab_inp_size = len(inp_lang.word_index)+1
vocab_tar_size = len(targ_lang.word_index)+1

dataset = tf.data.Dataset.from_tensor_slices((input_tensor_train, target_tensor_train)).shuffle(BUFFER_SIZE)
```

input tensor의 경우 크기가 16인 텐서들로 쪼개져서 준비가 된다. 16은 input tensor의 최대 길이다. 마찬가지로 target tensor는 크기가 11인 텐서들로 쪼개진다.


```python
print(dataset)
```

    <ShuffleDataset element_spec=(TensorSpec(shape=(17,), dtype=tf.int32, name=None), TensorSpec(shape=(10,), dtype=tf.int32, name=None))>
    

원하는 batch size로 데이터셋을 iterator하게 만들어준다.



```python
dataset = dataset.batch(BATCH_SIZE, drop_remainder=True)
```


```python
print(dataset)
```

    <BatchDataset element_spec=(TensorSpec(shape=(64, 17), dtype=tf.int32, name=None), TensorSpec(shape=(64, 10), dtype=tf.int32, name=None))>
    


```python
example_input_batch, example_target_batch = next(iter(dataset))
example_input_batch.shape, example_target_batch.shape
```




    (TensorShape([64, 17]), TensorShape([64, 10]))


<br>

## 4. attention 구현


예제 코드에서 attention에 대한 소개가 나와있지만, 위키독스에서 더 명료하게 개념을 알 수 있다.

https://wikidocs.net/22893

우선 인코더의 구조를 보면 Embedding layer와 GRU layer로 구성된 것을 확인할 수 있다.



```python
class Encoder(tf.keras.Model):
  def __init__(self, vocab_size, embedding_dim, enc_units, batch_sz):
    super(Encoder, self).__init__()
    self.batch_sz = batch_sz
    self.enc_units = enc_units
    self.embedding = tf.keras.layers.Embedding(vocab_size, embedding_dim)
    self.gru = tf.keras.layers.GRU(self.enc_units,
                                   return_sequences=True, # True이면 매 시점마다 은닉 상태를 출력
                                   return_state=True, # True이면 마지막 시점의 은닉 상태를 출력
                                   recurrent_initializer='glorot_uniform')

  def call(self, x, hidden):
    x = self.embedding(x)
    output, state = self.gru(x, initial_state = hidden)
    return output, state

  # 첫번째 은닉 상태를 0으로 찬 행렬로 초기화화해주는 것
  def initialize_hidden_state(self):
    return tf.zeros((self.batch_sz, self.enc_units))
```

Encoder class의 출력값으로 attention 매커니즘에 입력하려는 output과 state가 준비가 되었다. 입력은 (batch_size, max_length, hidden_size)의 형태로 이루어진 인코더 결과와 (batch_size, hidden_size)쌍으로 이루어진 인코더 은닉 상태(hidden state)이 된다.



```python
encoder = Encoder(vocab_inp_size, embedding_dim, units, BATCH_SIZE)

# 샘플 입력
sample_hidden = encoder.initialize_hidden_state()
sample_output, sample_hidden = encoder(example_input_batch, sample_hidden)
print ('Encoder output shape: (batch size, sequence length, units) {}'.format(sample_output.shape))
print ('Encoder Hidden state shape: (batch size, units) {}'.format(sample_hidden.shape))
```

    Encoder output shape: (batch size, sequence length, units) (64, 17, 1024)
    Encoder Hidden state shape: (batch size, units) (64, 1024)
    

다음은 바다나우 어텐션을 구현현할 것이다.

바나다우 어텐션 참고 위키독스 https://wikidocs.net/73161


```python
class BahdanauAttention(tf.keras.layers.Layer):
  def __init__(self, units):
    super(BahdanauAttention, self).__init__()
    self.W1 = tf.keras.layers.Dense(units)
    self.W2 = tf.keras.layers.Dense(units)
    self.V = tf.keras.layers.Dense(1)

  def call(self, query, values):
    # 쿼리 은닉 상태(query hidden state)는 (batch_size, hidden size,)쌍으로 이루어져 있습니다.
    # query_with_time_axis은 (batch_size, 1, hidden size)쌍으로 이루어져 있습니다.
    # values는 (batch_size, max_len, hidden size)쌍으로 이루어져 있습니다.
    # 스코어(score)계산을 위해 덧셈을 수행하고자 시간 축을 확장하여 아래의 과정을 수행합니다.
    query_with_time_axis = tf.expand_dims(query, 1)

    # score는 (batch_size, max_length, 1)쌍으로 이루어져 있습니다.
    # score를 self.V에 적용하기 때문에 마지막 축에 1을 얻습니다.
    # self.V에 적용하기 전에 텐서는 (batch_size, max_length, units)쌍으로 이루어져 있습니다.
    score = self.V(tf.nn.tanh(
        self.W1(query_with_time_axis) + self.W2(values)))

    # attention_weights는 (batch_size, max_length, 1)쌍으로 이루어져 있습니다. 
    # attention_weigths를 종합하여 attention distribution이라고 부른다.
    attention_weights = tf.nn.softmax(score, axis=1)

    # 덧셈이후 컨텍스트 벡터(context_vector)는 (batch_size, hidden_size)쌍으로 이루어져 있습니다.
    context_vector = attention_weights * values
    context_vector = tf.reduce_sum(context_vector, axis=1)

    return context_vector, attention_weights
```


```python
attention_layer = BahdanauAttention(10)
attention_result, attention_weights = attention_layer(sample_hidden, sample_output)

print("Attention result shape: (batch size, units) {}".format(attention_result.shape))
print("Attention weights shape: (batch_size, sequence_length, 1) {}".format(attention_weights.shape))
```

    Attention result shape: (batch size, units) (64, 1024)
    Attention weights shape: (batch_size, sequence_length, 1) (64, 17, 1)
    

decoder를 정의한다.


```python
class Decoder(tf.keras.Model):
  def __init__(self, vocab_size, embedding_dim, dec_units, batch_sz):
    super(Decoder, self).__init__()
    self.batch_sz = batch_sz
    self.dec_units = dec_units
    self.embedding = tf.keras.layers.Embedding(vocab_size, embedding_dim)
    self.gru = tf.keras.layers.GRU(self.dec_units,
                                   return_sequences=True,
                                   return_state=True,
                                   recurrent_initializer='glorot_uniform')
    self.fc = tf.keras.layers.Dense(vocab_size)

    # 어텐션을 사용합니다.
    self.attention = BahdanauAttention(self.dec_units)

  def call(self, x, hidden, enc_output):
    # enc_output는 (batch_size, max_length, hidden_size)쌍으로 이루어져 있습니다.
    context_vector, attention_weights = self.attention(hidden, enc_output)

    # 임베딩층을 통과한 후 x는 (batch_size, 1, embedding_dim)쌍으로 이루어져 있습니다.
    x = self.embedding(x)

    # 컨텍스트 벡터와 임베딩 결과를 결합한 이후 x의 형태는 (batch_size, 1, embedding_dim + hidden_size)쌍으로 이루어져 있습니다.
    x = tf.concat([tf.expand_dims(context_vector, 1), x], axis=-1)

    # 위에서 결합된 벡터를 GRU에 전달합니다.
    output, state = self.gru(x)

    # output은 (batch_size * 1, hidden_size)쌍으로 이루어져 있습니다.
    output = tf.reshape(output, (-1, output.shape[2]))

    # output은 (batch_size, vocab)쌍으로 이루어져 있습니다.
    x = self.fc(output)

    return x, state, attention_weights
```

`vocab_tar_size`는 target language의 vocab size 개수이다.

`tf.random.uniform((BATCH_SIZE, 1))`로 batch_size 크기만큼 균일분포 난수를 발생시킨다.



```python
decoder = Decoder(vocab_tar_size, embedding_dim, units, BATCH_SIZE)

sample_decoder_output, _, _ = decoder(tf.random.uniform((BATCH_SIZE, 1)),
                                      sample_hidden, sample_output)

print ('Decoder output shape: (batch_size, vocab size) {}'.format(sample_decoder_output.shape))
```

    Decoder output shape: (batch_size, vocab size) (64, 4368)
    

<br>

## 5. 최적화, 손실함수 정의



```python
optimizer = tf.keras.optimizers.Adam()
loss_object = tf.keras.losses.SparseCategoricalCrossentropy(
    from_logits=True, reduction='none')

def loss_function(real, pred):
  mask = tf.math.logical_not(tf.math.equal(real, 0))
  loss_ = loss_object(real, pred)

  mask = tf.cast(mask, dtype=loss_.dtype)
  loss_ *= mask

  return tf.reduce_mean(loss_)
  
checkpoint_dir = './training_checkpoints'
checkpoint_prefix = os.path.join(checkpoint_dir, "ckpt")
checkpoint = tf.train.Checkpoint(optimizer=optimizer,
                                 encoder=encoder,
                                 decoder=decoder)
```

<br>

## 6. 모델 훈련



```python
@tf.function
def train_step(inp, targ, enc_hidden):
  loss = 0

  with tf.GradientTape() as tape: # gradient의 중간 연산 과정을 tape에 기록해주는 기능능
    enc_output, enc_hidden = encoder(inp, enc_hidden)

    dec_hidden = enc_hidden

    dec_input = tf.expand_dims([targ_lang.word_index['<start>']] * BATCH_SIZE, 1) # decoder의 첫번째 input이 되는 <start> 지정

    # 교사 강요(teacher forcing) - 다음 입력으로 타겟을 피딩(feeding)합니다.
    for t in range(1, targ.shape[1]):
      # enc_output를 디코더에 전달합니다.
      predictions, dec_hidden, _ = decoder(dec_input, dec_hidden, enc_output)

      loss += loss_function(targ[:, t], predictions)

      # 교사 강요(teacher forcing)를 사용합니다.
      dec_input = tf.expand_dims(targ[:, t], 1)

  batch_loss = (loss / int(targ.shape[1]))

  variables = encoder.trainable_variables + decoder.trainable_variables

  gradients = tape.gradient(loss, variables)

  optimizer.apply_gradients(zip(gradients, variables))

  return batch_loss
```

GPU 사용 설정 안 하면 런타임이 너무 오래 걸리므로 꼭 런타임 -> 런타임 유형변경 -> GPU 설정을 해주자.



```python
EPOCHS = 10

for epoch in range(EPOCHS):
  start = time.time()

  enc_hidden = encoder.initialize_hidden_state()
  total_loss = 0

  for (batch, (inp, targ)) in enumerate(dataset.take(steps_per_epoch)):
    batch_loss = train_step(inp, targ, enc_hidden)
    total_loss += batch_loss

    if batch % 100 == 0:
      print('Epoch {} Batch {} Loss {:.4f}'.format(epoch + 1,
                                                   batch,
                                                   batch_loss.numpy()))
  # 에포크가 2번 실행될때마다 모델 저장 (체크포인트)
  if (epoch + 1) % 2 == 0:
    checkpoint.save(file_prefix = checkpoint_prefix)

  print('Epoch {} Loss {:.4f}'.format(epoch + 1,
                                      total_loss / steps_per_epoch))
  print('Time taken for 1 epoch {} sec\n'.format(time.time() - start))
```

    Epoch 1 Batch 0 Loss 4.5049
    Epoch 1 Batch 100 Loss 1.9858
    Epoch 1 Batch 200 Loss 1.8657
    Epoch 1 Batch 300 Loss 1.7295
    Epoch 1 Loss 1.9973
    Time taken for 1 epoch 35.93959403038025 sec
    
    Epoch 2 Batch 0 Loss 1.6057
    Epoch 2 Batch 100 Loss 1.3966
    Epoch 2 Batch 200 Loss 1.4414
    Epoch 2 Batch 300 Loss 1.3836
    Epoch 2 Loss 1.4501
    Time taken for 1 epoch 24.8153715133667 sec
    
    Epoch 3 Batch 0 Loss 1.4028
    Epoch 3 Batch 100 Loss 1.2178
    Epoch 3 Batch 200 Loss 1.1824
    Epoch 3 Batch 300 Loss 1.2187
    Epoch 3 Loss 1.2367
    Time taken for 1 epoch 24.458247423171997 sec
    
    Epoch 4 Batch 0 Loss 1.1026
    Epoch 4 Batch 100 Loss 1.1283
    Epoch 4 Batch 200 Loss 1.0214
    Epoch 4 Batch 300 Loss 1.0363
    Epoch 4 Loss 1.0774
    Time taken for 1 epoch 25.350347995758057 sec
    
    Epoch 5 Batch 0 Loss 0.9924
    Epoch 5 Batch 100 Loss 0.9963
    Epoch 5 Batch 200 Loss 1.0086
    Epoch 5 Batch 300 Loss 1.0192
    Epoch 5 Loss 0.9414
    Time taken for 1 epoch 25.34527015686035 sec
    
    Epoch 6 Batch 0 Loss 0.8075
    Epoch 6 Batch 100 Loss 0.8436
    Epoch 6 Batch 200 Loss 0.8031
    Epoch 6 Batch 300 Loss 0.7657
    Epoch 6 Loss 0.8130
    Time taken for 1 epoch 25.47455406188965 sec
    
    Epoch 7 Batch 0 Loss 0.6936
    Epoch 7 Batch 100 Loss 0.6561
    Epoch 7 Batch 200 Loss 0.6778
    Epoch 7 Batch 300 Loss 0.5930
    Epoch 7 Loss 0.6724
    Time taken for 1 epoch 25.008379220962524 sec
    
    Epoch 8 Batch 0 Loss 0.5179
    Epoch 8 Batch 100 Loss 0.5352
    Epoch 8 Batch 200 Loss 0.5811
    Epoch 8 Batch 300 Loss 0.5572
    Epoch 8 Loss 0.5125
    Time taken for 1 epoch 25.497196197509766 sec
    
    Epoch 9 Batch 0 Loss 0.3020
    Epoch 9 Batch 100 Loss 0.4005
    Epoch 9 Batch 200 Loss 0.3143
    Epoch 9 Batch 300 Loss 0.3462
    Epoch 9 Loss 0.3585
    Time taken for 1 epoch 24.915047645568848 sec
    
    Epoch 10 Batch 0 Loss 0.2077
    Epoch 10 Batch 100 Loss 0.2646
    Epoch 10 Batch 200 Loss 0.2277
    Epoch 10 Batch 300 Loss 0.2367
    Epoch 10 Loss 0.2423
    Time taken for 1 epoch 25.524725914001465 sec
    
    

<br>

## 7. 모델 번역

훈련 과정에서는 교사 강요를 사용했지만, 이제는 평가를 해야 하기 때문에 교사 강요를 하지 않고 예측값을 디코더의 input으로 사용한다.



```python
def evaluate(sentence):
  attention_plot = np.zeros((max_length_targ, max_length_inp))

  sentence = preprocess_sentence(sentence)

  inputs = [inp_lang.word_index[i] for i in sentence.split(' ')]
  inputs = tf.keras.preprocessing.sequence.pad_sequences([inputs],
                                                         maxlen=max_length_inp,
                                                         padding='post')
  inputs = tf.convert_to_tensor(inputs)

  result = ''

  hidden = [tf.zeros((1, units))]
  enc_out, enc_hidden = encoder(inputs, hidden)

  dec_hidden = enc_hidden
  dec_input = tf.expand_dims([targ_lang.word_index['<start>']], 0)

  for t in range(max_length_targ):
    predictions, dec_hidden, attention_weights = decoder(dec_input,
                                                         dec_hidden,
                                                         enc_out)

    # 나중에 어텐션 가중치를 시각화하기 위해 어텐션 가중치를 저장합니다.
    attention_weights = tf.reshape(attention_weights, (-1, ))
    attention_plot[t] = attention_weights.numpy()

    predicted_id = tf.argmax(predictions[0]).numpy()

    result += targ_lang.index_word[predicted_id] + ' '

    if targ_lang.index_word[predicted_id] == '<end>': # <end>라는 단어가 나오면 예측 중지지
      return result, sentence, attention_plot

    # 교사강요 대신에 예측된 ID를 모델에 다시 피드합니다.
    dec_input = tf.expand_dims([predicted_id], 0)

  return result, sentence, attention_plot
```


```python
# 어텐션 가중치를 그리기 위한 함수입니다.
def plot_attention(attention, sentence, predicted_sentence):
  fig = plt.figure(figsize=(10,10))
  ax = fig.add_subplot(1, 1, 1)
  ax.matshow(attention, cmap='viridis')

  fontdict = {'fontsize': 14}

  ax.set_xticklabels([''] + sentence, fontdict=fontdict, rotation=90)
  ax.set_yticklabels([''] + predicted_sentence, fontdict=fontdict)

  ax.xaxis.set_major_locator(ticker.MultipleLocator(1))
  ax.yaxis.set_major_locator(ticker.MultipleLocator(1))

  plt.show()
```


```python
def translate(sentence):
  result, sentence, attention_plot = evaluate(sentence)

  print('Input: %s' % (sentence))
  print('Predicted translation: {}'.format(result))

  attention_plot = attention_plot[:len(result.split(' ')), :len(sentence.split(' '))]
  plot_attention(attention_plot, sentence.split(' '), result.split(' '))
```


```python
# checkpoint_dir내에 있는 최근 체크포인트(checkpoint)를 복원합니다.
checkpoint.restore(tf.train.latest_checkpoint(checkpoint_dir))
```




    <tensorflow.python.training.tracking.util.CheckpointLoadStatus at 0x7fa85ce12550>




```python
translate(u'Ils jouent au foot.') # they play football
```

    Input: <start> ils jouent au foot . <end>
    Predicted translation: they play soccer . <end> 
    


    
<img src="https://user-images.githubusercontent.com/115082062/229085841-ca2abf1a-2b28-4015-8108-bbe578aba201.png">

<br>
    



```python
translate(u'il a trois chiens.') # he has three dogs
```

    Input: <start> il a trois chiens . <end>
    Predicted translation: he has three dogs . <end> 
    


    
<img src="https://user-images.githubusercontent.com/115082062/229085930-5eb7ed64-3e83-42ab-a523-d9c6e03dd2ae.png">

<br>
    


아래처럼 약간 의역을 하는 경우도 있는 것 같다.


```python
translate(u'nous ne pouvons pas les laisser gagner.') # we can not let them win
```

    Input: <start> nous ne pouvons pas les laisser gagner . <end>
    Predicted translation: we can t give up . <end> 
    


    
<img src="https://user-images.githubusercontent.com/115082062/229085941-b8f8d908-8db1-4547-a765-bc17d351cc79.png">

<br>
    


아래처럼 OOV 문제는 어떻게 해야할지 잘 판단이 안 선다.  train 되는 단어가 한정적이어서 keyerror가 발생하는 경우가 빈번한 것 같다.



```python
translate(u'Mbappé est un célèbre footballeur français.') # Mbappe is a famous French soccer player.
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    <ipython-input-43-4db59a2dbb65> in <module>
    ----> 1 translate(u'Mbappé est un célèbre footballeur français.') # we can not let them win
    

    <ipython-input-35-bd54cd790ad2> in translate(sentence)
          1 def translate(sentence):
    ----> 2   result, sentence, attention_plot = evaluate(sentence)
          3 
          4   print('Input: %s' % (sentence))
          5   print('Predicted translation: {}'.format(result))
    

    <ipython-input-33-a83e9578d47f> in evaluate(sentence)
          4   sentence = preprocess_sentence(sentence)
          5 
    ----> 6   inputs = [inp_lang.word_index[i] for i in sentence.split(' ')]
          7   inputs = tf.keras.preprocessing.sequence.pad_sequences([inputs],
          8                                                          maxlen=max_length_inp,
    

    <ipython-input-33-a83e9578d47f> in <listcomp>(.0)
          4   sentence = preprocess_sentence(sentence)
          5 
    ----> 6   inputs = [inp_lang.word_index[i] for i in sentence.split(' ')]
          7   inputs = tf.keras.preprocessing.sequence.pad_sequences([inputs],
          8                                                          maxlen=max_length_inp,
    

    KeyError: 'mbappe'

<br>