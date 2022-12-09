---
layout: single
title: "[R 야구 데이터] baseballr 패키지로 statcast 트래킹 데이터 분석"
excerpt: "baseballr 패키지는 Baseball Savant의 statcast 데이터를 제공한다. 이 패키지로 오타니, 무키 베츠, 블게주의 데이터를 분석해보자"

published: true

categories:
  - R

tags: [R, baseball, baseballr, Baseball Savant]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2022-12-09 13:50:00
last_modified_at: 2022-12-09 13:50:00
---

<br>

## Statcast란?
Statcast는 방대한 양의 최첨단 야구 트래킹 데이터이다. 메이저리그 야구장에 설치된 카메라와 레이더들로 경기장에서 벌어지는 투구와 타구, 주자의 움직임 야수의 움직임 등 다양한 플레이들을 정량 데이터로 남긴다. [MLB 공식 사이트에서 Statcast를 소개하는 글](https://www.mlb.com/glossary/statcast)을 확인할 수 있다.

<br>

## 라이브러리 설치

분석을 위한 초기 설정 명령어다. 필자는 baseballr 패키지를 다운 받아 Baseball Savant에서 제공되는 statcast 데이터를 다룰 것이다.
baseballr 패키지에 대한 reference는 아래 링크를 참조하자.
<br>
[baseballr 홈페이지 링크](https://billpetti.github.io/baseballr/index.html)

```r
install.packages("Lahman")
install.packages("dplyr") 
install.packages("baseballr")
install.packages("patchwork") #그래프를 이어붙이게 해주는 패키지다.
library(Lahman)
library(dplyr)
library(ggplot2) 
library(baseballr)
library(patchwork)
```

이제 선수 개인의 statcast data를 찾아오자. `statcast_search` 함수로 원하는 선수의 원하는 시점 stacast를 불러올 수 있다. 
이 때 선수 고유의 playerid를 알아야만 하는데, 이는 `playerid_lookup` 함수로 찾을 수 있다.

```r
playerid_lookup("Ohtani")

mookie <- statcast_search(start_date = "2021-01-01", 
                          end_date = "2021-12-31", 
                          playerid = 605141, 
                          player_type = 'batter')


altuve <- statcast_search(start_date = "2021-01-01", 
                          end_date = "2021-12-31", 
                          playerid = 514888, 
                          player_type = 'batter')

vladimir <- statcast_search(start_date = "2021-01-01", 
                          end_date = "2021-12-31", 
                          playerid = 665489, 
                          player_type = 'batter')

ohtani <- statcast_search(start_date = "2021-01-01", 
                            end_date = "2021-12-31", 
                            playerid = 660271, 
                            player_type = 'pitcher')
```



statcast search 각 변수명에 대한 설명은 아래 링크를 통해 확인 가능하다.

[Baseball Savant Statcast Search CSV Documentation](https://baseballsavant.mlb.com/csv-docs)

---

<br>

## 오타니 구종 구사율 파이차트

❓ **상황.** 
```
2021시즌 투수 오타니의 0-0count, full-count의 상황에서 구종 구사율 파이차트를 그려라.
```

💡 **코드.**
```r
ohtani <- ohtani %>%
  mutate(ballcount = case_when(balls==0&strikes==0~"0-0",
                               balls==1&strikes==0~"1-0",
                               balls==2&strikes==0~"2-0",
                               balls==3&strikes==0~"3-0",
                               balls==0&strikes==1~"0-1",
                               balls==1&strikes==1~"1-1",
                               balls==2&strikes==1~"2-1",
                               balls==3&strikes==1~"3-1",
                               balls==0&strikes==2~"0-2",
                               balls==1&strikes==2~"1-2",
                               balls==2&strikes==2~"2-2",
                               balls==3&strikes==2~"3-2"))

# 카운트를 인수로 입력하면 그 카운트의 구종 구사율 파이차트를 그려주는 함수
create_graph <-function(b,s) {
  ohtani_bc <- ohtani %>%
    filter(ballcount==paste0(b, "-", s)) %>% #함수 인수에 따라 조건문 바뀌도록
    group_by(pitch_type) %>% #구종을 기준으로 그룹화
    summarise(pitch_type_count=n()) #구종의 빈도를 계산
  ggplot(ohtani_bc, aes(x='', y=pitch_type_count, fill=pitch_type)) +
    geom_col(width=1) +
    coord_polar('y') + #원그래프 만들기 위해 축을 y로
    ggtitle(paste0(b,"-",s,"count")) + #그래프 제목
    labs(fill = "pitch_type")
}

count00 <- create_graph(b=0,s=0)
count32 <- create_graph(b=3,s=2)

count00+count32 #양옆으로 이어붙이기
```

<img src="https://user-images.githubusercontent.com/115082062/206605931-4f585079-8541-4aee-839e-3ade44655b5e.png">


---

<br>

## 오타니 구종별 땅볼 창출률

❓ **상황.** 
```
2021시즌 투수 오타니의 구종별 땅볼 창출률을 구해보자. 
가령 포심의 땅볼 창출률은 ‘땅볼로 연결된 포심 수/인플레이 타구로 연결된 포심 수’이다.
```

💡 **코드.**
```r
ohtani_pitch_name <- ohtani %>%
  filter(type=='X') %>% #인플레이 타구만 추출
  group_by(pitch_name) %>% #구종별 그룹화
  summarise(pitch_name_count = n()) #구종별 빈도

ohtani_ground <- ohtani %>%
  filter(bb_type=='ground_ball') %>% #땅볼만 추출
  group_by(pitch_name) %>% #구종별 그룹화
  summarise(ground_pitch_name_count=n()) %>% #구종별 땅볼로 연결된 개수
  left_join(ohtani_pitch_name, by=('pitch_name')) %>% #앞선 테이블과 결합
  mutate(ground_product_rate=ground_pitch_name_count/pitch_name_count) %>% #땅볼 창출률 변수 추가
  arrange(-ground_product_rate)

ohtani_ground

#결과
pitch_name      ground_pitch_name_count  pitch_name_count  ground_product_rate
  <chr>                             <int>            <int>               <dbl>
1 Split-Finger                         34               52               0.654
2 4-Seam Fastball                      66              129               0.512
3 Cutter                               20               53               0.377
4 Slider                               29               79               0.367
5 Curveball                             1               10               0.1

#스플리터의 땅볼 창출능력이 가장 높고, 포심이 그 뒤를 잇고 있다.

#추가적으로, 오타니의 인플레이타구 중 그라운드볼의 비율을 확인해보니 0.464였다.
#즉 오타니의 높은 땅볼 비율에 스플리터가 크게 공헌하고 있음을 알 수 있다.
ohtani %>%
  filter(type=='X') %>%
  group_by(bb_type) %>%
  summarise(bb_type_count=n()) %>%
  mutate(total_count=sum(bb_type_count),rate=bb_type_count/total_count) %>%
  arrange(-rate)

bb_type     bb_type_count  total_count   rate
  <chr>               <int>       <int>  <dbl>
1 ground_ball           150         323  0.464 
2 fly_ball               77         323  0.238 
3 line_drive             72         323  0.223 
4 popup                  24         323  0.0743
```

---

<br>

## 무키 베츠 타구속도, 발사각 산점도


❓ **상황.**
```
2021시즌 무키 베츠의 데이터를 이용하여 x축을 타구속도, y축을 발사각으로 하는 산점도를 그려보자. 
모든 인플레이 타구를 데이터로 하고, 아웃된 타구는 세모 표시, 안타가 된 타구는 원 표시를 하자. 
이 때 각 타구 결과별로 색깔을 달리 한다.
```

💡 **코드.**
```r
mookie_events <- mookie %>%
  as_tibble() %>%
  filter(description=='hit_into_play') %>%
  filter(events!='field_error', events!='fielders_choice_out', events!='double_play') #에러, 야수선택, 땅볼이 아닌 병살을 제외 %>%
  mutate(result = ifelse(events=='single'|
                           events=='double'|
                           events=='triple'|
                           events=='home_run','hit','out')) #안타로 이어진 타구와 아웃 타구를 분류하기 위한 변수 추가

mookie_launch_events <- mookie_events %>%
  #색깔은 타구 결과별로, 모양은 안타여부로
  ggplot(aes(x=launch_speed, y=launch_angle, color=events, shape=result)) +
  geom_point()

ggplotly(mookie_launch_events) #인터랙티브 그래프로 표현
```

<img src="https://user-images.githubusercontent.com/115082062/206606688-0ac4ccbb-cb27-4f46-b525-dc9ab9c2a27f.jpg">

---

<br>

## 무키 베츠 타구속도, 발사각 산점도(안타가 된 타구만)

❓ **상황.**
```
2021시즌 무키 베츠의 데이터를 이용하여 x축을 타구속도, y축을 발사각으로 하는 산점도를 그려보자. 
안타만 된 타구를 데이터로 한다.
```

💡 **코드.**

```r
mookie_events2 <- mookie %>%
  as_tibble() %>%
  filter(description=='hit_into_play') %>%
  filter(events!='field_error', events!='fielders_choice_out', events!='double_play') %>%
  mutate(result = ifelse(events=='single'|
                           events=='double'|
                           events=='triple'|
                           events=='home_run','hit','out')) %>%
  filter(result=='hit')

mookie_launch_events2 <- mookie_events2 %>%
  ggplot(aes(x=launch_speed, y=launch_angle, color=events)) +
  geom_point()

ggplotly(mookie_launch_events2)
```

<img src="https://user-images.githubusercontent.com/115082062/206606778-7992a1c1-4157-4181-a645-ceb517b49701.jpg">

홈런타구는 발사속도가 최소 95마일은 되어야 하고, 발사각은 25도~35도에서 형성되고 있음을 알 수 있다. 이보다 발사각이 살짝 낮으면 2루타 혹은 3루타로 연결이 된 것을 볼 수 있다.

---

<br>

## 무키 베츠 타구 종류에 따른 타구속도, 발사각 산점도

❓ **상황.**
```  
2021시즌 무키 베츠의 데이터를 이용하여 x축을 타구속도, y축을 발사각으로 하는 산점도를 그려보자. 
이 때 타구 종류(팝업, 라인드라이브, 플라이볼, 그라운드볼)를 대상 데이터로 한다.
```
  

💡 **코드.**
```r
mookie_bbtype <- mookie %>%
  as_tibble() %>%
  filter(type=='X')

mookie_launch_bbtype <- mookie_bbtype %>%
  ggplot(aes(x=launch_speed, y=launch_angle, color=bb_type)) +
  geom_point()

ggplotly(mookie_launch_bbtype)
```

<img src="https://user-images.githubusercontent.com/115082062/206606835-92598b5e-510c-433a-8b56-8ab3f44a67ed.jpg">

---

<br>

## 무키 베츠 타구 좌표 산점도

❓ **상황.**
```
2021시즌 무키베츠의 타구 좌표를 산점도로 그리고, 타구 결과에 따라 색깔을 달리 하라.
```

💡 **코드.**
```r
#타구좌표는 hc_x, hc_y변수에 저장되어 있다.
mookie_hit <- mookie %>%
  filter(type=='X') %>%
  filter(events!='field_error', events!='fielders_choice_out', events!='double_play')

mookie_location<-mookie_hit %>%
  ggplot(aes(x=hc_x,y=hc_y,color=events)) +
  geom_point()

ggplotly(mookie_location)
```

<img src="https://user-images.githubusercontent.com/115082062/206606876-986565f5-ab9b-4714-bdd7-7f90fd5a9723.jpg">

야구장이 뒤집힌 모양이다.

---
<br>


## 블게주 히팅 존 시각화

❓ **상황.**
```
2021시즌 블라디미르 게레로 주니어의 단타,2루타,3루타,홈런 별 히팅 존을 산점도로 그려라. 
이 때 구종별로 색깔을 달리한다.
```

💡 **코드.**
```r
#분석에 앞서 2021시즌 블라디미르의 스트라이크 존을 어림잡기 위해 선행작업을 했다.
#스트라이크 볼 여부에 따라 색깔을 나누어 산점도를 찍고, 존이 대강 어떻게 형성되는지 알아보았다.
vladimir %>%
  filter(description=='called_strike'|
           description=='ball') %>%
  ggplot(aes(x=plate_x, y=plate_z, color=description)) +
  geom_point() +
  annotate(
    geom='rect',
    xmin=1,
    xmax=-1,
    ymin=1.48,
    ymax=3.3,
    color='white',
    alpha=.1,
    linetype='dashed'
  ) +
  coord_fixed()

```

<img src="https://user-images.githubusercontent.com/115082062/206606931-25c07f6c-21b1-4227-81a8-28532419578b.jpg">


```r
#존을 알아냈으니 본격적으로 분석을 해보자
vladimir <- read.csv("vladimir.csv",fileEncoding = "UCS-2LE") %>%
  as_tibble()

vladimir %>%
  filter(events=='single'|
           events=='double'|
           events=='triple'|
           events=='home_run') %>%
  ggplot(aes(x=plate_x, y=plate_z, color=pitch_name)) +
  geom_point() +
  annotate(
    geom='rect',
    xmin=1,
    xmax=-1,
    ymin=1.48,
    ymax=3.3,
    color='white',
    alpha=.1,
    linetype='dashed'
  ) +
  facet_grid(~events) +
  coord_fixed()
```

<img src="https://user-images.githubusercontent.com/115082062/206606973-4ae99d59-1368-4160-9058-fd6ada72a14a.jpg">

---

<br>

## 블게주 헛스윙 존 시각화


❓ **상황.**
```
2021시즌 블게주의 헛스윙한 공을 스트라이크 존 상에 산점도로 찍어 나타내고, 구종별로 색깔을 달리하여 나타내라. 
이 때 좌투 우투를 구분하여 만든다.
```

💡 **코드.**

```r
vladimir %>%
  filter(description=='swinging_strike') %>%
  ggplot(aes(x=plate_x, y=plate_z, color=pitch_name)) +
  geom_point() +
  annotate(
    geom='rect',
    xmin=1,
    xmax=-1,
    ymin=1.48,
    ymax=3.3,
    color='white',
    alpha=.1,
    linetype='dashed'
  ) +
  facet_grid(~p_throws)
  coord_fixed()
```

<img src="https://user-images.githubusercontent.com/115082062/206607056-ad9610d4-9ec4-42e6-8c0c-40e19ae2990f.jpg">

하이패스트볼, 우투수의 바깥쪽으로 흘러나가는 슬라이더, 좌투수의 멀어지는 체인지업에 약한 것을 확인할 수 있다.

---

<br>

## 블게주 3-0 적극성 확인

❓ **상황.**
```
블게주는 3-0카운트에서도 스윙을 거침없이 하는 스타일인지를 알아보자. 
비교를 위해 알투베, 무키 베츠의 스타일도 알아보자.
```

💡 **코드.**

```r
vladimir %>%
  filter(balls==3&strikes==0) %>% #3-0카운트인 경우만 추출
  #볼, 사구, 루킹스트라이크, 바운드볼의 경우 no_swing으로, 스윙한 경우 swing으로 하는 변수 생성
  mutate(swing=ifelse(description=='ball'|
                        description=='hit_by_pitch'|
                        description=='blocked_ball'|
                        description=='called_strike', 'no_swing', 'swing')) %>%
  ggplot(aes(x=plate_x, y=plate_z, color=swing)) +
  geom_point() +
  annotate(
    geom='rect',
    xmin=1,
    xmax=-1,
    ymin=1.48,
    ymax=3.3,
    color='white',
    alpha=.1,
    linetype='dashed'
  ) +
  coord_fixed()
```

<img src="https://user-images.githubusercontent.com/115082062/206607121-7c30962f-66db-4f47-aab9-00d67d0e46da.jpg">

밑에 나올 베츠와 알투베와 비교할 수 없을 정도로 적극성을 보여준다.

<img src="https://user-images.githubusercontent.com/115082062/206607173-8de6f7ef-d5db-4d8f-912d-eb8b89ba70a3.jpg">

<img src="https://user-images.githubusercontent.com/115082062/206607203-50c368d7-a710-47d7-8d23-6422130dc4ba.jpg">

---

<br>

## 느린 구속 vs 빠른 구속, 배럴타구 비교 

❓ **상황.**
```
‘느린 공은 치기 쉬울까?’라는 의문에서 출발하여 
‘구속이 느린 패스트볼과 구속이 빠른 패스트볼 중 무엇이 정타가 나오기 쉬울까?’라는 의문을 품게 되었다. 
2021시즌 블게주의 데이터를 가져와서 패스트볼을 
총 5가지 구간(88마일 이하, 89~91마일, 92~94마일, 95~97마일, 98마일 이상)으로 나누고,
이에 따른 발사각, 타구속도 산점도를 그려라. 
이 때 배럴타구가 나오는 존을 산점도 그래프 위에 얹어 그려라.
```

💡 **코드.**

```r
vladimir_ff_launch <- vladimir %>%
  filter(pitch_type=='FF' & description=='hit_into_play') %>%
  mutate(speed = case_when(release_speed>=98~"98~",
                           release_speed>=95&release_speed<98~"95~97",
                           release_speed>=92&release_speed<95~"92~94",
                           release_speed>=89&release_speed<92~"89~91",
                           release_speed<89~"~88"))

vladimir_ff_launch %>%
  ggplot(aes(x=launch_speed, y=launch_angle)) +
  geom_point() +
  #배럴타구
  annotate(
    geom='rect',
    xmin=98,
    xmax=120,
    ymin=26,
    ymax=30,
    color='red',
    alpha=.1,
    linetype='dashed'
  ) +
  #배럴타구의 하위버전
  annotate(
    geom='rect',
    xmin=94,
    xmax=120,
    ymin=23,
    ymax=35,
    color='green',
    alpha=.1,
    linetype='dashed'
  ) +
  facet_grid(~speed)
```

<img src="https://user-images.githubusercontent.com/115082062/206607565-9fc732cf-2ac1-4dad-a5b7-92f7000adaf9.jpg">

공이 마냥 느리다고 하여 배럴타구, 즉 정타가 많이 나오는 건 아니다. 88마일 이하 패스트볼 중에 배럴타구는 커녕 하위 배럴타구 존에 들어온 타구는 한 개도 없었다. 
오히려 패스트볼 평균 구속 혹은 그 이상은 되어야 배럴타구 존에 데이터가 몰리는 것을 확인할 수 있다.
물론 이는 한 선수만의 데이터로 분석한 것이기에, 다른 선수들의 데이터까지 종합적으로 확인할 필요가 크다.

<br>
