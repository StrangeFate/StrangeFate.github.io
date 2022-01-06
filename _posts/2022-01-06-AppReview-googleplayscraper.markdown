---
layout: post
title:  "구글 플레이스토어 앱 리뷰 크롤링: Google Play Scraper"
date:   2022-01-06 15:30:00 +0900
image:  Google_Play_Scraper.PNG
tags:   앱리뷰 구글 구글플레이스토어 플레이스토어 리뷰 크롤링 앱 스크레이퍼 google googleplaystore playstore review crawling appreview app scraper
---


### 기존 리뷰 크롤링 코드의 작동 중단

2022년 새해가 밝았다. 

신년 기념으로 기존 코드들의 작동을 확인했고 과거에 포스팅한 구글 플레이스토어 앱 리뷰 크롤링 코드가 정상 작동하지 않음을 알게 되었다.

기존 코드의 경우 과거 앱 리뷰를 불러오기 위해 사이트 최하단으로 이동하고 리뷰 더 보기 버튼을 자동으로 눌러 리뷰를 표시하는 방식으로 모든 리뷰를 구글 플레이스토어에서 불러왔다.
이 부분이 문제가 되었는데, 과거 리뷰를 불러오기 위해 스크롤을 하는 과정에서 일정 수준에 도달하면 리뷰가 표시되는 것이 아닌 사이트 최 상단으로 이동해 리뷰 페이지 접속시 처음 만나는 리뷰들만 표시되도록 구글 플레이스토어 웹페이지가 변경되어 더 이상 과거 리뷰를 불러올 수 없게 된 것이다.

이를 해결하고 싶었으나 최근, 한림대학교 자연어 처리 연구실에 석사과정으로 입학하게 되어 과거 코드에 시간을 할애하기 어렵다고 판단했다.
하지만 이 블로그의 주된 유입이 구글 플레이스토어 앱 리뷰 크롤링 코드이기에 관련 정보가 필요한 사람들이 많다는 것을 알고 있고 그렇기에 해당 코드를 대체할 수 있는 파이썬 라이브러리를 소개하고자 한다.


***

### Google Play Scraper

[Google Play Scraper](https://github.com/JoMingyu/google-play-scraper)

Google Play Scraper는 [facundoolano](https://github.com/facundoolano/google-play-scraper)가 과거 Node.js를 활용해 만든 구글 플레이스토어 앱 리뷰 크롤러를 기반으로 [PlanB](https://github.com/JoMingyu)가 파이썬을 활용해 재작성한 라이브러리이다. 

이 라이브러리는 작년 하반기까지 유지보수되었고 이 글이 작성된 2022-01-06 15시 기준 정상작동함을 확인했다. 라이브러리 설치는 pip install google-play-scraper 를 통해 가능하다.

Google Play Scraper는 4개의 기능을 제공하는데, 우선 앱 이름을 추출해야 사용할 수 있다.

앱 이름 = 패키지 명 이라고 생각하면 되는데, 구글 플레이스토어의 카카오톡 링크를 예로 들자면, "https://play.google.com/store/apps/details?id=com.kakao.talk&hl=ko&gl=US" 다음과 같은 링크가 있을때, details?id= 다음에 오는 "com.kakao.talk" 이 패키지 명이라고 생각하면 된다. 해당 부분을 복사해두고 아래 기능을 작동시킬때 사용하면 된다.

그럼 지금부터 기능을 설명하겠다.(실행 코드 예제는 깃허브의 readme 에서 복사해왔다.)

우선, 앱 설명(app)이다.

<pre>
<code>
{% raw %}

from google_play_scraper import app

result = app(
    'com.nianticlabs.pokemongo',
    lang='en', # defaults to 'en'
    country='us' # defaults to 'us'
)

{% endraw %}
</code>
</pre>

인자로는 앱 이름(앞에서 설명한 앱 링크에서 추출한 것), 언어, 국가를 사용하며 실행시 해당 앱과 관련된 거의 모든 정보를 추출한다.(결과 예제는 위 깃허브 링크에서 확인하기 바란다.)

다음은 앱 리뷰(reviews)이다.

<pre>
<code>
{% raw %}

from google_play_scraper import Sort, reviews

result, continuation_token = reviews(
    'com.fantome.penguinisle',
    lang='en', # defaults to 'en'
    country='us', # defaults to 'us'
    sort=Sort.MOST_RELEVANT, # defaults to Sort.MOST_RELEVANT
    count=3, # defaults to 100
    filter_score_with=5 # defaults to None(means all score)
)

# If you pass `continuation_token` as an argument to the reviews function at this point,
# it will crawl the items after 3 review items.

result, _ = reviews(
    'com.fantome.penguinisle',
    continuation_token=continuation_token # defaults to None(load from the beginning)
)

{% endraw %}
</code>
</pre>

인자로는 앱 이름, 언어, 국가, 정렬(관련성, 최신 등), 카운트(리뷰 개수), 별점 필터, continuation_token을 사용한다. 각 인자를 통해 원하는 만큼의 리뷰를 원하는 순서대로 정렬하여 별점을 선택해 리뷰를 불러올 수 있다. 이때 continuation_token은 앞서 3개의 리뷰를 크롤링 했다면 그 개수가 저장되어 다음 크롤링시에 3개 이후의 것(네 번째 리뷰) 부터 크롤링 한다.

결과로는 사용자 이름, 사진, 본문, 별점 등 리뷰와 관련한 거의 모든 정보를 불러온다.

위 앱 리뷰와 비슷한 인자를 가진 모든 앱 리뷰(reviews_all)은 몇개의 리뷰가 있던 모든 리뷰를 불러오는 기능이다.

<pre>
<code>
{% raw %}

from google_play_scraper import Sort, reviews_all

result = reviews_all(
    'com.fantome.penguinisle',
    sleep_milliseconds=0, # defaults to 0
    lang='en', # defaults to 'en'
    country='us', # defaults to 'us'
    sort=Sort.MOST_RELEVANT, # defaults to Sort.MOST_RELEVANT
    filter_score_with=5 # defaults to None(means all score)
)

{% endraw %}
</code>
</pre>

위 기능과 비슷한 인자들이 보이는 가운데 count 인자가 빠지고 sleep_miliseconds 인자가 들어간 것을 볼 수 있다. sleep_miliseconds 인자의 경우 대기 시간과 비슷하다 보면 되는데, 다량의 리뷰를 한번에 불러오다보면 대량의 요청을 웹사이트에 보내게 되고 이 부분에서 많은 트래픽이 발생하게 되어 일부 문제가 발생할 수 있고 이를 방지하기 위해 설정하는 것이다. 기본값은 0인데 본인이 카카오톡 리뷰를 크롤링 할 때는 sleep_miliseconds를 1000으로 설정해야 잘 작동했다. continuation_token은 모든 리뷰를 불러오는 이번 기능의 특성상 필요하지 않아 빠졌다.

마지막 기능인 앱 권한(permissions)이다.

<pre>
<code>
{% raw %}

from google_play_scraper import permissions

result = permissions(
    'com.spotify.music',
    lang='en', # defaults to 'en'
    country='us', # defaults to 'us'
)

{% endraw %}
</code>
</pre>

인자로는 앱 이름, 언어, 국가를 사용하고 말 그대로 해당 앱이 요구하는 권한들을 불러온다.

***

Google Play Scraper는 셀레니움을 이용한 코드와 다르게 웹사이트를 표시하지 않으며 빠른 속도를 보여준다. 특히 구글 콜랩에서 사용했을때 그 속도가 극대화 됨을 알 수 있었고 68만개 리뷰 데이터를 불러오는데 40분 정도가 걸림을 확인할 수 있었다. 

개인적으로 과거 앱 리뷰 크롤링을 위해 긴 시간을 들여 셀레니움 코드를 작성하고 또 그 코드가 작동하는 것을 확인하던 시간이 갑자기 아까워진 기분이지만, 그럼에도 불구하고 이런 라이브러리와 방법도 존재하는구나 하는 좋은 공부가 되었던 시간이기도 했다. 

이 라이브러리를 작성하고 배포한 [PlanB](https://github.com/JoMingyu)님께 감사를 표하며 앱 리뷰 크롤링 관련 정보가 필요했던 분들께 이 글이 도움이 되었길 바란다.

