---
layout: page
title: 웹에 있는 데이터 작업
subtitle: 데이터 입수
minutes: 15
---
> ## 학습목표 {.objectives}
>
> *   간단한 REST API를 활용해서 데이터셋을 다운로드하는 파이썬 프로그램을 작성한다.

점점 더 많은 조직에서 [REST](reference.html#rest)로 불리는 형식으로 웹상에 데이터셋을 공개한다. REST의 상세한 사항(그리고 사상)이 중요한 것은 아니다; 중요하는 것은 REST가 언제 사용되고, 모든 데이터셋이 URL로 식별되고, [응용 프로그래밍 인터페이스(application programming interface, API)](reference.html#api)라고 불리는 함수집합으로 접근할 수 있다는 점이다.

예제로 세계은행 [기후 데이터 API(Climate Data API)]((http://data.worldbank.org/developers/climate-data-api))로 제공되는 15개 지구순환모델로 생성되는 데이터를 사용한다. [API 홈페이지](http://data.worldbank.org/developers/climate-data-api)에 따르면, 다양한 값에 대한 연평균치를 담고 있는 데이터셋은 URL 형태로 식별된다:


<code>http://climatedataapi.worldbank.org/climateweb/rest/v1/country/cru/<u><em>var</em></u>/year/<u><em>iso3</em></u>.<u><em>ext</em></u></code>

각 요소는 다음과 같다:

*   *var*는 `pr` (강수량) 혹은 `tas` ("지표면 온도")다;
*   *iso3* 국제표준기구(International Standards Organization, ISO)가 정한 [국가별 3-문자 코드]((http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3))다. 예를 들어 "CAN"은 캐나다, "BRA"는 브라질이다.
*   *ext* (확장 "extension" 줄임말)은 다운로드받을 데이터 형식을 지정한다. 형식에 대한 몇가지 선택지가 있지만, 가장 간단한 것이 [CSV(comma-separated values,콤마구분값)]((reference.html#csv))로, 각 레코드는 행이고, 각 행 값은 콤마로 구분된다. (CSV는 스프레드쉬트 데이터에 자주 사용된다.)

예를 들어, 캐나다 연평균기온을 CSV 파일형식으로 원한다면, URL은 다음과 같다:

~~~
http://climatedataapi.worldbank.org/climateweb/rest/v1/country/cru/tas/year/CAN.csv
~~~

상기 URL을 웹브라우져에 붙여넣고 엔터를 치면, 다음이 화면에 출력된다.

~~~
year,data
1901,-7.67241907119751
1902,-7.862711429595947
1903,-7.910782814025879
...
2007,-6.819293975830078
2008,-7.2008957862854
2009,-6.997011661529541
~~~

> ## 무대 뒤에서(Behind the Scenes) {.callout}
>
> 상기 특정 데이터셋은 세계은행 서버에 파일형태로 저장될 수도 있고, 서버가 다음 작업을 수행한다:
>
>  1. 사용자가 입력한 URL을 받는다.
>  2. URL을 조각으로 나눈다.
>  3. 3개 핵심 필드를 추출한다.(변수(var), iso3(국가코드), ext(파일형식)).
>  4. 데이터베이스에서 요청 데이터를 가져온다.
>  5. 데이터를 CSV 형식으로 저장한다.
>  6. 작업결과를 웹브라우져에 전송한다.
>
> 세계은행(World Bank)이 URL을 변경하지 않는다면, 
> 어떤 방법을 사용하는지 알 필요없이, 
> 프로그램을 변경하지 않고도 원하는 데이터를 조건만 변경하여 다운로드할 수 있다.

만약 2개 혹은 3개 국가에 대한 데이터만 살펴본다면, 
하나씩 파일을 다운로드하기만 하면 된다. 
하지만, 다른 많은 나라에 데이터를 비교하고자 한다면 프로그램을 작성해야 한다.

파이썬에는 URL 작업을 위한 `urllib2` 라이브러리가 있지만, 사용하기가 좀 투박하다. 
그래서 많은 사람들은 [Requests](http://docs.python-requests.org)라는 제3자 라이브러리를 선호한다. 
설치하기 위해서 다음 명령어를 실행한다.

~~~ {.bash}
$ pip install requests
~~~

> ## Pip 로 설치하기 {.callout}
>
> `pip` 는 그 자체로 프로그램임에 유의한다.
> 그래서, 상기 명령어는 쉘에서 실행되어야만 되고,
> 파이썬 내부에서 실행되면 *안* 된다.

만약 Request가 아직 설치되지 않았다면, 
`pip` 출력결과는 다음과 같다:

~~~ {.output}
Downloading/unpacking requests
  Downloading requests-2.7.0-py2.py3-none-any.whl (470kB): 470kB downloaded
Installing collected packages: requests
Successfully installed requests
Cleaning up...
~~~

만약 이미 설치했다면, 출력결과는 다음과 같다:

~~~ {.output}
Requirement already satisfied (use --upgrade to upgrade): requests in /Users/swc/anaconda/lib/python2.7/site-packages
Cleaning up...
~~~

어떤 방식이든지, 다음과 같이 원하는 데이터를 얻을 수 있다:

~~~ {.python}
import requests
url = 'http://climatedataapi.worldbank.org/climateweb/rest/v1/country/cru/tas/year/CAN.csv'
response = requests.get(url)
if response.status_code != 200:
    print('Failed to get data:', response.status_code)
else:
    print('First 100 characters of data are')
    print(response.text[:100])
~~~
~~~ {.output}
First 100 characters of data are
year,data
1901,-7.67241907119751
1902,-7.862711429595947
1903,-7.910782814025879
1904,-8.15572929382
~~~

프로그램 첫번째 행은 `requests` 라이브러리를 가져온다. 
두번째 행은 가져올 데이터에 대한 URL을 정의한다; 
세번째 행에 `requests.get` 호출에 인자로 URL을 넘길 수도 있지만, 
변수로 대입하는 것이 찾기가 더 쉽다.

사실 `requests.get` 메쏘드가 데이터를 가져온다. 좀더 구체적으로 살펴보면:

*   `climatedataapi.worldbank.org` 서버에 연결을 생성한다;
*   서버에 URL로 `/climateweb/rest/v1/country/cru/tas/year/CAN.csv`을 전송한다;
*   서버 응답결과를 담기 위해서 사용자 컴퓨터 메모리에 객체를 생성한다;
*   요청결과가 성공했는지 아닌지를 식별하기 위해서 객체의 `status_code` 멤버 변수에 숫자를 할당한다.
*   웹서버에서 전송받은 데이터를 객체 `text` 멤버 변수에 할당한다.

서버는 다른 많은 [상태코드(status codes)](reference.html#http-status-code)를 반환할 수 있다; 
가장 일반적인 것은 다음과 같다:

|코드|명칭                              |의미                                                                      |
|----|----------------------------------|--------------------------------------------------------------------------|
|200 |성공(OK)                          |요청 성공.                                                                |
|204 |콘텐츠 없음(No Content)           |서버가 요청을 수행했지만, 어떤 데이터도 반환할 필요가 없다.               |
|400 |잘못된 요청(Bad Request)          |서버가 요청의 구문을 인식하지 못했다.                                     |
|401 |권한없음(Unauthorized)            |이 요청은 인증이 필요하다.                                                |
|404 |찾을 수 없음(Not Found)           |서버가 요청한 페이지를 찾을 수 없다.                                      |
|408 |요청시간초과(Timeout)             |서버의 요청 대기가 시간을 초과하였다.                                     |
|418 |I'm a teapot                      |No, really...                                                             |
|500 |내부서버오류(Internal Server Error)|서버에 오류가 발생하여 요청을 수행할 수 없다.                            |

상기 상태코드 중에서 정말 신경쓸 것은 단하나 200다; 만약 다른 것을 보게 된다면, 응답은 아마도 실제 데이터를 포함하고 있지 않는다. (오류 메시지를 포함할 수도 있지만)

> ## 몇몇 사람은 규칙을 지키지 않는다. {.callout}
>
> 불행하게도, 몇몇 사이트는 유의미한 상태코드를 반환하지 않는다. 
> 대신에 *모두* 200를 반환하고, 응답 텍스트에 오류 메시지를 넣는다. 
> 결과가 사람에게 보여진다면 의미가 있지만, 
> 실질적으로 읽을 수 없는 프로그램이 "독자"가 되어 읽게 된다면 프로그램이 중단된다.


> ## REST API 정의 {.challenge}
>
> REST API는 다음과 같다:  
> 1.  데이터 형식이다.  
> 2.  URL을 경유하여 데이터에 접속하는 방식이다.  
> 3.  서버에 적은 부하를 준다.  
> 4.  Requests같은 파이썬 라이브러리를 경유해서만 접근할 수 있다.

> ## 과테말라(Guatemala) 에 대한 데이터 수집 {.challenge}
> 
> 과테말라에 대한 온도정보를 가져오도록 프로그램을 일부 변경한다.

> ## 아프카니스탄(Afghanistan)이 얼마나 더울까? {.challenge}
> 
> 기후 데이터 API에 나온 [documentation](http://data.worldbank.org/developers/climate-data-api)를 읽고 나서,
> 1980년부터 1999년 사이 아프카니스탄 연평균온도를 알아내는 URL을 작성한다.
