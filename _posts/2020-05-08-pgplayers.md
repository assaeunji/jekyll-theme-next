---
layout: post
title: "[프로그래머스] 완주하지 못한 선수 파이썬 풀이"
date: 2020-05-08
categories: [Python]
tag: [coding-test,hash]
comments: true
photos:
    - "../../images/pg-players.png"
---

* 풀이시간: 20분
* 분류: 해시    
* 링크: [`[link]`](https://programmers.co.kr/learn/courses/30/lessons/42576){: target="_blank"}

----
## Inputs & Outputs

| participants                                      | completion                               | return   |
| :------------------------------------------------ | :--------------------------------------- | :------- |
| ["leo", "kiki", "eden"]                           | ["eden", "kiki"]                         | "leo"    |
| ["marina", "josipa", "nikola", "vinko", "filipa"] | ["josipa", "filipa", "marina", "nikola"] | "vinko"  |
| ["mislav", "stanko", "mislav", "ana"]             | ["stanko", "ana", "mislav"]              | "mislav" |


---
## Idea

`participants`와 `completion`이 한 원소만 차이나기 때문에 (완주하지 못한 선수가 항상 1명) 
1. `completion`에 `"z"*21`을 추가했다. 그 이유는 `participants`와 길이를 맞춰주기 위함이다.
2. `participants`와 `completion`을 이름 순으로 정렬한다.
   1. `completion`의 가장 마지막 원소는 항상 `"z"*21`가 된다. (문제에서 이름 길이가 20자까지 가능하다 해서 21을 곱했다)
3. 이제 각 원소에 대해 비교해서 `False`값이 나오는 인덱스를 뽑아 리턴한다.   



---
## Solution

### My Solution

```python
def solution(participants, completion):
    completion.append("z"*21)

    participants.sort()
    completion.sort()

    idx = list(map(str,[x == y for x, y in zip(sorted(participants), sorted(completion))]))

    return participants[idx.index("False")]
```

### Other Solution

크... 간결하다. `collections`의 `Counter` 함수는 각 원소 이름을 key, 원소 별 개수를 value에 저장하는 딕셔너리를 반환한다.


```python
import collections

def solution(participant, completion):
    answer = collections.Counter(participant) - collections.Counter(completion)
    return list(answer)[0]
```


예를 들어 입력 2의 `participant`와 `completion`으로 `collections.Counter`를 해보면 다음과 같은 원리로 답을 낼 수 있다.w

```python
participants = ["marina","josipa","nikola", "vinko","filipa"]
completion =["josipa","filipa","marina","nikola"]
import collections
print(collections.Counter(participants))
> Counter({'marina': 1, 'josipa': 1, 'nikola': 1, 'vinko': 1, 'filipa': 1})

print(collections.Counter(completion))
> Counter({'josipa': 1, 'filipa': 1, 'marina': 1, 'nikola': 1})

print(collections.Counter(participants)-collections.Counter(completion))
> Counter({'vinko': 1})

print(list(collections.Counter(participants)-collections.Counter(completion))[0])
> vinko
```