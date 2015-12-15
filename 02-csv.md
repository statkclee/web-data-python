---
layout: page
title: 웹에 있는 데이터 작업
subtitle: CSV 데이터 처리
minutes: 15
---
> ## 학습목표 {.objectives}
>
> *   `csv` 라이브러리를 사용해서 CSV 데이터 파싱.
> *   여러줄 문자열로 된 CSV 파일을 파싱하는 프로그램 작성.

앞에서 작성한 작은 프로그램이 원하는 데이터를 물어다준다. 
하지만, 숫자 리스트보다는 한줄로 된 긴 문자열로 반환한다. 
긴 문자열을 숫자 리스트로 변환하는 방법이 두가지 있다:

Our little program gets the data we want,
but returns it as one long character string rather than as a list of numbers.
There are two ways we could convert the former to the latter:

*   함수를 작성해서 개행문자로 문자열을 쪼개서 행을 생성하고 나서, 
    콤마로 행을 다시 쪼개고 나서, 마지막으로 콤마로 구분된 각 부분을 숫자로 변환한다.
*   사용자를 위해서 상기 작업을 해주는 파이썬 라이브러리를 사용한다.


대부분 경험 많은 프로그래머는 두번째 접근법이 더 쉽다고 말하지만, "쉽다"는 것은 상대적이다: 
만약 라이브러리가 존재한다는 것을 인지하고, 충분히 알고 있어서, 
라이브러리가 수행하는 것으로 문제를 해결하는 방식을 알고 있다면, 
표준 라이브러리를 사용하는 것이 실무에서 좀더 효과적이 된다.

두가지 방식을 함께 시도해 보자. 
시작하려면, 다음 세줄을 담고 있는 `test01.csv` 파일을 생성한다:

~~~
1901,12.3
1902,45.6
1903,78.9
~~~

파일을 줄마다 읽고 (예를 들어) 각 줄에 대한 길이정보를 출력하기가 쉽다:

~~~ {.python}
with open('test01.csv', 'r') as reader:
    for line in reader:
        print(len(line))
~~~
~~~ {.output}
10
10
10
~~~

각 줄을 콤마로 쪼개서 각 줄을 문자열 조각 리스트로 변환한다:

~~~ {.python}
with open('test01.csv', 'r') as reader:
    for line in reader:
        fields = line.split(',')
        print(fields)
~~~
~~~ {.output}
['1901', '12.3\n']
['1902', '45.6\n']
['1903', '78.9\n']
~~~

날짜 정보는 올바르지만, 모든 값이 `\n`으로 끝난다.
[이스케이프 시퀀스(escape sequence)](reference.html#escape-sequence)로 
각 줄 마지막에 붙는 개행 문자다.
이것을 제거하려면, 콤마로 쪼개기 전에, 각 줄로부터 선두 및 꼬리 여백(whitespace)을 벗겨내야 된다:

~~~ {.python}
with open('test01.csv', 'r') as reader:
    for line in reader:
        fields = line.strip().split(',')
        print(fields)
~~~
~~~ {.output}
['1901', '12.3']
['1902', '45.6']
['1903', '78.9']
~~~

이제 몇몇 표준 파이썬 라이브러리를 통해서 데이터를 파싱하는 방법을 살펴보자. 
사용할 라이브러리는 `csv`다. 
`csv` 라이브러리는 그 자체로 데이터를 읽어들이지 않는다: 
대신에, 무언가로 읽어온 행을 받고서, 콤마로 쪼개고 리스트 값으로 변환한다.
다음에 `csv` 라이브러리를 사용하는 방법이 나와있다:

~~~ {.python}
import csv

with open('test01.csv', 'r') as raw:
    cooked = csv.reader(raw)
    for record in cooked:
        print(record)
~~~
~~~ {.ouptut}
['1901', '12.3']
['1902', '45.6']
['1903', '78.9']
~~~

여기서, `raw`는 정상적인 방식으로 데이터를 읽어온다.
반면에 `cooked`는 [래퍼(wrapper)](reference.html#wrapper)로 
텍스트 한줄을 받아들이고, 이를 필드 리스트로 변환한다:

동일하게 `csv.reader` 메쏘드에 파일이 아닌 문자열 리스트를 줄 수도 있다:

~~~ {.python}
import csv

with open('test01.csv', 'r') as raw:
    lines = raw.readlines()
cooked = csv.reader(lines)
for record in cooked:
    print(record)
~~~
~~~ {.output}
['1901', '12.3']
['1902', '45.6']
['1903', '78.9']
~~~

`csv` 라이브러리를 사용하는 것이 문자열만 쪼개는 것보다 더 간단해 보이지 않니잠,
다음과 같은 데이터를 만났을 때, 무슨 일이 발생하는지 살펴보라:

~~~
"Meltzer, Marlyn Wescoff",1922,2008
"Spence, Frances Bilas",1922,2012
"Teitelbaum,Ruth Lichterman",1924,1986
~~~

단순한 문자열 쪼개기를 하면, 출력결과는 다음과 같다:

~~~ {.output}
['"Meltzer', ' Marlyn Wescoff"', '1922', '2008']
['"Spence', ' Frances Bilas"', '1922', '2012']
['"Teitelbaum', 'Ruth Lichterman"', '1924', '1986']
~~~

이중 인용부호가 여전히 있고,
사람 각각 이름을 포함하고 있는 필드는 조각으로 나눠줬다.
반면에, 만약 `csv` 라이브러리를 사용한다면, 결과는 다음과 같다:

~~~ {.output}
['Meltzer, Marlyn Wescoff', '1922', '2008']
['Spence, Frances Bilas', '1922', '2012']
['Teitelbaum,Ruth Lichterman', '1924', '1986']
~~~

왜냐하면, 라이브러리가 콤마(그리고 더 많은 뭔가)를 포함하는 텍스트 필드를 처리하는 방법을 이해하고 있기 때문이다.

기후 데이터에 `csv` 를 사용하기 전에 한가지 더 작업을 할 필요가 있다.
세계은행 API를 사용해서 특정 국가에 대한 데이터를 얻을 때, 긴 한줄 문자열로 반환된다:

~~~
year,data
1901,-7.67241907119751
1902,-7.862711429595947
1903,-7.910782814025879
...
~~~

`csv.reader`에 긴 한줄 문자열을 넣기 전에 줄로 쪼개야만 되고,
얼마전에 마주한 이스케이프 시퀀스에 동일하게 있는 문자열을 쪼갬으로써 작업을 수행할 수 있다.
이 방식이 제대로 동작하는지 알아내기 위해서, `test01.csv` 파일을 읽어서 메모리에 넣고,
쪼개서 조각낸다:

~~~ {.python}
with open('test01.csv', 'r') as reader:
    data = reader.read()
    lines = data.split('\n')
    print(lines)
~~~
~~~ {.output}
['1901,12.3', '1902,45.6', '1903,78.9', '']
~~~

*거의* 맞게 처리됐지만, 리스트 끝에 빈 문자열이 왜 있을까요?
정답은 파일 마지막 줄은 개행(newline)으로 끝난다는데 있다.
그래서, 아래 예제처럼 파이썬도 동일하게 동작한다:

~~~ {.python}
fields = 'a-b-'.split('-')
print(fields)
~~~
~~~ {.output}
['a', 'b', '']
~~~

다시 한번 해법은 쪼개기 전에 시작과 끝단 여백(whitespace)을 벗겨내는 것이다:

~~~ {.python}
with open('test01.csv', 'r') as reader:
    data = reader.read()
    lines = data.strip().split('\n')
    print(lines)
~~~
~~~ {.output}
['1901,12.3', '1902,45.6', '1903,78.9']
~~~

이 모든 것을 한군데 모으게 되면, 다음과 같이 캐나다에 대한 데이터를 얻을 수 있게 된다:

~~~ {.python}
import requests
import csv

url = 'http://climatedataapi.worldbank.org/climateweb/rest/v1/country/cru/tas/year/CAN.csv'
response = requests.get(url)
if response.status_code != 200:
    print('Failed to get data:', response.status_code)
else:
    wrapper = csv.reader(response.text.strip().split('\n'))
    for record in wrapper:
        print(record)
~~~
~~~ {.output}
['year', 'data']
['1901', '-7.67241907119751']
['1902', '-7.862711429595947']
['1903', '-7.910782814025879']
['1904', '-8.155729293823242']
['1905', '-7.547311305999756']
...
~~~

진전된 것 같아 보인다. 그래서 문자열에서 실제로 원하는 숫자로 변환하자:

~~~ {.python}
import requests
import csv

url = 'http://climatedataapi.worldbank.org/climateweb/rest/v1/country/cru/tas/year/CAN.csv'
response = requests.get(url)
if response.status_code != 200:
    print('Failed to get data:', response.status_code)
else:
    wrapper = csv.reader(response.text.strip().split('\n'))
    for record in wrapper:
        year = int(record[0])
        value = float(record[1])
        print(year, value)
~~~
~~~ {.error}
Traceback (most recent call last):
  File "api-with-naive-converting.py", line 11, in <module>
    year = int(record[0])
ValueError: invalid literal for int() with base 10: 'year'
~~~

데이터 첫번째 줄 때문에 오류가 발생했다:

~~~
year,data
~~~

문자열 `'year'`을 정수형으로 변환할 때, 파이썬에서 바로 항의가 들어온다. 
오류수정은 복잡하지 않다: 
단어 year로 시작하는 행을 무시하고 넘어간다. 

When we try to convert the string `'year'` to an integer,
Python quite rightly complains.
The fix is straightforward:
we just need to ignore lines that start with the word `year`:

~~~ {.python}
import requests
import csv

url = 'http://climatedataapi.worldbank.org/climateweb/rest/v1/country/cru/tas/year/CAN.csv'
response = requests.get(url)
if response.status_code != 200:
    print('Failed to get data:', response.status_code)
else:
    wrapper = csv.reader(response.text.strip().split('\n'))
    results = []
    for record in wrapper:
        if record[0] != 'year':
            year = int(record[0])
            value = float(record[1])
            print(year, value)
~~~
~~~ {.output}
1901 -7.67241907119751
1902 -7.862711429595947
1903 -7.910782814025879
1904 -8.155729293823242
1905 -7.547311305999756
...
~~~

> ## CSV 파일 구성 {.challenge}
>
> CSV 파일이 다음과 같이 구분될 필요가 있다:
>
> 1.  레코드(필드) 그리고 나서 행(줄).
> 2.  행(줄) 그리고 나서 레코드(필드).
> 3.  개행(Newline) 문자.
> 4.  콤마와 기타 문자.


> ### 이스케이프 시퀀스(Escape Sequences) {.callout}
> 
> 문자열에 인용부호, 이중 인용부호, 그리고 다른 특수 문자를 표현할 방법이 필요하다. 이를 위해서 이스케이프 시퀀스(escape sequences)를 사용한다. `\'`는 단일 인용부호, `\"`는 이중 인용부호, `\n`는 개행문자. 등등  

~~~ {.shell}
'This can\'t be\nwritten without\n\"escape sequences\".' 이 의미하는 바는 다음과 같다.
~~~

~~~ {.output}
This can't be 
written without 
"escape sequences". 
~~~
