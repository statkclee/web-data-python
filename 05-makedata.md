---
layout: page
title: 웹에 있는 데이터 작업
subtitle: 데이터 게시
minutes: 15
---
> ## 학습목표 {.objectives}
>
> *   정적 데이터셋을 공유하는 파이썬 프로그램을 작성한다.

이제 다른 국가에 대한 기온데이터를 다운로드하고, 평균 온도차를 알아내는 함수를 작성했다.
다음 단계는 생성한 데이터셋을 게시함으로써 세상사람들과 발견한 것을 공유하는 것이다.
이를 위해서, 다음 세가지 질문에 답을 해야만 된다:

*   어떻게 데이터를 저장할 것인가?
*   사람들이 데이터를 어떻게 다운로드 받을 것인가?
*   사람들이 데이터를 어떻게 찾을 것인가?

첫번째 두가지 질문은 답하기가 쉽다: `diff_records`는 CSV 파일로 저장할 수 있는 (year, difference) 리스트 짝을 반환한다:

~~~ {.python}
import csv

def save_records(filename, records):
    '''Save a list of [year, temp] pairs as CSV.'''
    with open(filename, 'w') as raw:
        writer = csv.writer(raw)
        writer.writerows(records)
~~~

> ## 학습한 교훈 {.callout}
>
> 데이터를 읽을 때 사용한 것과 동일한 이유로 `csv` 라이브러리를 사용한다:
> `csv` 라이브러리는 올바르게 (콤마를 포함한 텍스트 같은)특수한 경우를 처리한다.

테스트 해보자:

~~~ {.python}
save_records('temp.csv', [[1, 2], [3, 4]])
~~~

`temp.csv` 파일을 열게되면, 기대한 대로 다음이 저장되어 있다:

~~~
1,2
3,4
~~~

이제, 이 파일이 어디로 가야할까? 대답은 분명하게 "서버(server)"가 되는데, 
이유는 노트북에 저장된 데이터는 작업할 때만 접속이 가능하기 때문이다.
(그리고, 대부분의 사람이 노트북에 웹서버를 실행하고 있지 않기 때문이다.)
하지만, 서버에서 어느 장소, 그리고 어떻게 호출해야 하나?

이런 질문에 대한 대답은 서버가 구축되는 방법에 달려있다.
사용자 다수가 사용하는 리눅스 컴퓨터에, 사용자는 홈 디렉토리 아래 `public_html` 같은
디렉토리를 생성할 수 있다. 웹서버는 자동적으로 이 디렉토리를 검색한다.
예를 들어, 만약 Nelle이 `public_html` 디렉토리에 `thesis.pdf` 이라는 파일을 갖고 있다면,
웹서버가 URL `http://the.server.name/~nelle/thesis.pdf` 요청을 받게될 때, 웹서버가 
해당 디렉토리를 찾게된다. (Nelle 이름 앞에 `~` 물결표시는 웹서버에게 Nelle의 
`public_html` 디렉토리를 찾도록 일러주는 역할을 한다.)
구체적인 사항은 컴퓨터마다 차이가 날 수 있지만, 기본적인 아이디어는 같다.

호출하는 방법에 대해서, REST 핵심 아이디어로 돌아간다:
모든 데이터셋은 "추측할 수 있는(guessable)" URL에 의해서 식별되어야만 된다.
여기 사례에서, `left-right.csv` 같은 명칭을 사용하는데, `left`와 `right`는
기온을 차이를 비교하는 3-문자 국가 코드다. 만약 호주와 브라질을 비교하려면,
`http://the.server.name/~nelle/AUS-BRA.csv` 에서 찾아야 된다고 사람들에게 일러줘야 된다.
(세계은행 API와 일관성 있게 대문자를 사용한다.)

하지만, 누군가 나쁜-이름으로 (따라서 찾을 수 없는) 파일을 생성하지 못하게 어떻게 할 수 있을까?
예를 들어, 누군가 `save_records('aus+bra.csv', records)` 부를 수 있다.
이런 일이 발생할 가능성을 줄이도록 하기 위해서,
`save_records`를 변경해서 매개변수로 국가식별자를 받도록 하자:

~~~ {.python}
import csv

def save_records(left, right, records):
    '''Save a list of [year, temp] pairs as CSV.'''
    filename = left + '-' + right + '.csv'
    with open(filename, 'w') as raw:
        writer = csv.writer(raw)
        writer.writerows(records)
~~~

다음과 같이 이제 호출할 수 있다:

~~~ {.python}
save_records('AUS', 'BRA', [[1, 2], [3, 4]])
~~~

그리고 나서, 올바른 출력파일이 생성되었는지 검사한다.
어쨌든 국가코드에 구속되어서(국가코드를 사용해서 데이터를 검색했다), 
사용자에게도 자연스러워 보여야 된다.

> ## 무엇을 검사할지 결정 {.challenge}
>
> `save_records`는 입력으로 모든 레코드가 정확하게 두 필드를 갖는지 검사해야만 되는가?
> 왜 혹은 왜 그렇지 않는가?
> 국가코드는 어떤가 - 국가코드는 실제 국가와 매칭되는 국가코드 목록을 담고 있어야 되는가
> 그리고 `left` 과 `right`이 해당 목록에 있는지 검사해야 되는가?

> ## 로컬에 구축하기 {.challenge}
>
> 학과 서버에 파일을 어떻게 게시할 수 있는지 알아내시오.

> ## 게시된 데이터 일관성 {.challenge}
>
> 게시된 데이터 파일 명칭이 일관성을 갖는게 중요한데 이유는 다음과 같다:
>
> 1.  일부 운영체제(예를 들어, 윈도우)는 공백을 다르게 처리한다.
> 2.  파일 명칭을 바꾸려고 하는데, 학과 서버에 접근할 수 없을 수 있다.
> 3.  `csv` 라이브러리가 일관성을 요구한다.
> 4.  프로그램은 올바르게 파일과 데이터가 있을 때만 처리할 수 있다.
