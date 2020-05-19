---
layout: post
title: Brawl Stars Python API 톺아보기
date: 2020-05-16
categories: [Data analysis]
tag: [data-analysis,brawlstats,brawlstars]
comments: true
photos:
    - "../../images/brawl-officialapi.png"

---

* 브롤스타즈 게임 로그로 데이터 분석을 해볼 수 없을까 고민하던 중 엄청난 블로그 두 개를 발견했습니다. novdov님의 브롤스타즈 분석([`[link]`](https://novdov.github.io/data%20anaylsis/2019/08/18/Brawl-Stars-API/))과 Alicia Li님의 분석 ([`[link]`](https://medium.com/@aliciali_7397/brawl-stars-data-analysis-best-worst-brawlers-99eb7684ad8)인데요. 저도 이렇게 데이터를 모아서 직접 분석해보면 의미가 있지 않을까 하여 게임 로그 수집을 시작하였습니다.
* novdov님의 브롤스타즈 분석([`[link]`](https://novdov.github.io/data%20anaylsis/2019/08/18/Brawl-Stars-API/)) 글에도 자세하게 전투 기록을 저장하는 방법에 대해 설명하고 있지만 API가 약간 바뀐 점도 있고, 더 설명하고 싶은 부분도 있어서 저도 이에 대해 포스팅하게 되었습니다.

---

## Introduction

제 브롤스타즈 전투 기록을 주기적으로 저장하기 위해서 필요한 단계는 다음과 같습니다.

1. Brawl Stars API 토큰 발급받기 & Python `brawlstats` 모듈 설치하기
1. AWS EC2 가상환경에서 전투기록 파싱하기
2. AWS EC2 가상환경에서 crontab으로 주기적으로 전투기록 저장하기

---
## 토큰 발급 및 Python `brawlstats` 모듈 설치

브롤스타즈가 기존에는 공식 API와 비공식 API를 둘 다 제공했는데 이제 **공식 API**로 통합되었습니다. **토큰**이란 파이썬 `brawlstats` 모듈에서 API에 접속하기 위한 비밀번호와 같은 것입니다. 자세한 과정은 다음과 같습니다.

1. [브롤스타즈 공식 API](https://developer.brawlstars.com/#/getting-started)에 접속하고 가입합니다.
2. My Keys > Create New Key를 클릭합니다.

    ![](../../images/brawl-officialapi3.png)

3. **KEY NAME**과 **DESCRIPTION**은 적당히 적으시면 되고, **ALLOWED IP ADDRESS**에는 브롤 스타즈 API에 접속할 때 사용하는 IP 주소를 적어줍니다. 이 주소 외에 다른 IP로 접속할 수 없습니다. 네이버에서 "내 IP 주소"라 검색하면 쉽게 현재 접속하고 있는 IP 주소를 알 수 있습니다. 

4. **Create Key**를 클릭하면 토큰이 발급됩니다. 이 토큰은 외부에 공개하지 않도록 주의합시다. 이제 이 토큰을 텍스트 파일로 적당한 곳에 저장해놓습니다.

5. 이제 Anaconda Prompt나 리눅스 커맨드 창에 다음과 같이 `brawlstats` 모듈을 설치합니다.

   ```python
   pip install brawlstats
   ```

---
## 전투 기록 파싱하기

Python의 `brawlstats` 모듈에 대한 공식 문서는 [이 곳](https://brawlstats.readthedocs.io/en/latest/)에서 찾을 수 있습니다.
이 과정에는 novdov님의 블로그와 GitHub의 도움을 많이 받았습니다.

먼저 전투 기록을 파싱하려면 앞서 발급받은 **토큰**과 내 게임 **태그**가 필요합니다. 게임 태그는 브롤 스타즈에 접속해서 좌측 상단의 내 닉네임을 클릭하시면 확인할 수 있습니다.
아래 사진에서 `#URC00YYV`가 제 게임 태그입니다. 

![](../../images/brawl-tag.jpg)

내 전투기록을 불러오려면 태그 가장 앞에 있는 `#`을 제외하고 입력하셔야 합니다. 다음 코드에서 `tag`에 해당합니다.

```python
import brawlstats
from pathlib import Path

# token 저장 경로
token = open(Path("C:/Users/user/Downloads/token.txt")).read().strip()
tag = "URC00YYV"

# API 접속
client = brawlstats.Client(token)
battles=client.get_battle_logs(tag)

```

이제 `battles`에는 최근 25건의 전투 기록이 저장되어 있습니다. `battles`를 출력하면 다음과 같이 클래스 정보가 나오고

```python
battles
> <brawlstats.models.BattleLog at 0x1633c36f408>
```

`battles[0]`로 가장 최근 게임 정보를 확인하면 다음과 같이 Box 형태로 출력됩니다. (~~하필 또 진 기록이네요..하핫~~)


```python
<Box: 
{
    'battle_time': '20200518T063643.000Z', 
    'event': {
        'id': 15000114, 
        'mode': 'gemGrab',
        'map': 'Escape Velocity'
             },
   'battle': 
    {
        'mode': 'gemGrab', 
        'type': 'ranked', 
        'result': 'defeat', 
        'duration': 88, 
        'trophy_change': -6, 
        'star_player': 
        {
            'tag': '#9G20VPG', 
            'name': '엘르트 바바리안', 
            'brawler': {'id': 16000013, 'name': 'POCO', 'power': 10, 'trophies': 599}
        }, 
        'teams': 
        [
            [
                {
                 'tag': '#PQ292Q88', 
                 'name': 'Yamada(極)', 
                 'brawler': {'id': 16000034, 'name': 'JACKY', 'power': 9, 'trophies': 595}
                }, 
                {
                 'tag': '#URC00YYV', 
                 'name': '응찌', 
                 'brawler': {'id': 16000013, 'name': 'POCO', 'power': 10, 'trophies': 595}
                }, 
                {
                 'tag': '#9CJ02PV89', 
                 'name': 'アリの餌食', 
                 'brawler': {'id': 16000004, 'name': 'RICO', 'power': 10, 'trophies': 629}
                }
            ], 

            [
                {
                 'tag': '#9G20VPG', 
                 'name': '엘르트 바바리안', 
                 'brawler': {'id': 16000013, 'name': 'POCO', 'power': 10, 'trophies': 599}
                }, 
                {
                 'tag': '#8GYVRJ0CC', 
                 'name': '찌님', 
                 'brawler': {'id': 16000017, 'name': 'TARA', 'power': 9, 'trophies': 609}
                }, 
                {
                 'tag': '#9VP99RRC', 
                 'name': 'グリズリー森山', 
                 'brawler': {'id': 16000004, 'name': 'RICO', 'power': 10, 'trophies': 594}
                }
            ]
        ]
    }
}>
```

이 기록은 브롤 스타즈 **전투 기록**의 첫 번째에 대응됩니다.

![](../../images/brawl-newest.jpg)

정보를 보면 

1. battle_time: 전투 시간
2. event: 게임 모드의 정보
   * id: 게임 모드의 id
   * mode: 게임 모드
   * map: 젬 그랩의 맵 
3. battle: 전투 기록
   1. mode: 게임 모드
   2. type: 게임 타입
   3. result: victory (승)/defeat (패)
   4. duration: 게임 지속 시간 (초)
   5. trophy_change: 게임 결과로 인한 내 캐릭터의 트로피 변의
   6. star_player: 스타플레이어, 이긴 팀에서 1명 뽑힘
      1. tag: 스타플레이어의 태그
      2. name: 스타플레이어 명
      3. brawler: 스타플레이어의 브롤러 (캐릭터)
      4. teams: 팀 정보
         1. 우리 팀
            1. 팀원 1
            2. 팀원 2
            3. 팀원 3
         2. 상대 팀
            1. 팀원 4
            2. 팀원 5
            3. 팀원 6

의 정보들을 담고 있습니다.

따라서 이를 이용한 파싱