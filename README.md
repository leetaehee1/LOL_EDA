# League Of Lengend 승패 예측 분석

# 1. 개 요
리그 오브 레전드(이하 LOL)는 전 세계에서 수많은 게이머들에게 사랑받고 있는 대표적인 컴퓨터 게임으로, e-sports 산업에서는 이미 그 영향력을 확인할 수 있는 대표적인 콘텐츠로 자리매김하고 있다. 특히, 2022년 항저우 아시안 게임에서는 LOL이 정식 종목으로 채택되어 게임의 글로벌한 인지도와 영향력을 증명하였다.

게임의 제작사인 Riot Games는 LOL에 대한 플레이어들의 흥미를 더욱 높이기 위해 게임 데이터를 무료로 제공하고 있다. Riot Games API를 통해 LOL 소환사의 개인적인 게임 정보부터 경기 데이터까지 다양한 정보에 접근할 수 있다[1]. 매일 24시간 동안 수집되는 약 20만 건의 경기 데이터는 풍부한 정보를 포함하고 있으며, 소정의 절차를 거치면 이 데이터에 쉽게 접근할 수 있다[2].

이번 프로젝트에서는 LOL 게임의 대규모 데이터셋 중 하루치 20만 건의 데이터를 활용하여 승패를 예측하는 모델을 개발하고자 한다. 이를 통해 게임의 다양한 측면을 분석하고, 어떤 플레이어 행동이 승리에 어떠한 영향을 미치는지에 대한 통찰력을 얻고자 합니다.

# 2. 데이터
## 2.1 데이터 소스
이번 프로젝트에 활용할 데이터는 온라인 게임 코칭 전문기업인 더매치랩(The Match LAB)에서 가공한 LOL 게임 데이터를 바탕으로 한다. 데이터는 2023년 8월 25일, 9월 15일, 9월 17일 각각 하루 동안 수집된 LOL 경기에 대한 세부 항목들로 구성되어 있다.

## 2.2 탐색적 데이터 분석
데이터는 5대5 솔로 랭크 경기 약 20만 건으로 구성되어 있으며, 포지션별로 데이터가 구분되어 있다. 5가지 포지션에 대한 내용은 다음과 같다.

![롤라인1](https://github.com/leetaehee1/LOL_EDA/assets/79897716/f11495fb-1528-4547-a923-ace72525416f)


* LOL 게임에 대한 간략한 설명
* 영상을 삽입하던지

| 포지션        | 역할 |
|-----|-------------|
| 탑(TOP)     | 팀 내의 최전방 탱커, 이니시에이팅(싸움을 여는 역할) |
| 정글(JUNGLE) | 오브젝트 관리(드래곤, 내셔남작 등), 라인 갱킹, 이니시에이팅 |
| 미드(MID)    | 팀 내의 메인딜러, 모든 싸움의 영향력이 높음 |
| 원딜(ADC)    | 후방에서 원거리 딜링 |
| 서폿(SPT)    | 시야확보 및 제거, 이니시에이팅, 제 2의 정글 |   

 *# 대부분은 이러한 역할을 가지고 있지만 아닌 경우도 존재하긴 한다.*

제공된 데이터의 항목은 총 185개로 구성되어 있으며, 전반적으로 다음과 같은 내용으로 정리해볼 수 있다.

| 항목           | 데이터 속성 |
|--------------|--------|
| 플레이어에 대한 데이터 | kills, deaths, assists, cs, kda ...|
| 팀에 대한 데이터    | baron, dragon, turretkills, goldearned ...    |
| 라인전 전후에대한 데이터 | diffdmg, diffgold, dratio, gratio, kratio ... |

여기서 중요하게 고려되는 항목들은 다음과 같다. (10개 이상)

| 데이터 속성         | 데이터의 의미                                    | 중요한 이유 |
|----------------|--------------------------------------------|--------|
| KDA            | Kill/Death/Assist의 비율                      | 전체적인 플레이어 성과를 측정하는 중요한 지표이다. |
| Dealt          | 챔피언에게 넣은 데미지              |
| Diffdpm        | DPM 격차                       | 전투에서 팀의 강세를 보여주는 중요한 지표이다. |
| Goldearned100  | 블루팀 총 획득 골드                                |
| Goldearned200  | 레드팀 총 획득 골드                                |
| Diffgold       | 팀 간의 Gold 격차                                    | 팀의 경제적 우위를 나타내어 아이템 구매, 경험치, 전투력 등에 영향을 미치며, 큰 차이일수록 전투에서 강세를 보일 수 있다.|
| Dragon         | 드래곤 처치 수                                   |
| Baron          | 내셔남작 처치 수                                  |
| Tower100       | 블루팀 타워 처치 수                                |
| Tower200       | 레드팀 타워 처치 수                                |
| Visionscore100 | 블루팀 시야점수                                   |
| Visionscore200 | 레드팀 시야점수                                   |

## 2.3 데이터 전처리
20만 건의 경기 데이터는 수준 별로 편차가 어느정도 존재할 것으로 예상된다. 따라서 일정 수준 이상의 데이터로 지표의 상관성을 통해 승패예측 모델을 만드는 것이 합리적일 것이다.

이에 이번 프로젝트에서는 다음과 같이 정의되는 LOL게임의 등급체계(tier)를 바탕으로, 모든 플레이어가 플래티넘 이상인 경기를 추출하여 분석해보고자 한다.

* 티어에 대한 정보 (표)
* 티어별 히스토그램
* 플래티넘 이상인 경기의 수

도출된 플래티넘 이상의 경기에 대해서, 핵심 데이터 속성으로 ???, ???, ??? 등을 추출했다. 그 이유는 ~~~때문이다. 그리고 승패를 예측하는 것이기 때문에, 경기 데이터를 기준으로 데이터 속성별 정규화(normalize)를 수행했다. 

## 2.4 데이터 프레임 설계

탐색적 데이터 분석과 데이터 전처리를 통해 다음과 같은 데이터 프레임을 만들고자 한다.

| Id  | 팀  | 주요지표 | TOP | JUG | MID | ADC | SPT |
|-----|-----|---------|-----|---|---|---|---|
| 고유번호 | Red/Blue | kda, dpd, ...| ... |...|

# 3. 승패예측 모델

## 3.1 모델 개요
CNN을 썼다. 단순한 합산을 썼다.

## 3.2 성능
0825
0915
0927
플래티넘 이상 4.5만건
데이터 전체 20만건

## 3.3 소결
* 성능에 대한 의미
* 핵심 데이터 항목에 대한 추정 및 분석

# 4. 결론 및 배운점
