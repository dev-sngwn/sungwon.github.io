---
layout: post
title: 한국어 OCR 해내기 (With Naver Cloud Platform) 1편
subtitle: ": 가뿐하게 OCR API를 만들고 쓰는 법"
tags: [Korean, KoreanOCR, OCR, NCP, NaverCloud, NaverCloudPlatform]
---

&nbsp;최근 한국어 OCR을 사용할 일이 생겼다. OCR 연구가 활발함은 어렴풋이 알고 있었고, 실생활에서도 몇몇 프로그램엔 이미 적용된 기술이니 수월할 거라고 지레 짐작을 했는데...  
<br>
<center>
<img src="https://github.com/mmsw0324/mmsw0324.github.io/blob/master/_posts/assets/tesseract.JPG?raw=true" width="100%"/><br>
<i>네 잘 지나섰니댜7 ^^</i><br>
</center>
<br>
&nbsp;쉽지 않았기 때문에 꽤나 삽질을 하게 되었고, 결과적으론 <strong>NAVER CLOUD PLATFORM(이하 NCP)</strong> 을 활용하는 게 정답이었다. 하지만 네이버 측에서 제공하는 가이드가 <del>(나 같은)</del> 클라우드 입문자에겐 다소 어려웠기에 보충 자료로써 본 글을 작성하고자 한다.
<br> 
<br>
&nbsp;<strong>1편</strong>에서는 쉽고 빠르게 사용할 수 있는 <strong>General OCR</strong>을 다루고, <strong>2편</strong>에서는 OCR 영역을 지정해 템플릿화해서 사용할 수 있는 <strong>Template OCR</strong>을 다룰 것이다.

<br>
<br>
<br>

### 0. 우선 NCP의 회원이 되십시오!<br>
<br>

<center>
  
<img src="https://github.com/mmsw0324/mmsw0324.github.io/blob/master/_posts/assets/NCP.png?raw=true" width="40%"/><br>
  
<i><a href="https://www.ncloud.com/">NAVER CLOUD PLATFORM</a></i>

</center>
<br>
&nbsp;위 링크에서 해야 할 일은 다음과 같다. <br>

>1\) <strong>회원가입</strong><br>
>2\) 상단의 메뉴에서 <strong>[서비스] → [Application Service] → [API Gateway]</strong>로 이동<br>
>3\) <strong>API Gateway 신청</strong>

까지 마치고 나면 아마 좌측에 메뉴가 있는 콘솔 페이지로 이동이 되어 있을 것이다.
좌측의 메뉴에서 <strong>[Products & Services] → [AI-Application Service] → [OCR]</strong>까지 이동한 후 본격적으로 시작!
<br>

<center>

<img src="https://github.com/mmsw0324/mmsw0324.github.io/blob/master/_posts/assets/01_start.jpg?raw=true"/><br>
<strong>[이용 신청]</strong> 클릭!<br>

</center>
<br>
<br>

### 1. 도메인 생성<br>

&nbsp;위 단계를 성공적으로 마치고 나면 <del>(실패하기엔 너무 간단하지만)</del> 아래와 같이 <strong>도메인 생성 창</strong>이 나타날 것이다.

<br>

<center>

<img src="https://github.com/mmsw0324/mmsw0324.github.io/blob/master/_posts/assets/02_domain.jpg?raw=true"/><br>
<i>적당한 도메인 명을 입력!</i>

</center>
<br>

&nbsp;필자는 뉴스 캡처에서 자막을 긁어오는 OCR API를 만들고자 하므로 도메인 이름을 <i>News_OCR</i>이라고 명명했다. 각자 적당한 도메인 이름을 지어 준 후, 1편에서는 General OCR을 다루기로 했으므로 서비스 타입은 General로 체크해준다.<br>
&nbsp;간단히 General과 Template의 차이를 언급하면 다음과 같다.

> <strong>General</strong>: Input 이미지에 포함된 <strong>모든 문자를 반환</strong><br>
> <strong>Template</strong>: Input 이미지에서 <strong>사용자가 지정한 영역이 포함하는 문자를 반환</strong>

&nbsp;혹시 와닿지 않더라도 문제는 없다, 두 타입을 모두 다룰 것이고 실제 데이터를 보면 감이 확 올테니 걱정 말고 다음 스텝으로 진행하자.
<br>
<br>
<br>

### 2. API 생성<br>

<center>

<img src="https://github.com/mmsw0324/mmsw0324.github.io/blob/master/_posts/assets/03_list.jpg?raw=true"/>
<strong>[Text OCR]</strong>을 클릭!

</center>
<br>

&nbsp;생성된 도메인의 동작 컬럼을 보면 <strong>Text OCR 버튼</strong>이 있다. 이를 누르면 다음과 같은 설정 창을 보게 된다.
<br>

<center>

<img src="https://github.com/mmsw0324/mmsw0324.github.io/blob/master/_posts/assets/04_config.jpg?raw=true"/>
<strong>[Text OCR]</strong>을 클릭!

</center>
<br>
&nbsp;<strong>Secret Key</strong>는 보안을 위해 필수로 사용되니 <strong>생성</strong>. 그리고 NCP는 너무도 편리하게 <strong>APIGW(API Gateway) 자동 연동</strong>을 지원하니 당장 사용하도록 하자. 클릭!
<br>

<center>

<img src="https://github.com/mmsw0324/mmsw0324.github.io/blob/master/_posts/assets/5_api.jpg?raw=true"/>
<i>생성된 Secret Key와 APIGW Invoke URL</i>

</center>
<br>
&nbsp;<strong>Secret Key</strong>와 <strong>APIGW Invoke URL(OCR Invoke URL이 아님)</strong>이 준비되었다면 General OCR을 사용할 준비는 끝이 났다. 네이버에서 제공하는 메인 가이드는 여기까지 적혀있어 <i>"그래서 usage는 어디에..."</i> 하는 마음이 든다. Jupyter Notebook이든 PyCharm이든 본인이 사용하는 파이썬 환경으로 이동하자.
<br>
<br>
<br>

### 3. API 사용<br>

&nbsp;설명서의 <i><a href="https://docs.ncloud.com/ko/ocr/ocr-1-3.html">OCR Custom API Spec.</a></i>을 잘 읽어보면 API가 요구하는 Request의 형태가 적혀 있다.
<br>

<center>

<img src="https://github.com/mmsw0324/mmsw0324.github.io/blob/master/_posts/assets/06_form.jpg?raw=true"/>
<i><del>필자는 이걸 못 찾아서 헤멨다...</del></i>

</center>
<br>

&nbsp;우리에게 중요한 것은 <i>image</i> 필드, 그중에서도 <i>image.data</i> 이다. <del>(나머지는 이름만 봐도 대충 느낌이 온다)</del> Description을 읽어보면 <strong>base64로 인코딩된 이미지 바이트</strong>를 요구하고 있다.
<br>

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/07_sample.jpg"/>
<i>예시 이미지 1</i>

</center>
<br>

&nbsp;뉴스 캡쳐에서 자막을 긁어오는 것이 목표이므로, 샘플 뉴스 이미지를 준비했다. 본 이미지를 우리가 만든 API에게 보내 OCR을 잘 해내는지 확인해보겠다. 기대되는 결과는 <strong>"올해의 액운을 막고 건겅과 풍년을 기원하였습니다."</strong>이다.
<br>

~~~python
import json
import base64
import requests

with open("./news_sample.png", "rb") as f:
    img = base64.b64encode(f.read())

 # 본인의 APIGW Invoke URL로 치환
URL = "https://<your url>.apigw.ntruss.com/custom/v1/000/<your url>/general"
    
 # 본인의 Secret Key로 치환
KEY = "<your secret key>"
    
headers = {
    "Content-Type": "application/json",
    "X-OCR-SECRET": KEY
}
    
data = {
    "version": "V1",
    "requestId": "sample_id", # 요청을 구분하기 위한 ID, 사용자가 정의
    "timestamp": 0, # 현재 시간값
    "images": [
        {
            "name": "sample_image",
            "format": "png",
            "data": img.decode('utf-8')
        }
    ]
}
data = json.dumps(data)
response = requests.post(URL, data=data, headers=headers)
res = json.loads(response.text)
~~~
<br>

~~~javascript
output:
{
  'version': 'V1',
  'requestId': 'sample_id',
  'timestamp': 1576660655544,
  'images': [{
    'uid': '123456a123456b1234567', /*private uid obfuscated*/
    'name': 'sample_image',
    'inferResult': 'SUCCESS',
    'message': 'SUCCESS',
    'fields': [
      {'inferText': 'GGDEAF', 'inferConfidence': 0.99862313},
      {'inferText': '올해의', 'inferConfidence': 0.99987835},
      {'inferText': '액운을', 'inferConfidence': 0.99987006},
      {'inferText': '막고', 'inferConfidence': 0.9997827},
      {'inferText': '건겅과', 'inferConfidence': 0.9919012},
      {'inferText': '풍년을', 'inferConfidence': 0.99932873},
      {'inferText': '기원하였습니다.', 'inferConfidence': 0.9997941},
      {'inferText': 'NEWS', 'inferConfidence': 0.99979496}
    ],
    'validationResult': {'result': 'NO_REQUESTED'}
  }]
}
~~~

&nbsp;서론의 망한 케이스를 기억한다면 이는 실로 놀라운 정확도다(심지어 오타까지 정확하게)! 다만 문제 될 부분이 보인다면 <i>GGDEAF NEWS</i>를 둘로 갈라놓은 데다가 심지어 붙어있지도 않다는 것이다<del>(예컨대 위에서부터 차례차례 인식해 Append 하는 방식일 것)</del>. 이는 꽤 큰 문제를 야기할 수 있는데, 예를 들어 아래와 같은 이미지를 OCR 한다고 생각해보자.

<br>

<center>

<img src="https://github.com/mmsw0324/mmsw0324.github.io/blob/master/_posts/assets/07_sample_bad.jpg?raw=true"/>
<i>예시 이미지 2</i>

</center>
<br>

&nbsp;직관적으로도 자막이 아닌 화면 속 글씨를 읽어올 것 같은 느낌이 오는데,
~~~javascript
output:
{
  'version': 'V1',
 'requestId': 'sample_id',
 'timestamp': 1576661468349,
 'images': [{'uid': '1215abe59a4d454aa9fa1311e749e1c5',
   'name': 'Sample',
   'inferResult': 'SUCCESS',
   'message': 'SUCCESS',
   'fields': [{'inferText': 'SRT', 'inferConfidence': 0.99957913},
    {'inferText': '알림서비스', 'inferConfidence': 0.999852},
    {'inferText': '12:47', 'inferConfidence': 0.9997815},
    {'inferText': '5월 20일', 'inferConfidence': 0.947222},
    {'inferText': '分', 'inferConfidence': 0.28402498},
    {'inferText': '현재', 'inferConfidence': 0.9984431},
    {'inferText': '아름씨가', 'inferConfidence': 0.99971795},
    {'inferText': '타고', 'inferConfidence': 0.99911255},
    {'inferText': '있는', 'inferConfidence': 0.9997971},
    {'inferText': '열차가', 'inferConfidence': 0.99986386},
    {'inferText': 'SIT', 'inferConfidence': 0.5262649},
    {'inferText': '달릴', 'inferConfidence': 0.56686825},
    {'inferText': '서북', 'inferConfidence': 0.8479211},
    {'inferText': '수서역', 'inferConfidence': 0.99913806},
    {'inferText': '근처에서', 'inferConfidence': 0.9999334},
    {'inferText': '화재가', 'inferConfidence': 0.9995774},
    {'inferText': '발생했습니다', 'inferConfidence': 0.99740624},
    ...하략
~~~

&nbsp;역시나 적중이다. 이 현상을 방지하기 위해선 <strong>OCR을 진행할 영역을 지정</strong>해주는 것이 필요하다. 그를 위해 <strong>Template OCR</strong>이 존재한다. 다음 편에서는 Template OCR 사용법에 대해 가이드하도록 하겠다.
<br>
<br>
<br>
<br>

### Reference
=============
<a href="https://docs.ncloud.com/ko/ocr/ocr-1-1.html">Naver Cloud Platform OCR 사용 가이드</a>
<a href="https://www.youtube.com/channel/UCy460AveLol6g_s3GZIKRkw">유튜브 경기농아방송</a>
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
                            
