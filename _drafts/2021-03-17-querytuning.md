---
layout: post
title: SQL 쿼리 튜닝 맛보기 
date: 2021-03-17
categories: [DB]
tag: [sql, db, easy-guide, query-optimization, query-tuning, honeytip]
comments: true
photos:
    - "../../images/honeytip-title.png"
---

* 오늘은 SQL 쿼리 간 비교를 통해 쿼리를 튜닝하는 방법에 대해 글을 쓰고자 합니다.
* 메인 소스는 [이 곳](https://www.slideshare.net/topcredu/sql-for-sqlsqltipsql)에서 확인하실 수 있습니다.


---
## 인덱스 컬럼을 포함하는 표현식, 함수, 계산식
> 인덱스 컬럼을 포함하는 표현, 함수, 계산은 인덱스를 사용하지 못한다. 인덱스 칼럼보다는 반대쪽에 변형을 가하자.

### 인덱스, 그게 뭐길래




  | 최적화 전 쿼리  | 최적화 후 쿼리  |
  |:---:|:---:|
  |where substr(ename, 1,3) = 'SCO'| where ename like 'SCO%'|
  |where trunc(hiredate) = trunc(sysdate)| where hiredate betwewen trunc(sysdate) and trunc(sysdate) + .99999
  |
  where en

---
## SELECT 절에서 DISTINCT 사용을 피하자


---
## WHERE 절 비교칼럼 데이터타입은 일치시켜라

---
## OUTER JOIN + IS NULL 대신 NOT IN, NOT EXISTS를 사용하자

---
## IN을 이용한 조인보다는 EXISTS를 사용하자

---
## UNION 보다는 UNION ALL을 사용하라

---
## 조인되는 건수가 작다면 일반적인 조인보다 스칼라 서브쿼리 사용도 고려하라.

