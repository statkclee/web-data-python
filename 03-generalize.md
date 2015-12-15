---
layout: page
title: 웹에 있는 데이터 작업
subtitle: 오류(Error) 처리와 일반화
minutes: 15
---
> ## 학습목표 {.objectives}
>
> *   스크립트를 함수로 바꾸기.
> *   오류를 명시적으로 처리함으로써 함수를 더 강건하게 만든다.

캐나다에 대한 데이터를 얻는 방법을 이제 알게되었으니, 
임의 나라에 대해서 동일한 작업을 수행하는 함수를 작성하자. 
절차는 단순하다: 

1.  작성한 코드를 복사해서 3-문자 국가코드를 매개변수로 받는 함수를 작성한다.
2.  국가코드를 적당한 곳에 URL에 삽입한다.
3.  화면에 출력하는 대신에 리스트로 결과를 반환한다.

작업결과 함수는 다음과 같다:

~~~ {.python}
def annual_mean_temp(country):
    ''' ("CAN" 처럼) 3-문자로 된 ISO코드로 특정 국가에 대한 연평균 기온정보를 얻어온다.'''
    url = 'http://climatedataapi.worldbank.org/climateweb/rest/v1/country/cru/tas/year/' + country + '.csv'
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
                results.append([year, value])
        return results
~~~

상기 함수는 다음과 같이 동작한다:

~~~ {.python}
canada = annual_mean_temp('CAN')
print('first three entries for Canada:', canada[:3])
~~~
~~~ {.output}
first three entries for Canada: [[1901, -7.67241907119751], [1902, -7.862711429595947], [1903, -7.910782814025879]]
~~~

하지만 문제가 있다. 유효하지 않는 국가 식별자를 매개변수로 전달할 때 무슨일이 발생하는지 살펴보자:

~~~ {.python}
latveria = annual_mean_temp('LTV')
print 'first three entries for Latveria:', latveria[:3]
~~~
~~~ {.output}
first three entries for Latveria: []
~~~

Latveria는 존재하지 않는다. 
그래서 왜 작성한 함수는 오류 메시지를 출력하는 대신에 빈 리스트를 반환할까? 
오류 메시지가 없다는 의미는 응답코드가 200을 의미한다: 
만약 그밖의 일이 있다면, `if` 분기로 가서
메시지를 출력하고 `None`을 반환한다(특정한 어떤 것도 반환하지 않고자 할 때 함수가 수행하는 작업).

그래서, 만약 응답코드가 200이고 어떤 데이터도 없다면, 지금 보고 있는 것이 설명된다.
검사해보자:

~~~ {.python}
def annual_mean_temp(country):
    ''' ("CAN" 처럼) 3-문자로 된 ISO코드로 특정 국가에 대한 연평균 기온정보를 얻어온다.'''
    url = 'http://climatedataapi.worldbank.org/climateweb/rest/v1/country/cru/tas/year/' + country + '.csv'
    print('url used is', url)
    response = requests.get(url)
    print('response code:', response.status_code)
    print('length of data:', len(response.text))
    if response.status_code != 200:
        print('Failed to get data:', response.status_code)
    else:
        wrapper = csv.reader(response.text.strip().split('\n'))
        results = []
        for record in wrapper:
            if record[0] != 'year':
                year = int(record[0])
                value = float(record[1])
                results.append([year, value])
        return results

latveria = annual_mean_temp('LTV')
print('number of records for Latveria:', len(latveria))
~~~
~~~ {.output}
url used is http://climatedataapi.worldbank.org/climateweb/rest/v1/country/cru/tas/year/LTV.csv
response code: 200
length of data: 0
number of records for Latveria: 0
~~~

다른 말로, 세계은행 사이트는 설사 실제로 그럴 수 없을때도, 항상 "여러분의 질의에 대답할 수 있어요"라고 말하고 있다.
약간 더 실험한 후에, 200 상태 코드를 *항상* 반환하는 것을 발견했다. 
실제 데이터가 있는지 없는지를 알 수 있는 유일한 방식은 `response.text`가 비었는지 점검하는 것이다. 
다음에 갱신된 함수가 있다:

~~~ {.python}
def annual_mean_temp(country):
    ''' 
    ("CAN" 처럼) 3-문자로 된 ISO코드로 특정 국가에 대한 연평균 기온정보를 얻어온다.
    만약 국가코드가 적법하지 않다면, 빈 리스트를 반환하라.
    '''
    url = 'http://climatedataapi.worldbank.org/climateweb/rest/v1/country/cru/tas/year/' + country + '.csv'
    response = requests.get(url)
    results = []
    if len(response.text) > 0:
        wrapper = csv.reader(response.text.strip().split('\n'))
        for record in wrapper:
            if record[0] != 'year':
                year = int(record[0])
                value = float(record[1])
                results.append([year, value])
    return results

print('number of records for Canada:', len(annual_mean_temp('CAN')))
print('number of records for Latveria:', len(annual_mean_temp('LTV')))
~~~
~~~ {.output}
number of records for Canada: 109
number of records for Latveria: 0
~~~

다른 국가에 대한 지상 기온을 얻을 수 있기 때문에, 
국가 온도를 비교하는 함수를 작성할 수 있다. 
(이제 궁극적으로 작업하려는 것에 대해서 명확해졌기 때문에, 바로 함수를 작성한다.) 
다음에 시도한 첫번째 함수가 있다:

~~~ {.python}
def diff_records(left, right):
    '''[year, value] 리스트 짝이 주어지면, [year, difference] 리스트 짝을 반환하라.'''
    num_years = len(left)
    results = []
    for i in range(num_years):
        left_year, left_value = left[i]
        right_year, right_value = right[i]
        difference = left_value - right_value
        results.append([left_year, difference])
    return results
~~~

여기서 루프 제어를 위해서 `left`에 항목 숫자를 사용한다. (항목갯수는 `len(left)`로 찾을 수 있다.)

~~~ {.python}
for i in range(num_years):
~~~

상기 표현식은 0부터 `num_years-1`까지 `i`를 실행한다. 
정확하게 `left` 인덱스와 상응한다. 
루프 내부에서 리스트 항목에서 왼쪽, 오른쪽 년도와 값을 풀어서 년도와 차이값을 `results`에 추가한다. 
그리고 마지막에 결과를 반환한다.

작성한 함수가 동작하는지 살펴보기 위해서, 몇개 가상으로 데이터를 만들어서 실행한다:

~~~ {.python}
print('one record:', diff_records([[1900, 1.0]],
                                  [[1900, 2.0]]))
print('two records:', diff_records([[1900, 1.0], [1901, 10.0]],
                                   [[1900, 2.0], [1901, 20.0]]))
~~~
~~~ {.output}
one record: [[1900, -1.0]]
two records: [[1900, -1.0], [1901, -10.0]]
~~~

매우 좋아 보인다— 하지만, 다음 테스트 케이스는 어떨까?

~~~ {.python}
print('mis-matched years:', diff_records([[1900, 1.0]],
                                         [[1999, 2.0]]))
print('left is shorter', diff_records([[1900, 1.0]],
                                      [[1900, 10.0], [1901, 20.0]]))
print('right is shorter', diff_records([[1900, 1.0], [1901, 2.0]],
                                       [[1900, 10.0]]))
~~~
~~~ {.error}
---------------------------------------------------------------------------
IndexError                                Traceback (most recent call last)
<ipython-input-15-7582f56db8bf> in <module>()
      4                                       [[1900, 10.0], [1901, 20.0]])
      5 print('right is shorter', diff_records([[1900, 1.0], [1901, 2.0]],
----> 6                                        [[1900, 10.0]]))

<ipython-input-13-67464343fd99> in diff_records(left, right)
      5     for i in range(num_years):
      6         left_year, left_value = left[i]
----> 7         right_year, right_value = right[i]
      8         difference = left_value - right_value
      9         results.append([left_year, difference])

IndexError: list index out of rangemis-matched years: [[1900, -1.0]]
left is shorter [[1900, -9.0]]
right is shorter
~~~

설사 년도가 매칭되지 않지 않더라도, 첫번째 테스트는 답을 제시한다: 
결과값을 얻었지만, 무의미하다. 
두번째 테스트 케이스는 부분적인 결과를 제시한다. 
이번에도 문제가 있다고 보고하지는 않는다. 
하지만, 세번째는 프로그램이 중단된다. 
왜냐하면 레코드 숫자를 결정하는데 `left`를 사용하지만, 
`right`는 그만큼의 숫자를 가지고 있지 않기 때문이다.

첫두 문제는 세번째 것보다 사실 더 나쁘다. 
왜냐하면 첫두 문제가 [침묵하는 실패(silent failures)](reference.html#silent-failure)의 전형이기 때문이다: 
함수가 잘못된 것을 수행하지만, 어떤 방식으로도 나타나고 있지 않는다.
버그를 수정해보자:

~~~ {.python}
def diff_records(left, right):
    '''
    [year, value] 리스트 짝이 주어지면, [year, difference] 리스트 짝을 반환하라.
    만약 입력이 정확하게 대응되는 년도가 아니라면 동작하지 않는다.
    '''
    assert len(left) == len(right), \
           'Inputs have different lengths.'
    num_years = len(left)
    results = []
    for i in range(num_years):
        left_year, left_value = left[i]
        right_year, right_value = right[i]
        assert left_year == right_year, \
               'Record {0} is for different years: {1} vs {2}'.format(i, left_year, right_year)
        difference = left_value - right_value
        results.append([left_year, difference])
    return results
~~~

작성한 "착한" 테스트 케이스는 통과했나요?

~~~ {.python}
print('one record:', diff_records([[1900, 1.0]],
                                  [[1900, 2.0]]))
print('two records:', diff_records([[1900, 1.0], [1901, 10.0]],
                                   [[1900, 2.0], [1901, 20.0]]))
~~~
~~~ {.output}
one record: [[1900, -1.0]]
two records: [[1900, -1.0], [1901, -10.0]]
~~~

이제 실패가 예상되는 세가지 테스트 케이스는 어떤가요?

~~~ {.python}
print('mis-matched years:', diff_records([[1900, 1.0]],
                                         [[1999, 2.0]]))
~~~
~~~ {.error}
---------------------------------------------------------------------------
AssertionError                            Traceback (most recent call last)
<ipython-input-18-c101917a748e> in <module>()
      1 print('mis-matched years:', diff_records([[1900, 1.0]],
----> 2                                          [[1999, 2.0]]))

<ipython-input-16-d41327791c15> in diff_records(left, right)
     10         left_year, left_value = left[i]
     11         right_year, right_value = right[i]
---> 12         assert left_year == right_year,                'Record {0} is for different years: {1} vs {2}'.format(i, left_year, right_year)
     13         difference = left_value - right_value
     14         results.append([left_year, difference])

AssertionError: Record 0 is for different years: 1900 vs 1999mis-matched years:
~~~

~~~ {.python}
print('left is shorter', diff_records([[1900, 1.0]],
                                      [[1900, 10.0], [1901, 20.0]]))
~~~
~~~ {.error}
---------------------------------------------------------------------------
AssertionError                            Traceback (most recent call last)
<ipython-input-19-682d448d921e> in <module>()
      1 print('left is shorter', diff_records([[1900, 1.0]],
----> 2                                       [[1900, 10.0], [1901, 20.0]]))

<ipython-input-16-d41327791c15> in diff_records(left, right)
      4     Fails if the inputs are not for exactly corresponding years.
      5     '''
----> 6     assert len(left) == len(right),            'Inputs have different lengths.'
      7     num_years = len(left)
      8     results = []

AssertionError: Inputs have different lengths. left is shorter
~~~
~~~ {.python}
print('right is shorter', diff_records([[1900, 1.0], [1901, 2.0]],
                                       [[1900, 10.0]]))
~~~
~~~ {.error}
---------------------------------------------------------------------------
AssertionError                            Traceback (most recent call last)
<ipython-input-20-a475e608dd70> in <module>()
      1 print('right is shorter', diff_records([[1900, 1.0], [1901, 2.0]],
----> 2                                        [[1900, 10.0]]))

<ipython-input-16-d41327791c15> in diff_records(left, right)
      4     Fails if the inputs are not for exactly corresponding years.
      5     '''
----> 6     assert len(left) == len(right),            'Inputs have different lengths.'
      7     num_years = len(left)
      8     results = []

AssertionError: Inputs have different lengths. right is shorter
~~~

정말 훌륭해요: 만약 올바르지 않은 형식이거나 불합치되는 데이터를 처리하려고 함면, 
추가한 가정대입문(assertion)이 이제 경고를 보낸다.

> ### 더 좋은 방법이 있다. {.callout}
> 
> 프로그램 셀 내부에서 각 테스트를 실행해야만 한다. 
> 왜냐하면, 가정대입문이 실패하자마자 코드 실행을 파이썬이 멈추기 때문이고, 
> 테스트 3개가 모두 실질적으로 실행되도록 확실히 하고자 한다. 
> 단위 테스트 (unit testing) 라이브러리가 사용자를 대신해서 처리하고, 
> 또한 그밖의 더 많은 것도 수행한다; 
> 파이썬 중급과정에서 단위 테스트 라이브러리를 다룬다.


>## 오류 처리 {.challenge}
>
> 파이썬 스크립트는 오류처리 코드를 가져야만 된다. 왜냐하면,
>
> 1.  파이썬은 태생적으로 신뢰성이 떨어지는 언어다.
> 2.  함수는 오류를 반환할 수 있다.
> 3.  제공된 데이터가 예상하는 바와 같다고 믿어사는 않된다.
> 4.  파이썬 스크립트는 오류가 나면 멈춘다. 그래서 작업이 완수되지 못한다.

> ## 언제 불평할까? {.challenge}
>
> 세계은행과 동일한 실수를 방금전에 실제로 저절렀다:
> 만약 누군가 `annual_mean_temp`에 적법하지 못한 국가 식별자를 넣게 되면,
> 오류를 보고하지 않고, 대신에 빈 리스트를 반환한다.
> 그래서 호출자가 어떻게든 알아야만 된다.
> 데이터를 다운로드 받지 못할 경우 실패하는 가정대입문을 사용해야 할까? 
> 왜 그럴까 혹은 왜 그렇지 않을까?

> ## 열거(Enumerating) {.challenge}
>
> 파이썬은 `enumerate`라는 함수를 포함하고 있는데, 흔히 `for` 루프에 사용된다.
> 루프가 다음과 같으면:
>
> ~~~ {.python}
> for (i, c) in enumerate('abc'):
>     print(i, '=', c)
> ~~~
>
> 다음을 출력한다.
>
> ~~~ {.output}
> 0 = a
> 1 = b
> 2 = c
> ~~~
>
> Rewrite `diff_records` to use `enumerate`.
