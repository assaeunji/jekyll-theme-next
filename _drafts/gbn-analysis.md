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
## Recap: GBN(Gaussian Bayesian Network)

[이전 포스팅](../2020-04-10-causal.md)에서 말씀드렸듯이 Bayesian Network 중 GBN는 노드가 정규분포를 따를 때 쓸 수 있는 방법입니다.

GBN은 데이터로부터 1. 인과 구조를 학습하고 그 구조를 바탕으로 2. 모수를 추정하는 두 단계로 이루어져 있습니다. 

---
### Parameter Tuning

먼저, 쉬운 **2. 모수 추정** 방법에 대해 먼저 설명드리겠습니다.
GBN은 $i$번째 노드인 $X_i$가 local하게 정규분포를 따른다 가정합니다.

   $$
   X_{i}=\mu_{X_{i}}+\Pi_{X_{i}} \boldsymbol{\beta}_{X_{i}}+\varepsilon_{X_{i}}, \quad \varepsilon_{X_{i}} \sim N\left(0, \sigma_{X_{i}}^{2}\right)
   $$

여기서 $\Pi_{X_{i}}$는 $X_i$의 직계 부모 집합 (a set of direct parents)을 의미합니다. 

만약 $X_1, X_2, X_3$의 노드에서 인과 관계가 $X_1 \rightarrow X_3$와 $X_2 \rightarrow X_3$와 같다면, 

$$
X_3 = \mu_{X_{3}}+\beta_1 X_1 + \beta_2 X_2 +\varepsilon_{X_{3}}, \varepsilon_{X_{3}} \sim N(0,\sigma^2_{X_{3}})
$$

과 같이 직계 부모인 $\Pi_{X_{3}}=X_1, X_2$들이 설명 변수로, 아이인 $X_3$이 반응 변수로 두고 선형 회귀식을 적합합니다. 회귀 계수의 추정은 MLE<sup>Maximum Likelihood Estimation</sup>을 통해 추정합니다.


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

Hill Climbing 알고리즘은 Score-based 의 방법 중 하나로, BN이 얼마나 데이터 셋 $\mathcal{D}$를 표현하고 있는 지를 점수로 표현합니다. DAG 구조를 $\mathcal{G}$라 가정할 때, 베이즈 정리에 따라 score는 다음과 같이 정의됩니다.

$$
\operatorname{BICscore}(G, \mathcal{D})=\log \operatorname{Pr}(\mathcal{D} | G)-\frac{d}{2} \log N
$$

여기서 $\log \operatorname{Pr}(\mathcal{D} | G)$는 $\mathcal{G}$의 인과 구조를 가졌을 때의 로그 가능도이고 $d$는 free parameter의 개수입니다. GBN 노드 각각의 분포는 정규분포이니 가능도 또한 정규분포이겠네요! 이 식에서 알 수 있듯이 BIC<sup>Bayesian Information Criteria</sup>는 높을수록 좋습니다. (참고로, 원래 회귀에서 쓰던 BIC는 로그 가능도에 "-"가 붙어있고 penalty는 양수여서 작을수록 좋습니다.)


이제 이 점수들을 갖고 Hill Climbing 알고리즘은 다음과 같은 순서로 진행합니다.

![](../../images/gbn-hillclimbing2.png)

위는 pseudo code인데 순서에 맞게 설명해보겠습니다.

1. $\mathcal{E}$는 초기 네트워크로, 비었거나 (연결 관계가 없거나) 아예 꽉 차있던가 아니면 랜덤하게 네트워크를 구성
2. $ProbabilityTable()$은 $\mathcal{E}$에서 만든 인과 구조 하에서 노드마다 local pdf를 구하고 MLE를 통해 모수들을 추정 
3. 첫 BN $\mathcal{B}$를 구성 (여기서 $\mathcal{U}$는 노드 집합)
4. `score` 초기화
5. 반복:
   1. maxscore를 score로 둠
   2. BN의 쌍 $(X,Y)$에 대해서 반복:
      1. 네트워크 구조 $\mathcal{E}$에서 
         * $\{X\rightarrow Y\}$를 추가해 $\mathcal{E}'$에 저장하거나
         * $\{X\rightarrow Y\}$를 제거해 $\mathcal{E}'$에 저장하거나
         * $\{X\rightarrow Y\}$ 구조를 $\{Y\rightarrow X\}$로 반대로 바꿔 $\mathcal{E}'$에 저장
         * $\mathcal{T}'$에 새롭게 구한 $\mathcal{E}'$로 다시 local pdf와 모수를 추정해 저장
      2. 새로운 BN $\mathcal{B}'$를 저장
      3. $\mathcal{B}'$를 가지고 BIC 점수를 갱신해 `newscore`에 저장
      4. 만약 `newscore`가 기존의 `score`보다 높다면 
         * $\mathcal{B}'$로 $\mathcal{B}$를 갱신하고, 
         * `score`도 `newscore`로 갱신
6. `newscore`가 더이상 갱신되지 않으면 중단
7. $\mathcal{B}$ 출력

핵심은 각 쌍 $(X,Y)$마다 인과관계를 추가해보거나 삭제해보거나 역으로 만들어보는 과정입니다. 이를 그림으로 나타내면 더 이해하기 쉽습니다.

![](../../images/gbn-hillclimbing.png)

하나의 인과 구조에서 한 엣지씩 붙여보고, 잘라보고, 바꿔보는 과정을 반복해 인과구조를 완성합니다.

---
### Grow-Shrink Markov Blanket Algorithm

제가 저번 포스팅에서 Markov Blanket에 대해 설명은 했지만 왜 이게 중요한 지 잘 이해가 안 갔었는데요. 중요한 이유가 이 것 때문이었습니다!


---
## 데이터 톺아보기






---
## References

* `R` 패키지 `bnlearn` tutorial [`[link]`](https://www.bnlearn.com/examples/useR19-tutorial/)
* `R` 패키지 `visNetwork` tutorial[`[link]`](https://datastorm-open.github.io/visNetwork/)
* D. Margaritis. Learning Bayesian Network Model Structure from Data. PhD thesis, School of Computer Science, Carnegie-Mellon University, Pittsburgh, PA, May 2003. Available as Technical Report CMU-CS-03-153. [`[pdf]`](https://www.cs.cmu.edu/~dmarg/Papers/PhD-Thesis-Margaritis.pdf)