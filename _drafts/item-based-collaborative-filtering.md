---
layout: post
title: "추천 시스템 (1): 아이템 기반 협업 필터링으로 영화 추천하기"
date: 2020-09-02
categories: []
tag: []
comments: true
---

* 이번 포스팅에서는 추천 시스템 중 아이템 기반의 협업 필터링에 대해 설명하고, MovieLens 데이터를 통해 손코딩 + Surprise 패키지를 통해 아이템 기반 협업 필터링에 대해 실습해보고자 합니다.
* (요즘 글쓰는 순서가 뒤죽박죽이네요... **시리즈 글**을 이어서 쓰는 건 참 어려운 것 같습니다. 끌리는대로! 씁니다 ㅎㅎ) 

---
## 추천 시스템 (Recommendation System) 개요





---
### 추천 시스템 (Recommendation System)의 유형

추천시스템은
* 콘텐츠 기반 필터링 (Contents based Filtering): 사용자가 특정 아이템을 매우 선호할 때, 그 아이템과 비슷한 콘텐츠를 가진 다른 아이템을 추천
* 협업 필터링 (Collaborative Filtering): 사용자가 매긴 평점이나 구매 이력으로 사용자의 행동 양식을 바탕으로 추천을 수행하는 방식을 의미
  * 최근접 이웃 기반 협업 필터링 (Nearest Neighbor Collaborative Filtering)
    * 사용자 기반 협업 필터링: 특정 사용자와 유사한 다른 사용자 N명을 뽑아 그들이 선호하는 아이템 추천
    * <mark style='background-color: #fff5b1'>아이템 기반 협업 필터링</mark>: 사용자들이 아이템을 좋아하는지/ 싫어하는지의 평가척도가 유사하는 아이템을 추천
  * 잠재 요인 협업 필터링 (Latent Factor Collaborative Filtering) -- 넷플릭스: 사용자 - 아이템 평점 매트릭스 속에 숨어있는 잠재요인을 추출해 추천 예측
으로 나뉩니다.


---
## 아이템 기반 협업 필터링을 예시로 알아보자

Movie Lens 데이터



---
## 손코딩


---
## Python Surprise 패키지