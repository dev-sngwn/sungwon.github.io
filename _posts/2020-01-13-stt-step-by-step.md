---
layout: post
title: 음성인식 코드 짜는 최단 경로 (With Naver Cloud Platform)
subtitle: ": 순식간에 STT 완성하기"
tags: [STT, SpeechToText, CSR, Clova, NCP, NaverCloud, NaverCloudPlatform]
---
&nbsp;불현듯 <strong>한본어 서비스</strong>를 만들어 보고 싶다는 생각이 들었다. 인터넷 방송이나 유튜브를 즐겨보시는 분은 익숙하실 텐데 한본어가 뭐냐면... 
<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-13-stt-step-by-step/hanbon.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<i><del>(한봉오가 몬지 모르씬다면 써르명해드리능게 인지산종)</del></i>
</center>

<br>
&nbsp;이렇게 한국어 발음을 일본어로 적고 Text-to-Speech(TTS)를 활용해 일본어스러운(?) 한국어를 만드는 것이다. 

><strong>한국어 목소리 → 일본어 STT → 텍스트 문장 → 일본어 TTS → EZ하게 한본어?!</strong> 

가 기본 아이디어였다.<br>
<br>
&nbsp;내가 아는 한 가장 쉽고 정확하게 STT와 TTS를 제공하는 곳은 <strong>네이버 클라우드 플랫폼(이하 NCP)</strong>이 분명하므로 이번에도 NCP를 활용해보았고, 그 과정을 정리해보고자 한다.<br>
<br>
<br>
<br>

### 1. 어플리케이션 등록
-------------------
&nbsp;우선 <a href="https://www.ncloud.com/"><strong>NCP</strong></a>로 이동하여 서비스를 신청하도록 하자. <strong>[서비스] → [AI Service] → [Clova Speech Recognition(CSR)]</strong>로 이동한 후 <strong>[이용 신청하기]</strong>를 눌러주길 바란다.
<br>
<br>

<center>
<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-13-stt-step-by-step/01_main.png"/>
</center>
<br>
&nbsp;이동하고 나면 위와 같은 화면을 만나게 될 텐데, <strong>[Application 등록]</strong>을 눌러 진행한다.
<br>
<br>

<center>
<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-13-stt-step-by-step/02_csr.png"/>
</center>
<br>
&nbsp;<strong>Application 이름</strong>을 입력한 후, 아래 리스트에서 원하는 서비스를 선택한다. NCP에서 제공하는 STT 모듈의 이름은 <span style="background-color: #ddffdd"><strong><i>Clova Speech Recognition(CSR)</i></strong></span> 임에 유의하자. 체크!
<br>
<br>

<center>
<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-13-stt-step-by-step/03_done.png"/>
</center>
<br>
&nbsp;하단으로 내려오면 <strong>서비스 환경 등록</strong> 부분이 있는데, <strong>요청을 보낼 IP</strong>를 적으면 된다. 이번 글에서는 로컬에서만 구동할 것이므로 <i>http://localhost</i> 라고 적은 후 URL을 <strong>추가</strong>하자. <i>Android와 iOS, 로컬 외의 Web은 이번 글에서는 다루지 않는다.</i>
<br>
<br>

<center>
<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-13-stt-step-by-step/04_list.png"/>
</center>
<br>
&nbsp;모든 과정을 마치고 나면 Application이 성공적으로 등록된 것을 확인할 수 있다.<br>
<br>
<br>
<br>

### 2. API 사용
-------------------
<center>
<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-13-stt-step-by-step/05_auth.png"/>
</center>
<center>
<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-13-stt-step-by-step/06_auth.png"/>
</center>
<br>
&nbsp;생성된 Application의 <strong>[인증 정보]</strong>를 눌러 <i>Client ID</i> 와 <i>Client Secret</i> 을 메모해둔다. 여기까지가 NCP에서 할 일! 이제 본인이 사용하는 파이썬 환경으로 이동하자. <br>
<br>
&nbsp;<i>*참고: 필자는 Jupyter Notebook에서 테스트를 진행했는데, 본 API는 허용할 URL을 지정해주는 방식이다 보니 그 외의 파이썬 환경에서는 동작하지 않을 수 있다.</i>
<br>
<br>

<center>
<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-13-stt-step-by-step/07_spec.png"/>
</center>
<br>
&nbsp;NCP에서 제공하는 [API 참조서](https://apidocs.ncloud.com/ko/ai-naver/clova_speech_recognition/stt/)에 첨부된 <strong>요청 정보</strong>이다. 요청을 보낼 URL, 파라미터와 헤더, 바디... 정말 쓰기 쉽게 잘 작성되어 있다! 이제 테스트를 진행하기 위한 목소리를 가져와보자.<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-13-stt-step-by-step/original.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<i>"네이버 클라우드 플랫폼의 CSR 테스트를 위한 데모 문장입니다."</i>
</center>
<br>
&nbsp;실험을 진행한 소스는 다음과 같다.

~~~python
import json
import requests

data = open("your/path/to/voice.mp3", "rb") # STT를 진행하고자 하는 음성 파일

Lang = "Kor" # Kor / Jpn / Chn / Eng
URL = "https://naveropenapi.apigw.ntruss.com/recog/v1/stt?lang=" + Lang
    
ID = "your_client_id" # 인증 정보의 Client ID
Secret = "your_secret" # 인증 정보의 Client Secret
    
headers = {
    "Content-Type": "application/octet-stream", # Fix
    "X-NCP-APIGW-API-KEY-ID": ID,
    "X-NCP-APIGW-API-KEY": Secret,
}
response = requests.post(URL,  data=data, headers=headers)
rescode = response.status_code

if(rescode == 200):
    print (response.text)
else:
    print("Error : " + response.text)
~~~

<br>

~~~javascript
// Expected: "네이버 클라우드 플랫폼의 CSR 테스트를 위한 데모 문장입니다."
output: // Original Vesion
{"text":"네이버 클라우드 플랫폼의 csr 테스트를 위한 템 5 문장입니다"}
~~~
<br>
&nbsp;이 정도 정확도라면 필자가 <i>"데모"</i> 를 <i>"템오"</i> 라고 발음했다고 인정하는 게 나을 것 같다... 일전에 STT 관련 업무를 하시는 분을 뵌 적이 있었는데, <strong><i>"잘못된 단어는 문맥상 더 맞는 단어로 고쳐주는 것이 맞지 않느냐"</i></strong> 라고 여쭙자 <strong><i>"우린 들리는 대로 그대로 적을뿐, 그 이상은 언어 처리하는 분이 해야 한다"</i></strong> 라는 답변을 받은 적이 있었다. 아마 이런 케이스에서 그것이 여실히 드러나는 것 같다.
<br>
<br>
&nbsp;한 문장만 하면 재미없으니, 두 가지 실험을 더 해보려고 한다.
<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-13-stt-step-by-step/fast.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<i>Fast Version</i>
</center>
<br>
<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-13-stt-step-by-step/tuned.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<i>Tune Version</i>
</center>

<br>
&nbsp;개발자로써 이게 얼마나 가혹한 테스트인지는 알고 있지만 ^*^ 그래도 혹시 놀라운 결과가 있을 지도 모르니... 소스는 동일한 소스에서 파일 경로만 변경하였다.<br>
<br>

~~~javascript
// Expected: "네이버 클라우드 플랫폼의 CSR 테스트를 위한 데모 문장입니다."
output: // Fast Vesion
{"text":"네이버 클라우드 플랫폼에 시작했으면 된 문장입니다"}
~~~
<br>
~~~javascript
// Expected: "네이버 클라우드 플랫폼의 CSR 테스트를 위한 데모 문장입니다."
output: // Tune Vesion
{"text":"네이버 클라우드 플랫폼 단어 문장입니다"}
~~~
<br>
&nbsp;음... 적어도 출신은 확실하다. 그리고 어느 정도의 Confidence를 갖는 게 아니라면 입력하지 않는 듯하다. 그 증거로 Tune Version에 <i>"CSR 테스트를 위한"</i> 부분이 생략되어 있다. 그리고 이를 확인할 수 있는 테스트가 또 있는데, 필자의 원래 목적을 상기해보자!<br>
<br>
&nbsp;바로 한본어 테스트인데, Tune Version을 입력으로 전달하되, <strong>Language 값을 <i>Kor</i> 에서 <i>Jpn</i> 으로 변경</strong>해보겠다!
<br>

~~~javascript
output: // Tune Vesion to JPN
{"text":"ニダー"}
~~~
<br>
&nbsp;입력에 비해 아주 짧은 결과가 나왔다. 이는 확실히 <strong>알아들을 수 없는 단어</strong>는 가장 유사한 단어로 치환하지 않고, <strong>생략해버림</strong>을 알 수 있는 대목이다. 그럼 마지막으로 Original Version에 대해 잘 동작하는지 확인해보고, 글을 마치도록 하겠다!
<br>

~~~javascript
output: // Original Vesion to JPN
{"text":"で僕がオタクだと飯です相手するビアンテも無駄に見た"}
~~~
<br>
&nbsp;뭔가 기대되는 긴 문장이 등장했고... 실제 발음은 어떤지 들어보도록 하자.
<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-13-stt-step-by-step/hanbon_2.mp3"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<i>음...</i>
</center>
<br>
&nbsp;마지막의 <i>"무단입니다"</i>를 제외하곤 무슨 말인지 전혀 모르겠다 ^*^ 손쉽게 한본어 서비스를 만들고자 했던 꿈은 여기까지인 걸로!
<br>
<br>
<br>

Reference
-------------
* <a href="https://apidocs.ncloud.com/ko/ai-naver/clova_speech_recognition/stt/">Naver Cloud Platform API 참조서 </a>
* <a href="https://docs.ncloud.com/ko/naveropenapi_v3/application.html">Naver Cloud Platform 설명서</a>
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
