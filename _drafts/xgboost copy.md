---
layout: post
title: "GBM (Gradient Boosting Machines) 에 대한 자세한 설명 (2): Classification"
date: 2020-08-30
categories: [ML]
tag: [tree algorithm,supervised-learning, boosting, gbm,xgboost, lightgbm]
comments: true
photos:
    - "../../images/tree-titleimage.jpg"
---

* 오늘은 [GBM에 대한 자세한 설명 (1): Regression](https://assaeunji.github.io/ml/2020-09-05-gbm/)편에 이어 GBM을 분류 (Classification)에 활용할 때 어떤 원리로 쓰이는지 알아보고자 합니다. 
* 이 또한 StatQuest with Josh Starmer YouTube "Gradient Boost Part 3: Classification" [link](https://www.youtube.com/watch?v=jxuNLH5dXCs)를 참조했습니다.

----
## Introduction 

GBM을 이용한 분류는 로지스틱 회귀와 비슷한 점이 많습니다.

1. Initial Prediction: log (odds) = log (4/2) = 0.69
   1. log odds를 로지스틱 함수에 넣으면
   2. P(love troll2) = e^log(odds) / (1+e^log(odds)) = 0.67
   3. 0.5보다 크기 때문에 모든 사람을 troll2를 모두 좋아한다고 분류한다.

2. Pseudo-Residual을 계산해서 얼마나 초기 예측이 틀렸는지를 계산
   1. (Observed - Predicted)
   2. y = p(loving troll 2) = .7
   3. Red, Blue = observed
   4. dot of line = predicted
   5. residual = 0.3 or -0.7
3. Tree 를 파봌ㄴ, 나무, 좋아하는 컬러로 tree를 만든다
4. sum residuals / sum (previous probaability * (1-p_i))