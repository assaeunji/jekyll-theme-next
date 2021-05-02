---
layout: post
title: "시계열 분석 시리즈 (1): AR / MA / ARIMA 모형"
date:
categories: []
tag: []
comments: true
photos:
    - "../../images/casual-title.png"
---

>  


* 이번 포스팅은 [실전 시계열 분석: 통계와 머신러닝을 활용한 예측 기법](http://www.yes24.com/Product/Goods/98576347) 책과 [Forecasting: Principles and Practice](https://otexts.com/fppkr/)책을 기반으로 AR, MA, ARIMA 모형을 정리하고자 합니다. 


---
## 본격적으로 들어가기 전 이것만은 알아두자.

통계적인 시계열 모형을 이용하려면, 항상 나오는 단어는 **자기 상관 (Autocorrelation)** , **정상성 (Stationarity)**, **백색 잡음 (White noise)**입니다. 이 들의 개념을 단단히 잡고 들어가야 시계열 모델들을 이해하는데 무리가 없을 것입니다.

### 시계열을 설명하는 세 가지: 추세, 계절성, 주기성

시계열을 설명하려면 **추세, 계절성, 주기성**에 대한 정의가 필요합니다.

* 추세 (trend): 데이터가 장기적으로 증가하거나 감소할 때 추세가 존재합니다. 주식 데이터가 대표적인 예이죠.
* 계절성 (seasonality): 해마다 어떤 특정한 때나 특정 요일에 나타나는 계절성 요인이 시계열에 영향을 줄 때 계절성이 존재합니다. 계절성의 빈도는 항상 일정하고 알려져 있습니다 (매 주 / 매 월 / 매 분기 등).
* 주기성 (cycle): 고정된 빈도가 아닌 형태로 증가하거나 감소하는 모습을 보일 때 주기가 나타납니다.

흔히들 주기적인 패턴과 계절적인 패턴을 혼동하지만, 사실 둘은 정말 다릅니다. 일정한 빈도로 나타나지 않는 요동은 주기적입니다. 빈도가 변하지 않고, 어떤 시기와 연관되어 있다면 그 요동은 계절성입니다.


![](../../images/arima-4patterns.png)

위 예제를 보고 각 데이터가 추세, 계절성, 주기성이 있는지 파악해볼까요?

* 미국 단독 주택 거래량: 매년 강한 계절성과 약 6 ~ 10년의 주기적 패턴을 보입니다. 그러나 전체 기간에 걸쳐 데이터가 증가하거나 감소하는 모습은 보이지 않기 때문에 추세가 있지는 않습니다.
* 미국 재무부 단기 증권 (treasury bill) 계약: 계절성 및 주기는 없지만, 아래로 내려가는 추세가 있습니다.
* 호주 분기별 전력 생산: 강한 계절성과 강한 증가 추세를 보입니다. 주기성은 보이지 않습니다.
* 구글 주식 종가 기준 일별 변동: 추세, 계절성, 주기성이 모두 없습니다.

### 자기 상관 (Autocorrelation)

상관 계수가 두 변수 사이의 선형 관계의 크기를 측정하는 것처럼, 자기 상관은 시계열에서 $y_t$와 $y_{t-k}$간의 선형 관계를 측정합니다. 이 두 값의 시차 (lag)는 $k$가 되고, **시차 $k$에서의 자기 상관 함수 (Autocorrelation function)** $r_k$는 다음과 같이 정의됩니다.

$$
r_k = \frac{\sum_{t=k+1}^T (y_t - \overline{y}) (y_{t-k} - \overline{y})}{\sum_{t=1}^T (y_t - \bar{y})^2}
$$

여기서 $T$는 전체 시계열의 길이입니다.

$X$와 $Y$의 상관 계수를 구할 때 공분산을 각자의 표준편차로 나눠주는 것처럼, 자기상관 함수도 $y_t$와 $y_{t-k}$ 간의 공분산을 각자의 표준편차로 나눠준 것과 같습니다.

즉, $\gamma(k) = Cov(y_t, y_{t-k})$로 $y_t$와 $y_{t-k}$의 **자기 공분산 함수 (Autocovariance function)**라 할 때, **자기 상관 함수 (Autocorrelation function)**는 다음과 같습니다.

$$
r_k = \frac{\gamma(k)}{\sqrt{Var(y_t)} \sqrt{Var(y_{t-k})}}
$$

여기서 분모에 $Var(y_t)$나 $Var(y_{t-k})$가 들어가는데, 분산은 자기 자신과의 공분산이므로 다음과 같이 나타낼 수 있습니다.
* $Var(y_t) = Cov (y_t, y_t)$
* $Var(y_{t-k}) = Cov(y_{t-k},y_{t-k})$

이때 자기 상관 함수가 $\gamma(k) = Cov(y_t, y_{t-k})$로 표현되므로, 
* $\gamma(0) = Cov(y_t, y_{t-0}) = Cov(y_t, y_t) = Var(y_t)$
* $\gamma(0) = Cov(y_{t-k}, y_{t-k-0}) = Var(y_{t-k})$
와 같이 표현됩니다.

결과적으로 위 식에서 분모는 $\gamma(0)$이 되어

$$
r_k = \frac{\gamma(k)}{\gamma(0)}
$$

로 단순화할 수 있습니다.

이 때 자기공분산 함수 $\gamma(k)$의 식은 다음과 같습니다.

$$
\gamma(k) = Cov(y_t, y_{t-k}) = E\left[ (y_t - \mu) (y_{t-k} - \mu) \right]
$$

이는 모집단에 대한 식이므로 표본에 대한 자기 공분산 함수 (Sample Autocovariance function)를 구하면 

$$
\gamma(k) = \frac{1}{T} \sum_{t=k+1}^T (y_t - \overline{y}) (y_{t-k} - \overline{y})
$$

가 되어, 결과적으로 자기 상관 함수는 이렇게 정의가 될 수 있는 것입니다.

$$
r_k = \frac{\gamma(k)}{\gamma(0)} =  \frac{\frac{1}{T} \times \sum_{t=k+1}^T (y_t - \overline{y}) (y_{t-k} - \overline{y})}{\frac{1}{T} \times \sum_{t=1}^T (y_t - \bar{y})^2} = \frac{\sum_{t=k+1}^T (y_t - \overline{y}) (y_{t-k} - \overline{y})}{\sum_{t=1}^T (y_t - \bar{y})^2} 
$$

**결과적으로 이 자기 상관 함수는 $y_t$와 $k$시점 전인 $y_{t-k}$ 간의 상관관계를 나타내는 식입니다.**
이 자기 상관 함수를 계산해 ACF 도표를 그리면 **시계열의 추세와 계절성**을 시각적으로 표현할 수 있습니다.






### 정상성 (Stationarity)



### 백색 잡음 (White noise)



--
## 자기 회귀 모델 (AR; Autoregressive Model)

자기 회귀 (AR; Autoregressive) 모델은 **과거가 미래를 예측한다**는 직관적인 사실에 의존합니다. 

가장 간단한 AR 모델인 AR(1) 모델은 다음과 같습니다.

$$
y_t = b_0 + b_1 y_{t-1} + \epsilon_t
$$

$t$ 시점에서의 값 $y_t$가 $y_{t-1}$의 회귀식으로 설명된다면, 이것이 AR(1) 모형입니다. 여기서 중요한 건 오차항 $\epsilon_t$는 **백색 잡음 (white noise)**이어야 합니다.

백색 잡음이란 자기 상관 (autocorrelation)이 없는 시계열