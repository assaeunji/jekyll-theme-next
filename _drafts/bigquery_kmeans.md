---
layout: post
title: 10분 만에 구현하는 BigQuery K-평균 클러스터링 모델
date: 2020-12-09
categories: [DB]
tag: [sql, db, cloud, BigQuery, k-means, clustering, machine-learning,ml, unsupervised-learning, easy-guide]
comments: true
---

![](../../images/bigquery_intro.png)
BigQuery ML에서는 표준 SQL 쿼리를 사용하여 BigQuery에서 머신러닝 모델을 만들고 실행할 수 있습니다. (참조: [`[link]`](https://cloud.google.com/bigquery-ml/docs/introduction?hl=ko))
{:.figure}

* 사진에서 볼 수 있듯이 이번 포스팅에서는 쉽게 BigQuery ML <sup>줄여서 "BQML"</sup>를 이용해 K-평균 클러스터링을 구현하는 방법에 대해 알아보고자 합니다.
* 자세한 내용은 [BigQuery ML k-평균 클러스터링 모델 만들기](https://cloud.google.com/bigquery-ml/docs/kmeans-tutorial?hl=ko) 페이지를 참조해주시길 바랍니다.
* 또한 성윤님의 [BigQuery의 모든 것 입문편](https://www.slideshare.net/zzsza/bigquery-147073606)도 보시는 것을 추천합니다!
  
----
## Introduction

요즘 빅쿼리를 사용할 일들이 종종 있어 "배워야지"하기만 하고 아직 실천을 하지 못했는데요. 최근에 또 [구글 빅쿼리 완벽 가이드](https://www.aladin.co.kr/m/mproduct.aspx?itemid=256496408)도 샀겠다, 구글 빅쿼리 강의도 ~~흘려~~ 들었겠다 하여 한 번 공부해볼까?하는 마음에 구글 빅쿼리 튜토리얼을 읽게 되었습니다.

그 도중 [BigQuery ML k-평균 클러스터링 모델 만들기](https://cloud.google.com/bigquery-ml/docs/kmeans-tutorial?hl=ko)가 딱 눈에 띄여서 한번 실습해봤는데, 생각보다 쉽게 할 수 있어서 이를 공유해야겠다는 기쁜 마음에 이 글을 쓰게 되었습니다.

기존에 SQL 언어에 익숙하신 분이라면 어렵지 않게 따라할 수 있을 것으로 보입니다.

---
## GCP 새 프로젝트 만들기

전 이미 프로젝트가 있었지만, 가이드를 위해 Google Cloud Platform <sup>GCP</sup> 콘솔 [`[link]`](https://console.cloud.google.com/home)에 들어가서 새 프로젝트를 생성해보았습니다.

먼저, 좌측 상단에서 `프로젝트 선택`을 클릭하고 `새 프로젝트`를 눌러 프로젝트를 생성합니다.

![](../../images/bigquery_first.png)

그럼 프로젝트 이름을 적당히 짓고 만들기를 클릭하면 프로젝트가 생성됩니다.
생성된 프로젝트에 가면 대시보드 형태로 내 프로젝트 정보를 볼 수 있는데요. 저희는 BigQuery를 이용하기 때문에 좌측 바에서 `빅데이터 > BigQuery`를 클릭해 편집기 창으로 이동합니다.

![](../../images/bigquery_new.png)


---
## 런던 자전거 공개 데이터세트 불러오기

BigQuery에서는 여러 데이터세트를 공개하고 있는데 이번 포스팅에서는 런던 자전거 데이터 세트를 이용해 K-means를 구현할 것입니다.

이를 위해서, 먼저 탐색기에서 **데이터 추가**를 누르고 **공개 데이터세트 탐색하기**를 눌러 "London Bicycle Hires"을 검색해 클릭합니다.

![](../../images/bigquery_dataset.png)

데이터세트를 불러오면 편집기의 탐색 창에서 bigquery-public-data라는 DB가 생성되었음을 볼 수 있습니다. 이 DB를 클릭해 스크롤을 내려서 `london_bicycles` 데이터세트가 있다면 잘 불러온 것입니다!

![](../../images/bigquery_folder.png)

---
## 모델 저장할 데이터세트 만들기

BigQuery로 K-평균 클러스터링 모델을 만드려면 이전에 모델을 저장할 데이터세트를 따로 만들어야 합니다. 이를 위해 다시 탐색기 창에서 프로젝트 ID 명을 가진 DB를 클릭한 후 (제 경우는 `assaeunji`) 우측의 **데이터세트 만들기**를 클릭합니다.

![](../../images/bigquery_datasetload.png)

다음과 같이 
* 데이터세트 ID에 bqml_tutorial, 
* 데이터 위치에서 유럽 연합(EU)을 선택합니다. 런던 자전거 공유 공개 데이터세트는 EU 멀티 리전 위치에 저장되기 때문에 데이터세트도 같은 위치에 있어야 합니다.

![](../../images/bigquery_makedataset.png)

다른 기본 설정은 그대로 두고 데이터세트 만들기를 클릭합니다.

----
## 데이터 톺아보기

저희가 사용할 데이터는 런던 자전거 데이터로, 다음 속성을 이용해 자전거 정거장을 클러스터링하는 것이 목표입니다.
* 대여 기간
* 일일 대여 횟수
* 도심에서의 거리

`london_bicycles` 데이터세트는 두 개의 테이블을 가지고 있습니다. 아래 그림처럼 해당 테이블의 스키마를 클릭하면 어떤 열을 갖고 있는 테이블인지 알 수 있습니다.
* cycle_hire: 대여할 때마다 기록되는 대여 정보 테이블입니다. 대여 ID (rental_id)와 자전거 ID(bike_ID)로 key를 갖고 있고, 각 대여 건마다 대여 시간 (duration), 
출발점과 종착점의 정보들이 나와있습니다.
    ![](../../images/bigquery_cyclehire.png)

* cycle_stations: 자전거 대여소에 대한 정보를 담고 있는 테이블입니다. 대여소 ID (id)의 위도 (latitude), 경도 (longitude), 자전거 보유 현황 (bikes_count) 등의 정보를 담고 있네요.
    ![](../../images/bigquery_cyclestations.png)

* 이 두 테이블에 JOIN KEY가 되는 건 cycle_stations의 대여소 ID와 cycle_hire의 출발점 혹은 종착점 id가 됩니다.

이 두 대여소 별 대여 기간, 일일 대여 횟수, 도심에서의 거리를 구하는 쿼리는 다음과 같습니다.

이제 이를 이용해서 먼저 대여 기간과 일일 대여 횟수는 

https://console.cloud.google.com/bigquery?project=tactile-rigging-270306&p=tactile-rigging-270306&d=bqml_tutorial&m=london_station_clusters&page=model