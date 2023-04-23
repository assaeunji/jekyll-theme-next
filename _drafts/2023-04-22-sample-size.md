---
layout: post
title: "분석 결과를 유의미하다고 판단할 수 있는 필수 요소: 표본 크기 (Sample size)"
date: 2023-04-22
categories: [Statistics]
tag: [statistics, sample-size, t-test, ab-test]
comments: true
photos:
    - "../../images/latte-1.jpeg"
---

* "**내 A/B 테스트는 몇 명의 표본을 필요로 할까?**"라 고민했던 분 없나요? 
* 이번 포스팅은 A/B 테스트 시 고려해야 할 요소 **표본 크기 (Sample Size)**에 대해 다루고자 합니다.
* 이 글에서 나온 정의들은 ChatGPT의 답변을 적극 활용했습니다. 초등학생한테 설명하는 컨셉으로 질문을 했더니 설명을 잘 하더라구요. 따봉 ChatGPT야 고마워!

----

## WHY: 표본 크기, 왜 중요할까? 

> 평균, 비율 등으로 내린 결과가 유의미하다고 말할 수 있기 위해선 **표본 크기**가 필요합니다.
 
저는 A/B 테스트를 많이 하지 못했지만, 테스트를 하지 않아도 통계적으로 유의하다고 말할 수 있는 표본 크기에 대해 관심이 많았습니다.
왜냐하면 표본이 적은 상태에서 결과를 해석할 때 "이거 우연히 나온 결과 아니에요? 정말 유의한게 맞을까요?"라는 피드백을 받은 경험이 많았기 때문이죠. 

그래서 집단마다 고객 수가 몇 명이 되어야 이 차이를 유의하게 볼 것인지에 대한 내용을 찾다 **표본 크기 (Sample Size)**에 대해 공부하게 되었습니다.
적절한 표본 크기를 확보하지 못하면 결과의 정확성과 신뢰도가 떨어지기 때문에, **간과하기 쉽지만 신뢰있는 분석 결과를 내기 위해선 체크해야할 요소**라 생각이 듭니다.

## WHAT: 표본 크기 정의

> 표본 크기란, 예를 들어 전국민 대상으로 어떤 설문 조사를 할 때, 전국민 대신 일부만 뽑아 몇 명을 선택할지를 말합니다. (Feat. ChatGPT)
  
그럼 자연스레 떠오르는 질문은 "그래, 표본 크기 중요한 건 알겠는데 이게 뭔데?"일 것입니다. 저는 부끄럽게도 통계학과이지만 구체적으로 표본 크기를 정의하기 어려워 이번 기회에 찾아보았습니다.

> **표본 크기란, 예를 들어 너희 반에서 특정 주제에 대한 조사를 할 때, 몇 명의 학생을 선택할지를 말해요.** 이렇게 선택된 학생들의 의견을 조사해서 그 주제에 대한 전체 학생들의 의견을 추측하게 됩니다. 이때, 선택된 학생들의 수가 적으면 전체 학생들의 의견을 정확하게 파악하기 어렵고, 선택된 학생들이 전체 학생들을 대표하지 못할 수도 있어요. 그래서 조사를 할 때는 적절한 표본 크기를 선택해서 조사를 하는 것이 중요합니다.

통계는 한글 번역이 참 와닿지 않은데, ChatGPT는 위와 같이 빗대어 말하니 아주 이해하기 쉽습니다. 이를 다시 통계의 용어를 써서 정의해보겠습니다.<sup>굳이? 라고 생각이 들 수 있지만, 결국 구글링할 땐 전문 용어가 필요하니까요 :) </sup>
표본 크기를 알기 위해선 세 개의 개념을 알면 됩니다.

* **모집단 (Population)**: 연구의 대상이 되는 전체 집단을 의미합니다. 모집단의 크기가 작으면 적은 표본 크기도 가능합니다. 하지만 대부분의 경우 모집단의 크기가 매우 크기 때문에 표본을 이용하여 연구를 진행합니다.
* **표본 (Sample)**: 모집단에서 일부를 추출한 작은 집단을 의미합니다. 표본은 모집단을 대표할 수 있는 방식으로 선택되어야 합니다.
* **표본 크기 (Sample Size)**: 표본의 크기를 의미합니다. 표본 크기가 적절하지 않으면 통계적으로 유의미한 결과를 얻기 어렵고, 표본이 모집단을 대표하지 못할 가능성이 높아집니다. 따라서 연구에서는 적절한 표본 크기를 결정하여 연구 결과를 신뢰할 수 있도록 해야 합니다.


![](../../images/sample-size-1.png)

위 그림에서 볼 수 있듯이, 모집단은 분석의 전체 집단, 표본은 그 중 일부만 뽑은 집단을 의미합니다. 표본 크기는 위 그림에서 5명이 되겠네요!
초등학생용 설명에서는 '반 전체 학생'이 모집단, '선택된 학생'이 표본이 되고, '선택된 학생 수'가 표본 크기가 됩니다.

전수 조사하면 되지 굳이 왜 표본을 뽑아야 하냐 하신다면... 시간과 비용의 문제일 수도 있고 불가능한 영역일 수도 있습니다. 예를 들어, 삼성전자 직원이 대한민국 국민들이 얼마나 갤럭시 핸드폰을 쓰는지 수요 조사를 한다고 생각해봅시다. 전국민 대상으로 갤럭시를 쓸 확률을 계산하는게 효율적일까요? 아마도 한 1,000만 명만 조사해도 전국민이 갤럭시를 쓸 확률과 비슷한 값을 낼 수 있을 것입니다. 

TMI로 갤럭시 점유율이 대략 30%쯤 될거라 생각했는데 [Counterpoint의 국내스마트폰 점유율 데이터](https://korea.counterpointresearch.com/%EA%B5%AD%EB%82%B4-%EC%8A%A4%EB%A7%88%ED%8A%B8%ED%8F%B0-%EC%A0%90%EC%9C%A0%EC%9C%A8-%EB%B6%84%EA%B8%B0%EB%B3%84-%EB%8D%B0%EC%9D%B4%ED%84%B0/)를 봤는데 63%나 되네요! 제 주변은 아이폰이 대부분이라 그런가봅니다.. 이렇게 선택 편향이 무섭습니다 ㅎㅎ


## HOW: 표본 크기, 그거 어떻게 하는 건데

다시 본론으로 돌아와서 다음은 표본 크기를 구하는 공식에 대해 설명해보고자 합니다.

사실 여러 사이트에서 표본 크기를 뚝딱 계산해줘서 손 계산은 필요없지만 그래도 궁금하지 않나요? 그리고 뚝딱 계산할 때 통계적인 개념 - MDE (Minimum Detectable Effect), 유의성 (Significance Level) - 등이 나와서 원리를 알고 쓰는 것이 좋다고 생각합니다.

박원우, 손승연, 박해신, & 박혜상. (2010). 적정 표본크기 (sample size) 결정을 위한 제언. Seoul Journal of Industrial Relations, 21, 51-85. [`링크`](https://s-space.snu.ac.kr/handle/10371/144993)를 읽으며 구체적으로 어떻게 표본 크기를 정해야할 지 감을 잡을 수 있었는데요. 짧게 추천사를 남겨보자면...

* 한글입니다. (엄청난 메리트!)
* 서울대라 그런지.. 왠지 모를 신뢰감이 있고 글도 잘 읽혀요.
* 각 분석 방법론별로 표본 크기 기준 (rule of thumb)를 쉽게 정의했습니다.

그럼 정말 본격적으로 어떻게 표본 크기를 정해야하는지 적어보겠습니다.

-----
### 첫째, 중심극한정리와 대수의 법칙

통계학의 만능키, **중심극한정리 (CLT, Central Limit Theorem)**는 데이터 한 번 만져봤다고 하면 모두 아실만한 개념인데요.

중심극한정리의 정의는 다음과 같습니다.

![](../../images/sample-size-2.png)

핵심은 
* 만약 우리가 많은 수의 표본을 선택하고 그들의 평균을 계산하면, 그 평균들의 분포는 대략적으로 정규 분포를 따르게 된다는 점
* 이렇게 표본을 여러 번 선택하여 그들의 평균을 계산하면, 이 평균들의 분포는 정규 분포에 가까워질 것

입니다. 
* 10명의 학생을 한 번만 뽑는게 아니라 여러 번 뽑아서 각 그룹 (표본)별 평균을 내고, 이를 히스토그램을 그리면 정규 분포와 같은 종 모양이 나오고
* 데이터가 **어떤 분포를 따르든간에** 표본 크기가 크다면 평균들의 분포가 정규 분포라는 것이죠.

정규 분포는 알려진 분포이기 때문에, 중심극한정리에 따라 표본들의 평균을 정규 분포라 하면 ㅇㅇㅇ


표본의 크기가 커질수록, 어떤 분포를 따르든 간에 표본에서 나온 평균의 분포는 정규 분포에 가까워진다.


## References
* 박원우, 손승연, 박해신, & 박혜상. (2010). 적정 표본크기 (sample size) 결정을 위한 제언. Seoul Journal of Industrial Relations, 21, 51-85. [`링크`](https://s-space.snu.ac.kr/handle/10371/144993)




https://www.qualtrics.com/kr/experience-management/research/determine-sample-size/


- 검정력: 효과가 실제로 있는데 있다고 말할 확률
    - 언제 쓰냐면 실험 기획할 때 적정할 실험의 샘플 수가 몇 개인지 정하는 것
    - 유의 수준, 검정력, effect size가 주어졌을 때 적정한 샘플 크기를 구할 수 있음

### A/B 테스트 비교

- [https://www.dynamicyield.com/lesson/bayesian-testing/](https://www.dynamicyield.com/lesson/bayesian-testing/)
- [https://abtestguide.com/calc/](https://abtestguide.com/calc/)
- [https://yozm.wishket.com/magazine/detail/1126/](https://yozm.wishket.com/magazine/detail/1126/)

### Sample Size Calculation

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2014d680-a01b-493f-9e22-f29e44281389/Untitled.png)

- 출처: [https://www.youtube.com/watch?v=KC1nwY7YCUE](https://www.youtube.com/watch?v=KC1nwY7YCUE)
- 계산식: [https://towardsdatascience.com/required-sample-size-for-a-b-testing-6f6608dd330a](https://towardsdatascience.com/required-sample-size-for-a-b-testing-6f6608dd330a)

- 결국 유의수준과 검정력, 원하는 효과 크기만 정한다면, 적절한 sample size를 구할 수 있음
    - 유의 수준은 5% / 검정력은 80% 정도가 default
    - 그런데 비현실적.
    - MDE (Minimum Detectable Effect): 최소한 탐지하고 싶은 Uplift에 대한 얘기 
    e.g. 50% → 55% 라면 5%p / 50% * 100= 10%
- sample size를 계산해주는 사이트
    - ABTesty [https://www.abtasty.com/sample-size-calculator/](https://www.abtasty.com/sample-size-calculator/)
    - Optimizely [https://www.optimizely.com/sample-size-calculator/#/?conversion=50&effect=10&significance=95](https://www.optimizely.com/sample-size-calculator/#/?conversion=50&effect=10&significance=95)

### 적용 사례

- 핵클: [https://docs-kr.hackle.io/docs/interpret-results](https://docs-kr.hackle.io/docs/interpret-results)
- VWO: [https://vwo.com/glossary/bayesian/](https://vwo.com/glossary/bayesian/)
- A/B테스트 Calculator: [https://abtestguide.com/bayesian/](https://abtestguide.com/bayesian/)

### Uplift / Expected Loss / Expected Revenue

- Uplift: (T-C) / C
- Expected Loss: max (C-T)
- Expected Revenue:
- [https://data-marketing-bk.tistory.com/entry/Python베이지안-AB-Test로-기대수익과-기대손실-계산하는-방법](https://data-marketing-bk.tistory.com/entry/Python%EB%B2%A0%EC%9D%B4%EC%A7%80%EC%95%88-AB-Test%EB%A1%9C-%EA%B8%B0%EB%8C%80%EC%88%98%EC%9D%B5%EA%B3%BC-%EA%B8%B0%EB%8C%80%EC%86%90%EC%8B%A4-%EA%B3%84%EC%82%B0%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)
- 기대 수익에 Gamma-Gamma를 사용하는 이유: [https://medium.com/analytics-vidhya/the-gamma-distribution-and-its-applications-in-the-mobile-app-industry-2ba891d8b7f4](https://medium.com/analytics-vidhya/the-gamma-distribution-and-its-applications-in-the-mobile-app-industry-2ba891d8b7f4)