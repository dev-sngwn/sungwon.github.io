---
title: "한국어 OCR 해내기 (With Naver Cloud Platform) 1편 (edit)"
date: 2019-12-17 13:50:30 -0400
categories: Guide
---

### 0. 들어가며
&nbsp;최근 한국어 OCR을 사용할 일이 생겼다. OCR 연구가 활발함은 어렴풋이 알고 있었고, 실생활에서도 몇몇 프로그램엔 이미 적용된 기술이니 수월할거라고 지레 짐작을 했는데...  
<br>
<center>
<img src="https://github.com/mmsw0324/mmsw0324.github.io/blob/master/_posts/assets/tesseract.JPG?raw=true" width="100%"/><br>
<i>네 잘 지나섰니댜7 ^^</i><br>
</center>
<br>
&nbsp;쉽지 않았기 때문에 꽤나 삽질을 하게 되었고, 결과적으론 <strong>NAVER CLOUD PLATFORM(이하 NCP)</strong> 을 활용하는 게 정답이었다. 하지만 네이버 측에서 제공하는 가이드가 <del>(나같은)</del> 클라우드 입문자에겐 다소 어려웠기에 보충 자료로써 본 글을 작성하고자 한다.
<br>
<br>
&nbsp;<strong>1편</strong>에서는 쉽고 빠르게 사용할 수 있는 <strong>General OCR</strong>을 다루고, <strong>2편</strong>에서는 OCR 영역을 지정해 템플릿화해서 사용할 수 있는 <strong>Template OCR</strong>을 다룰 것이다.

<br>
<br>

### 1. 우선 NCP의 회원이 되십시오!<br>

<center>
  
<img src="https://github.com/mmsw0324/mmsw0324.github.io/blob/master/_posts/assets/NCP.png?raw=true" width="30%"/>
  
[<i>NAVER CLOUD PLATFORM</i>](https://www.ncloud.com/)

</center>
<br>
위 링크에서 <strong>회원가입</strong>을 진행한다. 회원가입을 마친 후, 상단의 메뉴에서 <strong>[서비스] → [AI Service] → [OCR]</strong>까지 이동한 후 본격적으로 시작!
