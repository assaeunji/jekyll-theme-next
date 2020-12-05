---
layout: post
title: XGBoost: Gradient Boost Machine
date: 2020-08-30
categories: [ML]
tag: [tree algorithm,supervised-learning, boosting, gbm,xgboost]
comments: true
photos:
    - "../../images/tree-titleimage.jpg"

---

## Introduction 

XGBoost는 eXtra Gradeint Boost의 준말
장점
* 뛰어난 예측 성능
* GBM 대비 빠른 수행 시간
* 다양한 성능 향상 기능
  * 규제 기능 탑재
  * Tree Pruning (가지치기)
* 다양한 편의 기능
  * 조기 중단 (Early Stopping):약한 학습기 개수만큼 돌아야 정상인데 더이상 성능 개선이 나지 않는다면 자체 중단
  * 자체 내장된 교차 검증
  * 결손값 처리


C++로 작성 -> 파이썬 Wrapper -> scikit-learn Wrapper

* XGBClassifier
* XGBRegresssor
* 학습과 예측을 fit, predict

## 하이퍼 파라미터 비교

그리 중요하지 않음

| 파이썬 Wrapper  | Scikit-Learn Wrapper  | 설명  |
|:---:|:---:|:---:|
|eta|learnig_rate| GBM 학습률. 0~1의 값|
|num_boost_rounds|n_estimators|약한 학습기 개수 (반복 수행 횟수)|
|min_child_weight|min_child_weight| 과적합 조절을 위해 사용되는 것으로, 트리에서 추가로 가지를 나눌지를 결정하기 위해 필요한 데이터들의 weight의 총합. 클수록 분할을 자제|
|max_depth|max_depth||
|sub_sample|subsample||
|lambda|reg_lambda|L2 규제 적용 값. 값이 클수록 규제값이 커짐|
|alpha|reg_alpha|L1 규제 적용 값. 값이 클수록 규제값이 커짐|
|colsample_bytree|colsample_bytree|GBM의 max_feature와 비슷, 트리 생성에 필요한 피처를 임의로 샘플링하는 데 사용|
|scale_pos_weight|scale_pos_weight|특정값으로 치우친 비대칭한 클래스를 구성된 데이터세트의 균형을 유지하기 위한 파라미터|
|gamma|gamma|트리의 리프 노드를 추가적으로 나눌지 결정한 최소 손실 감소 값. 해당 값보다 큰 손실이 감소된 경우에 리프 노드를 분리|

## XGBoost 조기 중단 기능 (Early Stopping)

* XGBoost는 특정 반복횟수만큼 더 이상 비용함수가 감소하지 않으면 지정된 반속횟수를 다 완료하지 않고 수행을 종료할 수 있음
* early_stopping_rounds: 더 이상 비용 평가 지표가 감소하지 않는 최대 반복횟수
* eval_metric: 반복 수행 시 사용하는 비용 평가 지표
* eval_set: Validation Set


## 


https://brunch.co.kr/@snobberys/137

https://arxiv.org/abs/1603.02754

https://xgboost.readthedocs.io/en/latest/

