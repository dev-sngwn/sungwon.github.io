---
layout: post
title: Back Translation 정리
subtitle: ": 번역기 성능 영혼까지 끌어모으기"
tags: [BackTranslation, Transformer, NMT, Translation, WMT]
---
<br>
> 직접 발표를 진행한 내용입니다,<br>
> 본 글보다 쉽게 설명하였으니 혹여나 내용이 어렵다면 영상을 먼저 보시길 권장합니다.
> Link: [my-slide-and-presentation](https://github.com/dev-sngwn/my-slide-and-presentation)
<br>

<center>
<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/01_sota.png" width="100%"/><br>
<i><a href="https://paperswithcode.com/task/machine-translation">Machine Translation Leaderboards</a></i>

</center>
<br>

&nbsp;[paperswithcode](https://paperswithcode.com/)는 머신러닝의 거의 모든 Task에 대해 오픈된 Dataset을 소개하고 각 Dataset에서 <i>State-of-the-Art</i>를 달성한 방법론을 <strong>논문과 소스로 묶어서 소개</strong>하는 사이트다. 관심 있는 Task는 종종 확인하며 트렌드를 쫓으면 좋은데, 최근 Machine Translation Task를 확인했다가 처음 보는 개념(Back Translation)이 있어 해당 논문을 읽고 정리를 하고자 한다.<br>

> 본 글은 <br>[\<Improving Neural Machine Translation Models with Monolingual Data\>](https://arxiv.org/pdf/1511.06709.pdf) 논문과 <br>[\<Understanding Back-Translation at Scale\>](https://arxiv.org/pdf/1808.09381.pdf) 논문을 참고하여 작성되었습니다.

<br>

### 0. Intro
-------------------
&nbsp;Back Translation이란 개념이 처음 소개된 것은 2016년 ACL에서였다. [\<Improving Neural Machine Translation Models with Monolingual Data\>](https://arxiv.org/pdf/1511.06709.pdf) 논문은 단일 언어 데이터(이하 단일 데이터)로 기계 번역의 유창성을 높이고자 하였다.<br>
<br>
<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/02_en_de.png" width="100%"/>

</center>
<br>
&nbsp; 기본적으로 기계 번역 모델은 Encoder-Decoder 구조를 이루며 <i>Sourece Sentence</i>가 Encoder에 입력되고, <i>Target Sentence</i>가 Decoder에 입력되며 훈련을 진행한다. 고로 두 문장이 한 쌍을 이루는 병렬 데이터가 불가피하다. 그런데 단일 데이터만으로 훈련을 하겠다니? 잘 상상이 가지 않는다. 저자들은 두 가지 방법을 제안했다.<br>
<br>
<br>
<br>

### 1-1. Dummy Source Sentence
------------
&nbsp; 첫 번째 방법은 Encoder의 입력으로 <i>Dummy</i> 값을 주는 것이다. 저자들은 <i><code>null</code></i> 토큰을 생성하여 <i>Target Sentence</i>의 <strong>단일 데이터와 한 쌍</strong>을 이루게끔 하였다. 그리고 Encoder의 모든 Parameter은 Freeze하여 <i>Dummy</i> 값에 대한 학습은 일절 이루어지지 않도록 하였다.<br>
<br>
<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/03_flu.png" width="100%"/>

</center>
<br>
&nbsp;이 경우 Decoder만 새로운 문장에 대해 추가 학습이 진행되는 것과 같은 효과이므로, 저자들이 말한 대로 <strong>유창한 번역을 만드는 데에 도움이 될 것 같은 느낌</strong>이 든다!<br>&nbsp;유의할 점은 단일 데이터가 병렬 데이터의 수를 넘어가게 되면 <i>(비율이 1:1을 초과하면)</i> Decoder가 <i>Source Sentence</i>로부터 추출한 정보를 잊어버리고, <i>Target Sentence</i>에 의존적인 양상을 보이게 된다고 한다. 이 문제점을 해결하고자 한 것이 두 번째 방법이고, 바로 <span style="background-color: #eeeeff"><strong><strong>Back Translation</strong></strong></span>이다.<br>
<br>
<br>
<br>
### 1-2. Synthetic Source Sentence (Back Translation)
-----------
<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/04_synthetic.png" width="100%"/>

</center>
<br>
&nbsp;아무런 정보도 담고 있지 않은 <i><code>null</code></i> 토큰을 입력으로 주느니, 완벽하지는 않더라도 <i>Target Sentence</i>를 보고 인공적인 <i>Source Sentence</i>를 만드는 방법론이다. 생성된 인공 데이터를 <i>Synthetic Source Sentence</i>라 칭한다. 그리고 인공 데이터를 생성하는 과정을 <span style="background-color: #eeeeff"><strong>Back Translation</strong></span>이라 정의하며, 그 과정을 다음과 같이 표현하고 있다.

> i.e. an automatic translation of the monolingual target text into the source language.<br>
> (즉, 단일 타겟 언어 문장을 소스 언어 문장으로 자동 번역하는 과정이다.)

&nbsp;<i>다수의 독자들은 이 방법이 정말 유용한지에 대해 의심할 것이라 생각한다 <del>(본인이 그랬기 때문에)</del>. 하지만 놀랍게도 인공적으로 생성된 데이터가 엉성할수록 성능이 높아진다는 연구 결과가 있다. 이에 대해선 후술하도록 하겠다.</i><br>
<br>
&nbsp;여기까지가 [\<Improving Neural Machine Translation Models with Monolingual Data\>](https://arxiv.org/pdf/1511.06709.pdf) 논문이 제안하는 방법이다. 해당 논문에서도 실험을 진행했지만, 인공 데이터와 실제 데이터의 비율이나 개수 측면에서 다소 경험적으로 보이는 <del>(때려 맞춘)</del> 부분이 있어 여기서 소개하지는 않겠다. 가장 눈에 띄는 결과는 <strong><i>German→English WMT 15</i></strong> 에서 <span style="background-color: #eeffee"><strong>3.6 - 3.7 BLEU</strong></span>를 증가시킨 부분이다 <i>(31.6 BLEU in newstest 2015 with ensemble of 4)</i>. 추가로 인공 데이터를 생성하는 방법에 대해서 <i>Greedy Decoding</i>과 <i>Beem Search</i>를 비교한 부분이 있지만, 이 역시 후술할 논문에서 상세하게 다루고 있으니, 어서 다음 파트로 넘어가자.<br>
<br>
<br>
<br>
### 2. Understanding Back-Translation at Scale
-----------
&nbsp;앞서 말했 듯이 Back Translation을 소개한 논문에는 다소 경험적으로 보이는 부분이 있었다. 즉, 분석이 부족했다. 이에 <span style="background-color: #eeeeff"><strong>Back Translation</strong></span>을 자세히 분석하려는 시도가 있었고, 해당 논문은 2018년 EMLNP에서 소개되었다. 무려 <strong>Facebook</strong>과 <strong>Google</strong>의 합작품인 [\<Understanding Back-Translation at Scale\>](https://arxiv.org/pdf/1808.09381.pdf)이다.<br>
<br>
&nbsp;저자들은 무려 <span style="background-color: #ffffee"><strong>6가지의 환경을 가정</strong></span>하고 실험을 진행했다.<br>
<br>
&nbsp;<strong><i>1) 인공 데이터 생성 방법 </i></strong>을 먼저 분석하고, 1)에서 최적이었던 결과가 <strong><i>2) 질 좋은 인공 데이터를 생성하는가 </i></strong>에 대해 고찰하였다 (언어적인 측면에서). 또, 인공 데이터를 생성하는 모델도 결국 병렬 데이터로 훈련을 진행해야 하기에 <strong><i>3) 병렬 데이터가 얼마나 있어야 효과적인가? </i></strong>에 대해 고려했다. <br>
<br>
&nbsp;여기서부턴 저자들이 'Back Translation의 후속 논문이 안 나오게 하려는구나...' 하는 생각이 들 정도의 디테일인데, <strong><i>4) Back Translation을 진행하는 단일 데이터의 도메인이 미치는 영향</i></strong> 과 너무 인공 데이터만 학습하면 산으로 갈 수 있으니 <strong><i>5) 학습 중에 실제 데이터를 학습하는 빈도</i></strong>, 마지막으로 이 모든 걸 종합하여 온 힘 다해 학습했을 때의 성능을 소개하며 논문을 마친다 <i>(V100 GPU 128개...)</i>.<br>
<br>
&nbsp;자, 하나하나 톺아보도록 하자.<br>
<br>
<br>
<br>
#### 1, 2) Synthetic Data Generation Methods & Analysis
----------------------
&nbsp;앞선 연구에서는 인공 데이터를 생성하는 데에 <i>Greedy Decoding</i>과 <i>Beam Search</i>를 사용했다. 저 두 방법은 모두 MAP(Maximum A-Posteriori)를 사용하며 <strong>가장 높은 확률을 갖는 데이터</strong>를 생성하게 된다. 저자들은 두 방법 외에 <strong>Non-MAP</strong> 방법을 비교해보고자 하였다. Non-MAP 방법으론 <i>Random Sampling</i>과 <i>Top-10 Sampling</i>, <i>Noise를 추가한 Beam Search (이하 Noise Beam)</i>이 있다.<br>
<br>
&nbsp;각 방법에 대한 간단한 설명과 예문을 첨부한다.
<br>
<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/05_ex.png" width="100%"/>

</center>

> <strong>Reference</strong>: 원본 문장.<br>
> <br>
> <strong>Beam Search</strong>: Source Sentence와 Target Sentence의 일부가 주어졌을 때, 다음 단어로 가장 높은 확률의 단어를 즉시 택하는 것을 Greedy Decoding이라 한다. Beam Search는 Greedy Decoding의 발전된 형태로, 두 번째, 세 번째(범위는 Beam Size에 의해 결정)로 높은 확률의 단어도 택해 문장을 '만들어 본다'. 그리고 최종적으로 만들어진 문장들의 확률의 총합을 구하여 가장 높은 값을 갖는 문장을 생성한다.<br>
> <br>
> <strong>Random Sampling</strong>: 모델의 확률 분포에 따라서 랜덤하게 단어를 선택한다. 예를 들어 모델의 확률 분포가 "I"의 다음 단어로 "am"을 90%, "will"을 10%로 가진다면 10문장을 생성했을 때 "I am" 은 9문장, "I will"은 1문장이 나오는 식이다.<br>
> <br>
> <strong>Top-10 Sampling</strong>: Random Sampling과 유사하지만, 각 단어들을 확률 내림차순으로 정렬했을 때 Top-10을 제외하곤 생성에 포함하지 않는다. 임의성이 Random Sampling보다 덜하므로 보다 문맥에 맞는 문장이 생성된다. 굳이 Top-10인 이유는 5, 20, 50으로 실험을 해봐도 비슷한 결과가 나오기에 그렇다.<br>
> <br>
> <strong>Beam + Noise</strong>: Beam Search로 생성된 결과에 Noise를 추가하였다. 각 단어를 10%의 확률로 지우고, 10%의 확률로 <i>filler token</i> 인 <code>BLANK</code>로 치환하였다. 

<br>
&nbsp;실험 결과는 다음과 같다.
<br>
<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/06_graph_1.png" width="100%"/>

</center>
<br>
&nbsp;총 500만 개의 병렬 데이터를 학습한 후, 인공 데이터를 수백만 개씩 더하며 2,900만 개까지 실험을 진행한 결과이다. 언어적으로 더 좋은 번역을 만드는 MAP 방법을 <strong>Non-MAP 방법이 능가</strong>한다는 사실은 다소 놀랍다. 그리고 가장 효과적인 방법 (<i>Noise Beam</i>)은 병렬 데이터만 학습했을 때보다 <span style="background-color: #eeffee"><strong>1.7 - 2.0 BLEU</strong></span>만큼 향상되었다.<br>
<br>
&nbsp;저자들은 위 현상이 <strong>Beam Search가 너무 그럴듯한 번역을 만들기 때문</strong>이라고 말한다. 이를 설명하기 위해 반대로 생각하면, Noise를 더한 Beam Search도 엄연히 MAP 방법인데 가장 좋은 성능을 보였다. 즉 문제는 <strong>MAP / Non-MAP가 아니라</strong>, 생성된 인공 데이터가 <strong>어떤 정보를, 얼마나 포함하느냐</strong>가 중요하다는 것이다. <i>(고로 이제부턴 MAP란 단어를 사용하지 않겠다)</i><br>
&nbsp;쉽게 말하면 번역 모델한테 문제집을 쥐여주는 셈이다. 심화 문제집을 많이 공부한 번역 모델은 그만큼 어려운 문제도 풀 수 있는 능력을 갖출 것이고 <i>(Sampling / Noise Beam)</i>, 쉬운 문제집만 풀어온 번역 모델은 현실에서 등장하는 문제조차도 풀기 어렵다는 것이다. 아래는 인공 데이터의 <i>Perplexity</i> 정보이다.

> Perplexity: 번역하면 당황도, 혼잡도 정도가 되는데, 번역이 얼마나 개판인지 평가해주는 지표이다. 높을수록 난해한 문장을 생성했다고 이해하면 된다.

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/07_graph_2.png" width="100%"/>

</center>
<br>
&nbsp;앞서 말한 예시대로라면 <strong>Perplexity는 번역 문제집의 난이도</strong>라고 표현할 수도 있겠다. 지표로 보니 <i>Sampling</i>과 <i>Noise Beam</i>은 심화 문제집이 확실하다.<br>
<br>
&nbsp;다만 저 문제집도 결국 <strong>문제 푸는 학생<i>(번역 모델)</i>이 만들었다는 점</strong>을 간과할 수 없다. 만약 충분히 학습되지 않은 모델이 어렵게만 인공 데이터를 만든다면, 그건 무작위의 단어를 나열한 것과 크게 다르지 않을 것이다. 따라서 저자들은 <i>Back Translation</i>이 효과적일 수 있는 <strong>최소한의 병렬 데이터 수</strong>를 알아내야 했다.<br>
<br>
<br>
<br>
#### 3)  Low Resource vs. High Resource
---------------------
&nbsp;앞서 진행한 실험은 500만 개에서 2,900만 개까지의 데이터에서 진행했기 때문에 어쩌면 객관성이 떨어질 수도 있다. 왜냐면 저 정도의 병렬 데이터는 그럴듯한 번역기를 만들기에 충분해서, <i>Random Sampling</i>조차도 이상적인 확률 분포에서 이루어졌을 수 있지 않은가? 그렇기 때문에 저자들은 병렬 데이터 수를 줄여가며 실험을 진행했고, 그 환경을 <strong>Low Resource</strong>라 칭하였다.<br>
<br>
<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/08_graph_3.png" width="100%"/>

</center>
<br>
&nbsp;위 그래프는 병렬 데이터를 8만 개까지 줄여가며 실험을 진행한 결과이다. <span style="color: #aa3333"><strong>붉은 선</strong></span>이 <i>Sampling</i>이고, <span style="color: #3333aa"><strong>푸른 선</strong></span>이 <i>Beam</i>. 결과는 한눈에 들어오는 편이다. <strong>적어도 <i>64만 개 이상의 병렬 데이터</i> 를 가지고 있을 때, Sampling이 효과를 보기 시작한다</strong>. 추가로, <strong>Back Translation 자체는 모든 경우에서 효과적이다</strong>. 이쯤에서 우린 번역 모델을 완성하고, 모든 훈련을 마친 후 쓸 수 있는 히든카드를 얻은 셈이다 <del>(중복 할인되는 쿠폰만큼이나 든든하다)</del>.<br>
<br>
<br>
<br>
#### 4) Domain of Synthetic Data
---------------------
&nbsp;Back Translation은 적당한 양의 병렬 데이터만 있으면 활용이 가능하다. 단일 데이터는 모으고자 하면 얼마든지 모을 수 있다. 오픈된 말뭉치 데이터를 써도 좋고, 웹을 크롤링할 수도 있고, 심지어 직접 타이핑을 해도 무방하다. 이때, <strong>단일 데이터의 출처(도메인)와 번역기의 성능은 연관이 있는가?</strong> 다시 말해, <strong>소설책으로 학습해도 뉴스에서 쓰일 수 있는가?</strong><br>
<br>
<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/09_graph_4.png" width="100%"/>

</center>
<br>
&nbsp;위 그래프는 <i>WMT</i>의 평가 데이터셋인 <i>newstest</i>로 실험을 진행한 결과이다. 이름에 명시되어 있듯이 뉴스에서 크롤링한 병렬 데이터이다. <span style="color: #aa7733"><strong>갈색 선</strong></span>이 <strong>뉴스 데이터로 Back Translation</strong>한 결과, <span style="color: #aa3333"><strong>붉은 선</strong></span>이 <strong>일반적인 데이터(정확히는 병렬 데이터의 일부)로 Back Translation</strong>한 결과, <span style="color: #3333aa"><strong>푸른 선</strong></span>이 <strong>그냥 병렬 데이터</strong>로 학습한 결과이다.<br>
<br>
&nbsp;특정 구간에서는 <strong>Back Translation이 실제 데이터를 능가</strong>하기도 한다! 물론 <span style="color: #aa3333"><strong>붉은 선</strong></span>은 성능이 떨어지는 걸로 보아 <strong>뉴스에 한정적임</strong>을 직감할 수 있다. <i>어느 분야에서나 그렇겠지만 도메인 의존적인 것은 지양해야 한다. 결국 목적은 실제 세상에 적용하고자 함이니, 온실 속 화초는 시들기 마련 아니겠는가?</i> 고로 저자들은 온실 속 화초를 야생에 내놓았다.<br>
<br>
<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/10_graph_5.png" width="100%"/>

</center>
<br>
&nbsp;위 그래프는 여러 가지 도메인을 섞은 데이터셋으로 실험을 진행한 결과이다. 뉴스에서 강건했던 <span style="color: #aa7733"><strong>갈색 선</strong></span>은 맥 없이 고꾸라지고 말았다. 대신 이 부분에선 첫 실험에서 <i>가려져 있던 빛나는 결과</i>를 볼 수 있다. 이전 실험도 그렇고 <span style="color: #aa3333"><strong>붉은 선</strong></span>이 <span style="color: #3333aa"><strong>푸른 선</strong></span>에 비해 성능이 크게 뒤처지지 않는다. 그 말은 곧, <span style="background-color: #eeffee"><strong>64만 개의 병렬 데이터만 확보된다면 500만 개가 있을 때의 성능을 유사하게 만들어 낼 수 있다는 것</strong></span>이다! 데이터가 적어 중도 포기한 번역 모델이 있다면, 오랜만에 다시 살펴보도록 하자.<br>
<br>
<br>
<br>

#### 5) Upsampling the Bitext
---------------------
&nbsp;이 부분을 <i>Upsampling</i>이라고 칭한 이유는 잘 모르겠지만(기존에 알던 개념과 연관을 찾기 어렵다), 우선 개념적으론 <strong>인공 데이터를 학습하는 도중, 실제 데이터를 방문하는 빈도</strong>이다. 앞서 언급한 표현을 빌려오면 Back-Translation은 결국 학생이 직접 만든 문제집을 푸는 셈이기 때문에, <strong>실제 데이터와 동떨어진 내용을 학습하게 될 우려</strong>가 있다. 따라서 실제 데이터를 <strong><span style="background-color: #eeeeee">방문</span></strong>하는 빈도에 대한 실험은 의의를 가진다.<br>

> 실제로 저자들은 방문(visit)이란 표현을 사용했다. 

&nbsp;<i>Upsampling Rate</i>가 1이면 <strong>인공 데이터를 1번 볼 때 실제 데이터도 1번 보는 것(1:1)</strong>이고, 8이면 <strong>인공 데이터를 1번 볼 때 실제 데이터를 8번 보는 것(1:8)</strong>이다. 즉, <i>Upsampling Rate</i>가 <strong>높을수록 실제 데이터를 많이 학습</strong>하게 된다.<br>
<br>

<center>
<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-07-back-translation/11_graph_6.png" width="100%"/>

</center>
<br>
&nbsp;<strong><span style="color: #3333aa">Greedy Decoding</span></strong>과 <strong><span style="color: #aa3333">Beam Search</span></strong>는 <i>Upsampling Rate</i>가 높아질수록 <strong>성능이 올라가는 모습</strong>을 보인다. 반면에 <strong><span style="color: #aa7733">Top-10 Sampling</span></strong>, <strong><span style="color: #333333">Random Sampling</span></strong>, <strong><span style="color: #ccaa11">Noise Beam</span></strong>은 성능에 큰 변화가 없거나 <strong>심지어 떨어지는 모습</strong>을 보인다.<br>
&nbsp;단순하게 <strong><span style="color: #3333aa">Greedy Decoding</span></strong>과 <strong><span style="color: #aa3333">Beam Search</span></strong>가 만드는 문제집이 <strong>너무 쉽기 때문</strong>이라고 생각할 수 있다. 반대로 나머지 방법론이 만든 문제집은 충분히 <strong>좋은 내용이 담긴 심화 문제집</strong>이었기에, 굳이 실제 데이터를 보지 않아도 성능이 잘 나오는 것이다. <br>
<br>
<br>
<br>
### 마치며
-----------
&nbsp;실험 결과는 서론에 언급했듯이 <i>WMT 2014</i>에서 <span style="background-color: #eeffee"><strong>35.0 BLEU</strong></span>로 <i>State-of-the-Art</i> 성능을 달성하였다. 자세한 실험 환경과 결과값이 궁금하다면 논문을 참고하길 바란다.<br>
&nbsp;본 논문은 기계 번역에 포커스를 맞춘 논문이지만, 어렴풋한 느낌으로는 많은 분야에 활용이 가능할 것 같다. <strong>인공 데이터가 의미 있으며 실제 데이터를 능가할 수도 있다는 것을 증명</strong>하였으니, 모두 각자 연구하는 분야에 적용시킬 궁리를 해보도록 하자!
<br>
<br>
<br>

<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://dev-sngwn.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
