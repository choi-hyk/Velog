[Velog로 가기](https://velog.io/@choi-hyk/LLM-Transformer)

released at 2025-08-17 18:03:08 KST

updated at 2025-08-28 20:32:07 KST

|[AI](https://velog.io/tags/AI)|[Deep Learning](https://velog.io/tags/Deep-Learning)|[LLM](https://velog.io/tags/LLM)|[transformer](https://velog.io/tags/transformer)|
|----|----|----|----|

# 🖱️ Transformer

오늘은 현대 LLM의 모델들이 활용중인 가장 중요한 요소인 **Transformer**에 대해서 알아보겠다. Transformer는 2017년 Google에서 발표한 **「Attention is All you need」** 논문에서 소개된 모델이다.

[Attention is All you need](https://arxiv.org/abs/1706.03762)

해당 논문에서는 Transformer 아키텍처가 고안된 이유를 RNN의 단점을 서술하면서 설명하고, Transformer 아키텍처의 구성 방식을 각 layer를 기준으로 설명을 한다. 내용이 너무 어려워서 여러가지 영상이랑, 해석본도 찾아보면서 최대한 정리를 해보았다...

---

## ✏️ Transformer가 고안된 이유

**「Attention is All you need」** 에서는 먼저 RNN의 단점을 이야기하는데, RNN은 3가지의 주요 단점이 있다.

- **순차적 처리**: RNN은 단어를 순서대로 처리하는 구조 때문에 병렬 처리가 불가능하다. 이는 대규모 데이터 학습에 많은 시간이 소요되는 원인이 된다.

- **장기 의존성(Long-term Dependency) 문제**: 문장이 길어질수록 초반부 단어의 정보가 점차 희미해진다. 이로 인해 문장의 앞부분에 있는 중요한 맥락 정보를 활용하기 어렵다.

- **고정된 컨텍스트 벡터**: RNN은 문장 전체의 정보를 하나의 고정된 크기 벡터에 압축하는데, 이 과정에서 정보 손실이 발생하여 복잡한 문장의 의미를 온전히 담기 어렵다.

간단하게 RNN 문장이 길어질수록 성능이 떨어지고, 문장을 순서대로 처리를 해야되기 때문에, 병렬처리가 불가능하다. 또한 RNN과 더불어 **CNN(Convolutional Neural Network)** 이라는 신경망 기술에 대해서도 설명을 하였는데, 간단하게 CNN은 "합성곱 신경망"이라는 기술이다. CNN은 이미지 및 비디오와 같은 2차원 또는 3차원 데이터 처리에 특화된 딥러닝 모델인데, 데이터의 특징을 추출하여 분류나 탐지 같은 작업에 강점을 가진다. 하지만 CNN 또한 문장 내 단어 간의 **장기적인 의존 관계** 를 학습하는 데는 한계가 있었다.

즉, RNN은 순차 처리와 장기 의존성 문제, CNN은 문장의 전체 맥락을 포착하기 어렵다는 문제를 가지고 있었다.

따라서 Transformer는 에서 설명한 RNN과 CNN의 문제점인 병럴처리, 장기 의존성 그리고 다양한 문장의 표현을 고려한 모델이다. 

---

## 🛠️ 구조

<img width="1520" height="2239" alt="Image" src="https://github.com/user-attachments/assets/d643741a-915e-4c5f-9aaf-a6fdeef848e3" />

위의 구조를 보면 머리가 좀 아파올 것 같은데, layer 별로 나눠서 이해해보면 좀 괜찮을 것이다. 그림은 Transformer에서 사용하는 **Encoder와 Decoder**의 구조를 나타낸 그림이다. **Encoder는 입력이 주어지면, 해당 입력을 기반으로 입력 데이터들 관의 관계를 파악**하고, **Decoder는 해당 입력과 Encoder에서 생성된 관계를 통해 다음 데이터를 예측**한다. 이제 이 그림을 각 layer마다 살펴보겠다.

---

### Input Embedding

제일 먼저 Encoding에서 보이는 **Input Embedding** 은 자연어를 **Tokenizer** 해서 벡터화를 한 데이터이다.  
**Tokenizer** 와 Embedding은 추후에 다른 글로 알아보도록하고, 

Embedding으로 벡터화를 하게 되면 각 토큰의 특징을 나타내는 차원이 생긴다. 예를 들어서 `바나나` 라는 토큰이 있으면, 해당 토큰은 여러개의 특징으로 나타낼 수 있다. 

각 특징은 벡터의 한 요소로 표현되며, 예를 들어 바나나라는 단어가 4차원 벡터로 임베딩 되었다고 하면,

바나나 → [0.12, -0.87, 0.33, 0.55]

이런 식으로 수치화된다. 이 벡터의 각 값은 단순히 숫자가 아니라, 의미적인 특징을 압축한 값이다. 어떤 값은 과일과 관련된 의미를, 또 어떤 값은 음식이라는 카테고리적 의미를, 또 다른 값은 다른 단어들과의 관계 속에서 파생된 의미를 담고 있다.

이렇게 만들어진 Input Embedding은 이후 **Positional Encoding** 과 결합된다. Transformer는 RNN처럼 순차적으로 단어를 처리하지 않기 때문에, 단어의 순서 정보를 따로 제공해야 한다. 이때 사용하는 것이 Positional Encoding(위치 인코딩) 인데, 잠시 후에 알아보겠다.

---

### Attention

Attention은 사실 Transformer에서 고안된 기법이 아니다. Attention은 벡터화 되어서, 각 특징들을 벡터로 나타내어진 자연어의 관계를 파악하는 기법이다. **「Neural Machine Translation by Jointly Learning to Align and Translate」이라는 2014년 논문에서 RNN을 통한 기계번역을 개선하기 위해 고안된 기술**이라고 한다.

그렇다면 드는 생각이, Transformer는 기존의 딥러닝 모델들과는 무엇이 다르냐는 것이다. Attention을 적용하는 기법에서 **Transformer는 Self Attention과 Multi Head Attention 이라는 발전된 기법을 사용**한다.

먼저 Attention의 기본 원리에 대해서 알아보자.

$$
Attention(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

위의 식은 **「Attention is All you need」** 에서 설명하는 Attention을 구하는 식이다. 

**Attention**은 크게 세 단계로 진행된다. 먼저 입력 데이터는 **Query(Q)**, **Key(K)**, **Value(V)** 세 가지로 변환된다. 이때 **Query는 현재 단어가 "무엇을 찾고 있는가"** 를 나타내고, **Key는 "어떤 정보를 가지고 있는가"** 를, **Value는 "그 정보 자체"** 를 의미한다. 

예를 들어서 검색창에 **LLM과 관련된 논문** 을 입력하면 해당 입력이 $Q$ 가 될 것이다. 그리고 검색 이후 나온 여러가지 웹 사이트와 논문들은 $K$ 가 되고 실제 논문 데이터는 $V$ 가 될 것이다. 

이러한 **$Query(Q)$**, **$Key(K)$**, **$Value(V)$** 를 만드는 방식이 가중치 **$W$** 를 적용하는 것이다. 입력 임베딩 $X \in \mathbb{R}^{n \times d_{\text{model}}}$에 대해, 각각의 행렬은 다음과 같이 정의된다.

$$
Q = XW^Q, \quad K = XW^K, \quad V = XW^V
$$

여기서 $W^Q \in \mathbb{R}^{d_{\text{model}} \times d_k}$, $W^K \in \mathbb{R}^{d_{\text{model}} \times d_k}$, $W^V \in \mathbb{R}^{d_{\text{model}} \times d_v}$ 는 모두 학습 가능한 파라미터이다.

즉, 하나의 입력 임베딩이 들어오더라도 서로 다른 가중치 행렬과 곱해지면서, **질문을 하는 벡터(Q)**, **조건을 제공하는 벡터(K)**, **실제 정보를 전달하는 벡터(V)** 로 투영된다.

그리고 $W^Q \in \mathbb{R}^{d_{\text{model}} \times d_k}$, $W^K \in \mathbb{R}^{d_{\text{model}} \times d_k}$, $W^V \in \mathbb{R}^{d_{\text{model}} \times d_v}$ 에서 보통 $d_k$, $d_v$는 $\frac{d_\text{model}}h$ 로 차원을 계산하게 되는데, 이와 관련해서 왜 가중치들의 집합의 크기가 $d_{\text{model}}\times d_k$, $d_{\text{model}}\times d_v$ 가 되는지 그리고 $h$가 무엇인지 궁금할 것이다. 일단 기억만 하고 있어라, 뒤에 **Multi Head Attention**에서 $h$ 가 무엇이고 집합의 크기가 왜 저렇게 나오는지, 그리고 차원을 맞추는 이유가 무엇인지 설명하겠다.

$Q$, $K$, $V를 구하는$ 과정은 단순한 선형 변환으로 보일 수 있지만, 학습을 통해 $W^Q, W^K, W^V$가 점차적으로 최적화되면서, **Attention이 각 단어 간의 관계를 더 정교하게 파악할 수 있도록 만드는 핵심 장치**가 된다. 결국 이 단계는 **"입력 임베딩을 서로 다른 관점에서 바라보는 방법"을 모델이 스스로 학습하는 과정** 이라고 이해할 수 있다.

그리고 이러한 가중치들은 최종적으로 **디코더를 통해 생성된 확률 분포를 기반으로 손실 계산을 수행**하고, 이 손실을 줄이기 위해 **역전파와 가중치 조정 과정을 거쳐 업데이트된다.** 이렇게 조정된 가중치는 모델의 성능을 향상시키며, 다음 번 예측의 정확도를 높이는 데 사용된다.

**새롭게 조정된 가중치로 모델은 다시 한번 입력 데이터를 받아 순전파를 시작**한다. 참고로 순전파는 우리가 오늘 알아보는 Transformer의 과정이다. 인코더와 디코더는 업데이트된 가중치를 활용하여 입력 데이터의 문맥을 다시 파악하고, 디코더는 이를 기반으로 **더 정확한 확률 분포를 생성**한다. 이 반복적인 훈련 과정은 모델이 충분히 학습되어 **손실값이 더 이상 줄어들지 않을 때까지 계속된다.**
	
이러한 가중치를 생성 및 조정하는 방법인 역전파는 다음글인 Fine-tuning에서 심도있게 정리해보겠다. 지금은 Attention에 집중해보도록 하자.

Attention은 위 세 가지 $Query(Q)$, $Key(K)$, $Value(V)$ 의 데이터들의 관계를 파악하는 기법이다. 다시 식으로 돌아가서, 각 데이터들은 임베딩 되어서 벡터로 표현된다고 했다. 그리고 각 벡터들은 차원(특징)을 가진다 했다. 여기서 하나의 $Q$는 여러 개의 $K$를 가질 것이다. 그리고 $Q$와 $K$의 개수는 $n{\text{ (토큰의 개수)}}\times d_k{\text{ (차원의 개수)}}$ 가 된다. 그리고 하나의 $Q$는 입력 값 $X$에서 생성된 같은 차원의 모든 $K$와 **내적을 수행하여 유사도를 계산하고, 그 결과를 기반으로 각 $V$에 가중치를 부여하여 최종 Attention 출력을 만든다.**  ${K^T}$는 전치 행렬을 의미한다. 전치 행렬로 만드는 이유는 아래의 식으로 설명하겠다.

$$
Q \in \mathbb{R}^{n \times d_k}, \quad K \in \mathbb{R}^{n \times d_k}
$$

위에서 말했다 싶이 $Q$와 $K$의 개수는 $n{\text{ (토큰의 개수)}}\times d_k{\text{ (차원의 개수)}}$ 이다. 이때 내적을 하기 위해서 행렬 곱을 하게 된다. 토큰의 행렬에서 행은 $n$을 열은 $d_k$를 나타낸다. 하지만 위의 크기로는 $Q$, $K$는 행렬곱을 하지 못한다. 따라서 전치를 통해 크기를 맞춰준다.

$$
K^T \in \mathbb{R}^{d_k \times n}
$$

$$
QK^T \in \mathbb{R}^{n \times n}
$$

이렇게 크기가 맞춰진 $Q$ 와 $K$는 $n\times n$ 크기의 어텐션 스코어(attention score)로 변환된다.

$$
\frac{QK^T}{\sqrt{d_k}}
$$

위의 식은 $Q$를 $K$와 내적을 한 값을 $\sqrt{d_k}$로 Scaling하는 것을 나타낸다. 

**내적** 은 $Query$가 $Key$와 얼마나 잘 맞는지를 나타내는 척도이며, 일종의 **유사도(similarity) 점수**라고 볼 수 있다. 고등학교때 배운 내적을 생각해보자

$Q = (1, 2, 3)$
$K_1 = (2, 0, 1)$
$K_2 = (-2, -1, 1)$
$K_3 = (0, 2, 2)$

이렇게 1개의 $Q$에 3개의 $K$가 있다고 해보자. 해당 벡터는 위에서 이야기한 가중치가 적용되어 3개의 차원으로 이루어진 값이다. 즉 ${d_k}$는 3이다.

$Q \cdot K_1 = 5$
$Q \cdot K_2 = -1$
$Q \cdot K_3 = 10$

*행렬로 나타낸 경우*

$$
QK^T = 
\begin{bmatrix}1 & 2 & 3\end{bmatrix}
\begin{bmatrix}
2 & -2 & 0 \\
0 & -1 & 2 \\
1 & 1 & 2
\end{bmatrix}
= \begin{bmatrix}5 & -1 & 10\end{bmatrix}
$$


위에서 내적의 결과를 보면, $K_3$가 10으로 $Q$ 와 가장 유사하다. 그리고 $K_2$가 -1로 가장 관련이 없다.

$\sqrt{d_k}$는 **스케일링(scaling)** 을 의미한다 앞에서 계산한 내적 결과는 $d_k$의 크기가 커질수록 값이 점점 커지게 된다 만약 차원이 수백 차원 이상으로 커진다면 내적 값은 지나치게 커지고 $softmax$ 함수에 넣었을 때 기울기가 매우 가팔라져 작은 차이에도 확률 분포가 한쪽으로 치우쳐 버린다

이를 방지하기 위해 내적 값을 차원의 제곱근으로 나누어 **정규화(normalization)** 를 해준다 예를 들어 위에서 $d_k = 3$이므로 $\sqrt{d_k} = \sqrt{3} \approx 1.73$ 이 된다

그럼 각각의 내적 값은 다음과 같이 스케일링된다

* $\frac{Q \cdot K_1}{\sqrt{3}} = \frac{5}{1.73} \approx 2.89$
* $\frac{Q \cdot K_2}{\sqrt{3}} = \frac{-1}{1.73} \approx -0.58$
* $\frac{Q \cdot K_3}{\sqrt{3}} = \frac{10}{1.73} \approx 5.77$

이 과정을 거치면 값의 크기가 안정화되어 $Softmax$에 넣었을 때 적절한 확률 분포를 얻게 된다 즉 스케일링은 **내적 값이 차원 수에 비례해 과도하게 커지는 문제를 제어하는 장치**라고 이해하면 된다

그러면 이제 $Q$로부터 각 3개의 $K$의 관계를 알게 되었다. 그 다음으로 적용되는 것이 $softmax$이다. $softmax$는  $\frac{QK^T}{\sqrt{d_k}}$ 에서 나온 값 들을 전부 합 하였을 때, 1로 만들어주는 함수이다. 위의 경우에서는 $K_3$가 1에서 가장 많은 비율을 차지할 것이다. 그리고 $K_2$가 가장 적은 비율을 차지할 것이다. 

마지막으로 가중합을 $V$에 적용하여 최종적인 정보의 관계를 생성하게 된다.
  
정리하자면, 특정 Query가 여러 Key들과 얼마나 관련성이 있는지를 Softmax를 통해 확률 값으로 바꾸게 되고, 이 확률 값이 바로 Attention에서 말하는 **가중치(weight)** 가 된다. 그리고 이 가중치는 Value $V$ 벡터에 곱해져 최종적으로 중요한 정보는 크게, 덜 중요한 정보는 작게 반영되도록 조절한다.

결과적으로 Attention 메커니즘은 “**Query와 Key의 내적으로 구한 유사도를 스케일링 후 Softmax로 정규화하여, Value에 가중합을 적용하는 과정**”이라고 정리할 수 있다.

### Self Attention

그렇다면 Self Attention은 무엇일까? 앞에서 예시는 검색엔진처럼 Query는 질문, Key는 문서의 제목, Value는 실제 내용으로 비유했다. 하지만 Self-Attention은 그 대상이 외부 데이터가 아니라, **같은 문장 안의 토큰들끼리 서로를 참고하는 방식이다**. 사실 앞에서 이야기한 "하나의 $Q$는 입력 값 $X$에서 생성된 같은 차원의 모든 $K$와 **내적을 수행하여 유사도를 계산한다"** 가 바로 Self Attention을 나타내는 말이었다. 그냥 Attention은 외부 데이터에서 $Q$, $K$, $V$를 각각 생성한다.

하나의 문장 `나는 학교에 간다`가 있다고 하면, 각 단어가 동시에 Q, K, V의 역할을 수행한다.

"나는" → Query를 만들고, Key와 Value도 만든다
"학교에" → Query, Key, Value를 모두 가진다
"간다" → 역시 Query, Key, Value를 가진다

이렇게 되면 문장 안에서 각 단어는 다른 단어와 자신 사이의 관련성을 계산할 수 있다. 이를 통해 처음에 말한 RNN에서 실현하지 못한 장기 의존성 그리고 다양한 문장의 표현을 해결이 가능하다. 그러면 마지막 목적인 병럴처리는 어떻게 실현이 가능할까? 바로 Multi Head Attention에 그 정답이 있다.

### Multi Head Attention

<img width="835" height="1282" alt="Image" src="https://github.com/user-attachments/assets/1991d59d-4881-4bba-8ac1-9e3b7c5e8be2" />

먼저 Multi Head Attention에 대해서 알아보려면 head가 무엇인지 알아야 한다. 처음에 나는 차원이랑 헤드가 헷갈렸는데, 차원은 위에서 이야기 한 것처럼 각 토큰의 특징을 나타낸 것이다. 

$d_{model} = 512$라면, 각 단어 토큰은 512개의 숫자로 표현된다.

"바나나"라는 토큰이 들어오면 벡터

  $$
  (0.12, -0.33, 0.98, \dots, 0.21) \in \mathbb{R}^{512}
  $$

이런 식으로 512차원의 공간에서 하나의 점으로 표현된다. 즉 $\mathbb{R}^{512}$ 는 모델의 용량이 된다.

헤드는 **Self-Attention을 여러 번 나눠서 병렬로 돌리는 단위**이다.
예를 들어, $d_{model} = 512$이고 Head 수가 $h = 8$이라면,
각 Head는 차원을 나눠서

  $$
  d_{head} = \frac{d_{model}}{h} = 64
  $$

의 크기로 Attention 연산을 한다.
전체 임베딩 512차원을 **8개의 시각(Head)으로 쪼개서 동시에 바라본다**고 이해하면 된다.

* **차원(dimension)** → 토큰 벡터의 "특징 공간" 크기 (정보량)
* **헤드(head)** → 그 특징 공간을 "여러 시각"으로 분할해 병렬로 학습하는 방법

즉 **Multi Head Attention**은 $h$번 만큼 Self Attention을 하는 것이다. 이를 통해 병렬처리를 실현 가능하다. 

여기서 앞에서 이야기한 가중치의 크기와 $d_k, d_v$를 $\frac{d_{model}}{h}$로 맞추는 이유가 여기서 나온다.

 $$
 X \in \mathbb{R}^{n \times d_{\text{model}}}
 $$

위의 식처럼 입력값 $X$의 크기는 $n$개의 토큰에서 차원 ($d_{\text{model}}$)을 곱한 값이다. 그러면 각 $Q$, $K$, $V$를 생성하기 위해 가중치와 곱하게 되는데 이때 $h$개 ($d_k$, $d_v$) head로 Self Attention을 하게 된다. 그 뜻은 원래 차원 각각에 $h$개의 특징 추가시키는 것이다. 위의 예시인 경우 512개의 차원에서 8개의 특징을 추가하여 512와 8을 곱한 98,304개가 된다.   

따라서 가중치들의 크기는 $d_{\text{model}}\times d_k$, $d_{\text{model}}\times d_v$  

$$
Q = XW^Q, \quad K = XW^K, \quad V = XW^V
$$

위에서 본 해당 식에서 각 행렬 곱을 하게 되면 
$$(n \times d_{\text{model}}) \times (d_{\text{model}} \times d_k) = (n \times d_k)$$
$$(n \times d_{\text{model}}) \times (d_{\text{model}} \times d_k) = (n \times d_v)$$


위의 예시에서 Self Attention의 각 차원은 64개 일것이다. 그리고 8번의 Self Attention을 하면 총 **512개의 차원이 다시 완성되는 것이다**. 이렇게 해야지 다음 과정인 Residual Connection(잔차 연결) 같은 구조에서 입력($d_{model}$)과 출력($d_{model}$)을 그대로 더할 수 있고, 다음 Feed-Forward Layer도 **동일한 차원에서 작동할 수 있게 된다.** 이렇게 $h$로 나누는 방법으로 차원을 맞추는 것이다.



$$
MultiHead(Q, K, V) = Concat(head_1, \dots, head_h)W^O
$$

$$
\text{where } head_i = Attention(QW_i^Q, KW_i^K, VW_i^V)
$$

위의 식은 Self Attention 이 $head$번 만큼 일어나고 각 결과를 더해서 **Multi Head Attention**을 구성하는 것을 나타낸다.

**Multi Head Attention**까지 실행하면, 아마 입력값에 대해 모델이 충분히 이해했다고 생각할 것이다. 하지만 하나 빠트린것이 있는데, 바로 입력 값의 위치에 대한 정보이다.

---

### Positional Encoding

자연어에서 입력값의 위치는 매우 중요하다. 그런데 위치를 고려하게 되면 RNN에서 장기 의존성(Long-term Dependency)와 같이 위치에 따른 정보 손실과 순차적 처리로 인해 병렬처리가 불가능하다. 이러한 위치에 따른 정보를 Transformer에서는 다른 기법으로 적용하였는데, 바로 **Positional Encoding**으로 적용을 한 것이다.

Transformer는 RNN처럼 순차적으로 단어를 처리하지 않기 때문에, 입력 토큰의 **순서 정보**가 사라지는 문제가 있다. 즉, "나는 학교에 간다"라는 문장이 들어와도, Transformer 입장에서는 단순히 4개의 벡터 집합일 뿐 "나는 → 학교에 → 간다"라는 순서 관계를 알 수 없다.

이를 해결하기 위해 **Positional Encoding(위치 인코딩)**을 추가한다. Positional Encoding은 각 토큰의 임베딩 벡터에 "해당 토큰이 문장 내 몇 번째 위치에 있는지"를 수학적으로 표현한 벡터를 더해주는 방식이다.


$$
Z = X + PE
$$

$X$는 원래의 단어 임베딩, $PE$는 위치 정보를 담은 인코딩 벡터이다.


논문에서 제안한 Positional Encoding은 **사인(sin)과 코사인(cos)** 함수를 이용한다.
특정 위치 $pos$와 차원 $i$에 대해 다음과 같이 정의된다.

$$
PE_{(pos, 2i)} = \sin \left( \frac{pos}{10000^{2i/d_{model}}} \right)
$$

$$
PE_{(pos, 2i+1)} = \cos \left( \frac{pos}{10000^{2i/d_{model}}} \right)
$$

여기서

* $pos$ : 단어의 위치 (0번, 1번, 2번 …)
* $i$ : 임베딩 벡터의 차원 인덱스
* $d_{model}$ : 임베딩 벡터의 총 차원 수

즉, 짝수 차원은 사인 함수, 홀수 차원은 코사인 함수로 값을 넣어준다.
나도 정확한 원리는 모르지만 사인 코사인으로 파형을 생성해서 무한한 길이도 위치를 알아낼 수 있는 형태로 바꿔진다고 한다. ~~잘 모름~~
 
---

### Feed Forward Network (FFN)

마지막으로 Multi-Head Attention과 Positional Encoding을 거친 후의 출력은 그대로 다음 레이어로 전달되지 않고, 한 번 더 **Feed Forward Network(포지션별 전결합 신경망)**을 거치게 된다. FFN은 모든 위치(pos)에 대해 동일하게 적용되는 두 개의 선형 변환과 비선형 활성화 함수로 구성된다.

수식으로 표현하면 다음과 같다.

$$
FFN(x) = \max(0, xW_1 + b_1)W_2 + b_2
$$

여기서 $W_1, W_2$와 $b_1, b_2$는 학습 가능한 가중치와 편향이다. 중간에 들어가는 $\max(0, \cdot)$는 ReLU 함수로, 비선형성을 부여하여 모델이 더 복잡한 패턴을 학습할 수 있게 한다.

FFN의 특징은 **각 토큰 위치마다 동일한 네트워크가 독립적으로 적용**된다는 것이다. 즉, 입력이 10개의 토큰이든 20개의 토큰이든, 각 토큰 벡터는 똑같은 FFN 구조를 거쳐 변환된다. 이로 인해 모델은 위치에 무관하게 동일한 변환을 수행하면서도, Attention으로 이미 반영된 단어 간의 관계를 기반으로 비선형적인 특징을 학습할 수 있다.

간단히 말해 Attention이 **단어들 사이의 관계** 를 학습하는 단계라면, FFN은 그 관계로부터 **복잡한 패턴을 추출하고 강화하는 단계**라고 볼 수 있다.

이 과정을 거친 출력은 다시 Residual Connection(잔차 연결)과 Layer Normalization을 통해 안정화되고, 다음 Encoder Layer로 전달된다. Encoder는 이런 구조를 여러 층 쌓아 올려 강력한 표현 학습 능력을 얻게 된다.

### Masking

이제 **Masking**에 대해서 알아보겠다. 마스킹은 decoder에만 적용되는 기법이다. Decoder는 해당 입력과 Encoder에서 생성된 관계를 통해 다음 데이터를 예측한다... 라고 위에서 설명했다. 그런데 생각해보자 Encoder는 단순히 모든 입력에 관한 관계를 정의하는 것이라서, Encoder로 생성된 정보들은 전부 Self Attention 을 통해 조금이라도 각자의 정보를 가지고 있을 것이다.

하지만 Decoder는 다르다. Decoder는 **순차적 예측(Autoregressive Generation)** 을 수행해야 한다. 예를 들어, `나는 학교에 간다`라는 문장을 생성한다고 할 때, 첫 번째 단계에서는 `나는`만 알고 있어야 하며, 두 번째 단계에서는 `나는 학교에`까지만 알고 있어야 한다. 만약 Decoder가 앞으로 나올 단어 `간다`를 미리 참고해버린다면, 모델은 학습 과정에서 미래 정보를 엿보는 **정보 누수(Information Leakage)** 가 발생하게 된다. 이를 해결해 주는 것이 Masking 기법이다.

Masking은 미래 시점의 단어를 가려서 현재 시점 이전의 단어들만 보이도록 하는 역할을 한다. 수학적으로는 Attention의 Softmax 단계에서, 미래 단어 위치에 -∞ 값을 추가하여 확률이 0이 되도록 만든다. 이렇게 하면 Decoder는 항상 앞에서 생성된 단어까지만 참고해서 다음 단어를 예측하게 된다.

---

### Add & Norm 

<img width="1520" height="2239" alt="Image" src="https://github.com/user-attachments/assets/d643741a-915e-4c5f-9aaf-a6fdeef848e3" />

그림을 다시 한번 봐보자 MultiHead Attention과 Feed Forward를 하고 나서 Add & Norm이라는 과정이 있고, 화살표 하나는 MultiHead Attention과 Feed Forward를 하지 않고 Add & Norm을 향하고 있다. 

MultiHead Attention과 Feed Forward를 하고 나면 원래 입력 정보 $X$가 변질 되거나 학습지 되지 않아, 아예 다른 출력이 나올 수도 있다. 또한 각 layer를 지날때마다 데이터 분포 층이 뒤틀릴 수도 있다. 이를 해결 해 주는 것이 Add & Norm이고, 따라서 각 layer를 지나고 나서 적용을 해준다.

즉, **Add & Norm은 "입력 + 출력"을 합쳐서 정규화하는 과정**이다.Add & Norm은 **Residual Connection + Layer Normalization**로 이루어진다. 

먼저 Residual Connection은 레이어의 입력 $X$와 해당 레이어의 출력 $F(X)$를 더한다.

$$
Y = X + F(X)
$$

이렇게 더하면, 깊은 네트워크에서도 원래 입력 정보 $X$가 손실되지 않고 그대로 흘러갈 수 있다.

그 다음에 Layer Normalization을 적용한다. 평균과 분산을 구해서 정규화하는 거다.

$$
\text{LayerNorm}(Y) = \frac{Y - \mu}{\sigma} \cdot \gamma + \beta
$$

최종 출력은 이렇게 된다.

$$
\text{Output} = \text{LayerNorm}(X + F(X))
$$

---

### BackPropagation

이제 위에서 구한 값을 통해 역전파를 진행한다. 이러한 역전파는 수천~수억번을 반복하여 모델을 최적화 하게 된다. 앞에서 말한 것처럼 역전파는 다음 글에서 알아보겠다.

---

### Decoder

위의 그림을 보면 디코더에서는 두 가지 입력이 필요하다.
첫 번째는 **아웃풋 임베딩(Output Embedding)**, 두 번째는 **인코더의 출력값**이다.

**아웃풋 임베딩**은 지금까지 생성한 단어들을 임베딩한 것이다. 예를 들어 번역 모델에서 이미 “I go”까지 만들었다면, 이게 디코더 입력으로 들어간다. 디코더는 이걸 바탕으로 다음 단어가 뭔지 예측한다.

**인코더 출력값**은 원문 문장의 의미 표현이다. 디코더는 단순히 자기 자신(아웃풋 임베딩)만 보고는 번역을 할 수 없다. 원래 문장 정보도 같이 참고해야 한다. 그래서 중간에 **Cross-Attention** 이라는 중간단계에서 인코더 출력값을 Key, Value로 삼고, 디코더 쪽에서 만든 Query와 결합한다. 이렇게 해야 “출력 문장이 원문을 잘 반영하도록” 만들 수 있다.

#### 디코더만 사용하는 모델들

GPT 같은 모델은 **디코더만 사용하는 구조**다. 인코더-디코더 구조가 아니기 때문에, **아웃풋 임베딩만 가지고 학습한다.** 방식은 다음과 같다. 디코더 모델은 입력 문장을 바로 디코더에 넣는다. 그리고 나서 마스크드 셀프 어텐션(Masked Self-Attention)으로, 앞으로 올 단어는 못 보고 과거 단어만 참고한다. 이렇게 해서 언어 모델링이 가능하다. 즉, 앞 단어가 주어졌을 때 다음 단어를 예측하는 방식이다. 따라서 디코더 생성형 모델이 사용하는 방식이다. 우리가 GPT를 통해 자연어를 입력하고 GPT가 응답을 생성하는 것이 이러한 토크나이저와 마스크드 셀프 어텐션으로 다음 단어의 확률을 통해 만드는 것이다.

---

## 😁 마무리

이번에는 Transformer에 대해서 알아보았는데, 사실 이 글을 5일에 걸쳐서 쓴것 같다. 중간에 틀린 내용도 고치고 논문 내용도 다시 살펴보느라 오래 걸렸는데, 이렇게 한번 정리하니 확실히 이해가 잘되는 것 같다. 다음에는 fine-tuning에 대해서 알아보겠다. 사실 우리가 LLM을 사용하면 이미 Transformer 가 적용된 LLM을 파인튜닝하거나 RAG, 프롬프팅을 적용하여 사용한다. 따라서 파인튜닝이야 말로 LLM을 실전에 사용할 수 있는 핵심적인 기술이다. 다음 글에서 파인튜닝에 대해서 심도있게 다뤄보겠다.

[[참고] Attention/Transformer 시각화로 설명, 임커밋 (YouTube)](https://www.youtube.com/watch?v=6s69XY025MU)











