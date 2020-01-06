---
layout: post
title: Back Translation 정리 (edit)
subtitle: ": 번역기 성능 영혼까지 끌어모으기"
tags: [BackTranslation, Transformer, NMT, Translation, WMT]
---

## Back Translation 정리

&nbsp;[paperswithcode](https://paperswithcode.com/)는 머신러닝의 거의 모든 Task에 대해 오픈된 Dataset을 소개하고 각 Dataset에서 <i>State-of-the-Art</i> 를 달성한 방법론을 <strong>논문과 소스로 묶어서 소개</strong>하는 사이트다. 관심 있는 Task는 종종 확인하며 트렌드를 쫓으면 좋은데, 최근 Machine Translation Task를 확인했다가 처음 보는 개념(Back Translation)이 있어 해당 논문을 읽고 정리를 하고자 한다.

<br>

> 본 글은 <br>[\<Improving Neural Machine Translation Models with Monolingual Data\> (ACL 2016)](https://arxiv.org/pdf/1511.06709.pdf) 논문과 <br>[\<Understanding Back-Translation at Scale\> (EMNLP 2018)](https://arxiv.org/pdf/1808.09381.pdf) 논문을 참고하여 작성되었습니다.

<br>

### 0. Intro

-------------------

&nbsp;Back Translation이란 개념이 처음 소개된 것은 2016년 ACL에서였다. [\<Improving Neural Machine Translation Models with Monolingual Data\>](https://arxiv.org/pdf/1511.06709.pdf) 논문은 단일 언어 데이터(이하 단일 데이터)로 기계 번역의 유창성을 높이고자 하였다.<br><center>

[Encoder Decoder Image]

</center>

&nbsp; 기본적으로 기계 번역 모델은 Encoder-Decoder 구조를 이루며 <i>Sourece Sentence</i>가 Encoder에 입력되고, <i>Target Sentence</i>가 Decoder에 입력되며 훈련을 진행한다. 고로 두 문장이 한 쌍을 이루는 병렬 데이터가 불가피하다. 헌데 단일 데이터만으로 훈련을 하겠다니? 잘 상상이 가지 않는다. 저자들은 두 가지 방법을 제안했다.

<br>

### 1-1. Dummy Source Sentence

------------

&nbsp; 첫 번째 방법은 Encoder의 입력으로 <i>Dummy</i> 값을 주는 것이다. 저자들은 <i><code><null></code></i> 토큰을 생성하여 <i>Target Sentence</i>의 <strong>단일 데이터와 한 쌍</strong>을 이루게끔 하였다. 그리고 Encoder의 모든 Parameter은 Freeze하여 <i>Dummy</i> 값에 대한 학습은 일절 이루어지지 않도록 하였다.<br><center>

[Dummy Parallel Sentence & Freezed Encoder Decoder Image]

</center>

<br>

&nbsp;이 경우 Decoder만 새로운 문장에 대해 추가 학습이 진행되는 것과 같은 효과이므로, 저자들이 말한 대로 <strong>유창한 번역을 만드는 데에 도움이 될 것 같은 느낌</strong>이 든다!<br>&nbsp;유의할 점은 단일 데이터가 병렬 데이터의 수를 넘어가게 되면 <i>(비율이 1:1을 초과하면)</i> Decoder가 <i>Source Sentence</i>로부터 추출한 정보를 잊어버리고, <i>Target Sentence</i>에 의존적인 양상을 보이게 된다고 한다. 이 문제점을 해결하고자 한 것이 두 번째 방법이고, 바로 <span style="background-color: #eeeeff"><strong><strong>Back Translation</strong></strong></span>이다.

<br>

### 1-2. Synthetic Source Sentence (Back Translation)

-----------

<center>
[Synthetic Source Sentence Encoder Decoder Image]
</center>

<br>

&nbsp;아무런 정보도 담고 있지 않은 <i><code><null></code></i> 토큰을 입력으로 주느니, 완벽하지는 않더라도 <i>Target Sentence</i>를 보고 인공적인 <i>Source Sentence</i>를 만드는 방법론이다. 생성된 인공 데이터를 <i>Synthetic Source Sentence</i>라 칭한다. 그리고 인공 데이터를 생성하는 과정을 <span style="background-color: #eeeeff"><strong>Back Translation</strong></span>이라 정의하며, 그 과정을 다음과 같이 표현하고 있다.

> i.e. an automatic translation of the monolingual target text into the source language.<br>
>
> (즉, 단일 타겟 언어 문장을 소스 언어 문장으로 자동 번역하는 과정이다.)

&nbsp;<i>다수의 독자들은 이 방법이 정말 유용한지에 대해 의심할 것이라 생각한다 <del>(본인이 그랬기 때문에)</del>. 하지만 놀랍게도 인공적으로 생성된 데이터가 엉성할 수록 성능이 높아진다는 연구 결과가 있다. 이에 대해선 (part)에서 후술하도록 하겠다.</i><br>

&nbsp;여기까지가 [\<Improving Neural Machine Translation Models with Monolingual Data\>](https://arxiv.org/pdf/1511.06709.pdf) 논문이 제안하는 방법이다. 해당 논문에서도 실험을 진행했지만, 인공 데이터와 실제 데이터의 비율이나 개수 측면에서 다소 경험적으로 보이는 <del>(때려 맞춘)</del> 부분이 있어 여기서 소개하지는 않겠다. 가장 눈에 띄는 결과는 <strong><i>German→English WMT 15</i></strong> 에서 <span style="background-color: #eeffee"><strong>3.6 - 3.7 BLEU</strong></span>를 증가시킨 부분이다 <i>(31.6 BLEU in newstest 2015 with ensemble of 4)</i>. 추가로 인공 데이터를 생성하는 방법에 대해서 <i>Greedy Decoding</i>과 <i>Beem Search</i>를 비교한 부분이 있지만, 이 역시 후술할 논문에서 상세하게 다루고 있으니, 어서 다음 파트로 넘어가자.

<br><br>

### 2. Understanding Back-Translation at Scale

-----------

&nbsp;앞서 말했 듯이 Back Translation을 소개한 논문에는 다소 경험적으로 보이는 부분이 있었다. 즉, 분석이 부족했다. 이에 <span style="background-color: #eeeeff"><strong>Back Translation</strong></span>을 자세히 분석하려는 시도가 있었고, 해당 논문은 2018년 EMLNP에서 소개되었다. 무려 <strong>Facebook</strong>과 <strong>Google</strong>의 합작품인 [\<Understanding Back-Translation at Scale\>](https://arxiv.org/pdf/1808.09381.pdf)이다.

<br>&nbsp;저자들은 무려 <span style="background-color: #ffffee"><strong>6가지의 환경을 가정</strong></span>하고 실험을 진행했다.

<br>

&nbsp;<strong><i>1) 인공 데이터 생성 방법 </i></strong>을 먼저 분석하고, 1)에서 최적이었던 결과가 <strong><i>2) 질 좋은 인공 데이터를 생성하는가 </i></strong>에 대해 고찰하였다 (언어적인 측면에서). 또, 인공 데이터를 생성하는 모델도 결국 병렬 데이터로 훈련을 진행해야 하기에 <strong><i>3) 병렬 데이터가 얼마나 있어야 Back Translation이 효과적인가? </i></strong>에 대해 고려했다. 

<br>

&nbsp;여기서부턴 저자들이 'Back Translation의 후속 논문이 안나오게 하려는구나...' 하는 생각이 들 정도의 디테일인데, <strong><i>4) Back Translation을 진행하는 단일 데이터의 도메인이 미치는 영향</i></strong> 과 너무 인공 데이터만 학습하면 산으로 갈 수 있으니 <strong><i>5) 학습 중에 실제 데이터를 학습하는 빈도</i></strong> <i>(논문에서는 visit이라고 표현하였다)</i>, 마지막으로 이 모든 걸 종합하여 <strong><i>6) 온 힘 다해 학습했을 때의 성능</i></strong> 을 소개하며 논문을 마친다 <i>(V100 GPU 128개...)</i>.

<br>

&nbsp;자, 하나하나 톺아보도록 하자.

<br>

#### 1, 2) Synthetic Data Generation Methods & Analysis

----------------------

&nbsp;앞선 연구에서는 인공 데이터를 생성하는 데에 <i>Greedy Decoding</i>과 <i>Beam Search</i>를 사용했다. 저 두 방법은 모두 MAP(Maximum A-Posteriori)를 사용하며 <strong>가장 높은 확률을 갖는 데이터</strong>를 생성하게 된다. 저자들은 두 방법 외에 <strong>Non-MAP</strong> 방법을 비교해보고자 하였다. Non-MAP 방법으론 <i>Random Sampling</i>과 <i>Top-10 Sampling</i>, <i>Noise를 추가한 Beam Search (이하 Noise Beam)</i>이 있다.<br>
&nbsp;각 방법에 대한 간단한 설명과 예문을 첨부한다.<br><center>

[Example Sentence]

[Method Simple Diagram]

</center>

> <strong>Random Sampling</strong>: 모델의 확률 분포에 따라서 랜덤하게 단어를 선택한다. 예를 들어 모델의 확률 분포가 "I"의 다음 단어로 "am"을 90%, "will"을 10%로 가진다면 10문장을 생성했을 때 "I am" 은 9문장, "I will"은 1문장이 나오는 식이다.<br>
>
> <strong>Top-10 Sampling</strong>: Random Sampling과 유사하지만, 각 단어들을 확률 내림차순으로 정렬했을 때 Top-10을 제외하곤 생성에 포함하지 않는다. 임의성이 Random Sampling보다 덜하므로 보다 문맥에 맞는 문장이 생성된다. 굳이 Top-10인 이유는 5, 20, 50으로 셋팅을 해봐도 비슷한 결과가 나오기에 그렇다.
>
> <br>
>
> <strong>Noise Beam</strong>: Beam Search로 생성된 결과에 Noise를 추가하였다. 각 단어를 10%의 확률로 지우고, 10%의 확률로 <i>filler token</i> 인 <code>BLANK</code>로 치환하였다. 

<br>&nbsp;실험 결과는 다음과 같다.<br><center>

[Graph]

</center>

<br>&nbsp;총 500만 개의 병렬 데이터를 학습한 후, 인공 데이터를 300만개, 600만개, 1200만개 씩 더하며 실험을 진행한 결과이다. 언어적으로 더 좋은 번역을 만드는 MAP 방법을 <strong>Non-MAP 방법이 능가</strong>한다는 사실은 다소 놀랍다. 그리고 가장 효과적인 방법 (<i>Noise Beam</i>)은 병렬 데이터만 학습했을 때보다 <span style="background-color: #eeffee"><strong>1.7 - 2.0 BLEU</strong></span>만큼 향상되었다. 

