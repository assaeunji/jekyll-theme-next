---
layout: post
title: Implicit Feedback 추천 시스템에 대한 친절한 설명
date: 2020-10-11
categories: [ML]
tag: [recommender-system, ALS, implicit-feedback, ml, python, pyspark]
comments: true
---

![Jesse님께 허락을 받고 쓴 글 인증](../../images/jesse-allow.png)

* 이 글은 Jessed님의 A Gentle Introduction to Recommender Systems with Implicit Feedback을 번역한 글입니다.

---

추천 시스템은 소셜 네트워크, 엔터테인먼트 산업 등 다양한 산업에서 매우 중요합니다. 당신이 선호할만한 곡을 추천하는 것부터, 읽을만한 책을 추천하고, 입을만한 옷을 추천하는 것까지 추천시스템은 고객들에게 제품을 더 쉽게 선택할 수 있도록 도와주고 있습니다.

의사결정에 도움을 주는 추천시스템은 왜 그리 중요할까요? 한 논문은 이틀간 잼 가판대에 관한 실험을 진행했습니다. 한 가판데는 24개의 잼 종류가 있는 반면 두번째 가판대는 6개만 잼이 놓여져 있었습니다. 24개의 종류를 가진 가판대에서는 3%의 고객들만이 구매한 반면, 6개의 종류를 가진 가판대는 30%의 고객들이 구매를 결정했다 합니다. 거의 10배에 가깝게 구매가 증가한 것이죠!

선택할 수 있는 종류가 정해져 있다면, 특히 온라인 쇼핑몰에서 추천 시스템은 매우 큰 효과를 가집니다. 과거에 Netflix에 종사하다 Quora에 재직하고 계신 Xavier Amatriain는 2014년에 카네기 멜론 대학에서 추천 시스템에 대한 강연을 했는데요. 그가 소개한 추천 시스템에 대한 주요 통계 자료는 다음과 같습니다.
* 넷플릭스에서는 고객들이 본 영화 중 2/3는 추천된 영화입니다.
* 구글에서는 뉴스 추천으로 CTR을 38%나 향상시켰습니다.
* 아마존에서는 구매의 35%가 추천으로부터 나옵니다.

이와 더불어 Hulu에서는 단순하게 가장 인기있는 상품을 추천했던 것보다 추천 시스템을 이용한 결과, CTR이 3배 이상 증가했다 보고했습니다. 잘만 활용한다면 추천시스템은 기업에게 큰 도움을 줄 수 있습니다.

자, 그럼 고객 정보와 아이템에 대해 충분한 정보가 있다 가정하고 추천 시스템을 구축한다 해봅시다. 어떻게 구축할 수 있을까요? 구축하기 위해서는 다음과 같은 요인들을 고려해야 합니다.
* 유저와 아이템에 대한 데이터의 종류
* scale 능력
* 추천의 투명성

이 포스트에서는 위의 세 가지 요인들에 대해 다루고, 고려할 요인에 필요한 기본적인 방법론에 대해 정리하고자 합니다.

---
### 콘텐츠 기반 추천 (Pandora)

스트리밍 뮤직 기업인 Pandora의 경우, 음악 게놈 프로젝트의 일환으로 자신들이 갖고 있는 음악들의 고유한 특징들을 추출하기로 결정했습니다. 대부분의 노래들은 450개 가까이 되는 특징을 가진 특징 벡터에 기반하고 있습니다. 이러한 특징들로 추천 시스템을 이진 분류 (binary classification) 문제로 치환해 모형을 구축할 수 있습니다. 이를 통해, 특정한 유저의 플레이리스트를 바탕으로 훈련 데이터를 만들어서 그 유저가 특정한 노래를 좋아할 확률을 추정할 수 있도록 전통적인 머신러닝 방법들을 사용할 수가 있는거죠. 즉, 콘텐츠 기반 추천은 이 곡을 좋아할 확률이 가장 높은 곡들로 추천하면 되는 일입니다.

그러나 대부분의 회사들은 제품의 특징에 대한 데이터를 따로 갖고 있지 않습니다. Pandora도 곡들의 특징을 추출하기까지 여러 해가 걸렸다고 합니다. 이런 한계로 인해 콘텐츠 기반 추천은 최우선 순위가 아닐 수 있습니다.

---
## 인구 통계 (Demographic) 기반 추천 (Facebook)

만약 Facebook이나 LinkedIn처럼 유저에 대한 인구 통계학 정보가 많다면, **비슷한 유저나 그들의 과거 행동**에 기반해서 추천을 할 수 있습니다. 콘텐츠 기반 방법과 유사하게 유저들의 고유한 특징 벡터를 뽑아내고, 특정한 아이템을 좋아할 확률을 예측할 수 있는 모델을 만들 수 있습니다.

그러나, 이 또한 유저들에 대한 정보가 많이 필요하기 때문에 콘텐츠 기반 추천과 마찬가지로 쉽지 않은 선택입니다.

따라서, 만약에 아이템과 유저에 기반한 자세한 정보에 대해 신경쓰고 싶지 않다면, 협업 필터링이 강력하고 효율적인 방법이 될 수 있습니다.

---
## 협업 필터링 (Collaborative Filtering)

협업 필터링은 유저와 아이템간의 관계를 사용하고, 그 자체의 정보에 대해서는 사용하지 않습니다. 필요한건 단지 각 유저와 아이템 간 상호작용 (interaction)으로 인해 나온 평점입니다. 이러한 상호작용은 명시적(explicit), 암시적(implicit) 상호작용으로 나눌 수 있습니다.

* 명시적 (Explicit): 점수, 평점, 좋아요와 같이 명시적으로 점수를 매긴 것
* 암시적 (Implicit): 클릭, 노출, 구매로 추정할 수 있는 선호도로, 명시적 상호작용만큼 명확하지 않음

가장 쉬운 예시는 숫자로 매길 수 있는 영화 평점입니다. 우리는 평점을 통해 유저가 영화를 즐겼는지를 알 수 있습니다. 그러나, 대부분 사람들은 평점을 전혀 남기지 않는다는 문제가 있습니다! 따라서, 사용가능한 데이터가 매우 희귀합니다. 대신에 Netflix는 최소한 유저가 어떤 콘텐츠를 시청했는지의 정보를 얻을 수 있습니다. 이 때, 이 유저가 시청 후에 콘텐츠를 좋아하지 않을 수도 있습니다. 따라서, 어떤 장르의 영화를 선호할지 추천하는 일이 더 어려워질 수 있는 것이죠.

이런 단점에도 불구하고, 암시적 피드백은 중요합니다. Hulu라는 회사에서는 [추천 시스템에 대한 기사](http://tech.hulu.com/blog/2011/09/19/recommendation-system/)에서 다음과 같이 언급합니다.

> Hulu에서의 암시적 데이터의 양은 명시적 피드백의 양을 압도하기 때문에, 우리의 추천 시스템은 암시적 피드백 데이터를 갖고 우선적으로 설계되어야 한다.

더 많은 데이터가 더 좋은 모형을 의미하기 때문에, 암시적 피드백 데이터를 구축하는데 힘써야한다는 것입니다. 암시적 피드백 데이터를 이용한 협업 필터링에는 많은 방법들이 있지만, 이 글에서는 스파크 라이브러리에서 쓰는 방법인 ALS (Alternative Least Squares)에 대해 서술하겠습니다.

---
## Alternative Least Squares

본격적으로 예시에 들어가기 전에, 이 방법이 어떻게 작동하는지 직관적으로 설명하고, 왜 스파크에서 유일하게 선택되었는지 설명하고자 합니다. 앞에서 협업 필터링은 유저나 아이템에 대한 정보를 전혀 이용하지 않는다고 말했습니다. 그러면 어떻게 유저와 아이템이 서로 연관있다고 정의할 수 있을까요?

정답은 행렬 분해 (Matrix Factorization)입니다. 행렬 분해는 차원 축소 (관련한 정보는 남기되 피처의 개수를 줄이고 싶을 때)에서 자주 사용되는 방법입니다. 이는 주성분 분석<sup>PCA; Principal Component Analysis</sup>과 고유값 분해 <sup>SVD; Singular Value Decomposition</sup>과 일맥상통하죠.

본질적으로, 매우 큰 유저-아이템 행렬로 숨겨진 피처들을 뽑아내서 이들을 훨씬 작은 유저들의 특징을 담은 행렬과 아이템 특징을 담은 행렬로 만들 수 있을까요? 이 작업이 바로 ALS가 행렬 분해를 통해 하고자 하는 일입니다.

아래 사진에서 보듯이, 행렬 $R$은 $M \times N$ 행렬입니다. $M$은 유저 수, $N$은 아이템 수를 의미합니다. 이 행렬은 매우 희소 (sparse)한데, 유저들이 몇 개들의 아이템과 상호작용했기 때문입니다. <sup>여기서 "상호작용"했다는 의미는 유저들이 해당 아이템을 소비하거나 구매하는 등의 행위를 의미합니다.</sup> 이 $R$행렬을 두 개의 더 작은 행렬로 분해할 수 있습니다: 한 행렬은 $M\times K$차원을 가진 각 유저의 특징들을 담은 잠재 유저 행렬 $U$이고, 두 번째 행렬은 $K\times N$행렬을 가진 아이템들의 특징을 담은 잠재 아이템 행렬 $V$입니다. 이 두 행렬을 곱하면 원본 행렬 $R$과 유사한 값을 내지만, 두 행렬은 아이템과 유저에 대한 $K$개의 잠재적 특징을 가진 빽빽한 (dense) 행렬입니다.

![](../../images/als_matrix.png)

$U$와 $V$행렬을 풀기 위해서는 SVD (매우 큰 행렬의 역행렬을 구해야하기 때문에 매우 컴퓨터 연산을 많이 차지하는)를 사용하거나, 이를 근사한 ALS를 적용할 수 있습니다. ALS의 경우, 한 번에 한 특징 벡터만 추출하기만 하면 되기 때문에 병렬적으로 처리할 수 있습니다 (그렇기 때문에 스파크에서 이 방법을 사용한 큰 이유라 생각합니다). 이를 하기 위해선 먼저, $U$를 랜덤하게 초기화하고 $V$의 해를 구합니다. 이 후 다시 돌아가 $V$의 해를 이용해 $U$의 해를 구합니다. 이처럼, $R$을 최대한 근사시키는 수렴 값을 얻을 때까지 반복합니다.

이 연산이 끝나면 단지 $U$와 $V$를 곱해 특정한 유저/아이템 상호작용에 대한 평점을 예측할 수 있습니다. 이것이 Hu, Koren, and Volinsky의 [Collaborative Filtering for Implicit Feedback Datasets](http://yifanhu.net/PUB/cf.pdf)의 방법입니다. 이제 이 논문에서 사용된 방법을 실제 데이터에 적용해보고, 추천 시스템을 구축해보겠습니다.

---
## 데이터 설명

예시에 사용할 데이터는 UCI Machine Learning repository에서 가져왔습니다. 이 데이터셋은 "Online Retail"이라 불리고, [여기](https://archive.ics.uci.edu/ml/datasets/Online+Retail)에서 찾을 수 있습니다. 설명에서 알 수 있듯이, 이 데이터는 영국에 있는 온라인 소매 기업에서 가져온 8개월 간의 구매 기록을 담고 있습니다.

먼저 필요한 라이브러리를 불러오겠습니다.

```python
import pandas as pd
import scipy.sparse as sparse
import numpy as np
from scipy.sparse.linalg import spsolve
import random

website_url = 'http://archive.ics.uci.edu/ml/machine-learning-databases/00352/Online%20Retail.xlsx'
retail_data = pd.read_excel(website_url)
retail_data.head()
```

![](../../images/implicit_retail.png)


해당 데이터는 구매 목록에 대한 송장 번호 (Invoice), 아이템 ID (StockCode), 아이템에 대한 설명 (Description), 구매 횟수 (Quantity), 구매 일자 (InvoiceDate), 구매 가격 (UnitPrice), 고객 ID (CustomerID), 고객의 국적 (Country) 컬럼으로 이루어져 있습니다.

---
## 데이터 전처리

먼저, 결측치가 있는지 확인하겠습니다.

```python
retail_data.info
```
![](../../images/implicit_info.png)

대부분의 컬럼에는 결측치가 없지만 CustomerID에 결측치가 좀 있군요. CustomerID가 결측이면 누가 어떤 항목을 샀는지 모르기 때문에 결측치를 제거해야 합니다.

```python
# 1. CustomerID가 NA인 것을 지워줌
cleaned_retail = retail_data[retail_data['CustomerID'].notna()]
cleaned_retail.info() # 총 406,829행
```

결측을 제거하니 406,829행이 되었습니다.

다음은 아이템 ID에 따른 아이템 설명 테이블 `item_lookup`을 만듭니다. 이 테이블을 만드는 이유는 ~~ 입니다.

```python
item_lookup = cleaned_retail[['StockCode','Description']].drop_duplicates()
item_lookup['StockCode'] = item_lookup['StockCode'].astype(str)
item_lookup.head()
```

ALS 알고리즘의 입력값 형태를 만드려면 데이터를 변형해야 합니다. 이를 위해 유니크한 고객 ID를 행렬의 행으로, 유니크한 아이템 ID를 행렬의 열로 만듭니다. 행렬의 값은 각 유저가 각 아이템을 구매한 총 횟수입니다. 이런 행렬을 **희소 행렬 (Sparse Matrix)**라 합니다.

```python
# matrix가 매우 크기 때문에 Sparse matrix로 바꾸어주어서
# zero가 아닌 값들의 위치와 그 값만 저장하도록 메모리 절약!
cleaned_retail['CustomerID'] = cleaned_retail['CustomerID'].astype(int)
# 1. 고객 ID, 아이템 ID, 구매량만 가져옴
cleaned_retail = cleaned_retail[['CustomerID','StockCode','Quantity']]
# 2. 고객과 아이템 별로 총 얼마나 구매했는지 
grouped_cleaned = cleaned_retail.groupby(['CustomerID','StockCode']).sum().reset_index()

# 3. 구매 수량 (Quantity)가 0인 데이터는 고객이 구매했으나 환불한 경우이기 때문에 구매 카운트로 쳐줘서 1로 바꿔줌
grouped_cleaned[grouped_cleaned['Quantity']==0] = 1

# 4. 구매한 애들만 뽑기
grouped_purchased = grouped_cleaned[grouped_cleaned['Quantity']>0]

grouped_purchased.head(5)
```

![](../../images/implicit_purchased.png)

`grouped_purchased` 테이블을 보면 고객 별로 특정 StockCode를 가진 항목들을 8개월 동안 얼마나 구매했는지를 Long Table 형태로 보여줍니다.

이제 이를 행은 고객 ID, 열은 아이템 ID (StockCode)로 두고 구매 수량 (Quantity)을 행렬 값으로 두는 희소 행렬을 만듭니다.

```python
customers = list(np.sort(grouped_purchased['CustomerID'].unique()))
products = list (grouped_purchased['StockCode'].unique())
quantity = list(grouped_purchased['Quantity'])

rows = grouped_purchased['CustomerID'].astype('category').cat.codes
cols = grouped_purchased['StockCode'].astype('category').cat.codes

print(len(customers)) # 4327
print(len(products))  # 3650

# csr: Compressed Sparse matrix by Row
purchase_sparse = sparse.csr_matrix((quantity, (rows, cols)), shape = (len(customers),len(products)))
purchase_sparse #4327 * 3650 행렬
```
`purchase_sparse` 행렬은 4,327명의 고객과 3,650개의 아이템으로 이루어져 있습니다. 이런 유저/ 아이템 간 상호작용은 265,221개에 달합니다. 이렇게 만든 희소 행렬이 얼마나 비었는지 보기 위해 희소성 (Sparsity)를 계산할 수 있습니다. 이는 행렬에 얼마나 0 값이 많은지를 보여줍니다.

```python
# Sparsity: 얼마나 비어있나?
matrix_size = purchase_sparse.shape[0]* purchase_sparse.shape[1]
num_purchases = len(purchase_sparse.nonzero()[0])
sparsity = 100 * (1 - (num_purchases / matrix_size))
sparsity
> 98.3207
```

상호작용 행렬의 98.3%이 비어있습니다. 협업 필터링을 구축하기 위해서 필요한 희소성은 약 99.5%까지 가능합니다. 저희가 구한 희소행렬은 98.3%이니까 협업 필터링을 무난하게 구축할 수 있습니다.

---
## Train, Validation 세트 만들기

일반적인 머신러닝에서는 훈련 데이터로 만든 모형이 얼마나 새로운 데이터에 잘 작동하는지 테스트해야 합니다. 훈련 데이터로부터 테스트 데이터를 따로 만들어서 할 수 있죠.

그러나 협업 필터링에서는 적절한 행렬 분해 (matrix factorization)을 하기 위해서 **모든** 유저/아이템 상호작용 데이터를 사용해야 합니다. 결과적으로 모형을 훈련시킬 때 일정한 확률로 랜덤하게 뽑힌 유저/아이템 상호작용을 숨겨야 합니다. 이후, 테스트 단계에서는 얼마나 유저가 실제로 추천된 아이템을 구매했는지 파악할 수 있습니다. 이상적으로는 A/B 테스트를 통해 추천 효과를 검증할 수 있습니다.

그럼 어떻게 일정한 확률로 데이터를 숨길 수 있을까요?

![](../../images/implict_train.png)

우리의 테스트 데이터는 원본 데이터의 복사본입니다. 그러나, 훈련 데이터는 랜덤하게 일정한 확률로유저/아이템 상호작용 몇 개를 가려 고객이 그 아이템을 구매한 적이 없는 것처럼 만듭니다 (= 몇 개의 구매 수량을 0으로 채워넣습니다). 이 후 테스트 데이터에서 얼마나 유저가 실제 구매한 아이템이 추천됐는지 파악할 수 있습니다. 만약 유저가 추천된 아이템을 실제 구매한 경우가 많을 경우 추천 시스템이 제대로 작동한다 말할 수 있겠죠!

자 이제 훈련 데이터를 만들어 봅시다.

```python
def make_train (matrix, percentage = .2):
    '''
    -----------------------------------------------------
    설명
    유저-아이템 행렬 (matrix)에서 
    1. 0 이상의 값을 가지면 1의 값을 갖도록 binary하게 테스트 데이터를 만들고
    2. 훈련 데이터는 원본 행렬에서 percentage 비율만큼 0으로 바뀜
    
    -----------------------------------------------------
    반환
    training_set: 훈련 데이터에서 percentage 비율만큼 0으로 바뀐 행렬
    test_set:     원본 유저-아이템 행렬의 복사본
    user_inds:    훈련 데이터에서 0으로 바뀐 유저의 index
    '''
    test_set = matrix.copy()
    test_set[test_set !=0] = 1 # binary하게 만들기
    
    training_set = matrix.copy()
    nonzero_inds = training_set.nonzero()
    nonzero_pairs = list(zip(nonzero_inds[0], nonzero_inds[1]))
    
    random.seed(0)
    num_samples = int(np.ceil(percentage * len(nonzero_pairs)))
    samples = random.sample (nonzero_pairs, num_samples)
    
    user_inds = [index[0] for index in samples]
    item_inds = [index[1] for index in samples]
    
    training_set[user_inds, item_inds] = 0
    training_set.eliminate_zeros()
    
    return training_set, test_set, list(set(user_inds))

 product_train, product_test, product_users_altered = make_train(purchase_sparse, 0.2)
```
자 이제 Hu, Koren, Volinsky의 논문에서 쓰인 ALS (Alternating Least Squares)를 구현해볼 시간입니다.

---
## 암시적 피드백을 위한 ALS 알고리즘 구현하기
[Collaborative Filtering for Implicit Feedback Datasets](http://yifanhu.net/PUB/cf.pdf)을 보면 ALS를 구현하기 위한 수식들을 보실 수 있습니다.

먼저, 희소 평점 행렬 (저희가 만든 `product_train`행렬에 해당합니다)을 신뢰 행렬 (Confidence matrix)로 만들어야 합니다.

$$
C_{ui} = 1 + \alpha r_{ui}
$$

여기서 $C_{ui}$는 $u$번째 유저의 $i$번째 아이템에 대한 신뢰 행렬입니다. $\alpha$는 선호도 (저희 예시에서는 구매량)에 대한 스케일링 term이고 $r_{ui}$는 저희의 구매량에 대한 원본 행렬에 해당합니다. 논문에서는 $\alpha$에 대한 초기값으로 40을 제안합니다.



