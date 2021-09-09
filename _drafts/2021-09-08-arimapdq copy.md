---
layout: post
title: "시계열 분석 시리즈 (3): Python auto_arima로 삼성 주가 제대로 예측하기"
date: 2021-09-08
categories: [Statistics]
tag: [time-series, python, easy-guide, data-analysis, stock-analysis] 
comments: true
photos:
    - "../../images/arimapdq-title.png"
---

* ARIMA (p,d,q) 모형에서 p와 d와 q 값을 고를 때 생각보다 "이게 최선이야?"라 생각이 들고, 위 그림처럼 예측이 전혀 안 될 때도 있습니다. 저 또한 괜히 애꿎은 p, d, q 를 바꿔가면서 수많은 모형으로 테스트를 해봤지만, 예측이 전혀 안되어서 당황했던 적이 있었는데요. 이번에 공부하면서 왜 안 되는지 깨닫게 되어 이 깨달음을 전파하고자 이 글을 씁니다.
* Python 실습을 위해 또 삼성전자의 주가를 가져와서 ARIMA로 예측해보았습니다. (~~응또삼; 응지 또 삼성전자 주가...~~)
* 역시 [실전 시계열 분석: 통계와 머신러닝을 활용한 예측 기법](http://www.yes24.com/Product/Goods/98576347){:target='_blank'} 책과 [Forecasting: Principles and Practice](https://otexts.com/fppkr/){:target='_blank'}책을 기반으로 글을 작성하였으며 ARIMA 의 이해가 부족하시다면 이전 포스트인 [시계열 분석 시리즈 (2): AR / MA / ARIMA 모형, 어디까지 파봤니?](https://assaeunji.github.io/statistics/2021-08-23-arima/){:target='_blank'}를 참고 바랍니다.

---
## ARIMA 모형에서의 p, d, q 의 의미

[이전 포스트](https://assaeunji.github.io/statistics/2021-08-23-arima/){:target='_blank'}에서도 소개했듯이 ARIMA (p,d,q)모형은 어떤 시계열을 d차 차분을 했을 때 AR (p) 모형과 MA (q) 모형이 합쳐진 ARMA (p,q) 모형을 따르는 형태입니다.
여기서
* **d차 차분**은 $y_t$에서 그 $d$만큼 과거 시점의 값인 $y_{t-d}$ 간의 차이를 계산한 것을 의미합니다. 
    
    $$
    y_t - y_{t-d}
    $$
    
    차분을 하는 목적은 차분한 시계열이 정상성 (stationarity)을 띠게 하여 시계열 예측을 가능케 하도록 함에 있습니다. (이 논의는 [시계열 분석 시리즈 (1): 정상성 (Stationarity) 뽀개기](https://assaeunji.github.io/statistics/2021-08-08-stationarity/){:target='_blank'}에서 자세히 확인하실 수 있습니다)
* **AR (p)모형**은 시계열 값이 과거 $p$시점만큼 앞선 시점까지의 값에 의존할 때 쓰는 모형으로, 현 시점의 값을 자기 자신의 과거의 값들로 설명하는 회귀모형을 의미합니다.
* **MA (q)모형**은 시계열 값이 과거 $q$시점만큼 앞선 시점까지의 오차에 의존할 때 쓰는 모형으로, 현 시점의 값을 과거의 오차 값들로 설명하는 회귀모형을 의비합니다. 오차는 "예측하기 어려운 shock"<sup>shock이란 갑자기 값이 평균에서 크게 벗어날 때 사용하는 시계열 용어</sup>로 해석이 가능하여, MA (q)는 과거의 shock이 현재 값에 영향을 줄 때 사용하는 모형입니다.

결국 ARIMA (p,d,q)는 이렇게 모형을 정의할 수 있습니다.

$$
\underbrace{y'_{t}}_{d차 차분값} = c + \underbrace{\phi_1 y'_{t-1} + \cdots + \phi_p y'_{t-p}}_{AR(p) 부분} + \underbrace{\theta_1 \epsilon_{t-1} + \cdots + \theta_q \epsilon_{t-q} + \epsilon_t}_{MA(q) 부분}
$$

여기서 $y'$는 차분을 구한 시계열입니다. (한 번 이상 차분을 구한 것일 수 있습니다) 또한 식을 자세히 보면 $p$차까지 AR처럼 과거의 $y'$값들이 들어가있고, $q$차까지 MA처럼 $\epsilon_t$가 전개되어있습니다. 따라서, 모수 $p, d, q$는 다음과 같은 의미를 같습니다.
* $p$: AR 부분의 차수
* $d$: 차분의 정도
* $q$: MA 부분의 차수

이를 후방 이동 연산자를 이용해서 식을 정리하면 다음과 같습니다.

$$
  (1-\phi_1B - \cdots - \phi_p B^p)(1-B)^d y_t = c + (1 + \theta_1 B + \cdots + \theta_q B^q)\epsilon_t
$$

예를 들어 $d=2$차 차분이라면 아래와 같이 전개가 됩니다.

$$
\begin{aligned}
  (1-\phi_1B - \cdots - \phi_p B^p)(1-B)^2 y_t &= c + (1 + \theta_1 B + \cdots + \theta_q B^q)\epsilon_t\\
   (1-\phi_1B - \cdots - \phi_p B^p) (1-2B+B^2) y_t &= c + (1 + \theta_1 B + \cdots + \theta_q B^q)\epsilon_t\\
   (1-\phi_1B - \cdots - \phi_p B^p) (y_t-2y_{t-1}+y_{t-2}) &= c + (1 + \theta_1 B + \cdots + \theta_q B^q)\epsilon_t\\
   \underbrace{(1-\phi_1B - \cdots - \phi_p B^p)}_{AR(p) 부분} \underbrace{((y_t- y_{t-1}) -(y_{t-1}+y_{t-2}))}_{2차 차분값} &= c + \underbrace{(1 + \theta_1 B + \cdots + \theta_q B^q)\epsilon_t}_{MA(q) 부분}\\
\end{aligned}
$$

---
## AIC: ARIMA 차수 선택과 모수 추정의 기준

자, 그럼 ARIMA 에는 p,d,q 라는 세 개의 차수가 존재하는 것까지 이해했습니다.
이제 이 p,d,q의 값을 적당히 잘 정해줘야하고,
이 각각에 대해 $c, \phi_1,\cdots, \phi_p$, $\theta_1,\cdots, \theta_q$ 회귀계수의 값을 추정해주어야 합니다.

이를 위해 AIC (Akaike’s information Criterion)<sup>아카이케 정보 기준</sup> 를 이용할 수 있습니다.
AIC 외에도 BIC, HQIC, OOB 등 다양한 정보 기준이 있지만, Python에서 기본값이 되는 AIC에 대해 한정하여 설명하겠습니다.

ARIMA에서 쓰는 AIC를 설명하기 전에 더 쉬운 예인 선형 회귀 모형을 생각해봅시다.

$y$를 예측하는데 필요한 변수 $x$들을 골라야 하는 경우 AIC를 이용하여 AIC를 가장 작게 하는 변수 조합으로 모형을 정하는데요. 예를 들어, 몸무게를 예측하는데 헬스 횟수, 음주 빈도, 하루 식단 칼로리와 같은 변수들이 후보로 있다면, 이런 식으로 변수 조합을 바꿔가면서 모형들을 세워볼 수 있습니다. 
* 몸무게 = a + b * 헬스 횟수 + c * 음주 빈도 + d * 하루 식단 칼로리
* 몸무게 = a + b * 헬스 횟수 + c * 음주 빈도 
* 몸무게 = a + b * 헬스 횟수 + d * 하루 식단 칼로리
* 몸무게 = a + c * 음주 빈도 + d * 하루 식단 칼로리
* $\vdots$

이 각 모형들에 대해 AIC라는 값을 구하고 AIC를 최소화하는 모형을 최종 모형으로 택할 수 있습니다. 
AIC는 아래와 같이 정의됩니다.

$$
AIC = -2\log (L) + 2p
$$

여기서 $\log(L)$은 로그 가능도, $p$는 추정된 모수의 개수를 의미합니다. **가능도 (Likelihood)**는 위에서 후보 모형들로 모형을 세웠을 때, 실제 관측된 데이터가 그 모형을 따를 확률을 의미합니다. 따라서, 관측한 데이터가 모형을 잘 따르고 잘 적합될수록 가능도는 큰 값을 가집니다. 

AIC의 식을 보면 $-2\log(L)$ 이라는 항 때문에 모형이 잘 적합될수록 AIC 값은 작아지고, $2p$라는 항 때문에 모형이 복잡할수록 AIC 값은 커집니다. 이 때 우리는 AIC를 최소화하는 모형을 고르는 것이기 때문에 **가능도를 최대화하는 동시에 모형은 좀 간단한** 모형을 고르게 됩니다. 여러 예측 변수들을 추가한 복잡한 모형이 간단한 모형보다 가능도가 높은 경향이 있는데, 복잡한 모형은 과적합하기 쉽습니다. 따라서 복잡한 모형을 지양하기 위해 불필요한 변수가 들어가면 AIC 값이 커지도록 페널티 $2p$를 부여하여 모델의 품질을 평가합니다.

이처럼 ARIMA (p,d,q)에서도 p, d, q를 바꿔가면서 $c, \phi_1,\cdots, \phi_p$, $\theta_1,\cdots, \theta_q$ 회귀계수의 값을 추정하고, 각 모형에 대해 AIC를 계산할 수 있습니다. ARIMA 모델에 대해 AIC는 다음과 같이 쓸 수 있습니다.

$$
\text{AIC} = -2\log(L) + 2(p+q+k+1)
$$

여기서 $k$는 ARIMA(p,d,q)모형의 상수항 여부에 따라 결정됩니다. 
상수항 $c$가 있으면 $k=1$, 없으면 $k=0$의 값을 갖습니다.

ARIMA 모형을 자세히 보면 AR(p)부분의 모수 $\phi_1,\cdots, \phi_p$ p개, MA(q)부분의 모수 $\theta_1,\ldots,\theta_q$ q개, 상수가 있을 경우 $c$라는 모수 1개, $\epsilon_t$에 대한 모수 1개이기 때문에 총 $p+q+k+1$개의 모수를 갖는 것을 확인하실 수 있습니다.


---
## `auto_arima` 과정 전에 할 일: 시각화

이처럼 AIC와 같은 정보 기준을 사용하여 가장 시계열 값을 잘 설명할 수 있는 차수 p, d, q와 회귀 계수들을 추정할 수 있습니다. 파이썬의 `pmdarima`모듈의 `auto_arima` (내지는 R의 `auto.arima` 함수)는 자동으로 p, d, q를 바꿔가면서 계수를 추정하고, 각 모형별로 정보 기준을 계산하여 최적의 값을 내는 모형을 계산합니다.

그러나 `auto_arima` 만 너무 맹신하면 큰 코 다칠 수 있습니다. 
* `auto_arima` 과정을 거치기 전에 시계열 자료를 시각화하는 과정과, 
* `auto_arima` 과정을 거친 후에 모델 적합 후의 잔차가 백색잡음인지 확인하는 과정. 
이 두 가지가 필수적입니다.이 과정을 거쳐 문제가 없으면 예측을 할 수 있는 것입니다. 

따라서 `auto_arima` 과정 전에 할 일인 "시각화"에 대해 먼저 알아보겠습니다.
시계열 자료를 시각화하는 것이 필요한 이유는 크게 두 가지입니다.

1. 데이터 상 특이한 관측치가 있는지? 패턴은 어떠한지? **추세 (trend) / 계절성 (seasonality) / 주기성 (cycle)** 이 눈으로 확인 가능한지? 에 대해 확인이 필요합니다. 만약 계절성이 있다면 `auto_arima`를 통해 적합시 `seasonal` 옵션으로 SARIMA (Seasonal ARIMA) 모형을 적합한다 명시해야할 수 있고, 주기가 있다면 `auto_arima`에서 `m` 옵션으로 주기를 명시해주어야 모형 적합을 잘 할 수 있기 때문입니다. 
2. 분산을 안정화해줄 필요가 있는지 파악해야합니다. 과거와 현재 시계열 자료의 분산이 다르다면 로그 변환 내지는 Box-Cox 변환을 고려해볼 수 있습니다.

그렇다면 시각화를 했을 때 어떤 시계열이 추세 / 계절성 / 주기성을 보일까요? 

* **추세**는 데이터가 장기적으로 증가하거나 감소할 때 추세가 존재합다 말합니다. 추세는 선형적일 필요는 없습니다.
    ![](../../images/arimapdq-trend.png)

* **계절성**은 해마다 어떤 특정한 때나 1주일마다 특정 요일에 나타나는 것 같은 계절성 요인이 시계열에 영향을 줄 때 계절성 패턴이 나타납니다. 계절성은 빈도의 형태로 나타내는데, 그 빈도는 항상 일정하다 알려져 있습니다.
    ![](../../images/arimapdq-seasonality.png)

* **주기성**은 고정된 빈도가 아닌 형태로 증가나 감소하는 모습을 보일 때 주기가 나타납니다. 아래처럼 빨간 박스 부분을 보면, 반복적으로 매 5년마다 비슷한 형태로 증가하는 모습을 보여 주기가 존재함을 알 수 있습니다.
    ![](../../images/arimapdq-cycle.png)

어떤 시계열이 분산이 다를까요?

![](../../images/arimapdq-ex.png)

위 그림에서 위쪽 그림을 자세히 보면 과거에 데이터가 변동하는 정도보다 현재에 더 큰 크기로 변동함을 알 수 있습니다. 그렇기 때문에 데이터의 분산을 안정화해주기 위해 아래쪽 그림처럼 로그 변환을 하였고, 그 결과 분산이 과거와 현재 간의 분산의 크기가 비슷함을 확인하실 수 있습니다.

---
## 자동화된 차수 선택 방법 (`auto_arima`) 의 원리

파이썬의 `auto_arima` (내지는 R의 `auto.arima` 함수)는 자동으로 p, d, q를 바꿔가면서 계수를 추정하고, 각 모형별로 정보 기준을 계산하여 최적의 값을 내는 모형을 계산합니다. 더 자세하게는 이 함수는 **힌드만-칸다카르 (Hyndman-Khandakar) 알고리즘**의 변형된 형태를 사용한다고 하는데요. `auto_arima`는 최적읜 ARIMA 모형을 얻기 위해 단위근 검정 <sup>정상성을 파악하기 위한 검정</sup>, AICc 최소화<sup>AICc는 수정된 형태의 AIC</sup>, MLE를 결합하여 사용합니다.

구체적인 **힌드만-칸다카르 알고리즘**은 다음과 같습니다.
1. 단위근 검정을 반복하여 차분 횟수 $0\le d \le 2$를 결정합니다.
2. 데이터를 $d$번 차분한 후 AICc를 최소화할 수 있는 $p$ 와 $q$를 고릅니다. 이 알고리즘에서는 $p$ 와 $d$의 모든 가능한 조합을 살펴보는 것이 아니라, 단계적 (stepwise) 탐색을 사용합니다.
   1. 4개의 초기 모델을 맞춥니다.
      * ARIMA (0,d,0)
      * ARIMA (2,d,2)
      * ARIMA (1,d,0)
      * ARIMA (0,d,1)
      $d=2$가 아니면 상수를 포함합니다.
   2. 2-1 단계에서 맞춘 가장 좋은 모델을 "현재 모델"로 둡니다.
   3. 현재 모형의 변형을 고려합니다.
      * 현재 모델에서 $p$와 $q$를 각각 $\pm 1$로 바꿔봅니다.
      * 상수 $c$를 현재 모델에 포함시켜보거나 제외해봅니다.
      * 현재 모델이나 변형된 모델 모두 고려하여 가장 좋은 모델이 새로운 현재 모델이 됩니다.
   4. 더 작은 AICc가 나오지 않을 때까지 2-3.을 반복합니다.

위 알고리즘을 이해해보면 다음과 같습니다.
1. 일단 ARIMA를 적합하려면 정상성을 띠어야하기 때문에 단위근 검정을 통해 알맞은 차분 횟수 $d$를 구해야 합니다. 시계열에서 차분을 했을 때 **단위근 검정**을 통해 정상성을 띠면 그것이 알맞은 차수가 될 것입니다.
2. **초기 모델**을 보면 $p$와 $q$의 값이 0~2 사이로 그리 크지 않음을 알 수 있습니다. **시계열 적합시 보통 $p+q <2, p\cdot q = 0$의 제약**만으로도 충분히 시계열 자료를 설명할 수 있다 합니다. 
3. 초기 모델을 변형시켜가면서 더 나은 모델은 없는지 AICc라는 **정보 기준**을 통해 최적의 모형을 찾습니다.


---
## 실전 ! 삼성전자 주가로 ARIMA 최적 모델 찾기

### Step 1. 시계열 자료 시각화

자, 이제 이론만 설명하면 너무 지겨우니 실제 삼성전자 주가로 `auto_arima`를 적용한 것에 대해 먼저 보여드리겠습니다.
이를 위해 20년 1월부터 21년 8월 30일까지의 일별 삼성전자 주가 데이터를 불러왔습니다.

```python
import FinanceDataReader as fdr
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
from pmdarima.arima import ndiffs
import pmdarima as pm


df_krx = fdr.StockListing('KRX') # 한국거래소 상장종목 전체
ticker = df_krx[df_krx.Name=='삼성전자']['Symbol'] # 티커
ss = fdr.DataReader(''.join(ticker.values),'2020-01-01', '2021-08-30')
```

삼성전자 종가를 시각화하면 다음과 같습니다.

```python
y_train = ss['Close'][:int(0.7*len(ss))]
y_test = ss['Close'][int(0.7*len(ss)):]
y_train.plot()
y_test.plot()
```

![](../../images/arimapdq-ssclose.png)

시각화를 보면서 느낀건 분산도 분산이지만, 20년 5월부터 21년 3월까지 강력한 상승 **추세**를 보인다는 것입니다. (이 때 주식을 샀어야 했는데...) 이런 추세형 그래프를 보고 들으면 좋은 생각은
* 정상성은 확실히 만족하지 않겠다 -> 차분이 꼭 필요하겠다 -> ARIMA의 차수 $d$가 1 이상이겠구나라 생각해볼 수 있고
* 상승 추세가 있기 때문에 상수항 $c$가 존재하는 확률 보행 모형이 아닐까? 라 예상해볼 수 있습니다. ($y_t = c + y_{t-1} + \epsilon_t$) 
* 또한, 계절성이나 주기성은 크게 보이진 않기 때문에 관련 모수는 auto_arima에 적용할 필요는 없겠다 예상해볼 수 있습니다.

정말 차분이 필요한지, 필요하다면 몇 차 차분이 최선인지 파악하려면 아래와 같이 `ndiffs` 함수를 이용할 수 있습니다.

```python
kpss_diffs = ndiffs(y_train, alpha=0.05, test='kpss', max_d=6)
adf_diffs = ndiffs(y_train, alpha=0.05, test='adf', max_d=6)
n_diffs = max(adf_diffs, kpss_diffs)

print(f"추정된 차수 d = {n_diffs}")
```

```python
> 추정된 차수 d = 1
```

분산 안정화가 필요한지의 여부는 모델이 너~무 적합이 안된다면 다시 시도해보도록 하겠습니다.

### Step 2. auto_arima 적용

자, 이제 모델 `auto_arima` 함수로 최적의 모형을 탐색해보겠습니다. 위에서 7:3의 비율로 train과 test 데이터를 나누었습니다.
train 데이터로 학습을 하고, test 데이터로 예측을 해봅니다.

```python
model = pm.auto_arima(y = y_train        # 데이터
                      , d = 1            # 필수는 아니나 있는게 좋음
                      , start_p = 0      # 생략 가능
                      , max_p = 3        # 생략 가능
                      , start_q = 0      # 생략 가능
                      , max_q = 3        # 생략 가능 
                      , m = 1            # 생략 가능 (Default)
                      , seasonal = False # 계절성 ARIMA가 아니라면 필수!
                      , stepwise = True  # 생략 가능 (Default)
                      , trace=True       # stepwise로 모델 적합할 때마다 결과를 프린트하고 싶을 때 사용
                      )
```

`auto_arima`에 주요한 옵션들을 설명하면 다음과 같습니다. 더 자세한 설명은 [공식 가이드](https://alkaline-ml.com/pmdarima/modules/generated/pmdarima.arima.auto_arima.html#pmdarima.arima.auto_arima){:target='_blank'}를 참조바랍니다.

* `y`: array 형태의 시계열 자료
* `d` (기본값 = none): 차분의 차수, 이를 지정하지 않으면 실행 기간이 매우 길어질 수 있음
* `start_p` (기본값 = 2), `max_p` (기본값 = 5): AR(p)를 찾을 범위 (start_p에서 max_p까지 찾는다!)
* `start_q` (기본값 = 2), `max_q` (기본값 = 5): AR(q)를 찾을 범위 (start_q에서 max_q까지 찾는다!)
* `m` (기본값 = 1): 계절적 차분이 필요할 때 쓸 수 있는 모수로 $m=4$이면 분기별, $m=12$면 월별, $m=1$이면 계절적 특징을 띠지 않는 데이터를 의미한다. `m=1`이면 자동적으로 seasonal 에 대한 옵션은 False로 지정된다.
* `seasonal` (기본값 = True): 계절성 ARIMA 모형을 적합할지의 여부
* `stepwise` (기본값 = True): 최적의 모수를 찾기 위해 쓰는 힌드만 - 칸다카르 알고리즘을 사용할지의 여부, False면 모든 모수 조합으로 모형을 적합한다.
* `trace` (기본값 = False): stepwise로 모델을 적합할 때마다 결과를 프린트하고 싶을 때 사용한다.

정말 간단하게 필수적인 옵션만 갖고 적합하면 아래와 같습니다. 
참고로 d의 값을 1로 둔 이유는, 위에서 `ndiffs`함수로 추정된 d의 값이 1이였기 때문입니다.

```python
model = pm.auto_arima (y_train, d = 1, seasonal = False, trace = True)
model.fit(y_train)
```

```
Performing stepwise search to minimize aic
 ARIMA(2,1,2)(0,0,0)[0] intercept   : AIC=4942.777, Time=0.13 sec
 ARIMA(0,1,0)(0,0,0)[0] intercept   : AIC=4934.821, Time=0.01 sec
 ARIMA(1,1,0)(0,0,0)[0] intercept   : AIC=4936.780, Time=0.02 sec
 ARIMA(0,1,1)(0,0,0)[0] intercept   : AIC=4936.799, Time=0.02 sec
 ARIMA(0,1,0)(0,0,0)[0]             : AIC=4934.428, Time=0.01 sec
 ARIMA(1,1,1)(0,0,0)[0] intercept   : AIC=4938.798, Time=0.15 sec

Best model:  ARIMA(0,1,0)(0,0,0)[0]          
Total fit time: 0.349 seconds
```

`auto_arima`를 사용한 결과 최적의 모델은 ARIMA (0,1,0) 모형으로 나왔습니다. 

이 모형을 풀어 말하면 1차 차분을 했을 때 백색 잡음 ($\epsilon_t \sim N(0,\sigma^2)$)임을 의미하고,
결국 아래 식처럼 **임의 보행 모형** (Random Walk Model)을 따른다는 것을 알 수 있습니다.

$$
\begin{aligned}
y_t - y_{t-1} &= \epsilon_t\\
y_t &= y_{t-1} + \epsilon_t
\end{aligned}
$$


또한 위 `Performing stepwise search to minimize aic` 부분을 자세히 보면 위에서 설명한 힌드만-칸다카르 알고리즘대로 초기 4개의 모델을 구성한 후에 하나씩 모수들을 바꿔가고, 
상수 (`intercept`)을 뺐다 껴보면서 stepwise 하게 차수를 찾고 있는 것을 확인할 수 있습니다.



이제 모형 summary 결과를 확인해봅시다.

```python
print(model.summary())
```

```
                              SARIMAX Results                                
==============================================================================
Dep. Variable:                      y   No. Observations:                  289
Model:               SARIMAX(0, 1, 0)   Log Likelihood               -2466.214
Date:                Tue, 07 Sep 2021   AIC                           4934.428
Time:                        22:29:59   BIC                           4938.091
Sample:                             0   HQIC                          4935.896
                                - 289                                         
Covariance Type:                  opg                                         
==============================================================================
                 coef    std err          z      P>|z|      [0.025      0.975]
------------------------------------------------------------------------------
sigma2      1.599e+06   9.63e+04     16.608      0.000    1.41e+06    1.79e+06
===================================================================================
Ljung-Box (Q):                       36.13   Jarque-Bera (JB):                46.72
Prob(Q):                              0.65   Prob(JB):                         0.00
Heteroskedasticity (H):               1.47   Skew:                             0.53
Prob(H) (two-sided):                  0.06   Kurtosis:                         4.67
===================================================================================
```

웬 SARIMAX라 할 수도 있겠지만, auto_arima가 비계절성 ARIMA 뿐 아니라 계절성 ARIMA (SARIMA)도 같이 지원하기 때문에 그렇습니다.
* ARIMA (0,1,0) 모형의 AIC는 4934.428로 가장 작았고, AR의 차수와 MA의 차수 p, q의 값이 모두 0이기 때문에 따로 회귀 계수에 대한 추정치들은 나오지 않았습니다.
* 또한 `sigma2`은 $\epsilon_t \sim N(0,\sigma^2)$ 부분에서 나온 $\sigma^2$의 추정치라 할 수 있습니다.
* Ljung-Box (Q) / Heteroskedasticity (H) / Jarque-Bera (JB)에 대한 부분은 **모두 잔차에 대한 검정 통계량**들입니다. 그리고 Prob (Q) / Prob (H) / Prob (JB)는 각 검정에 대한 p-value이구요!
  *  Ljung-Box (Q) <sup>융-박스 검정 통계량</sup>는 잔차가 백색잡음인지 검정한 통계량입니다.
  *  Jarque-Bera (JB) <sup>자크-베라 검정 통계량</sup>은 잔차가 정규성을 띠는지 검정한 통계량입니다.
  *  Heteroskedasticity (H) <sup>이분산성 검정 통계량</sup>은 잔차가 이분산을 띠지 않는지 검정한 통계량입니다.

---
## auto_arima 후에 할 일 1: 잔차의 백색 잡음 여부 및 정규성 확인

> 잔차에 대해서는 모형을 적합하고 남은 잔차가 1. 백색 잡음 (white noise)인지, 2. 정규성을 만족하는지를 확인해야 합니다.

위  Ljung-Box (Q) <sup>융-박스 검정 통계량</sup>에서 Prob (Q) 값을 보면 0.65이므로 유의수준 0.05에서 귀무가설을 기각하지 못합니다.
Ljung-Box (Q) 통계량의 귀무가설은 "잔차(residual)가 백색잡음(white noise) 시계열을 따른다"이므로, 위 결과를 통해 **시계열 모형이 잘 적합되었고 남은 잔차는 더이상 자기상관을 가지지 않는 백색 잡음**임을 확인할 수 있습니다.

또한, 위 Jarque-Bera (JB) <sup>자크-베라 검정 통계량</sup>에서 Prob(JB)값을 보면 0.00으로 유의 수준 0.05에서 귀무가설을 기각합니다.
Jarque-Bera (JB) 통계량의 귀무가설은 "잔차가 정규성을 만족한다"이므로, 위 결과를 통해 "잔차가 정규성을 따르지 않음"을 확인할 수 있습니다. 

또한, 잔차가 정규분포를 따른다면, 경험적으로
* 비대칭도 (Skew)는 0에 가까워야 하고 
* 첨도 (Kurtosis)는 3에 가까워야 합니다.

위 Summary 결과를 통해 비대칭도는 0.53으로 0에 가깝지만 첨도는 4.67로 3보다 더 높은 값을 가지고 있음을 알 수 있습니다.

이를 아래와 같이 그래프로도 확인하실 수 있습니다.

```python
model.plot_diagnostics(figsize=(16, 8))
plt.show()
```

![](../../images/arimapdq-residualplot.png)

잔차가 백색 잡음을 따르는지 보여주는 플랏은 (1,1)과 (2,2)에 위치한 그림입니다. (1,1)은 잔차를 그냥 시계열로 그린 것이고, (2,2)의 그림은 잔차에 대한 ACF입니다. 백색 잡음의 특성상 시계열이 평균 0을 중심으로 무작위하게 움직이는 것을 볼 수 있고, ACF도 허용 범위 안에 위치함을 알 수 있습니다.

잔차가 정규성을 만족하는지 보여주는 플랏은 (1,2)와 (2,1)에 위치한 그림입니다. (1,2)는 잔차의 히스토그램을 그려 정규 분포 N(0,1)과 밀도를 추정한 그래프를 같이 겹쳐서 보여줍니다. 위 비대칭도와 첨도에서 확인하셨던 것처럼 정규분포와 비슷한 평균을 갖지만, 첨도가 더 뾰족하게 솟아오른 것을 알 수 있습니다. (2,1)그래프는 Q-Q 플랏으로 정규성을 만족한다면 빨간 일직선 위에 점들이 분포해야 합니다. 그러나, 양 끝 쪽에서 빨간 선을 약간 벗어나는 듯한 모습을 보입니다.

**결과적으로 저희가 적합한 ARIMA (0,1,0)으로 남은 잔차는 백색 잡음이지만, 정규성은 따르지 않는다 볼 수 있습니다.**

그래서 어떻게 하냐! 글쎄요... 여기에는 정답은 없습니다.
저 같으면 일단 넘어가고 예측에 실패하면 다시 모형 설정을 바꿔서 테스트할 것 같습니다. 그래프를 봐도 알 수 있듯이 그렇게 티나게 정규분포를 벗어나는 것 같지 않아서이죠! (주관입니다)
예측에 실패한다면 위 설명 드린 `auto_arima` 모수인 p,d,q의 max를 늘려볼 수도 있고, train데이터를 더 최근만 가져올 수도 있고, 아니면 분산 안정화를 시도해볼 수도 있습니다.

---
## auto_arima 후에 할 일 2: 예측




That is, it fits a non-seasonal (that's the trailing (0,0,0)[0] part, and it's not surprising, since you specified seasonal=False) ARIMA(0,1,0) model. This is an ARMA(0,0) model on first differences, or

𝐵𝑦𝑡=𝑦𝑡−𝑦𝑡−1=𝜖𝑡,

where 𝐵 is the backshift operator, and 𝜖𝑡∼𝑁(0,𝜎2). Alternatively,

𝑦𝑡=𝑦𝑡−1+𝜖𝑡.

That is, a random walk.

In forecasting, you substitute the expected value for the innovations 𝜖𝑡, which is zero. Thus, your forecasts are simply the last observation. In particular, the forecasts do not vary over time, so you get a flat line.

Now you will probably wonder why auto_arima() fits a random walk. As Tim writes, there is no obvious cycles or trends in your data, and the stepwise AIC optimization does not find meaningful autocorrelation or moving average dynamics in your time series. So auto_arima() indeed believes a random walk is the best description of your data.

You may want to look through previous questions on flat ARIMA forecasts. Or at Is it unusual for the MEAN to outperform ARIMA? A flat forecast - whether from the overall average, as discussed in the last link, or whether from a random walk model - is surprisingly often the best forecast you can make. If there is no structure to be found, then there is no structure to be found.

---
## 실전! 삼성 주가 예측하기

