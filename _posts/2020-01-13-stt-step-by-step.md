---
layout: post
title: 음성인식 코드 짜는 최단 경로 (With Naver Cloud Platform) (edit)
subtitle: ": 순식간에 STT 완성하기"
tags: [STT, SpeechToText, CSR, Clova, NCP, NaverCloud, NaverCloudPlatform]
---
<br>

&nbsp;불현듯 <strong>한본어 서비스</strong>를 만들어 보고 싶다는 생각이 들었다. 인터넷 방송이나 유튜브를 즐겨보시는 분은 익숙하실 텐데 한본어가 뭐냐면... 
<br>

<center>
<audio controls="controls">
  <source type="audio/mp3" src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-13-stt-step-by-step/hanbon.mp3"></source>
  <source type="audio/ogg" src="filename.ogg"></source>
  <p>Your browser does not support the audio element.</p>
</audio>
<br>
<i><del>(한봉오가 몬지 모르씬다면 써르명해드리능게 인지샨종)</del></i>
</center>
<br>
&nbsp;이렇게 한국어 발음을 일본어로 적고 Text-to-Speech(TTS)를 활용해 일본어스러운(?) 한국어를 만드는 것이다. 

><strong>내 목소리 → 일본어 STT → 문장 → 일본어 TTS → 쉽고 빠르게 한본어?!</strong> 

가 기본 아이디어였다.<br>
<br>
&nbsp;내가 아는 한 가장 쉽고 정확하게 STT와 TTS를 제공하는 곳은 <strong>네이버 클라우드 플랫폼(이하 NCP)</strong>이 분명하므로 이번에도 NCP를 활용해보았고, 그 과정을 정리해보고자 한다.<br>
<br>
<br>
### 1. 어플리케이션 등록
-------------------
&nbsp;우선 <i><a href="https://www.ncloud.com/">NCP</a></i>로 이동하여 서비스를 신청하도록 하자. <strong>[서비스] → [AI Service] → [Clova Speech Recognition(CSR)]</strong>로 이동한 후 <strong>[이용 신청하기]</strong>를 눌러주길 바란다.
<br>

<center>
<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-01-13-stt-step-by-step/01_start.jpg"/>
</center>
<br>
&nbsp;이동하고 나면 위와 같은 화면을 만나게 될텐데, <strong>[Application 등록]</strong>을 눌러 진행한다.

