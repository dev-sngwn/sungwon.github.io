---
layout: post
title: 얼굴 인식과 닮은 꼴 찾기를 한 번에! (With Naver Cloud Platform)
subtitle: ": 컴퓨터 비전 1도 몰라도 가능?"
tags: [Clova, FaceRecognition, Recognition, FaceSimilarity, Similarity, NCP, NaverCloud, NaverCloudPlatform]
---
<strong>이전 글</strong><br>

* [<i>한국어 OCR 해내기 1편: 가뿐하게 OCR API를 만들고 쓰기</i>](https://dev-sngwn.github.io/2019-12-17-korean-ocr-step-by-step-1/)<br>
* [<i>한국어 OCR 해내기 2편: 입맛대로 커스텀한 OCR 만들기</i>](https://dev-sngwn.github.io/2019-12-17-korean-ocr-step-by-step-2/)<br>
* [<i>음성인식 코드 짜는 최단 경로: 순식간에 STT 완성하기</i>](https://dev-sngwn.github.io/2020-01-13-stt-step-by-step/)<br>
* [<i>텍스트를 음성 mp3로 간단하게 변환하기: 딥러닝 몰라도 TTS 하는 법</i>](https://dev-sngwn.github.io/2020-02-16-tts-step-by-step/)
* [<i>성능 좋은 번역기, Papago 빌려쓰기 : 내 번역기에 네이버 버프 주는 법</i>](https://dev-sngwn.github.io/2020-03-15-papago-step-by-step/)

<br>
&nbsp;하루가 다르게 인공지능에 기반한 새로운 서비스들이 등장하고 있다. 사실 공부하는 입장에서는 <i>'내가 나중에 취업해서 할 일을 그렇게 쉽게 만들어 버리면 어떡합니까^^;;;;;'</i> 라는 생각이 들지만 동시에 더 앞서가야겠다는 각성의 계기가 되기도 한다!<br>
<br>
&nbsp;이번에 사용해 본 것은 Naver Cloud Platform (이하 NCP) 의 얼굴 인식 서비스인 <span style="background-color: #ddffdd"><strong><i>Clova Face Recognition (CFR) </i></strong></span>이다. 필자는 자연어 처리를 메인으로 공부해서 컴퓨터 비전에 대한 지식은 거의 없는 상태인데, 그럼에도 쉽게 사용할 수 있었는지! <span style="color: #aaaaaa">(물론 그랬으니 글을 적는 거긴 하다!)</span> 가장 빠른 경로로 살펴보도록 하겠다. 
<br>
<br>
<br>

#### 1. 애플리케이션 등록
--------------------------
&nbsp;이번 글부터는 반복되는 파트를 가급적 생략하려고 한다.<br>
<br>
&nbsp;필자가 [이전에 작성한 글](https://dev-sngwn.github.io/2020-02-16-tts-step-by-step/)에 동일한 과정이 나와 있으니 <strong>1. 애플리케이션 등록</strong> 까지 마친 후 <code>Client ID</code>와 <code>Client Secret</code>을 확보하길 바란다! <strong>이용할 서비스만 CFR로 변경해주면 된다.</strong>
<br>
<br>
<br>

#### 2. API 사용
--------------------------
&nbsp;CFR은 <strong>1) 유명인 얼굴 인식 API</strong>와 <strong>2) 얼굴 감지 API</strong>를 지원한다. 각각 살펴보도록 하자!<br>
<br>
<br>

#### ▶ 유명인 얼굴 인식 (Celebrity) API
<br>
&nbsp;우선 테스트엔 아래 이미지를 사용하였다.
<br>

<center>
<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-05-05-cfr-step-by-step/sample.jpg"/>
<br>
<i>▲ Photo by kevin laminto on Unsplash</i>
</center>

<br>
&nbsp;API가 요구하는 데이터는 <strong>Binary 타입의 2MB 이하의 이미지 데이터</strong>이다. 우린 이를 위해 <code>pillow</code>로 이미지를 불러와 <code>io</code>로 Binary 변환을 할 것이다. <br>
<br>

> <i>NCP 측에서 제공하는 가이드에서는 이미지를 애초에 Binary로 읽어오는데, </i><br>
> <i>그대로 사용했다가 <code>Timeout Error</code>를 마주했다... 2MB가 넘지 않는데 왜?ㅠ</i>

<br>

~~~ python
import io
from PIL import Image

def convert_to_byte_array(image):
    image_byte = io.BytesIO()
    image.save(image_byte, format='PNG')
    
    return image_byte.getvalue()

img = Image.open("./sample.jpg")
w, h = img.size
print("Original Shape:", img.size)

resize_const = 8
img = img.resize((w // resize_const, h // resize_const))
print("Resized Shape:", img.size)

image_byte = convert_to_byte_array(img)
~~~
<br>
~~~ markdown
out:

Original Shape: (5974, 3988)
Resized Shape: (746, 498)
~~~
<br>
&nbsp;사용한 이미지가 <code>5974 x 3988</code> 이라는 괴랄한 해상도를 가지고 있어 1/8 수준인 <code>746 x 498</code>로 낮춘 후, Binary로 변환하였다.<br>
<br>
&nbsp;테스트 과정에서 <code>cv2</code>를 사용하기도 했었는데, <i>OpenCV</i> 가 RGB를 BGR로 뒤집어 처리하는 특성 때문에 오류를 야기하는 듯해서 편한 <code>pillow</code> 길로 갔다. 혹시라도 <code>cv2</code>를 적용했는데 성능이 이상하다면 이미지 채널 상태를 살펴보시길!<br>
<br>
&nbsp;API가 요구하는 데이터가 준비되었으니, 이제 보낼 일만 남았다. 아까 준비해둔 <code>Client ID</code>와 <code>Client Secret</code>를 꺼내어 아래 소스에 적용시키도록 하자!<br>
<br>

~~~ python
import os
import sys
import json
import requests

client_id = "<your_client_id>"
client_secret = "<your_client_secret>"

url = "https://naveropenapi.apigw.ntruss.com/vision/v1/celebrity"

files = {'image': image_byte}
headers = {'X-NCP-APIGW-API-KEY-ID': client_id,
           'X-NCP-APIGW-API-KEY': client_secret}
response = requests.post(url,  files=files, headers=headers)
rescode = response.status_code

if (rescode==200):
    res = json.loads(response.text)
    print(res)
    
else:
    print("Error Code:" + response.text)
~~~
<br>
~~~ markdown
out:

{
   "info":{
      "size":{
         "width":746,
         "height":498
      },
      "faceCount":1
   },
   "faces":[
      {
         "celebrity":{
            "value":"김민석",
            "confidence":0.01
         }
      }
   ]
}

~~~
<br>
&nbsp;일단 너무 쉽게 결과에 도달한 것에 <strong>박수 짝짝</strong>이다. 그리고 뜬금없는 김민석 1프로ㅋㅋㅋㅋㅋㅋㅋ에 잠시 웃고... 원인을 파악해보자. 입력의 크기가 문제인 걸까? 크기를 이래저래 조절해보았다.<br>
<br>

| resize_const | celeb  | confidence |
|--------------|--------|------------|
| 4            | 이다희 | 0.18552    |
| 5            | 이다희 | 0.15012    |
| 6            | 이다희 | 0.310799   |
| 8            | 김민석 | 0.01       |
| 10           | 김희선 | 0.236089   |
| 12           | 김희선 | 0.118982   |
| 16           | 이다희 | 0.271809   |
| 20           | 이연희 | 0.0796587  |

<br>
&nbsp;참고로 <code>찾지 못함</code> 결과도 존재한다. 위 결과가 어떻게든 찾아진 결과임에 유의하자.<br>
<br>
&nbsp;실제 셀럽 사진을 그대로 전달했을 때에는 <code>confidence</code>가 1이 나오기도 한다. 모델이 문제가 있는 건 절대 아니고, 올바르게 동작하는 조건이 있는 듯 하다 <i>(실제 셀럽 사진도 크기 조절에 따라 결과가 상이하다)</i>. 원인은 클로바 팀에서 추후에 발견하여 보완할 것이라 생각하고, 최소한 <strong>해상도에 대한 가이드</strong>라도 주어졌다면 어땠을까 생각한다. 어쨌건, 찜찌름하지만 제법 동작한다! 
<br>
<br>

<br>

#### ▶ 얼굴 감지 (Face) API
<br>
&nbsp;얼굴 감지는 그야말로 얼굴을 <strong>감지하여 분석</strong>한다. 잘은 모르지만 분석된 정보를 바탕으로 이렇게 저렇게하면 Face ID가 완성되지 않을까????? 비전에 종사하는 여러분, 화이팅이다.<br>
<br>
&nbsp;코드는 앞서 사용한 소스에서 <code>url</code>만 <code>"https://naveropenapi.apigw.ntruss.com/vision/v1/face"</code>로 변경해주면 된다. 거듭 느끼지만 사용법이 정말 간단하다!<br>
<br>
&nbsp;결과는 아래와 같다.
<br>

~~~ markdown
out:

{
   "info":{
      "size":{
         "width":746,
         "height":498
      },
      "faceCount":1
   },
   "faces":[
      {
         "roi":{
            "x":294,
            "y":178,
            "width":96,
            "height":96
         },
         "landmark":"None",
         "gender":{
            "value":"female",
            "confidence":0.998822
         },
         "age":{
            "value":"19~23",
            "confidence":0.670971
         },
         "emotion":{
            "value":"neutral",
            "confidence":0.645446
         },
         "pose":{
            "value":"part_face",
            "confidence":0.832075
         }
      }
   ]
}
~~~

<br>
&nbsp;<code>roi</code>를 이미지 상에 그려보면...

> <i>ROI: Region Of Interest</i>

<br>

~~~ python
from PIL import ImageDraw

height = res["faces"][0]["roi"]["height"]
width  = res["faces"][0]["roi"]["width"]
x      = res["faces"][0]["roi"]["x"]
y      = res["faces"][0]["roi"]["y"]

draw = ImageDraw.Draw(img)
draw.rectangle([(x,y), (x + width, y + height)], outline="red")

img
~~~

<br>
<center>
<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-05-05-cfr-step-by-step/sample_roi.png"/>
<br>
<i>Output</i>
</center>

<br>
&nbsp;호오... 아주 정확하다. 그 외 다른 값들도 딱 봐도 설득력 있는 결과가 나타났다. <code>emotion</code>이나 <code>age</code>정보는 활용하기도 좋다! <code>landmark</code>가 <code>None</code>인 것에 대해 가이드에도 설명이 없는 것이 다소 아쉽지만, 예컨대  <code>pose</code>가 <code>part_face</code>여서 그런 것 같다 <i>(얼굴의 일부분임을 알고도 좌표를 찍으려면 에러가 날 수 있으니)</i>. 확인을 위해 정면을 보고 있는 다른 사진으로 테스트를 해보았다.
<br>

<br>
<center>
<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-05-05-cfr-step-by-step/sample2.jpg"/>
<br>
<i>▲ Photo by Mason Wilkes on Unsplash</i>
</center>
<br>

~~~ markdown
...
"landmark":{
   "leftEye":{
      "x":358,
      "y":204
   },
   "rightEye":{
      "x":399,
      "y":198
   },
   "nose":{
      "x":369,
      "y":231
   },
   "leftMouth":{
      "x":361,
      "y":251
   },
   "rightMouth":{
      "x":405,
      "y":246
   }
}
...
~~~
<br>
~~~ python
from PIL import ImageDraw

height = res["faces"][0]["roi"]["height"]
width  = res["faces"][0]["roi"]["width"]
x      = res["faces"][0]["roi"]["x"]
y      = res["faces"][0]["roi"]["y"]

draw = ImageDraw.Draw(img)
draw.rectangle([(x,y), (x + width, y + height)], outline="red")

if res["faces"][0]["landmark"] is not None:
    def parse_points(key):
        return res["faces"][0]["landmark"][key]["x"],\
                res["faces"][0]["landmark"][key]["y"]
    
    draw.point(xy=[parse_points("leftEye"),
                   parse_points("rightEye"),
                   parse_points("nose"),
                   parse_points("leftMouth"),
                   parse_points("rightMouth")], fill="red")

img
~~~
<br>

<center>
<img src="https://raw.githubusercontent.com/dev-sngwn/dev-sngwn.github.io/master/_posts/assets/2020-05-05-cfr-step-by-step/sample2_roi.png"/>
<br>
<i>(쥐똥만한 <code>landmark</code> 좌표가 찍혀있다)</i>
</center>

<br>
&nbsp;이번엔 제대로 감지된 것으로 보아 가설이 들어 맞는 것 같다! 하지만 앞서 언급한 Face ID에 대한 얘기는 취소해야겠는게, 이정도 정보로 본인 인증을 한다면 시스템이 너무 취약... 그래도 보안이 관련된 고오-급 기술에 적용은 어려울 지 몰라도 스티커를 씌우거나 얼굴형을 다듬는 등의 <strong>유틸리티적인 개발엔 쉽게 적용</strong>할 수 있을 것 같다!<br>

<br>
<br>
<br>

Reference
-------------
* <a href="https://apidocs.ncloud.com/ko/ai-naver/clova_face_recognition/">Naver Cloud Platform API 참조서 </a>
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