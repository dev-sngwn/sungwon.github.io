---
layout: post
title: 텍스트를 음성 mp3로 간단하게 변환하기 (With Naver Cloud Platform)
subtitle: ": 딥러닝 몰라도 TTS 하는 법"
tags: [TTS, TextToSpeech, CSS, Clova, NCP, NaverCloud, NaverCloudPlatform]
---
<strong>이전 글</strong><br>

* [<i>한국어 OCR 해내기 1편: 가뿐하게 OCR API를 만들고 쓰기</i>](https://dev-sngwn.github.io/2019-12-17-korean-ocr-step-by-step-1/)<br>
* [<i>한국어 OCR 해내기 2편: 입맛대로 커스텀한 OCR 만들기</i>](https://dev-sngwn.github.io/2019-12-17-korean-ocr-step-by-step-2/)<br>
* [<i>음성인식 코드 짜는 최단 경로: 순식간에 STT 완성하기</i>](https://dev-sngwn.github.io/2020-01-13-stt-step-by-step/)

<br>
&nbsp;Naver Cloud Platform(이하 NCP)은 AI 서비스를 API 형태로 제공한다. 성능은 그들도 신이 아니기에 100% 완벽하지는 않지만, 직접 만들자 생각해보면 답없... 적어도 <strong>현존하는 서비스 중에는 가장 높은 수준</strong>을 보이지 않나 생각한다(실제로 수많은 Task들에 이름을 올린 것으로 증명했다).<br>
<br>
&nbsp;이번엔 NCP의 Text-to-Speech(TTS) 모듈인 Clova Speech Synthesis(CSS)를 사용해 본 후기를 공유하고자 한다.<br>
<br>
<br>
<br>

#### 1. 애플리케이션 등록
--------------------------
&nbsp;언제나와 같이 <a href="https://www.ncloud.com/"><strong>NCP</strong></a>로 이동하여 서비스를 신청하도록 하자. <strong>[서비스] → [AI Service] → [Clova Speech Synthesis(CSS)]</strong>로 이동한 후 <strong>[이용 신청하기]</strong>를 눌러주길 바란다.
<center>
[Service Main]
</center>
<br>
&nbsp;TTS는 STT와 굉장히 밀접한 관계를 가지고 있다. CSR과 CSS 또한 그러하여, 사용법이 굉장히 유사하다. 먼저 언제나 그랬듯이 <strong>[Application 등록]</strong>을 눌러 서비스를 생성하길 바란다.<br>
<br>

<center>
[Create Service]
</center>

<br>
&nbsp;<strong>Application 이름</strong>을 입력한 후, 아래 리스트에서 사용하고자 하는 서비스를 선택한다. NCP에서 제공하는 TTS 모듈의 이름은 CSS임에 유의한다. 체크해준 후, 아래로 쭉쭉!<br>
<br>

<center>
[IP]
</center>

<br>
&nbsp;Android와 iOS는 이번 글에서는 다루지 않는다(<del>자꾸 CSR 때 했던 말을 반복하는 것 같다</del>). 로컬 환경에서만 요청을 주고 받을 것이므로 <strong>서비스 환경 등록</strong> 부분에는<i>https://localhost</i>를 입력하고 추가하면 서비스를 생성할 수 있다.<br>
<br>

<center>
[Auth]
</center>

<br>
&nbsp;생성된 서비스의 인증 정보를 메모해두면 NCP에서 할 일은 끝났다. 정말 간단하다!<br>
<br>
<br>
<br>

#### 2. API 사용
--------------------------
<center>

[Data Table]

</center>

<br>
&nbsp;이번 모듈은 JSON 형태로 사용하는 방법을 알아내지 못했다(필자의 견문이 부족해서...). 하지만 Usage만 봐도 원하는 대로 커스텀할 수 있을 정도로 코드가 간단하니, 걱정은 말자!<br>
<br>
&nbsp;아래 소스를 실행하면 <code>text</code>의 텍스트가 음성 파일로 변환되어 소스 경로에 저장된다.
<br>
&nbsp;<i>* 참고: 필자는 Jupyter Notebook에서 테스트를 진행했으며 그 외의 파이썬 환경에서는 동작하지 않을 수 있다.</i>

~~~ python
import os
import sys
import urllib.request
import time

client_id = "your_client_id" # 인증 정보의 Client ID
client_secret = "your_client_secret" # 인증 정보의 Client Secret

speaker = "jinho"
"""
음성 합성에 사용할 목소리 종류

mijin      : 한국어, 여성 음색
jinho      : 한국어, 남성 음색
clara      : 영어, 여성 음색
matt       : 영어, 남성 음색
shinji     : 일본어, 남성 음색
meimei     : 중국어, 여성 음색
liangliang : 중국어, 남성 음색
jose       : 스페인어, 남성 음색
carmen     : 스페인어, 여성 음색

"""
speed = 0 # -5(Fast) to 5(Slow)

text = urllib.parse.quote("네이버 클라우드 플랫폼의 CSS 모듈 시험 문장입니다.")
data = "speaker=" + speaker + "&speed=" + str(speed) + "&text=" + text

url = "https://naveropenapi.apigw.ntruss.com/voice/v1/tts"
request = urllib.request.Request(url)

request.add_header("X-NCP-APIGW-API-KEY-ID", client_id)
request.add_header("X-NCP-APIGW-API-KEY", client_secret)

response = urllib.request.urlopen(request, data=data.encode('utf-8'))

if response.getcode()==200:
    
    print("CSS 성공! 파일을 저장합니다.")
    
    now = time.localtime()
    response_body = response.read()
    file_name = speaker + "_" + str(speed) + "_" + \
                "%04d%02d%02d_%02d%02d%02d" % \
                (now.tm_year, now.tm_mon, now.tm_mday,
                 now.tm_hour, now.tm_min, now.tm_sec)
    
    with open(file_name + ".mp3", 'wb') as f:
        f.write(response_body)
        
    print("파일명:", file_name)
        
else:
    print("Error Code:" + rescode)
~~~
~~~ markdown
CSS 성공! 파일을 저장합니다.
파일명: jinho_0_20YYMMDD_HHMMSS # 코드가 포함된 경로에 저장됩니다.
~~~
<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-02-16-tts-step-by-step/sample.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<i>네이버 클라우드 플랫폼의 CSS 모듈 시험 문장입니다.</i>
</center>

<br>
&nbsp;오... <strong>제법 자연스러운 음성</strong>이 생성되었다.<br>
<br>
&nbsp;하지만 Clova의 음성 생성 프로젝트들은 직접 <strong>[더빙](https://clova.ai/voice)</strong>을 하거나 <strong>[전화 상담](https://www.facebook.com/ClovaAIResearch/posts/1013208905715490?__xts__%5B0%5D=68.ARAQPbA8sd7SEbZkQ7eGVv7xOGG-NwzoyvSwQ_mENlR3TCo77FpK1lAZ7q7AVWYKqdyqrG0i7NbLj120-Sj5tTtwyrmCCSUE8iw3Jpx5t042iyCteesT-kVEdfaStY6ps3FncLtuX8-oCdY_b2i-ssgEFeclwOTtXrQ1rEnvt1AnhaDiPv0_O3ytNbJ11fsyJO9FewElhRk5R0mRdn0NvCk2l6o-S6bFA-xEFDhAyeH6ODYQLa7i6XIDbTYivhbTcO8egrFO5RQxBrNKBnyEBmQZia8PUco6wdcaKohlCg3rXqSIIbOu7zF_OLAFvspXGlCPOss0_VQjiXqmx5sjSMAg-Q&__tn__=-R)</strong>을 할 정도로 수준이 높아 그만큼 기대가 되었는데, 그에 비하면 조금 아쉬운 결과긴 하다. 사실 요청 오는 고작 한 문장으로 감정 상태를 파악하는 게 절대 쉽지 않아서(솔직히 불가능이라서) 이보다 자연스러운 억양은 자동으로는 안될 것 같고, 감정 상태를 직접 정의해주는 방식으론 가능하지 않을까 생각한다. <i>작성 중에 알아보니 Clova Premium Voice에서 해당 기능을 지원하는 듯 하다, 추후 다뤄보겠다.</i>..<br>
<br>
&nbsp;가지고 놀만한 것은 <strong>Speaker</strong>! <del>꽤 많은 음성이 준비되었다고 생각했는데, 직접 해보니 지원되지 않는 음성이 많았다.</del> <strong>한국어를 지원하는 음성이 있고, 올바른 언어를 전달해줘야만 동작하는 언어가 있다(대부분).</strong> 각자의 음성을 들어보도록 하자!<br>

<details markdown="1">
<summary>모든 음성 펼치기</summary>

<center>

<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-02-16-tts-step-by-step/mijin.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<code>mijin (한국어, 여성 음색)</code>
</center>

<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-02-16-tts-step-by-step/jinho.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<code>jinho (한국어, 남성 음색)</code>
</center>

<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-02-16-tts-step-by-step/clara.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<code>clara (영어, 여성 음색)</code>
</center>

<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-02-16-tts-step-by-step/matt.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<code>matt (영어, 남성 음색)</code>
</center>

<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-02-16-tts-step-by-step/shinji.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<code>shinji (일본어, 남성 음색)</code>
</center>

<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-02-16-tts-step-by-step/meimei.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<code>meimei (중국어, 여성 음색)</code>
</center>

<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-02-16-tts-step-by-step/liangliang.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<code>liangliang (중국어, 남성 음색)</code>
</center>

<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-02-16-tts-step-by-step/jose.mp3"></source>
  <p>Your browser does not support the audio element.</p>
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-02-16-tts-step-by-step/carmen.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<code>jose (스페인어, 남성 음색)</code>
</center>

<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-02-16-tts-step-by-step/carmen.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<code>carmen (스페인어, 여성 음색)</code>
</center>

<br>

</center>

</details>

<br>
&nbsp;다 각국의 언어로 TTS를 진행했지만 한국어를 지원하는 음성은 뭐였냐면...<br>
<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-02-16-tts-step-by-step/hanbon.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<i>...?</i>
</center>

<br>
&nbsp;정말 이해할 수 없지만 이게 돼버렸다! 영어로 발음 기호를 적어서 준 것도, 일본어로 변환해서 준 것도 아니고... <strong>그냥 한국어를 전달</strong>했는데 TTS가 된다! 다른 언어들은 한국어만 전달하면 인식할 수 있는 단어가 없어 <code>HTTPError 400</code>을 발생시킨다. <strong>오직 <code>shinji</code> , 일본어에만 가능하다!</strong>
<br>
&nbsp;텍스트를 발음 기호로 변환 후 처리하는 방식이 아닐까? 라는 생각을 했는데, 그런 방식이면 어떤 언어던 다 작동되는게 맞고... 정말정말 이해할 수 없다... 어쩌면 클로바 팀도 한본어를 꿈꾸고 계셨던걸까 ㅋㅅㅋ?<br>
<br>
&nbsp;여하튼 한본어가 성공했으니 저 말을 스페인 억양으로 너무 시켜보고 싶은데 그게 안돼서 아쉽다...... 일본어도 해냈으니 스페인어도 해내실 수 있습니다...... 클로바 화이팅...!
<br>
<br>
<br>

Reference
-------------
* <a href="https://apidocs.ncloud.com/ko/ai-naver/clova_speech_synthesis/tts/">Naver Cloud Platform API 참조서 </a>
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