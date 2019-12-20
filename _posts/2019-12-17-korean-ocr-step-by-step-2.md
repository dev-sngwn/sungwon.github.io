---
layout: post
title: 한국어 OCR 해내기 (With Naver Cloud Platform) 2편
subtitle: ": 입맛대로 커스텀한 OCR 만들기"
tags: [Korean, KoreanOCR, OCR, NCP, NaverCloud, NaverCloudPlatform]
---

이전 글:
<a href="https://dev-sngwn.github.io/2019-12-17-korean-ocr-step-by-step-1/">한국어 OCR 해내기 (With Naver Cloud Platform) 1편: 가뿐하게 OCR API 만들기</a>

&nbsp;1편에서 쉽고 빠르게 OCR API를 만들어서 사용했지만 원치 않는 부분에 대해서도 OCR이 진행되어 결과에 노이즈가 발생하는 문제가 있었다. 2편에서는 이 문제를 해결할 수 있는 <strong>Template OCR</strong>에 대해 다룰 것이다.<br>

> 모든 설명은 1편을 선행했다고 가정하니, 혹시라도 건너뛰었다면 위 링크로 이동해 <span style="background-color: #ffeeee"><strong><i>&nbsp;0. 우선 NCP의 회원이 되십시오! </i></strong></span>&nbsp;부분까지는 따라 하고 오길 권장한다.

<br style="line-height:10px">
<br style="line-height:10px">

### 1. 도메인 생성
-------------

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/01_domain.png"/>
<i>Template 도메인 생성</i>

</center>

&nbsp;General OCR을 만들 때와 동일하게 도메인을 먼저 생성한다. 서비스 타입은 <strong>Template</strong>로 선택!


<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/02_list.png"/>

</center>

&nbsp;그러면 1편에서 생성한 General 도메인과 새로 만든 Template 도메인을 확인할 수 있다. <strong>[템플릿 빌더]</strong>를 눌러 템플릿 작업을 시작하도록 하자!

<br style="line-height:10px">
<br style="line-height:10px">

### 2. 템플릿 생성
-------------

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/03_template_main.png"/>
<i>생성한 도메인을 관리할 수 있다</i>

</center>

&nbsp;도메인을 생성했으니 커스텀을 해 줄 차례다. 좌측에 <strong>[템플릿 목록]</strong>을 눌러 템플릿을 생성하러 가보자!
<br>

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/04_template_list.png"/>
<i>텅 빈 템플릿 리스트</i>

</center>

&nbsp;<strong>[템플릿 생성]</strong>을 눌러 이동하면 아래와 같은 화면을 볼 수 있다.
<br>

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/05_template_base.png"/>

</center>
<br>
&nbsp;이동된 화면에서 <strong>(1)템플릿 명을 입력</strong>하고 <strong>(2)샘플 이미지를 업로드</strong>하면 위와 같은 화면을 볼 수 있다. 가장 먼저 해야 하는 것은 <span style="background-color: #ddddff"><strong>대표 샘플을 지정</strong></span>하는 것이다.

> <strong>대표 샘플</strong>: API가 완성된 후, OCR 결과는 <code>{"title": {"title name": "title result"}, "field": ["field name": "field result", ...]}</code> 와 같이 응답된다. 이 때 <code>title result</code>를 지정해주는 과정으로 <strong>결과 분류</strong>에 사용될 수 있다.


&nbsp;이를테면 위의 이미지에서 좌측 상단의 <i>비포 선라이즈</i> 영역을 대표 샘플 영역으로 지정하면 <code>title result</code>가 <i>"비포 선라이즈"</i>가 아닐 경우 <span style="background-color: #ffdddd"><strong>템플릿에 어긋나는 형태</strong></span>라고 판단하여 제외할 수 있다는 것이다. 설명한 영역을 대표 샘플 영역으로 지정하고, 영화 자막 영역을 <i>subtitle</i> 영역으로 지정하도록 하겠다.
<br>

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/06_template_done.png"/>
<i>완성된 템플릿</i>

</center>
<br>
&nbsp;원하는 영역을 모두 지정한 후, <strong>[저장]</strong>을 눌러 템플릿을 저장하도록 하겠다. 여기서 지정해 준 필드 이름은 후에 결과의 각 필드별 key값으로 전달되니 헷갈리지 않도록 예쁘게 명명해주도록 하자.
<br>
<br>

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/07_template_list.png"/>

</center>
&nbsp;저장 후, 좌측 메뉴의 <strong>[템플릿 목록]</strong>을 확인해보면 방금 만든 템플릿이 저장되어 있는 것을 볼 수 있다. 이때 <strong>템플릿 ID</strong>은 후에 사용될 수 있으므로 기억해 두도록 하자(여러 개의 템플릿을 사용하는 경우).

<br style="line-height:10px">
<br style="line-height:10px">


### 3. API 생성
-------------

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/08_template_main.png"/>

</center>

&nbsp;좌측 메뉴 상단부의 <strong>[설정]</strong>을 눌러 도메인 정보 페이지로 돌아온다. 그리고 <strong>[외부 연동]</strong> 탭을 클릭해 API 설정 페이지로 넘어가자!
<br>

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/09_api.png"/>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/10_api_config.png"/>

</center>

&nbsp;General OCR을 진행했을 때와 동일하게, <strong>Secret Key를 생성</strong>한다. <strong>APIGW Invoke URL</strong>은 1편을 진행했다면 이미 생성되어 있을 것이고, 그렇지 않았다면 <strong>[자동 연동]</strong>을 눌러 URL을 생성해준다.

<br>
<br>

### 4. API 배포
-------------

&nbsp;General OCR은 <strong>Secret Key</strong>와 <strong>APIGW Invoke URL</strong>만으로도 사용할 수 있었기에 저 페이지에서 섣불리 <i>"다했다~"</i> 를 해버릴지도 모른다. 하지만 생성된 URL로 보내보면 아래와 같은 에러를 마주할 것이다.

~~~javascript
{
  'code': '1021',
  'message': 'Not Found Deploy Info',
  'path': '/external/v1/488/abc123abc123abc123/infer', /*private uid obfuscated*/
  'timestamp': 1576746215096
}
~~~

&nbsp;이는 API가 배포되지 않아 발생하는 에러이다. 어서 마무리 작업을 확실히 하고 마침표를 찍도록 하자.
<br>

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/11_to_release.png"/>

</center>

&nbsp;연동 설정을 마무리하고 좌측 메뉴 상단부의 <strong>[배포 관리]</strong>로 이동한다.

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/12_beta.png"/>

</center>

&nbsp;생성한 템플릿을 체크하고, <strong>[베타 배포]</strong>를 눌러 템플릿을 배포한다.

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/13_beta.png"/>

</center>

&nbsp;<i>현재 배포 상태</i> 가 <span style="background-color: #ddffdd"><strong>Beta</strong></span> 상태로 변경된 것을 확인할 수 있다. 이는 테스트 단계이므로 아직 배포가 완료된 것이 아니다. 확실한 서비스를 위해 <span style="background-color: #ddffdd"><strong>베타 테스트</strong></span>를 진행한 후, 배포를 완료하도록 하자.

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/14_test.png"/>

</center>

&nbsp;좌측 메뉴의 <strong>[테스트]</strong>을 눌러 테스트 페이지로 이동한다.

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/15_test.png"/>

</center>

&nbsp;<span style="background-color: #ddffdd"><strong>베타 테스트</strong></span>를 진행할 예정이므로 테스트 조건은 <span style="background-color: #ddffdd"><strong>베타</strong></span>로 선택. 이미지를 업로드한 후, 테스트!
<br>

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/16_result.png"/>
<i>테스트 결과</i>

</center>
<br>

&nbsp;우리가 의도한 대로 <strong>지정한 영역에 대해서만 OCR</strong>을 진행함을 알 수 있다, 성공적이다! 어서 배포를 마치고 코드에서도 정상적으로 동작하는지 확인해보자. 페이지 최상단의 메뉴 바에서 <strong>[서비스 배포]</strong>를 눌러 최종 배포를 마무리하길 바란다. 
<br>

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/17_release.png"/>
<i>얼른 배포하고 코딩하러 가자..!</i>

</center>

<br>
<br>
<br>


### 5. 테스트
-------------

<center>

<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2019-12-17-korean-ocr-step-by-step-2/movie_sample.png"/>
<i>예시 이미지</i>

</center>
<br>

&nbsp;테스트 환경은 General OCR 때와 동일하다, 다만 유의점이라면 여러 개의 템플릿을 사용할 경우, <code>"templateIds"</code> 컬럼을 포함해 템플릿을 지정해줘야 한다는 것. 물론 컬럼을 포함하지 않을 경우 자동으로 맞는 템플릿을 찾지만 명확하게 지정하는 것을 권장한다. 1편에서 사용한 소스를 그대로 첨부하겠다.

~~~python
import json
import base64
import requests

with open("./movie_sample.png", "rb") as f:
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
          # "templateIds": [400]  # 설정하지 않을 경우, 자동으로 템플릿을 찾음 
        }
    ]
}
data = json.dumps(data)
response = requests.post(URL, data=data, headers=headers)
res = json.loads(response.text)
~~~

<br>

~~~javascript
{
  'version': 'V1',
  'requestId': 'sample_id',
  'timestamp': 1576748247494,
  'images': [{
    'uid': '123456a123456b1234567', /*private uid obfuscated*/
    'name': 'sample',
    'inferResult': 'SUCCESS',
    'message': 'SUCCESS',
    'matchedTemplate': {'id': 444, 'name': 'Movie_Subtitle'},
    'title': {
      'name': 'movie_title',
      'bounding': {'top': 45.31195, 'left': 144.9561, 'width': 189.04782, 'height': 109.76625},
      'inferText': '비포\n선라이즈', 'inferConfidence': 0.9905509},
   'fields': [{
     'name': 'Movie_subtitle',
     'bounding': {'top': 607.4006, 'left': 448.08237, 'width': 501.85226, 'height': 99.57385},
     'valueType': 'ALL',
     'inferText': '오늘 비엔나에 왔는데\n재밌게 놀 곳을 찾고 있어요',
     'inferConfidence': 0.9994903
   }],
   'validationResult': {'result': 'NO_REQUESTED'}
  }]
}
~~~

&nbsp;General OCR보다 훨씬 깔끔하게 결과가 나오는 것을 알 수 있다. 정형화된 데이터를 다루는 게 목적이라면 이보다 적합한 것은 없을 정도로... 현재 Naver Cloud Platform의 OCR Service는 무료로 제공되고 있으므로, OCR이 필요하다면 활용하는 것을 <strong>적극적으로 권장</strong>한다.
<br>
<br>
<br>

Reference
-------------
* <a href="https://docs.ncloud.com/ko/ocr/ocr-1-1.html">Naver Cloud Platform OCR 사용 가이드</a>
* <a href="https://www.youtube.com/channel/UCy460AveLol6g_s3GZIKRkw">유튜브 경기농아방송</a>
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


