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
* **Score based**: Hill Climbing Algorithm
* **Constraint based**: Grow-Shrink Markov Blanket, Incremental Association, Fast Incremental Association, Interleaved Incremental Association
* **Hybrid**: Max-Min Hill Climbing

**Score based**는 약간 heuristic한 방법인데요. 선형회귀에서 단계 별로 변수를 선택할 때 AIC나 BIC를 기준으로 최적의 모형을 선택하듯이 score based도 네트워크에서 한 연결 (edge)을 끊어보기도, 붙여보기도 하면서 각 DAG의 score를 비교하는 방식입니다. 마찬가지로 score도 BIC를 사용합니다.

**Constraint based**는 조건부 독립 검정 (CI; Conditional Independence test)을 통해 인과 관계를 파악하는 알고리즘입니다.

마지막으로 **Hybrid**은 score based와 constraint based를 적당히 섞은 알고리즘입니다. 

저는 여기서 가장 기본이 되는 Score based의 **Hill Climbing Algorithm**과 Constraint based의 **Grow-Shrink Markov Blanket Algorithm**에 대해 알아보겠습니다.

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
### Grow-Shrink Markov Blanket Algorithm (GSMB Algorithm)

제가 저번 포스팅에서 Markov Blanket (MB)에 대해 설명은 했지만 왜 이게 중요한 지 잘 이해가 안 갔었는데요. 중요한 이유가 이 알고리즘 때문이었습니다.
다시 설명하자면 **Markov Blanket**은 세 가지 요소로 이루어져 있습니다.

1. 노드의 직계 부모
2. 노드의 직계 자식
3. 노드의 직계 자식의 부모

![](../../images/causal-dag.png)
이미지 출처: [`[link]`](https://steemit.com/dag/@cryptodreamers/dag-dag-directed-acyclic-graph)
{:.figure}

앞의 그림에서 
* $B$의 MB는 $\{A, C\}$, 
* $D$의 MB는 $\{C, E, F\}$가 되겠네요! 

이를 수식으로 표현하면 $\mathbf{BL}(B) = \{A,C\}$, $\mathbf{BL}(D) = \{C,E,F\}$가 됩니다.

MB는 MB를 조건으로 했을 때 다른 노드들은 해당 노드와 **조건부 독립**이 되는 성질을 가지고 있습니다. 즉 $B$의 경우 

$$
P(B, E\vert \mathbf{BL}(B)) = P(B, D\vert \mathbf{BL}(B))=P(B, F\vert \mathbf{BL}(B))=P(B\vert \mathbf{BL}(B)),\text{where }\mathbf{BL}(B) = \{A,C\}
$$

GSMB 알고리즘은 Growing (성장)과 Shrinking (축소)를 통해 노드마다 Markov Blanket을 찾는 게 핵심입니다.

![](../../images/gbn-gsmarkov.png)

마찬가지로 pseudo code인데요. 설명하자면 다음과 같습니다.
1. 공집합 $\mathbf{S}$에서 시작
2. [**성장 단계**] $X$노드를 제외한 모든 노드 $Y$에 대해서 하나씩 반복: $\mathbf{S}$를 조건으로 했을 때 $X$노드와 $Y$ 독립이 아니라면 $\mathbf{S}$에 $Y$ 포함
3. [**축소 단계**] $\mathbf{S}$에서 한 노드씩 빼가면서 조건부 독립인 지 확인하고, 독립이면 $\mathbf{S}$에 $Y$ 제외
4. X의 markov blanket인 $\mathbf{BL}(X)$에 $\mathbf{S}$ 저장
---

이를 그림으로 그리면 다음과 같습니다. 
![](../../images/gbn-gsmarkov-2.png)

실제 인과 관계 구조는 점선이고, 하나씩 노드를 골라 조건부 독립인 지 확인하는 **성장 단계**를 거칩니다.
그림에서 보시는 것과 같이 $A$의 MB를 구하는 첫 과정으로 $A$를 제외한 노드에 대해서 조건부 독립인 지 테스트합니다. 
* (a)와 (b): 공집합 $\mathbf{S}=\emptyset$을 조건으로 했을 때 $A$와 $B$가 독립인 지 테스트한 과정입니다. 결과적으로 독립이 아니기 때문에 $\mathbf{S}$에 $\{B\}$가 포함됩니다.
* (c)와 (d): $\mathbf{S}=\{B\}$를 조건으로 했을 때 $A$와 $F$가 독립인 지 확인합니다. 그 결과 독립이기 때문에 $\mathbf{S}$는 그대로 유지됩니다.
* 이를 $A$를 제외한 모든 노드에 대해 테스트합니다.

성장 단계가 끝나고 $A$의 MB($\mathbf{S}$)는 $\{B, G, C, K, D, H, E\}$로 구성됩니다.
그 다음에 MB 중 불필요한 노드는 없는 지 확인하기 위해 **축소 단계**를 거칩니다.

![](../../images/gbn-gsmarkov-3.png)

* (a)와 (b): 먼저 MB에서 $G$를 빼고 조건으로 하여 $A$와 $G$가 독립인 지 확인합니다. 그 결과 독립이기 때문에 $\mathbf{S}$에서 $G$를 제외합니다.
* (c)와 (d): 마찬가지로 MB에서 $K$를 빼고 조건으로 하여 $A$와 $K$가 독립인 지 확인합니다. 그 결과 독립이기 때문에 $\mathbf{S}$에서 $K$를 제외합니다.
  
최종적으로 성장과 축소 단계를 거쳐 $A$의 MB를 $\{B, C, D, H, E\}$로 정합니다.

![](../../images/gbn-gsmarkov-4.png)

이는 $A$ 노드에 대한 과정이고, $B~L$의 노드도 마찬가지로 MB를 구하면 끝입니다.


**정리하면...**

* 인과 관계 추론의 핵심은 구조를 학습해서 모수를 추정하는 두 단계, 핵심은 "구조 학습"!
* Score based인 Hill climbing 알고리즘은 각 노드의 쌍마다 연결을 끊어보고, 연결해보고, 바꿔보는 과정을 반복해 인과 구조 학습
* Constraint based인 Grow-Shrink Markov Blanket 알고리즘은 각 노드마다 성장 단계와 축소 단계를 거쳐 MB를 구해 인과 구조 학습 


---
## Data Analysis: Malocclusion Data

이제 이 GBN을 이용해서 실제 데이터를 분석한 예에 대해 알아봅시다.
데이터 출처는 무려 Nature의 Scientific Report 잡지에 실린 [Scutari et al. (2017)](https://www.nature.com/articles/s41598-017-15293-w.pdf)논문이네요! $(143\times 9)$의 작은 데이터입니다.

단어들이 좀 어려운데 핵심은 이겁니다.
1. $T_1$과 $T_2$ 두 시점에 측정된 143명의 부정교합 환자 자료로, 이 두 시점의 차이 (`dT`)와 부정 교합과 관련한 측정 값들의 **차이**(`dANB`~`dCoGo`)를 변수로 선정하였습니다. 이들은 다음과 같습니다.
   *  `dT`: difference in times
   *  `dANB`: difference in angle between Down's points A and B (degrees).
   *  `dIMPA`: difference in incisor-mandibular plane angle (degrees).
   *  `dPPPM`: difference in palatal plane - mandibular plane angle (degrees).
   *  `dCoA`: difference in total maxillary length from condilion to Down's point A (mm).
   *  `dGoPg`: difference in length of mandibular body from gonion to pogonion (mm).
   *  `dCoGo`: difference in length of mandibular ramus from condilion to pogonion (mm).
   ![](../../images/gbn-data.png)

    결국 부정 교합과 관련있는 거리나 각도들을 $T_1$과 $T_2$시점에 측정해서 그 차이들을 변수로 만들었습니다. 

2. 중요한 변수는 **처리** (`Treatment`)와 **부정교합의 상태** (`Growth`)입니다. 처리는 받지 않았으면 0, 받았으면 1의 값을 갖고, Growth는 부정교합 상태가 악화됐으면 0, 완화됐으면 1의 값을 갖습니다. 
3. 목표는 두 가지로 요약됩니다.
   * 기존에 가졌던 가정과 인과 관계 결과가 일치하는 지
   * "만약 이 변수가 이 값을 가졌다면 처리 효과는 어떻게 될까?"와 같은 인과 관계 추정

이 데이터를 통해 GBN을 사용하면 다음과 같습니다.  $\varepsilon_{\Delta Y} \sim N\left(0, \sigma_{\Delta Y}^{2}\right), \Delta T=T_{2}-T_{1} , \Delta Y=Y_{T_{2}}-Y_{T_{1}}$라 할 때,

$$
\frac{\Delta Y}{\Delta T}=\mu^{*}+\frac{\text { Growth }}{\Delta T} \beta_{G}^{*}+\frac{\text { Treatment }}{\Delta T} \beta_{T R}^{*}+\frac{\Delta X_{1}}{\Delta T} \beta_{2}^{*}+\ldots+\varepsilon_{\frac{\Delta Y}{\Delta T}}
$$

여기서 $\Delta Y$는 `dANB~dCoGO`의 6개 변수에 해당합니다. 이제부터 이 변수를 "**안면 관련 변수**" (craniofacial features)라 부르겠습니다.

---
### EDA

먼저, 필요한 R 라이브러리입니다.

```r
library(GGally)
library(bnlearn)
library(Rgraphviz)
library(penalized)
library(visNetwork)
library(dplyr)
```

변수에 대해 잘 모르기 때문에 EDA가 더 중요합니다. 
`diff`로 파일을 불러오고 시간 차이 $\Delta T$로 나눠준 자료를 `diff_delta`에 저장해서 상관 관계와 pairs 플랏을 그려봅니다.

```r
diff=read.csv("dental.csv")
diff_delta = cbind(Treatment=diff$Treatment,
                   Growth=diff$Growth,
                   sapply(diff[, 1:6], function(x) x / diff$dT))
ggcorr(diff_delta,palette="RdBu",label=TRUE,label_round=3)
ggpairs(diff[,c("dANB", "dPPPM", "dIMPA", "dCoA", "dGoPg", "dCoGo")])
```

![](../../images/dental-pairs.png)
![](../../images/dental-cor.png)


상관관계와 산점도를 보면 
* `dCoGo`, `dGoPg`, `dCoA` 간의 상관관계
* `Treatment`, `dANB`, `dCOA` 간의 상관관계

가 돋보입니다. 산점도도 보면 일직선에 가까운 모습이죠. 특히 `Treatment`와 `dANB`, `dCOA`는 아래 턱점 A와 관련이 있기 때문에 의학적으로 흥미로운 결과라 합니다. 또 처리가 주로 영향을 미치는 변수가 이 두 개라 추측할 수 있습니다. 


---
### Step1: Black List and White List

인과 관계 구조를 학습하는 데에 앞서 이어지지 말아야 할 연결을 **black list**에, 꼭 이어져야 할 연결을 **white list**에 지정합니다.
[Scutari et al. (2017)](https://www.nature.com/articles/s41598-017-15293-w.pdf)에서는 다음과 같이 black list와 white list를 지정했습니다.
* Black list: 총 22개의 연결을 제외합니다.
  * (18개) 안면 관련 변수(`dANB~dCoGO`)에서 `dt`, `Treatment`, `Growth`로 오는 arc는 모두 제외해야 합니다. 
  왜냐하면 시간의 차이(`dt`)나 처리 (`Treatment`)로 안면 관련 변수의 측정값이 변화할 수는 있어도 그 반대는 성립하지 않기 때문입니다. 마찬가지로 부정교합의 상태(`Growth`)에 따라 안면 관련 변수의 측정값이 바뀌는 것이기 때문입니다. 
  * (2개) 처리(`Treatment`)는 시간의 차이(`dT`)의 원인이 될 수 없고 그 반대도 성립하면 안되기 때문에 제외합니다.
  * (2개) 부정교합의 상태(`Growth`)는 처리(`Treatment`)와 시간 차이(`dT`)의 원인이 될 수 없습니다.
  
  따라서 black list를 다음과 같이 지정합니다.

  $$
  \begin{aligned}
  \text{안면 관련 변수} \rightarrow \mathtt{dt}
  \text{안면 관련 변수} \rightarrow \mathtt{Growth}
  \text{안면 관련 변수} \rightarrow \mathtt{Treatment}
  \mathtt{Treatment} \rightarrow \mathtt{dT}
  \mathtt{dT} \rightarrow \mathtt{Treatment}
  \mathtt{Grwoth} \rightarrow \mathtt{Treatment}
  \mathtt{Grwoth} \rightarrow \mathtt{dT}
  \begin{aligned}
  $$

* White list: 총 3개의 연결을 포함합니다.
  * (2개) 사전 지식에 따라 `dANB` $\rightarrow$ `dIMPA` $\leftarrow$ `dPPPM` 연결을 포함합니다.
  * (1개) 시간에 따라 부정교합의 상태가 변화해야하므로 `dT` $\rightarrow$ `Growth` 연결을 포함합니다.

```r
# DAG에 포함되지 말아야할 엣지들
black_list = tiers2blacklist(list("dT", "Treatment", "Growth",
                                  c("dANB", "dPPPM", "dIMPA", "dCoA", "dGoPg", "dCoGo")))
black_list = rbind(black_list, c("dT", "Treatment"))
# DAG에 포함되어야할 엣지들
white_list=matrix(c("dANB", "dIMPA","dPPPM", "dIMPA","dT", "Growth"),
                  ncol = 2, byrow = TRUE, dimnames = list(NULL, c("from", "to")))
```

웃긴 건 `bnlearn`의 패키지에서 black list는 `tiers2blacklist` 함수를 통해 지정할 수 있는데 white list는 직접 행렬을 만들어줘야 하네요.

---
### Step2: Causal Structure Learning

이제 black list와 white list를 포함해서 Hill-Climbing 알고리즘과 Grow-Shrink Markov Blanket 알고리즘을 통해 인과 관계를 학습합니다. `bnlearn`에서는 각각 `hs`, `gs`함수를 사용합니다.

```r
DAG  = hc(diff, whitelist = white_list, blacklist = black_list)
DAG2 = gs(diff, whitelist = white_list, blacklist = black_list)
```

이들의 결과를 보려면 `RGraphviz`나 `visNetwork` 패키지를 사용할 수 있습니다. 둘 다 써본 결과 
`RGraphviz`는 사용법이 쉽지만 그래프가 예쁘지 않다는 것이고, `visNetwork`는 사용법이 조금 복잡하지만 그래프가 예쁩니다. 

전 그래서 `visNetwork`을 사용했고 다음의 함수를 통해 그래프를 그리도록 했습니다.

```r
"plot_network" = function(dag,strength_df=NULL,undirected=FALSE,
                          group=NA,title=NULL,height=NULL,width=NULL)
    {
    edge_size = ifelse(is.null(strength_df),NA,
                   right_join(strength_df, data.frame(dag$arcs[,c(1,2)]))$strength)
    
    nodes = names(dag$nodes)
    nodes = data.frame(id   = nodes,
                       label= nodes,
                       size = 16,
                       font.size= 18,
                       shadow   = TRUE,
                       group    = group)
    
    edges = data.frame(from   = dag$arcs[,1],
                       to     = dag$arcs[,2],
                       value  = edge_size,
                       arrows = list(to=list(enabled=TRUE,scaleFactor=.5)),
                       shadow = TRUE)
    
    if(is.na(group[1]))     nodes = nodes[,-6] # without group
    if(is.na(edge_size)) edges = edges[,-3] # without edge_size
    if(undirected)       edges$arrows.to.enabled=FALSE
    
    network=visNetwork(nodes,edges,main=title,height=height, width=width)%>% 
        visOptions(highlightNearest = TRUE, nodesIdSelection = TRUE)
    return(network)
}
```
함수의 인자에 대해 간단히 설명드리면 다음과 같습니다.
* `dag`: 인과관계를 학습한 object
* `strength_df`: Bootstrap 후 강도를 나타내는 dataframe
* `undirected`: 방향성이 있는 지의 여부
* `group`: Network 그림에서 쓰이는 그룹
* `title`: Network 그림의 제목
* `height`, `width`: Network 그림의 높이, 너비

이 함수를 통해 그린 네트워크는 다음과 같습니다.

```r
group = ifelse(names(DAG$nodes)%in%c("Treatment","Growth"),2,1)
plot_network(DAG,group=group,title="Hill-Climbing")
plot_network(DAG2,group=group,title="Markov Blanket")
```
{% include_relative ../../images/test.html %}

![](../../images/dental-hs.png)
![](../../images/dental-gsmb.png)


---
## References

* `R` 패키지 `bnlearn` tutorial [`[link]`](https://www.bnlearn.com/examples/useR19-tutorial/)
* `R` 패키지 `visNetwork` tutorial[`[link]`](https://datastorm-open.github.io/visNetwork/)
* D. Margaritis (2003), Learning Bayesian Network Model Structure from Data. PhD thesis, School of Computer Science, Carnegie-Mellon University, Pittsburgh, PA, Available as Technical Report CMU-CS-03-153. [`[pdf]`](https://www.cs.cmu.edu/~dmarg/Papers/PhD-Thesis-Margaritis.pdf)
* Scutari, M., Auconi, P., Caldarelli, G., & Franchi, L. (2017). Bayesian networks analysis of malocclusion data. *Scientific reports*, 7(1), 1-11. [`[pdf]`](https://www.nature.com/articles/s41598-017-15293-w.pdf)
