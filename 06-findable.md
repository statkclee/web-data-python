---
layout: page
title: 웹에 있는 데이터 작업
subtitle: 데이터를 찾을 수 있게 만들기

minutes: 15
---
> ## 학습목표 {.objectives}
>
> *   메타데이터(metadata)를 제공해서 데이터셋을 더 유용하게 만든다.

파일명칭을 생성하는데 어떤 규칙이 있는지 일러주는 것으로 충분하지 못하다.
왜냐하면, 실제로 어떤 데이터셋을 생성해내는지 말해주지 않기 때문이다.
따라서, 이번 수업의 마지막 단계는 생성한 데이터를 찾을 수 있도록 무슨 파일이 존재하는지
일러주는 [색인(index)](reference.html#index)을 생성한다.

다음에 사용할 형식이 나와 있다:

~~~
2014-05-26,FRA,TCD,FRA-TCD.csv
2014-05-27,AUS,BRA,AUS-BRA.csv
2014-05-27,AUS,CAN,AUS-CAN.csv
2014-05-28,BRA,CAN,BRA-CAN.csv
~~~

칼럼은 데이터셋이 생성된 일자, 비교되는 두 국가에 대한 식별자,
데이터파일 명칭이다. 날짜를 포함해서 어느 시점에 갱신되었는지 알기 쉽게 한다.
하지만, 왜 귀찮게 파일명칭을 포함할까?
결국, 두 국가코드가 주어지면 쉽게 파일을 다시 생성할 수 있다.
대답은 *작성자 우리*는 생성되는 파일명칭에 대한 규칙을 알고 있지만, 
다른 사람의 프로그램은 그럴 필요가 없다는 것이다.

다음에 새로운 데이터 파일을 생성할 때마다 인덱스 파일을 갱신하는 함수가 나와 있다:

~~~ {.python}
import time

def update_index(index_filename, left, right):
    '''Append a record to the index.'''

    # Read existing data.
    with open(index_filename, 'r') as raw:
        reader = csv.reader(raw)
        records = []
        for r in reader:
            records.append(r)
    
    # Create new record.
    timestamp = time.strftime('%Y-%m-%d')
    data_filename = left + '-' + right + '.csv'
    new_record = (timestamp, left, right, data_filename)
    
    # Save.
    records.append(new_record)
    with open(index_filename, 'w') as raw:
        writer = csv.writer(raw)
        writer.writerows(records)
~~~

프로그램을 테스트해보자.
만약 색인 파일이 다음을 포함하고 있다면:

~~~
2014-05-26,FRA,TCD,FRA-TCD.csv
2014-05-27,AUS,BRA,AUS-BRA.csv
2014-05-27,AUS,CAN,AUS-CAN.csv
2014-05-28,BRA,CAN,BRA-CAN.csv
~~~

그리고, 실행하면:

~~~ {.python}
update_index('data/index.csv', 'TCD', 'CAN')
~~~

그리고 나면 색인 파일은 이제 다음을 담고 있게된다:

~~~
2014-05-26,FRA,TCD,FRA-TCD.csv
2014-05-27,AUS,BRA,AUS-BRA.csv
2014-05-27,AUS,CAN,AUS-CAN.csv
2014-05-28,BRA,CAN,BRA-CAN.csv
2014-05-29,TCD,CAN,TCD-CAN.csv
~~~

이제 모든 것이 제자리에 있기 때문에, 우리 자신이나 다른 사람도 작성한 데이터로 새롭고
흥미로운 것을 수행하기 쉽다. 예를 들어, 
쉽게 작은 프로그램을 작성해서 어떤 데이터셋이 특정 국가에 관한 정보를 포함하고 있는지,
마지막 검사한 이래로 게시되었는지 알수 있게 된다:

~~~ {.python}
def what_is_available(index_file, country, after):
    '''What data files include a country and have been published since 'after'?'''
    with open(index_file, 'r') as raw:
        reader = csv.reader(raw)
        filenames = []
        for record in reader:
            if (after <= record[0]) and (country in (record[1], record[2])):
                filenames.append(record[3])
    return filenames

print what_is_available('data/index.csv', 'BRA', '2014-05-27')
~~~
~~~ {.output}
['AUS-BRA.csv', 'BRA-CAN.csv']
~~~

> ## 새로운 유형의 과학 {.callout}
>
> 이것이 돌파구처럼 보이지는 않을지도 모르지만,
> 실제로 웹이 어떻게 과학연구자를 도와서 새로운 유형의 과학을 수행하는지에 관한 한 사례다.
> 약간 더 작업을 하면, 
> *본인* 컴퓨터에 파일을 생성해서 언제 마지막으로 데이터를 생성한 각기 다른 사이트에 대해서
> `what_is_available` 프로그램을 실행했는지 기록할 수 있다.
> 매번 프로그램을 실행할 때마다, 작성한 프로그램은 다음을 수행한다:
>
> *   로컬 컴퓨터에 있는 "점검목록" 파일을 읽어들인다;
> *   각 데이터에 대해 어떤 새로운 데이터가 있는지 물어본다;
> *   해당 데이터를 다운로드하고 처리한다; 그리고
> *   사용자에게 결과를 요약하여 제시한다.
>
> 이것이 정확하게 블로그 프로그램이 동작하는 방식이다.
> 모든 블로그 리더(reader) 프로그램은 점검하기로 된 블로그 URL 목록을 간직하고 있다.
> 프로그램이 실행될 때, 각 사이트로 가서 (`feed.xml`처럼 불리는 전형적인) 색인 파일에 물어본다.
> 그리고 나서, 이미 살펴본 로컬 기록과 해당 색인에 등재된 기사를 점검한다.
> 그리고 나서, 새로운 기사를 다운로드한다.
> 이런 과정을 자동화함으로써, 블로깅 도구가 실제로 볼 가치가 있는 것에만 집중하도록 돕고 있다.

> ## 색인(Indexing) {.challenge}
>
> 생성된 데이터에 대한 색인을 항상 생성해야만 된다. 왜냐하면:
>
> 1.  변경에 대해서 자동화된 방식으로 점검을 할 수 있다.
> 2.  웹서버는 색인없이 디렉토리를 보여주지 않는다.
> 3.  REST API는 함수에 색인을 요구한다.
> 4.  너무나 복잡해서 프로그램 스스로 계산할 수 없다.

> ## 메타데이터에 대한 메타데이터 {.challenge}
>
> 인덱스 파일의 첫번째 줄이 칼럼 명칭정보를 부여하는 헤더여야만 되는가?
> 왜 혹은 왜 아닌가?

> ## 자동화할 것인가 말 것인가 {.challenge}
>
> `save_records` 내부에서 `update_index`을 호출되어서
> 새로운 데이터가 생성될 때마다 자동으로 색인이 갱신되어야 될까?
> 왜 혹은 왜 아닌가?

> ## 불필요한 쓸모 없는 것 제거 {.challenge}
>
> `update_index` 와 `save_records` 모두 데이터 파일 명칭을 만들어낸다.
> 리팩토링해서 이런 불필요함을 제거하라.
