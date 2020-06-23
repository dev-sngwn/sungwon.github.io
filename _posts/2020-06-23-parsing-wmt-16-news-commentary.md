---
layout: post
title: WMT News Commentary 데이터의 병렬 쌍이 안 맞는 문제 (edit)
subtitle: ": 멘붕과 삽질의 과정"
tags: [WMT, MachineTraslation, Translation, NMT, XLIFF]
---

&nbsp;<strong>WMT</strong>는 <i>Workshop on Statistical Machine Translation</i> 의 약어로 2006년부터 개최되어 올해로 15회차 <i>(2020.11.19-20)</i> 를 맞이한다. 전통이 깊은 만큼 매년 데이터셋을 업데이트하며 <strong>WMT 시리즈를 공개</strong>하는데, 이 데이터셋들은 현재 기계 번역의 주축이 되어주고 있다!<br>
<br>
&nbsp;다만 논문을 쓰는 입장에서 너무 최신 데이터는 과거 모델의 Baseline 성능 측정을 직접 해야한다는 부담이 있기 때문에 <strong>대체로 WMT2014 - 2016 부근의 데이터를 사용</strong>하는 듯하다. 필자도 같은 맥락으로 WMT2016 데이터셋을 활용하고자 했는데... 문제가 발생하여 이를 해결한 과정을 적어보고자한다.<br>
<br>
&nbsp;우선 WMT 2016 공식 홈페이지는 [여기](https://www.statmt.org/wmt16/translation-task.html)를 눌러 방문할 수 있다. 우리는 데이터를 다운로드해 활용하길 원하므로 스크롤을 쭉쭉 내리다보면 아래와 같은 부분을 만날 수 있다.<br>
<br>
[Image]
<br>
<br>
&nbsp;문제가 발생한 것은 바로 <u>News Commentary v11</u>이다. 보기엔 전혀 문제가 없지만 압축을 해제하고 데이터를 직접 살펴보면...<br>
<br>

> 혹시 Windows 환경에서 tgz 파일을 다루지 못해 강제 Ubuntu행을 당했다면 7-Zip으로 구원받도록 하자.

<br>
~~~python
def data_len(path, encoding="utf-8"):
    
    with open(path, "r", encoding=encoding) as f:
        dataset = f.read().splitlines()
        
    return len(dataset)

import os
import glob

files = glob.glob("./training-parallel-nc-v11/*")

for file in files:
    print(os.path.basename(file), "Length:", data_len(file))
~~~
~~~Markdown
news-commentary-v11.cs-en.cs Length: 191865
news-commentary-v11.cs-en.en Length: 192052
news-commentary-v11.de-en.de Length: 243330
news-commentary-v11.de-en.en Length: 243480
news-commentary-v11.ru-en.en Length: 196844
news-commentary-v11.ru-en.ru Length: 196587
~~~
<br>
&nbsp;분명 병렬 데이터인데 <strong>단 한 세트도 병렬이 일치하지 않는다!</strong> 어디서부터 병렬 쌍이 틀어진 것인지 알 수 없기 때문에 함부로 데이터를 선별할 수도 없고... <br>
<br>
&nbsp;필자는 <i>de-en</i> 데이터가 필요했기에 혹시 <strong>길이가 0인 데이터가 포함되어 있나</strong> 싶어서 이를 제거해봤는데 제거하고 난 개수는 각각 <code>243068</code>, <code>241791</code>로 여전히 일치하지 않았다. 사실 길이가 0인 데이터는 <strong>완전한 노이즈</strong>인데 그것도 정제가 되지 않았다는 것에서 <strong>다운로드한 파일이 신뢰할만한가</strong>에 대해 생각하기 시작했다.<br>
<br>

> 추가로 구분자가 "\n"이 아니라던가... 도 생각해봤지만 이 또한 답이 없는 접근이었다.

<br>
&nbsp;



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