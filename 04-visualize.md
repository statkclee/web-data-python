---
layout: page
title: 웹에 있는 데이터 작업
subtitle: 시각화
minutes: 15
---
> ## 학습목표 {.objectives}
>
> *   `pyplot`을 사용해서 간단히 시각화한다.


긴 숫자 목록은 그다지 유용하지 못하다.
하지만, 이제 국가별로 기온차이를 시각화하는데 필요한 모든 도구를 갖게 되었다:

~~~ {.python}
from matplotlib import pyplot as plt

australia = annual_mean_temp('AUS')
canada = annual_mean_temp('CAN')
diff = diff_records(australia, canada)
plt.plot(diff)
plt.show()
~~~

![First Plot](fig/plot-01.png)

이것은 원하는 바가 아니다: 
`pyplot` 라이브러리가 리스트 짝을 하나의 곡선에 대한 (x,y) 좌표라기 보다는 두개 상응하는 곡선으로 해석했다. 
(year, difference) 리스트 짝을 NumPy 배열로 변환하자:

~~~ {.python}
import numpy as np
d = np.array(diff)
~~~

그리고 나서, 두번째 칼럼에 대해서 첫번째 칼럼을 도식화한다:

~~~ {.python}
plt.plot(d[:, 0], d[:, 1])
plt.show()
~~~

![Second Plot](fig/plot-02.png)

차이(difference)가 천천히 줄어드는 것처럼 보이지만, 
신호(signal)에 잡음(noise)이 많다. 
이 지점에서, 만약 실질적 해답을 구한다면, 
곡선적합(curve-fitting) 라이브러리를 사용하거나 유의미한 통계량을 계산할 시점이다.

> ## 시각화 변경하기 {.challenge}
>
> 플롯(plot) 명령어를 변경해서 Y축 스케일을 0에서 32로 바꿔라.
> 이렇게 스케일을 변경하는 것이 데이터에 대한 좀더 정확한 혹은 덜 정확한 시점정보를 제공한다고 생각합니까?
