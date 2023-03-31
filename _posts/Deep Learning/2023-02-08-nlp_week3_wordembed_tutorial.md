---
title: "[NLP] FastText와 Word2Vec 비교"
excerpt: "Word2Vec와 FastText로 각각 토큰화를 하고 단어 유사도를 비교해보자"
toc: true
toc_label: "목차"
toc_sticky: true

tags: [Deep Learning, NLP, Word2Vec, FastText]

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

## 1. word embedding in Pytorch


```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim

torch.manual_seed(1) # 랜덤 시드 고정
```




    <torch._C.Generator at 0x7f7fdff5dad0>



pytorch에서 제공하는 `Embedding(V, D)`를 사용하면 워드 임베딩을 할 수 있다. V는 Vocabulary의 개수이고 D는 벡터의 Dimensional을 의미한다. 즉 아래 예시는 2개의 단어를 각각 5차원의 벡터로 만든다는 것이다.

`tensor()` 함수를 사용하여 hello에 매핑된 인덱스를 tensor 객체로 만들어준다.

이 때 dtype을 `long`으로 설정하면 64-bit-integer로 지정된다.


```python
word_to_ix = {"hello": 0, "world": 1} # 단어를 정수에 매핑
embeds = nn.Embedding(2, 5)  # 2 words in vocab, 5 dimensional embeddings
lookup_tensor = torch.tensor([word_to_ix["hello"]], dtype=torch.long) #tensor 객체로
```

아래처럼 hello라는 단어가 5차원의 임베딩 벡터로 바뀐 모습을 확인할 수 있다.


```python
hello_embed = embeds(lookup_tensor)
print(hello_embed)
```

    tensor([[ 0.6614,  0.2669,  0.0617,  0.6213, -0.4519]],
           grad_fn=<EmbeddingBackward0>)
    

<br>

## 2. N-gram Language model

이제 셰익스피어의 Soneet의 한 구절을 사용하여 N-gram 언어모델을 구현해볼 것이다. N-gram 언어모델의 원리는 n개의 context_size가 주어졌을 때 i번째 단어를 알기 위해 `P(w_i | w_{i-1}, w_{i-2}, ..., w_{i-n+1} )`라는 조건부 확률을 구하는 것이다. 즉, n개의 앞 문맥 단어가 주어졌을 때 다음 단어가 나올 확률을 예측하는 과정이다.


```python
CONTEXT_SIZE = 2
EMBEDDING_DIM = 10
# We will use Shakespeare Sonnet 2
test_sentence = """When forty winters shall besiege thy brow,
And dig deep trenches in thy beauty's field,
Thy youth's proud livery so gazed on now,
Will be a totter'd weed of small worth held:
Then being asked, where all thy beauty lies,
Where all the treasure of thy lusty days;
To say, within thine own deep sunken eyes,
Were an all-eating shame, and thriftless praise.
How much more praise deserv'd thy beauty's use,
If thou couldst answer 'This fair child of mine
Shall sum my count, and make my old excuse,'
Proving his beauty by succession thine!
This were to be new made when thou art old,
And see thy blood warm when thou feel'st it cold.""".split() # tese sentence를 단어 단위로 쪼갠다.
```

아래 반복문을 순회하면 위 test_sentence에 있는 각 단어들을 target word로 하고 context_size의 개수만큼 앞 단어가 같이 출력되는 tuple이 생성된다. 현재 context_size는 2개로 설정되어 있으니, 3번째 단어인 winters부터 cold까지 총 113개의 target word로 하는 튜플이 생성될 것이다.


```python
ngrams = [
    (
        [test_sentence[i - j - 1] for j in range(CONTEXT_SIZE)],
        test_sentence[i]
    )
    for i in range(CONTEXT_SIZE, len(test_sentence))
]
```


```python
print(len(ngrams))
print(ngrams[0]) #winters 앞의 두 단어인 when과 forty가 함께 출력됐다.
```

    113
    (['forty', 'When'], 'winters')
    


```python
vocab = set(test_sentence)
word_to_ix = {word: i for i, word in enumerate(vocab)} # 각 단어에 대해 정수 인코딩을 한다.
```

이제 이 N-gram 언어 모델을 신경망 형태로 클래스화 시킬 것이다.

- embedding층: 입력은 단어 사전의 크기만큼, 출력은 원하는 임베딩 벡터 차원만큼 설정한다.
- linear1층: context_size, 즉 앞 문맥의 단어 개수만큼 벡터 차원에 곱해서 linear한 층을 만들어준다.
- linear2층: 완성된 벡터를 다시 단어 사전의 크기로 출력시켜 softmax 함수에 통과시킬 준비를 한다.


```python
class NGramLanguageModeler(nn.Module):

    def __init__(self, vocab_size, embedding_dim, context_size):
        super(NGramLanguageModeler, self).__init__()
        self.embeddings = nn.Embedding(vocab_size, embedding_dim)
        self.linear1 = nn.Linear(context_size * embedding_dim, 128)
        self.linear2 = nn.Linear(128, vocab_size)

    def forward(self, inputs):
        embeds = self.embeddings(inputs).view((1, -1))
        out = F.relu(self.linear1(embeds))
        out = self.linear2(out)
        log_probs = F.log_softmax(out, dim=1)
        return log_probs
```

`nn.NLLLoss()`함수는 `nn.CrossEntropyLoss()`와 마찬가지로 corss-entropy를 구하는 손실함수이다. 차이점으로는 `CrossEntropyLoss()`는 함수 자체에 softmax를 적용시키고, `NLLLoss()`는 함수 자체에 softmax가 없어서 직접 모델 레이어에 softmax를 추가시켜야 한다는 차이가 있다.

옵티마이저로는 SGD를 사용한다.


```python
losses = []
loss_function = nn.NLLLoss()
model = NGramLanguageModeler(len(vocab), EMBEDDING_DIM, CONTEXT_SIZE)
optimizer = optim.SGD(model.parameters(), lr=0.001)
```

이제 신경망을 학습시킨다.


```python
for epoch in range(10):
    total_loss = 0
    for context, target in ngrams:

        # 단어들을 정수로 인코딩한 후 tensor 자료형으로 변환해준다.
        context_idxs = torch.tensor([word_to_ix[w] for w in context], dtype=torch.long)

        # gradients 값을 0으로 초기화시켜준다. gradients가 누적되지 않고 리셋되어야
        # 역전파가 올바르게 수행되기 때문이다.
        model.zero_grad()

        # forward(순전파)를 실행한다.
        log_probs = model(context_idxs)

        # softmax를 통해 산출된 최종 확률과 실제 인코딩된 정답 간의 오차를 계산한다.
        loss = loss_function(log_probs, torch.tensor([word_to_ix[target]], dtype=torch.long))

        # 오차를 바탕으로 gradinet를 업데이트한다.
        loss.backward()
        # updated gradient를 바탕으로 weight를 업데이트한다.
        optimizer.step()

        total_loss += loss.item()
    losses.append(total_loss) # 각 epoch마다 나온 오차를 종합한다.
print(losses)  # 학습시에 오차는 epoch를 거칠수록 점점 작아진다.

# beauty라는 단어의 임베팅 벡터이다. 설정한 바와 같이 차원이 10이다.
print(model.embeddings.weight[word_to_ix["beauty"]])
```

    [517.5080122947693, 514.8285989761353, 512.166300535202, 509.5199177265167, 506.8893609046936, 504.2742302417755, 501.6708755493164, 499.07986402511597, 496.49976110458374, 493.9293167591095]
    tensor([-0.3964, -1.9279,  0.9253,  0.9139, -0.7897, -0.5820, -0.7818, -1.4230,
             1.6084, -0.0329], grad_fn=<SelectBackward0>)
    

<br>

## 3. CBOW
CBOW는 앞 문맥만을 고려하는 것이 아니라 앞, 뒤 문맥을 모두 고려하여 사이에 있는 target word를 예측하는 작업이다. N-gram 언어모델과 달리 전통적 통계학 기반의 모델이 아니다. 때문에 확률론적인 해석은 어렵지만 낮은 연산량으로 효과를 보는 모델이다.

`context_size`를 2로 설정하게 되면 앞 2 단어, 뒤 2 단어를 참고하게 된다.


```python
CONTEXT_SIZE = 2 
raw_text = """We are about to study the idea of a computational process.
Computational processes are abstract beings that inhabit computers.
As they evolve, processes manipulate other abstract things called data.
The evolution of a process is directed by a pattern of rules
called a program. People create programs to direct processes. In effect,
we conjure the spirits of the computer with our spells.""".split()
```

아래 과정은 앞서 N-gram의 원리와 동일하다.


```python
vocab = set(raw_text)
vocab_size = len(vocab)

word_to_ix = {word: i for i, word in enumerate(vocab)}
data = []
for i in range(CONTEXT_SIZE, len(raw_text) - CONTEXT_SIZE):
    context = (
        [raw_text[i - j - 1] for j in range(CONTEXT_SIZE)]
        + [raw_text[i + j + 1] for j in range(CONTEXT_SIZE)]
    )
    target = raw_text[i]
    data.append((context, target))
print(data[:5])
```

    [(['are', 'We', 'to', 'study'], 'about'), (['about', 'are', 'study', 'the'], 'to'), (['to', 'about', 'the', 'idea'], 'study'), (['study', 'to', 'idea', 'of'], 'the'), (['the', 'study', 'of', 'a'], 'idea')]
    

위 N-gram 모델에서 쓰인 신경망을 거의 그대로 사용했다. 다만, 주의해야 할 것은 context_size가 앞 뒤로 두번씩 count 되기에 layer1에서 input에 2를 곱해줘야 한다.


```python
class CBOW(nn.Module):

    def __init__(self, vocab_size, embedding_dim, context_size):
        super(CBOW, self).__init__()
        self.embeddings = nn.Embedding(vocab_size, embedding_dim)
        self.linear1 = nn.Linear(context_size * embedding_dim *2, 128)
        self.linear2 = nn.Linear(128, vocab_size)

    def forward(self, inputs):
        embeds = self.embeddings(inputs).view((1, -1))
        out = F.relu(self.linear1(embeds))
        out = self.linear2(out)
        log_probs = F.log_softmax(out, dim=1)
        return log_probs
```


```python
losses = []
loss_function = nn.NLLLoss()
model = CBOW(len(vocab), EMBEDDING_DIM, CONTEXT_SIZE)
optimizer = optim.SGD(model.parameters(), lr=0.001)
```


```python
for epoch in range(10):
    total_loss = 0
    for context, target in data:

        # 단어들을 정수로 인코딩한 후 tensor 자료형으로 변환해준다.
        context_idxs = torch.tensor([word_to_ix[w] for w in context], dtype=torch.long)

        # gradients 값을 0으로 초기화시켜준다. gradients가 누적되지 않고 리셋되어야
        # 역전파가 올바르게 수행되기 때문이다.
        model.zero_grad()

        # forward(순전파)를 실행한다.
        log_probs = model(context_idxs)

        # softmax를 통해 산출된 최종 확률과 실제 인코딩된 정답 간의 오차를 계산한다.
        loss = loss_function(log_probs, torch.tensor([word_to_ix[target]], dtype=torch.long))

        # 오차를 바탕으로 gradinet를 업데이트한다.
        loss.backward()
        # updated gradient를 바탕으로 weight를 업데이트한다.
        optimizer.step()

        total_loss += loss.item()
    losses.append(total_loss) # 각 epoch마다 나온 오차를 종합한다.
print(losses)  # 학습시에 오차는 epoch를 거칠수록 점점 작아진다.

# beauty라는 단어의 임베팅 벡터이다. 설정한 바와 같이 차원이 10이다.
print(model.embeddings.weight[word_to_ix["process"]])
```

    [227.15311574935913, 225.7848310470581, 224.42582035064697, 223.07474970817566, 221.73145699501038, 220.39596724510193, 219.06667685508728, 217.74286723136902, 216.42530012130737, 215.1130495071411]
    tensor([-0.3668, -0.5739, -1.1252, -0.5992, -1.3084, -0.8383,  0.3383,  1.4071,
             1.4924,  1.4587], grad_fn=<SelectBackward0>)
    
