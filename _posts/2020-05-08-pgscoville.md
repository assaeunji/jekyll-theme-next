---
layout: post
title: "[프로그래머스] 더 맵게! 파이썬 풀이"
date: 2020-05-08
categories: [Python]
tag: [coding-test,heap,sorting-algorithm]
comments: true
photos:
    - "../../images/pg-scoville.png"
---

* 풀이시간: 20분
* 분류: 힙    
* 링크: [`[link]`](https://programmers.co.kr/learn/courses/30/lessons/42626){: target="_blank"}

----
## Inputs & Outputs

| scoville             | K    | return |
| :------------------- | :--- | :----- |
| [1, 2, 3, 9, 10, 12] | 7    | 2      |

---
## Idea

처음으로 힙 구조를 접했는데, 이게 힙이 필요한 이유는 
`scoville`의 길이는 100만 이하, `K`는 최대 10억까지 가기 때문에 시간 복잡도를 낮출 방법이 필요하다.

따라서 $O(N\log N)$의 시간 복잡도를 갖는 힙 정렬을 사용하였다.



---
## Solution

1. `heapq` 라이브러리를 불러오고
2. `scoville`에 `heapify`를 통해 힙 정렬을 수행한다.  
3. 이제 무한 반복인데:
   1. `first`에 제일 작은 값을 꺼내고
   2. 이 최솟값이 `K`보다 크다면 반복을 멈춘다.
   3. 만약 `scoville`의 길이가 0이 된다면 -1을 반환한다.
   4. 두번째 값 `second`를 또 꺼내고
   5. `first+second*2`로 새로운 값을 `scoville`에 다시 넣는다. 
   6. `answer`를 1씩 증가시킨다.
4. 결과 `result` 반환

```python
import heapq as hq

def solution(scoville, K):

    hq.heapify(scoville)
    answer = 0
    while True:
        first = hq.heappop(scoville)
        if first >= K:
            break
        if len(scoville) == 0:
            return -1
        second = hq.heappop(scoville)
        hq.heappush(scoville, first + second*2)
        answer += 1  

    return answer
```