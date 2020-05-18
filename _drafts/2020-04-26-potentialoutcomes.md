---
layout: post
title: Causal Inference using Potential Outcomes
date: 2020-04-26
categories: [bayesian]
tag: [causal-inference, bayesian, potential-outcomes, probability,data-analysis]
comments: true
---

* Potential Outcomes는 인과 관계를 추론하는 방법 중 하나로, [이전 포스팅](../2020-04-10-causal)에서 설명했습니다. 
* 이번 포스팅에서는 Potential Outcomes의 구체적인 방법과 그 예제에 대해 설명드리고자 합니다. 
<!-- * 전체 코드와 데이터는 [이 곳](https://github.com/assaeunji/causal-bayesian-network)에서 확인하실 수 있고, [`R` 패키지 `bnlearn` tutorial](https://www.bnlearn.com/examples/useR19-tutorial/)를 주로 참조했음을 알립니다. -->

---

## Review

이전 포스팅에서 Potential outcomes에 대해 간단히 살펴봤는데요. 더 구체적인 예시로 설명해보고자 합니다.

인과 관계 추론의 목적은 처리 (Treatment) $Z$가 결과 (Outcome) $Y$에 미치는 효과를 추정하는 것입니다.
그러나 교란변수 (Confounder) $X$는 $Z$와 $Y$에 모두 영향을 주어 $Z$의 순수 효과를 추정하는 것을 방해합니다. 따라서 $Z\rightarrow Y$의 인과관계를 추정하기 위해서는 $X\rightarrow Z$의 종속 관계를 끊는 것이 중요합니다. 즉, 

$$
Z\perp \!\!\! \perp X
$$

무작위 통제 환경 (Randomized experiments)에서는 처리를 랜덤하게 배치함으로써 교란변수 $X$가 처리 $X$에 영향을 주지 않도록 (독립이도록) 만듭니다.

그러나 무작위 통제 환경이 아닌 관측 환경에서는 이렇게 처리를 랜덤하게 배치하는 것이 불가능합니다. 이미 처리를 받은 사람에게는 "처리가 일어나지 않았을 때의 결과"가 존재하지 않고 처리를 받지 않은 사람에게는 "처리가 일어났을 때 결과"를 얻을 수 없죠.
따라서, Potential outcomes들은 이런 **일어나지 않은 결과에 대한 추론**에 집중한 방법입니다.


이를 이해하기 위해 쉬운 예제를 [이 곳](https://www.slideshare.net/lumiamitie/causal-inference-primer-20190601)에서 가져왔습니다.


![](../../images/potentialoutcomes-ex.png)

다음과 같이 상황이 주어졌다 가정합시다.
케빈이 얼마나 이득/손해를 보았는 지 정확히 구하려면 
* 1년 전 그 날, 케빈이 주식을 사는 대신 **다른 선택**을 해보고 (ex. 적금, 비트코인...)
* 새로운 선택을 한 세계에서  1년 간 케빈이 벌어들인 수익을 구하고
* 이쪽 세계의 케빈이 삼성바이오로직스 주식을 벌어들인 수익/손해와 비교하는 것입니다.

즉 현실 케빈의 선택을 Y0, 저쪽 세계의 케빈의 선택을 Y1이라 하면,
케빈의 선택으로 인과적인 영향은 간단하게 **Y0-Y1**을 계산하면 구할 수 있습니다.

그러나 케빈은 1년 전 그 시간으로 돌아가서 다른 선택을 해볼 수가 없습니다. 
이런 상황을 **관측 환경** (Observational Studies)이라 합니다. 이런 환경에서는 관측된 결과만 확인할 수 있지만 인과관계를 추론하기 위해서는 **관측되지 않은 결과 (Potential Outcomes)** 을 알아야 합니다. 즉 케빈이 1년 전 삼성바이오로직스 주식 대신 다른 선택을 했을 때 벌어들인 수익이 바로 관측되지 않은 결과입니다.


![](../../images/potentialoutcomes-ex2.png)

따라서 관측되지 않은 결과를 채워넣는 게 Potential outcomes 방법론의 핵심입니다. 
또한 케빈 개인이 아니라 보통 전체 투자자의 효과를 알고자 하는 게 목적이므로 
Average Treatment Effect  (ATE)로 모든 사람의 효과를 구하고 이의 평균을 계산합니다.

> Potential Outcomes의 목표: Potential Outcomes을 추정해서 ATE 추정

그럼 Potential Outcomes의 방법은 어떤 게 있을까요?

1. 처리 배치 (Treatment Assignment)를 추정하는 것에 기반
    1. Matching: 가장 비슷한 데이터끼리
    2. Stratification: 가장 비슷한 집단끼리
    3. Inverse Probability Weighting

2. 반응 표면 (Response Surface)을 추정하는 것에 기반
    * Regression
    * BART(Bayesian Regression Additive Trees)

저는 이 포스팅에서 "처리 배치"를 추정하는 것에 기반하는 Matching과 Stratification, IPW에 맞춰 말씀드리고자 합니다.

---
## Matching

매칭 방법은 처리를 받은 사람들과 처리를 받지 않은 사람들 중 **비슷한 조건**을 가진 사람들끼리 짝지어 그 차이를 계산하는 방법입니다.


![](../../images/potentialoutcomes-matching.png)

위의 예시에서, 삼성바이오로직스 주식에 투자하지 않은 사람들 중에서 케빈과 가장 유사한 조건의 사람을 찾아서 비교하는 게 매칭의 핵심입니다. 

만약, 
* 케빈은 1년 전 삼성바이오로직스 주식을 구매해 2천만 원의 손실을 냈는데
* 케빈과 유사한 조건을 가진 친구 '케빙'이 1년 전 삼성바이오로직스 주식에 투자하지 않고 비트코인을 구매해 1천만 원의 수익을 냈다면 

삼성바이오로직스 주식의 인과 효과는 -3천만 원이 되는 식입니다.

따라서 $i$번째 데이터와 $j$번째 데이터 간의 **거리**를 재서 $\epsilon$보다 작으면 (즉, 거리가 가까운 사람들끼리) 매칭이 됩니다. 

$$
\begin{aligned}
Distance(X_i, X_j) &<\epsilon\\
Distance(X_i, X_j) &= \sqrt{(\vec{x}_i-\vec{x}_j)^\top S^{-1}(\vec{x}_i-\vec{x}_j)}
\end{aligned}
$$

매칭에도 여러 종류가 있는데 가장 기본적인 매칭의 거리 척도는 **Mahalanobis 거리**입니다.


---
## Stratification<sup>층화추출</sup>

매칭이 개인 단위에서 비슷한 데이터를 찾는 것이었다면, 층화추출은 **집단** 단위에서 비슷한 데이터를 찾아 집단 별로 차이를 계산합니다.







---
## References
 * 카카오 데이터분석가 이민호님 발표: Causal Inference [`[link]`](https://www.slideshare.net/lumiamitie/causal-inference-primer-20190601)
 * https://causalinference.gitlab.io/kdd-tutorial/methods.html
 * https://onedrive.live.com/View.aspx?resid=FB9A18AE325D3EFB!5374&wdSlideId=1800&wdModeSwitchTime=1587882572368&authkey=!APDx8SBOro95IR8
 * http://www.degeneratestate.org/posts/2018/Mar/24/causal-inference-with-python-part-1-potential-outcomes/