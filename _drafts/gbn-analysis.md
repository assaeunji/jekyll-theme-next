---
layout: post
title: Causal Inference using Gaussian Bayesian Network with Example
date:
categories: []
tag: []
comments: true
---

* Gaussian Bayesian Network는 인과 관계를 추론하는 방법 중 하나로, [이전 포스팅](../2020-04-10-causal.md)에서 설명했습니다. 
* 이번 포스팅에서는 `R`의 `bnlearn`패키지를 이용해 부정교합 데이터를 Gaussian Bayesian Network로 분석한 결과에 대해 쓰고자 합니다. 
전체 코드와 데이터는 [여기]()에서 확인하실 수 있고,  `R` 패키지 `bnlearn` tutorial [`[link]`](https://www.bnlearn.com/examples/useR19-tutorial/)를 주로 참조했음을 알립니다.

---
## Recap: Gaussian Bayesian Network

[이전 포스팅](../2020-04-10-causal.md)에서 말씀드렸듯이 Bayesian Network 중 Gaussian Bayesian Network는 노드가 정규분포를 따를 때 쓸 수 있는 방법입니다.

또한 Bayesian Network는 1. 인과 구조를 학습하고 그 구조를 바탕으로 2. 모수를 추정하는 두 단계로 이루어져 있습니다. 

---
### Parameter Tuning

먼저, 쉬운 **2. 모수 추정** 방법에 대해 먼저 설명드리겠습니다.
Gaussian Bayesian Network에서는 $i$번째 노드인 $X_i$가 다음의 구조를 따른다 가정합니다.

   $$
   X_{i}=\mu_{X_{i}}+\Pi_{X_{i}} \boldsymbol{\beta}_{X_{i}}+\varepsilon_{X_{i}}, \quad \varepsilon_{X_{i}} \sim N\left(0, \sigma_{X_{i}}^{2}\right)
   $$

여기서 $\Pi_{X_{i}}$는 $X_i$의 직계 부모 집합 (a set of direct parents)을 의미합니다. 

만약 $X_1, X_2, X_3$의 노드에서 인과 관계가 $X_1 \rightarrow X_3$와 $X_2 \rightarrow X_3$와 같다면, 

$$
X_3 = \mu_{X_{3}}+\beta_1 X_1 + \beta_2 X_2 +\varepsilon_{X_{3}}, \varepsilon_{X_{3}} \sim N(0,\sigma^2_{X_{3}})
$$

과 같이 직계 부모인 $\Pi_{X_{3}}=X_1, X_2$들이 설명 변수로, 아이인 $X_3$이 반응 변수로 두고 선형 회귀식을 적합합니다. 회귀 계수의 추정은 MLE<sup>Maximu Likliehood Estimation</sup>을 통해 추정합니다.


---
### Causal Structure Learning

1. **인과 구조를 학습**하는 과정은 학습 방법에 따라 크게 세 가지 방법이 있습니다.
* **Score based**: Hill Climbing Algorithm,
* **Constraint based**: Grow-Shrink Markov Blanket, Incremental Association, Fast Incremental Association, Interleaved Incremental Association
* **Hybrid**: Max-Min Hill Climbing

**Score based**는 약간 heuristic한 방법인데요. 선형회귀에서 단계 별로 변수를 선택할 때 AIC나 BIC를 기준으로 최적의 모형을 선택하듯이 score-based algorithms도 네트워크에서 한 연결 (edge)을 끊어보기도, 붙여보기도 하면서 각 DAG의 score를 비교하는 방식입니다. 마찬가지로 score도 BIC를 사용합니다.

**Constraint based**는 조건부 독립 검정 (CI; Conditional Independence test)을 통해 인과 관계를 파악하는 알고리즘입니다.

마지막으로 **Hybrid**은 score based와 constraint based를 적당히 섞은 알고리즘입니다. 

저는 여기서 가장 기본이 되는 Score based의 Hill Climbing Algorithm과 Constraint based의 Grow-Shrink Markov Blanket Algorithm에 대해 알아보겠습니다.

---
### Hill Climbing Algorithm





---
## 데이터 톺아보기






---
## References

* `R` 패키지 `bnlearn` tutorial [`[link]`](https://www.bnlearn.com/examples/useR19-tutorial/)
* `R` 패키지 `visNetwork` tutorial[`[link]`](https://datastorm-open.github.io/visNetwork/)
* D. Margaritis. Learning Bayesian Network Model Structure from Data. PhD thesis, School of Computer Science, Carnegie-Mellon University, Pittsburgh, PA, May 2003. Available as Technical Report CMU-CS-03-153. [`[pdf]`](https://www.cs.cmu.edu/~dmarg/Papers/PhD-Thesis-Margaritis.pdf)