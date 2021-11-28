---
layout: post
title: "Dirichlet Distribution: Conjugate Prior for Multinomial Distribution"
date: 2019-01-05 
categories: [Bayesian]
tag: [dirichlet, bayesian, multinomial]
comments: true
---

## **Prior: Dirichlet Distribution**
디리슐레 분포 (Dirichlet dist.)는 다항분포 (Multinomial dist.)의 확률을 모델링하는 분포로, 베타분포의 확장판이다. 베타분포가 이항분포의 사전분포로 쓰여 켤레 관계를 형성하는 것처럼 multivariate 버전으로 디리슐레 분포가 다항분포의 *사전분포*로 쓰인다.
또한, 디리슐레 분포의 무한대 확장판 "디리슐레 과정"은 클러스터 개수를 조정하는 사전분포로 쓰인다. 이는 다음 포스트에서 설명할 것이다. 


 디리슐레 분포의 정의는 다음과 같다.

### Definition
$$G_1,G_2,\ldots, G_{k+1}$$개의 **독립**인 확률변수가 각각 $$\alpha_i$$와 $$\beta=1$$의 모수를 가진 감마분포를 따른다 하자. ($$i=1,\ldots,k$$). 즉,

$$
\begin{aligned}
G_1 &\sim \text{Gam}(\alpha_1, 1)\\
G_2 &\sim \text{Gam}(\alpha_2, 1)\\
&\vdots\\
G_k &\sim \text{Gam}(\alpha_k, 1)\\
G_{k+1} &\sim \text{Gam}(\alpha_{k+1}, 1)
\end{aligned}
$$

이때, $$\theta_i = \frac{G_i}{G_1+G_2+ \ldots +G_{k+1}}$$, $$\theta_{k+1} = \sum_{i=1}^{k+1} G_i$$ 라 정의하면, 다음의 성질을 만족한다.
* $$\theta_i = \frac{G_i}{\sum_{i=1}^{k+1}G_i}=\frac{G_i}{\theta_{k+1}},\ i=1,\ldots,k$$, $$\theta_{k+1} = \sum_{i=1}^{k+1} G_i$$
* $$(\theta_1,\theta_2,\ldots,\theta_k) \sim \text{Dirichlet} (\alpha_1,\alpha_2,\ldots,\alpha_k, \alpha_{k+1})$$, $$\theta_{k+1} \sim \text{Gam}(\sum_{i=1}^{k+1} \alpha_i,1)$$

 여기서 주의할 점은, $$k$$개의 확률 $$\theta_i$$ ($i=1,\ldots,k$)을 모델링하는 데 필요한 모수는 $$\alpha_1,\ldots, \alpha_{k+1}$$로 $$k+1$$개라는 점이다.

이에 대한 $$\boldsymbol{\theta}=(\theta_1,\ldots,\theta_k)$$의 결합 p.d.f.는 다음과 같다:

$$
\begin{aligned}
p(\theta_1\ldots,\theta_k|\alpha_1,\ldots,\alpha_k,\alpha_{k+1}) &= \frac{\Gamma(\alpha_1+\ldots+\alpha_k+\alpha_{k+1})}{\Gamma(\alpha_1)\cdots \Gamma(\alpha_k)\Gamma(\alpha_{k+1})} \theta_1^{\alpha_1-1}\cdots \theta_k^{\alpha_k-1}\left(1-\sum_{i=1}^{k} \theta_i \right)^{\alpha_{k+1}-1}\\ 
p(\boldsymbol{\theta}|\boldsymbol{\alpha})&=\frac{\Gamma(\sum_{i=1}^{k+1}\alpha_i)}{\prod_{i=1}^{k+1}\Gamma(\alpha_i)} \prod_{i=1}^{k}\theta_i^{\alpha_i-1} \propto \prod_{i=1}^{k}\theta_i^{\alpha_i-1}
\end{aligned}
$$

cf. 베타분포는 감마분포를 따르는 확률변수 $$2$$개로 구성된다. 즉, $$G_1,\ G_2$$가 각각 $$G_1\sim \text{Gam}(\alpha_1,1)$$, $$G_2 \sim \text{Gam}(\alpha_2,1)$$를 따른다 할 때, $$\theta = \frac{X_1}{X_1+X_2} \sim \text{Beta}(\alpha_1, \alpha_2)$$이다.

---
## **Likelihood: Multinomial Distribution**

다항분포도 디리슐레분포와 마찬가지로, 이항분포의 multivariate 버전이다. 즉, 2개의 선택지 대신 $$k$$개의 선택지가 있을 때, 각각 선택지가 $$\theta_i$$의 확률로 뽑힌다. 명확한 정의는 다음과 같다.

### Definition
확률 벡터 $$\mathbf{X} = (X_1,\ldots,X_k)$$가 모수 $$n$$과 $$\boldsymbol{\theta} = (\theta_1,\ldots, \theta_k)$$를 가진 다항분포일 때, 다음의 p.d.f를 가진다:

$$
f(\boldsymbol{x}|n,\boldsymbol{\theta}) = \binom{n}{x_1,\ldots, x_k} \theta_1^{x_1} \cdots \theta_k^{x_k} = \binom{n}{x_1,\ldots, x_k} \prod_{i=1}^{k} \theta_i^{x_i} 
$$
* $$\sum_j p_j = 1$$

---
## **Conjugacy: Dirichlet & Multinomial Distribution**
디리슐레 분포(Dirichlet distribution)는 다항분포 (Multinomial distribution)의 확률 $$\theta_i$$를 모델링하는데 쓰이는 사전분포이다. 여기서 Conjugacy가 성립한다.  그렇다면 여기서 conjugacy를 어떻게 해석해야 할까? 사전분포가 디리슐레 분포이고, 가능도가 다항분포이면 사후분포도 다시 디리슐레 분포를 따를 때, (즉, 사전분포와 사후분포의 형태가 같을 때) "켤레성"(conjugacy)을 띤다고 말한다.

---

## **Posterior: Dirichlet Distribution**
사후분포 (posterior)는 베이즈 정리에 따라 사전분포 (prior)와 가능도 (likelihood)의 곱에 비례한다. 여기서의 사전분포와 가능도는 앞에서 정의한 것과 같이 
* 사전분포: $$k$$개의 선택지에 대한 확률 $$\theta_i$$의 분포인 **디리슐레 분포**

$$
p(\boldsymbol{\theta}|\boldsymbol{\alpha}) = \prod_{i=1}^k \theta_i^{\alpha_i-1}
$$

* 가능도: $k$개의 선택지 중 $i$번째 선택지를 뽑는 횟수 $$X_i$$의 분포인 **다항 분포** 

$$
p(\mathbf{X}|\boldsymbol{\theta}) = \prod_{i=1}^{k} \theta_i^{x_i}
$$

이다. 따라서 사후분포를 도출하면,

$$
\begin{aligned}
\text{Posterior} &\propto \text{Likelihood}\cdot \text{Prior}\\
P(\boldsymbol{\theta}|\mathbf{X}, \boldsymbol{\alpha}) &\propto p(\mathbf{X}|\boldsymbol{\theta}) \cdot p(\boldsymbol{\theta}|\boldsymbol{\alpha})\\
&\propto \prod^k_{i=1} \theta_i^{\alpha_i+x_i-1}\\
&\sim \text{Dirichlet}(\alpha_1+x_1, \cdots, \alpha_k+x_k)
\end{aligned}
$$

의 결과를 도출할 수 있다.