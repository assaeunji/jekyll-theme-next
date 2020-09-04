---
layout: post
title: GBM (Gradeint Boosting Machines) 에 대한 자세한 설명 
date: 2020-08-30
categories: [ML]
tag: [tree algorithm,supervised-learning, boosting, gbm]
comments: true
photos:
    - "../../images/tree-titleimage.jpg"

---

* 이번 포스팅은 나무 모형 시리즈의 세 번째 글입니다. 이전 글은 [AdaBoost에 대한 자세한 설명](https://assaeunji.github.io/ml/2020-08-14-adaboost/)과 [배깅 (Bagging)과 부스팅 (Boosting)의 원리](https://assaeunji.github.io/ml/2020-08-06-tree/)에서 확인하실 수 있습니다.
* GBM은 LightGBM, CatBoost, XGBoost가 기반하고 있는 알고리즘이기 때문에 해당 원리를 아는 것이 중요합니다.


---
## Introduction



---
## GBM의 원리

Gradeint Boosting Machines (이하 GBM<sup>Gradient Boosting Model</sup>로 통칭하겠습니다)는 AdaBoost와 마찬가지로 Boosting 계열의 알고리즘이기 때문에 순차적으로 약한 학습기 (weak learner)를 만들어 가며 이전 학습기의 잔차 (resuidual)를 보완하는 방식입니다.


이전 학습기의 잔차 (resuidual)를 줄여나가기 위해서
* AdaBoost는 이전에 잘못 분류한 데이터에 **가중치**를 더 많이 주고, 잘 분류한 학습기에 더 많은 가중치를 주어 해결하지만
* Gradient Boosting은 **gradient descent**를 이용해서 잔차 (resuidual)를 최소화

한다는 점에서 차이가 있습니다.


---
### Gradient Descent란?

그럼 gradient descent가 무엇인지 알아야겠죠?

![](../../images/gbm-gradient.png)

Gradient descent를 한국말로 풀면 **경사 하강법**입니다. 

경사하강법은 **미분 가능한 convex한 함수**, 즉 $y=x^2$과 같은 오목 함수에서 $y$가 최소값 갖게 하는 $x$를 찾기 위한 알고리즘입니다. (위의 그림에서는 $x$가 $w$로 표시돼 있네요!) 

이를 위해 Gradient Descent는 함수의 기울기 혹은 경사 (gradient)를 구하여 기울기가 낮은 쪽으로 $x$값을 계속 이동시켜서 $y$가 **극소값**을 갖도록 반복합니다.

참고로
* 극소값은 local minimum으로 가장 작은 함수값이라 말할 순 없지만 주변에 비해 작은 값
* 최소값은 global minimum으로 전체 범위에서 가장 작은 함수값

라는 점에서 차이를 보입니다.


구체적인 Gradient Descent 알고리즘은 다음과 같습니다.

초기값 $x^{(0)}$을 선택하고, 다음의 식을 반복적으로 실행해서 임의의 조건을 만족하면 종료하게 됩니다.

$$
x^{(t)} = x^{(t-1)} -\eta_t \Delta f(x^{t-1}),\ t=1,2,...
$$

이처럼 $t$번째 단계의 $x^{(t)}$는 이전 단계의 값 $x^{(t-1)}$에서 $\eta_t$와 $\Delta f(x^{(t-1)})$를 곱한 값을 빼주어 값을 지속적으로 업데이트합니다.

여기서 $\eta_t$는 학습률 (learning rate), 스텝의 크기 (step size)라 불리우는데, 한 번 업데이트할 때 얼마나 큰 크기로 업데이트할지 조정하는 값으로 사용자에 의해 지정되는 값입니다.

그리고 $\Delta f(x^{(t-1)})$이 바로 기울기 (gradient)에 해당합니다. 기울기는 함수를 **1차 미분**한 것과 같습니다.

이를 무한 반복하는 것은 아니고 $t$번째 값인 $x^{(t)}$와 $t-1$번째 값인 $x^{(t-1)}$ 번째 값이 **거의 같으면** 그 값이 수렴했음을 의미하기 때문에 반복을 멈춥니다. 

예를 들어 $f(x) = x^4 - 3x^3 +2$에서 극값을 찾기 위해 Gradient Descent를 구현해봅시다.

```python
x_t_1 = 0
x_t   = 6 #초기값
eta   = 0.01 #step size
precision = 1e-6 #정밀도

def delta_f(x):
    return 4 * x**3 - 9 * x**2

while abs (x_t - x_t_1) > precision:
    x_t_1 = x_t
    x_t = x_t_1 - eta * delta_f (x_t_1)

print (str(x_t))
```
여기서 `precision`부분은 "$x^{(t)}$와 $t-1$번째 값인 $x^{(t-1)}$ 번째 값이 **거의 같으면**" 이라는 조건을 수식화해 표현한 것입니다. 
이 두 값의 차가 `precision`값인 0.000001보다 작으면 반복을 멈추게 되어 있기 때문입니다.

---
### Boosting과 Gradient Descent의 관계

**자, 그래서 Gradient Descent는 알았는데... Boosting과는 어떻게 연관이 있는거죠?**

예측값과 실제 값의 차이를 나타내는 손실 함수 (Loss Function)를 MSE <sup>Mean Squared Error</sup>로 정의하면 이 손실함수의 1차 미분값 (gradient)의 음수를 취한게 바로 잔차 (residual)이 됩니다.

$$
\begin{aligned}
L(y_i, f(x_i)) &= (y_i - f(x_i))^2\\
L'(y_i,f(x_i))
\end{aligned}
$$



먼저 Boosting은 여러 개의 약한 학습기로 나온 예측값에 가중치를 주어 




---
### 



https://jinwonsohn.github.io/ml/nonparametric/2019/02/25/Boosting.html


https://3months.tistory.com/368

https://wikidocs.net/19037
