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

* 이번 포스ㅇㅔㅓㅡ 
* Python 실습을 위해 또 삼성전자의 주가를 가져와서 ARIMA로 예측해보았습니다. (~~응또삼; 응지 또 삼성전자 주가...~~)
* 역시 [실전 시계열 분석: 통계와 머신러닝을 활용한 예측 기법](http://www.yes24.com/Product/Goods/98576347){:target='_blank'} 책과 [Forecasting: Principles and Practice](https://otexts.com/fppkr/){:target='_blank'}책을 기반으로 글을 작성하였으며 ARIMA 의 이해가 부족하시다면 이전 포스트인 [시계열 분석 시리즈 (3): auto_arima를 잘 쓰기 위한 배경 지식](https://assaeunji.github.io/statistics/2021-09-08-arimapdq/){:target='_blank'}를 참고 바랍니다.


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

