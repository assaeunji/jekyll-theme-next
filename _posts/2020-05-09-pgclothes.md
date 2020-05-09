---
layout: post
title: "[프로그래머스] 위장 파이썬 풀이"
date: 2020-05-09
categories: [Python]
tag: [coding-test, hash]
comments: true
photos:
    - "../../images/pg-clothes.png"

---


* 풀이시간: 20분
* 분류: 해시    
* 링크: [`[link]`](https://programmers.co.kr/learn/courses/30/lessons/42578){: target="_blank"}

----
## Inputs & Outputs

| clothes                                                                                    | return |
| :----------------------------------------------------------------------------------------- | :----- |
| [["yellow_hat", "headgear"], ["blue_sunglasses", "eyewear"], ["green_turban", "headgear"]] | 5      |
| [["crow_mask", "face"], ["blue_sunglasses", "face"], ["smoky_makeup", "face"]]             | 3      |

----
## Idea

이 문제의 아이디어는 **조합**을 어떻게 구성하느냐에 따라 있다.
예를 들어 `"headgear"`의 옷 종류가 2가지고, `"eyewear"`의 종류가 3가지라고 하면, 가능한 경우의 수는
$(2+1) \times (3+1) -1$이 된다. 왜냐하면 `"headgear"`의 선택지는 2개이지만 **안 입은 경우**도 고려하면 3 가지가 되고, 
`"eyewear"`도 마찬가지로 **안 입은 경우**를 고려하면 4 가지가 된다. 마지막에 1을 빼주는 이유는 아예 아무 것도 안입었을 경우를 빼줘야 하기 때문이다.

---
## Solution

### Solution 1

간단하게 구현하는 방법은 역시 `collections.Counter`를 이용하는 것이다. 
1. `clothes`에서 각 요소 별 두 번째에 해당하는 종류 `cate`에 대한 빈도를 계산하고,
2. 빈도를 나타내는 `values`들에 대해서 
   1. (빈도 + 1)을 차례대로 곱해주면 된다.
3. 이후 전라 상태를 제외해야 하므로 `answer-1`을 해준다.

```python
def solution(clothes):
  answer=1
  import collections 
  closet = collections.Counter([cate for name, cate in clothes])        
  for value in closet.values():
      answer*= (value+1)
  return answer-1
```

### Solution 2

이보다 더 간단한 솔루션은 `reduce`를 이용하는 것이다.

```python
def solution(clothes):
   answer=1
   import collections 
   from functools import reduce
   closet = collections.Counter([kind for name, kind in clothes])
   # reduce(집계함수, 순회 가능한 데이터, 초기값)
   return reduce(lambda x, y: x*(y+1), closet.values(),1) - 1
```


빈도수에서 `for`문을 돌리는 대신에 `reduce`함수를 이용하면 `for`문이 돌듯이 누적해서 함수를 계산해준다.

예를 들어, 다음의 예제에서 6이 나오는 이유는
1. `x=1`, `y=2`가 되어 (1*2)를 계산하고
2. `x=(1*2)`, `y=3`이 되어 `(1*2)*3`을 계산해 결국 6이 된다.

```python
reduce(lambda x,y:x*y, [1,2,3])
> 6
```

근데 위의 Solution은 `reduce`에 세 번째 인자 `1`이 있다. 이는 **초기값**을 의미하는데,
`x`에 배열의 첫번째 원소가 오지 않고 초기값이 먼저 오는 것을 의미한다.

즉 위의 예제에서 
1. `x=10`, `y=1`
2. `x=(10*1)`, `y=2`
3. `x=(10*1*2)`, `y=3`이 되기 때문에 `60`을 출력한다.

```python
reduce(lambda x,y:x*y, [1,2,3],10)
> 60
```

따라서 위의 솔루션에 세 번째 인자 `1`을 써주지 않으면 다른 답이 나온다.