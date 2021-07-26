---
layout: post
title: Python brawlstats으로 브롤 스타즈 전투 기록 수집하기
date: 2020-05-20
categories: [Data Analysis]
tag: [brawlstats,brawlstars,parsing,python,data-collection,aws,ec2]
comments: true
photos:
    - "../../images/brawl-officialapi.png"

---

* 마침 Python의 `brawlstats` 라이브러리가 브롤 스타즈 API wrapper를 제공하고 있어 전투 기록을 모으는 게 가능해졌습니다. 따라서 이 포스팅에서는 다음과 같이 전투 기록을 주기적으로 모으는 방법에 대해 설명하고자 합니다.
* 관련 코드는 [이 곳](https://github.com/assaeunji/brawlstars-analysis){:target="_blank"}에서 확인하실 수 있습니다.

---

## Introduction

     이 맵은 이 조합 (ex. 탱, 딜, 힐)이다!
     이 브롤러의 스타 파워 or 가젯은 이 맵에서 매우 강력하다 / 미약하다!
     플레이 시간이 길수록 이길 확률은 낮아진다!

게임을 오래 하다 보면 위와 같이 가설을 갖곤 합니다. 브롤 스타즈 장기 유저 & 나름의 데이터 분석가로서 이런 가설들을 실제로 검증하고 싶다는 생각을 항상 해왔습니다.

마침 Python의 `brawlstats` 라이브러리가 브롤 스타즈 API wrapper를 제공하고 있어 전투 기록을 모으는 게 가능해졌습니다. 따라서 이 포스팅에서는 다음과 같이 전투 기록을 주기적으로 모으는 방법에 대해 설명하고자 합니다.

1. Brawl Stars API 토큰 발급받기 & Python `brawlstats` 설치하기
2. AWS EC2 가상환경에서 전투기록 파싱하기
3. AWS EC2 가상환경에서 crontab으로 주기적으로 전투기록 저장하기


본격적으로 들어가기 전에 설명드릴 개념은 다음과 같습니다.

![](../../images/brawl-info.png)

1. **젬 그랩(Gem Grab)**: 3 대 3으로 팀을 이루고, 팀 총합 보석을 10개 이상 얻고 카운트다운이 종료될 때까지 버틴 팀이 승리하는 게임
3. **맵**: 젬 그랩에서 싸우는 장소, 하루 정도 지나면 맵이 바뀌고, 맵에 따라 맞는 브롤러들이 다름
2. **브롤러**: 젬 그랩에서 싸우는 캐릭터, 유저 별로 가진 브롤러 중 하나를 택해 젬 그랩에 참여할 수 있음
3. **트로피**: 게임에서 이길 때마다 브롤러 트로피가 8씩 더해지고, 지면 브롤러의 랭크에 따라 감소됨 
4. **스타파워** & **가젯**
   1. 스타 파워는 브롤러 파워레벨이 10일 때 가지는 추가 능력 (패시브 스킬)
   2. 가젯은 브롤러 파워레벨이 7 이상일 때 가지는 추가 능력이고 게임 당 2~3번씩 발동 가능. 보통은 스타 파워에 비해 미미하나 브롤러에 따라 매우 강력할 수 있음

---
## 토큰 발급 및 Python `brawlstats` 설치

브롤스타즈가 기존에는 공식 API와 비공식 API를 둘 다 제공했는데 이제 **공식 API**로 통합되었습니다. **토큰**이란 파이썬 `brawlstats` 모듈에서 API에 접속하기 위한 비밀번호와 같은 것입니다. 자세한 과정은 다음과 같습니다.

1. [브롤스타즈 공식 API](https://developer.brawlstars.com/#/getting-started){:target="_blank"}에 접속하고 가입합니다.
2. My Keys > Create New Key를 클릭합니다.

    ![](../../images/brawl-officialapi3.png)

3. **KEY NAME**과 **DESCRIPTION**은 적당히 적으시면 되고, **ALLOWED IP ADDRESS**에는 브롤 스타즈 API에 접속할 때 사용하는 IP 주소를 적어줍니다. 이 주소 외에 다른 IP로 접속할 수 없습니다. 네이버에서 "내 IP 주소"라 검색하면 쉽게 현재 접속하고 있는 IP 주소를 알 수 있습니다. 

4. **Create Key**를 클릭하면 토큰이 발급됩니다. 이 토큰은 외부에 공개하지 않도록 주의합시다. 이제 이 토큰을 텍스트 파일로 적당한 곳에 저장해놓습니다.

5. 리눅스 커맨드 창에 다음과 같이 `brawlstats` 모듈을 설치합니다.

   ```python
   pip install brawlstats
   ```

---
## AWS EC2 가상환경에서 전투기록 파싱하기

Python의 `brawlstats` 라이브러리에 대한 공식 문서는 [이 곳](https://brawlstats.readthedocs.io/en/latest/){:target="_blank"}에서 찾을 수 있습니다.

전투 기록을 파싱하려면 앞서 발급받은 **토큰**과 내 게임 **태그**가 필요합니다. 게임 태그는 브롤 스타즈에 접속해서 **좌측 상단**의 제 닉네임을 클릭하시면 확인할 수 있습니다.
아래 사진에서 `#URC00YYV`가 제 게임 태그입니다. 

![](../../images/brawl-tag.png)

전투 기록을 불러오려면 태그 가장 앞에 있는 `#`을 제외하고 입력하셔야 합니다. 다음 코드에서 `tag`에 해당합니다.

```python
import brawlstats
from pathlib import Path

# token 저장 경로
token = open(Path("C:/Users/user/Downloads/token.txt")).read().strip()
tag = "URC00YYV"

# API 접속
client = brawlstats.Client(token)
```

제 경우 위에서 발급받은 토큰을 `Downloads` 폴더에 `token.txt` 텍스트 형태로 넣어놓고 불러왔습니다.


---
### `get_battle_logs`: 최근 25건의 전투 기록 불러오기

![](../../images/brawl-newest.jpg)

`brawlstats`의 `get_battle_logs` 함수를 이용하면 위와 같은 최근 25건의 전투 기록을 로그 형태로 불러올 수 있습니다.

```python
battles=client.get_battle_logs(tag)
```

`client`에 `get_battle_logs`함수를 적용하면 해당 `tag`를 가진 유저의 최근 25 건의 전투 기록을 불러옵니다. `battles`를 출력하면 `<brawlstats.models.BattleLog at 0x1633c36f408>`라 클래스 정보가 나오고, `battles[0]`로 가장 최근 게임 정보를 확인하면 다음과 같이 Box 형태로 출력됩니다. (~~하필 또 진 기록이네요..하핫~~)

<details>
<summary>펼쳐 보기</summary>
<div markdown="1">
<br/>

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
</div>
</details>


이 기록은 브롤 스타즈 **전투 기록**의 첫 번째에 대응됩니다. `<Box>`형태로 되어 있는 전투 기록을 살펴보면 다음의 내용을 포함하고 있습니다.

1. battle_time: 전투 시간 (UTC +0:00)
2. event: 게임 모드의 정보
   * id: 게임 모드의 id
   * mode: 게임 모드
   * map: 젬 그랩의 맵 
3. battle: 전투 기록
   1. mode: 게임 모드
   2. type: 게임 타입
   3. result: victory (승)/defeat (패)
   4. duration: 게임 지속 시간 (초)
   5. trophy_change: 게임 결과로 인한 내 캐릭터의 트로피 변화
   6. star_player: 스타플레이어, 이긴 팀에서 1명 뽑힘
      1. tag: 스타플레이어의 태그
      2. name: 스타플레이어의 이름
      3. brawler: 스타플레이어의 브롤러
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

---
### `get_profile`: 유저 정보 불러오기

![](../../images/brawl-user.png)

`brawlstats`의 `get_profile`함수를 이용하면 위와 같이 **모드별 승리 횟수**나 **브롤러 보유 정보**에 대해 불러올 수 있습니다.

```python
profile = client.get_profile(tag)
```

`profile`을 출력하면 `<Player object name='응찌' tag='#URC00YYV'>`라 나오고, `profile.raw_data`라 입력하면 다음과 같이 제 모든 게임 정보들이 나옵니다.

<details>
<summary>펼쳐 보기</summary>
<div markdown="1">
<br/>

```python
{'tag': '#URC00YYV',
 'name': '응찌',
 'nameColor': '0xffffce89',
 'trophies': 13455,
 'highestTrophies': 13686,
 'highestPowerPlayPoints': 300,
 'expLevel': 133,
 'expPoints': 92316,
 'isQualifiedFromChampionshipChallenge': False,
 '3vs3Victories': 5907,
 'soloVictories': 2,
 'duoVictories': 9,
 'bestRoboRumbleTime': 0,
 'bestTimeAsBigBrawler': 0,
 'club': {'tag': '#CPUPP8LC', 'name': '츠냥응찌월드'},
 'brawlers': [{'id': 16000000,
   'name': 'SHELLY',
   'power': 5,
   'rank': 15,
   'trophies': 319,
   'highestTrophies': 323,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000001,
   'name': 'COLT',
   'power': 10,
   'rank': 21,
   'trophies': 542,
   'highestTrophies': 559,
   'starPowers': [{'id': 23000138, 'name': 'Magnum Special'},
    {'id': 23000077, 'name': 'Slick Boots'}],
   'gadgets': [{'id': 23000273, 'name': 'Speedloader'}]},
  {'id': 16000002,
   'name': 'BULL',
   'power': 5,
   'rank': 11,
   'trophies': 176,
   'highestTrophies': 178,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000003,
   'name': 'BROCK',
   'power': 10,
   'rank': 22,
   'trophies': 523,
   'highestTrophies': 635,
   'starPowers': [{'id': 23000079, 'name': 'Incendiary'},
    {'id': 23000150, 'name': 'Rocket No. Four'}],
   'gadgets': []},
  {'id': 16000004,
   'name': 'RICO',
   'power': 4,
   'rank': 16,
   'trophies': 363,
   'highestTrophies': 371,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000006,
   'name': 'BARLEY',
   'power': 8,
   'rank': 20,
   'trophies': 518,
   'highestTrophies': 536,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000007,
   'name': 'JESSIE',
   'power': 10,
   'rank': 22,
   'trophies': 601,
   'highestTrophies': 646,
   'starPowers': [{'id': 23000083, 'name': 'Energize'},
    {'id': 23000149, 'name': 'Shocky'}],
   'gadgets': []},
  {'id': 16000008,
   'name': 'NITA',
   'power': 10,
   'rank': 22,
   'trophies': 573,
   'highestTrophies': 605,
   'starPowers': [{'id': 23000136, 'name': 'Hyper Bear'},
    {'id': 23000084, 'name': 'Bear With Me'}],
   'gadgets': []},
  {'id': 16000009,
   'name': 'DYNAMIKE',
   'power': 7,
   'rank': 21,
   'trophies': 543,
   'highestTrophies': 561,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000010,
   'name': 'EL PRIMO',
   'power': 7,
   'rank': 15,
   'trophies': 325,
   'highestTrophies': 325,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000011,
   'name': 'MORTIS',
   'power': 1,
   'rank': 5,
   'trophies': 40,
   'highestTrophies': 40,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000012,
   'name': 'CROW',
   'power': 8,
   'rank': 21,
   'trophies': 548,
   'highestTrophies': 567,
   'starPowers': [],
   'gadgets': [{'id': 23000243, 'name': 'Defense Booster'}]},
  {'id': 16000013,
   'name': 'POCO',
   'power': 10,
   'rank': 22,
   'trophies': 543,
   'highestTrophies': 641,
   'starPowers': [{'id': 23000144, 'name': 'Screeching Solo'},
    {'id': 23000089, 'name': 'Da Capo!'}],
   'gadgets': [{'id': 23000267, 'name': 'Tuning Fork'}]},
  {'id': 16000014,
   'name': 'BO',
   'power': 10,
   'rank': 22,
   'trophies': 623,
   'highestTrophies': 640,
   'starPowers': [{'id': 23000148, 'name': 'Snare a Bear'},
    {'id': 23000090, 'name': 'Circling Eagle'}],
   'gadgets': [{'id': 23000263, 'name': 'Super Totem'}]},
  {'id': 16000015,
   'name': 'PIPER',
   'power': 5,
   'rank': 18,
   'trophies': 399,
   'highestTrophies': 422,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000016,
   'name': 'PAM',
   'power': 7,
   'rank': 20,
   'trophies': 506,
   'highestTrophies': 524,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000018,
   'name': 'DARRYL',
   'power': 2,
   'rank': 8,
   'trophies': 111,
   'highestTrophies': 111,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000019,
   'name': 'PENNY',
   'power': 10,
   'rank': 23,
   'trophies': 567,
   'highestTrophies': 666,
   'starPowers': [{'id': 23000142, 'name': 'Balls of Fire'},
    {'id': 23000099, 'name': 'Last Blast'}],
   'gadgets': [{'id': 23000248, 'name': 'Pocket Detonator'}]},
  {'id': 16000020,
   'name': 'FRANK',
   'power': 5,
   'rank': 17,
   'trophies': 406,
   'highestTrophies': 416,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000021,
   'name': 'GENE',
   'power': 7,
   'rank': 20,
   'trophies': 542,
   'highestTrophies': 542,
   'starPowers': [],
   'gadgets': [{'id': 23000252, 'name': 'Lamp Blowout'}]},
  {'id': 16000022,
   'name': 'TICK',
   'power': 10,
   'rank': 21,
   'trophies': 525,
   'highestTrophies': 591,
   'starPowers': [{'id': 23000114, 'name': 'Well Oiled'}],
   'gadgets': [{'id': 23000253, 'name': 'Backup Mine'}]},
  {'id': 16000024,
   'name': 'ROSA',
   'power': 8,
   'rank': 20,
   'trophies': 518,
   'highestTrophies': 537,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000025,
   'name': 'CARL',
   'power': 10,
   'rank': 21,
   'trophies': 505,
   'highestTrophies': 578,
   'starPowers': [{'id': 23000129, 'name': 'Power Throw'},
    {'id': 23000145, 'name': 'Protective Pirouette'}],
   'gadgets': []},
  {'id': 16000026,
   'name': 'BIBI',
   'power': 3,
   'rank': 14,
   'trophies': 283,
   'highestTrophies': 289,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000027,
   'name': '8-BIT',
   'power': 6,
   'rank': 20,
   'trophies': 519,
   'highestTrophies': 527,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000029,
   'name': 'BEA',
   'power': 4,
   'rank': 16,
   'trophies': 348,
   'highestTrophies': 352,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000030,
   'name': 'EMZ',
   'power': 9,
   'rank': 21,
   'trophies': 519,
   'highestTrophies': 565,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000031,
   'name': 'MR. P',
   'power': 8,
   'rank': 21,
   'trophies': 565,
   'highestTrophies': 565,
   'starPowers': [],
   'gadgets': [{'id': 23000269, 'name': 'Service Bell'}]},
  {'id': 16000032,
   'name': 'MAX',
   'power': 5,
   'rank': 18,
   'trophies': 403,
   'highestTrophies': 427,
   'starPowers': [],
   'gadgets': []},
  {'id': 16000034,
   'name': 'JACKY',
   'power': 6,
   'rank': 20,
   'trophies': 502,
   'highestTrophies': 502,
   'starPowers': [],
   'gadgets': []}]}
```
</div>
</details>


---
### Python으로 전투 기록 파싱하기

따라서 이를 이용해 젬 그랩의 전투 기록을 파싱하는 함수는 다음과 같습니다.


```python

def parse_single_user(self,tag):
    parsed_logs = []
    battles = self.client.get_battle_logs(tag)

    # 25건의 전투에 대해 반복
    for battle in battles:
        # 젬그랩 모드만 추출
        if battle["event"]["mode"] == "gemGrab":
            
            battle_info = {
                "battle_time": dateutil.parser.parse(battle['battle_time']).astimezone(pytz.timezone("Asia/Seoul")),
                "map_id": battle['event']['id'],
                "battle_map": battle['event']['map'],
                "duration": battle['battle']['duration']
            }

            star_player_tag = battle['battle']['star_player']['tag']
            teams           = battle['battle']['teams']

            # battle의 2개의 팀에 대해 반복
            for team in teams:

                # players: 해시태그 (#)만 제외한 팀 내의 3 명의 유저의 태그
                players = [log['tag'][1:] for log in team]
                side = 1 if tag in players else 0 # 1: 우리 팀, 2: 상대 팀

                # 1개 팀 내의 3명의 유저에 대해 반복
                for player in team:

                    # 유저마다 프로파일을 가져오고 브롤러 정보 트로피에 따라 정렬
                    player_info     = self.client.get_profile(player['tag'][1:])
                    player_brawlers = sorted(player_info.brawlers, key = lambda x: x['trophies'],reverse=True)
                    
                    # 유저가 가진 브롤러 중 해당 게임에 쓴 브롤러의 정보 (가젯 및 스타파워 여부)를 가져옴
                    for brawler in player_brawlers:
                        if brawler["id"] == player["brawler"]["id"]:
                            gadget     = 1 if brawler["gadgets"] else 0
                            star_power = 1 if brawler["star_powers"] else 0

                    # 팀 정보 저장
                    team_info = {
                            "player_tag" : player['tag'],
                            "player_name": player["name"],
                            "player_trophies": player_info.trophies,
                            "player_top_brawler": player_brawlers[0]["name"],
                            "star_player": 1 if player["tag"] == star_player_tag else 0,
                            "team": side,
                            "result": battle["battle"]["result"],
                            "brawler_id": player["brawler"]["id"],
                            "brawler_name": player["brawler"]["name"],
                            "brawler_power": player["brawler"]["power"],
                            "brawler_star_power": star_power,
                            "brawler_gadget": gadget,
                            "brawler_trophies": player["brawler"]["trophies"]
                    }
                    # battle_info와 team_info merge 후 parsed_logs 리스트에 append
                    parsed_logs.append({**battle_info,**team_info})
    return parsed_logs
```

![](../../images/brawl-data.png)

이렇게 저장된 정보는 총 17개의 변수를 가지고, 유저 당 최대 150개 (25번의 전투 $\times$ 6명)의 행을 갖게 됩니다. 
이를 확장해 여러 유저의 전투 기록을 파싱하는 코드를 다음과 같이 작성했습니다.

```python
    def parse_all_users(self,tags):

        # tags가 하나일 경우
        if type(tags) ==str:
            parsed_logs = parse_single_user(self,tags)

        # tags가 리스트 형태로 제공될 경우 (여러 유저의 태그일 경우)
        else:
            parsed_logs = []
              
            for tag in tqdm(tags[1:]):
                try:
                    parsed_logs.extend(parse_single_user(self,tag))
                except:
                    pass

        return parsed_logs
```

가끔 로그를 모으다 오류가 나는 경우가 있어 그 경우는 `try`와 `except`구문으로 `for`문을 멈추지 않고 다음 유저로 넘어가도록 했습니다.

---
## AWS EC2 가상환경에서 crontab으로 주기적으로 전투기록 저장하기

윈도우에서도 작업 스케줄러나 cron으로 원하는 시간마다 파이썬 코드가 실행되도록 예약 작업을 할 수 있지만 ([참조](http://hleecaster.com/task-scheduling-in-windows-using-cron/){:target="_blank"}), 항상 컴퓨터를 켜놓아야 되는 부담 때문에 AWS EC2 가상환경에서 crontab을 이용하기로 했습니다. crontab에 대한 자세한 설명과 사용 방법은 [이 포스팅](https://assaeunji.github.io/aws/2020-05-18-crontab/){:target="_blank"}을 참조하시면 됩니다. crontab을 이용해 주기적으로 전투기록을 저장하는 방법은 다음과 같습니다.

1. 위의 함수들을 `brawlparser`라는 `Class`에 저장해놓고 `bsparser.py`로 저장
2. `main.py`에 `crontab`을 실행할 파서 작성 
3. AWS EC2 커맨드 창에서 `crontab -e`로 crontab에 진입
4. `Shift + G`, `i`를 차례대로 누름 $\rightarrow$ 다음과 같이 작성 $\rightarrow$ `Esc`, `:wq` 차례대로 눌러 crontab 저장

    ```
    0 17 * * * sudo python3 main.py --token_path token --output_dir ./rankers-data --tag URC00YYV --tag_path rankers.csv
    0 17 * * * sudo python3 main.py --token_path token --output_dir ./my-data --tag URC00YYV
    ```
 
5. 데이터를 저장할 경로인 `output_dir`로 rankers-data 폴더와 my-data 폴더 생성




`main.py`는 다음과 같이 argument로 4개를 입력하도록 만들었습니다.

```python
import argparse
from pathlib import Path
import pandas as pd
from bsparser import brawlparser
from datetime import datetime

def get_datetime(fmt="%Y%m%d%H%M"):
 return datetime.strftime(datetime.now(), fmt)

def get_parser(_=None):
 parser = argparse.ArgumentParser()
 parser.add_argument("--token_path", type=str, help="path of API token")
 parser.add_argument("--output_dir", type=str, help="output directory")
 parser.add_argument("--tag_path", type=str, help="path of tags")
 parser.add_argument("--tag", type=str, help="initial tag")
 return parser


if __name__ == "__main__":
 args, _ = get_parser().parse_known_args()
 tags = pd.read_csv(Path(args.tag_path).expanduser())['0'].tolist() if args.tag_path else args.tag

 print(args)
 
 # expanduser(): 절대경로로 대체
 init        = brawlparser(Path(args.token_path).expanduser(), args.tag)
 all_logs    = init.parse_all_users(tags)
 
 df_all_logs = pd.DataFrame(all_logs)
 df_all_logs.to_csv(Path(args.output_dir).expanduser().joinpath(f"log_{get_datetime()}.csv"), index=False) 
```

* `token_path`: 앞서 발급받은 토큰의 경로
* `output_dir`: 데이터를 저장하는 경로
*  `tag_path`: 태그가 있는 경로. 저는 상위 200명의 랭커들의 태그를 `rankers.csv`에 저장해놓고, 이를 `tags`로 불러왔습니다.
*  `tag`: Python 브롤 스타즈 API에 접속할 태그. 저는 제 태그인 `URC00YYV`를 넣었습니다.

만약 제 전투기록만 불러오고 싶다면 `tag_path`의 flag를 쓰지 않으면 됩니다! 이 후 crontab에는 매일 오후 5시 (젬그랩 맵이 바뀌는 시간)에 저장하도록 했습니다. 
전체 코드는 [이 곳](https://github.com/assaeunji/brawlstars-analysis){:target="_blank"}에서 확인하실 수 있습니다. 이렇게 한 열흘 간 데이터를 모아서 여러 주제로 분석해볼 예정입니다. 재밌는 인사이트가 나오면 좋겠군요!



---
## References
* 브롤스타즈 공식 API [`[link]`](https://developer.brawlstars.com/#/getting-started){:target="_blank"}
* Python BrawlStats Library Document [`[link]`](https://brawlstats.readthedocs.io/en/latest/){:target="_blank"}
* novdov님의 브롤스타즈 Python API로 전투 기록 확인하기 [`[link]`](https://novdov.github.io/data%20anaylsis/2019/08/18/Brawl-Stars-API/){:target="_blank"}
