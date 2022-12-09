---
title: "[MySQL] MySQL 설치, 테이블 생성 및 데이터 입력"
excerpt: "MySQL Workbench를 설치하고 테이블을 생성하고 원하는 데이터를 입력해보자"

toc: true
toc_label: "목차"
toc_sticky: true

published: true

categories:
  - MySQL
tags: [MySQL, yalco, Workbench]

date: 2022-12-09 13:35:00
last_modified_at: 2022-12-09 13:35:00
---

<br>

<div class="notice--primary" markdown="1">
💡 유튜브 채널 ‘얄팍한 코딩사전’에서 제공하는 강좌를 보고 학습했습니다. <br>
SQL의 기초적인 내용을 웹에서 실습해볼 수 있는 강좌입니다. <br>
강좌에서 배운 개념을 바탕으로 직접 문제를 만들고, 이를 풀어보는 식으로 독학했습니다.
</div>

<br>

> 강좌 링크

[얄코의 갖고 노는 MySQL 데이터베이스 강좌](https://www.yalco.kr/lectures/sql/)

<br>

# Section3. 데이터 조작하기

## 3-1 MySQL 설치하기

[MySQL :: Begin Your Download](https://dev.mysql.com/downloads/file/?id=510039)

윈도우의 경우 Community Server와 Workbench, Sample Datebase를 한번에 설치할 수 있다. 설치가 끝나면 Wokrbench를 실행한다.

<img src="https://user-images.githubusercontent.com/115082062/206624457-e3421791-761c-4250-81d5-efa66b07e6d8.jpg">

화면 좌측에 SCHEMAS라는 것이 테이블 여러 개의 묶음이다.

새롭게 DATABASE를 만들어보자. 왼쪽 상단 SQL+ 아이콘을 누르면 된다. 

```sql
CREATE SCHEMA `mydatabase` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci 
# 'mydatabase'라는 schema를 생성한다.

DROP DATABASE 'mydatabase'
# 'mydatabase'라는 데이터를 삭제한다.
```

코드를 작성하고 상단 번개모양 아이콘(혹은 Ctrl+엔터)을 누르고 왼쪽 탭에 활성화 아이콘을 누르면 새로운 스키마 ‘mydatabase’가 생성되어 있다. 아니면 명령어가 아니라 왼쪽 탭에서 우클릭하여 create schema를 눌러서 만들 수도 있다. 이 때 `utf8mb4`는 한글을 포함한 전세계 문자와 이모티콘을 사용 가능하게 한단 의미이다.

---

❓ **문제.** 
```
주어진 sakila 데이터베이스로 문제를 풀어보자. acotr_id에 따라서 그 actor가 출연한 영화를 보여주는 코드를 작성하라.
```

💡 **답.**
```sql
SELECT concat_ws(' ', A.first_name, A.last_name) AS actor_name, F.title FROM actor A
JOIN film_actor FA on A.actor_id = FA.actor_id
JOIN film F on F.film_id = FA.film_id
WHERE A.actor_id = 1;
```

---

Workbench는 엄밀히 말하면 MySQL이 아니라, 쉽게 작동시키기 위한 클라이언트 툴이다. 진짜 MySQL은 **C:\Program Files\MySQL\MySQL Server 8.0\bin** 에서 MySQL shell을 실행해야 한다.

---

<br>

## 3-2 테이블 만들고 데이터 입력하기

`CREATE TABLE` 명령어로 테이블을 만들 수 있다. player이란 이름의 테이블을 만들어보자.

```sql
CREATE TABLE player (
	player_id INT, -- 정수 자료형
    player_name VARCHAR(10), -- 문자열(최대10글자)
    age TINYINT, -- 작은 숫자 자료형
    debut DATE); -- 날짜 자료형
```

물론 왼쪽 탭에서 우클릭 후 create table을 누르면 아래처럼 명령어 없이 테이블을 만들 수도 있다.

<img src="https://user-images.githubusercontent.com/115082062/206624319-68e83e59-ea5d-4d42-b4ee-e082f9ab875c.jpg">

이제 `ALTER TABLE`로 테이블을 변경해보자.

```sql
alter table player rename to friends, -- 테이블명 변경
change column person_id person_id tinyint, -- 컬럼 자료형 변경
change column person_name person_nickname varchar(10), -- 컬럼명 변경
drop column debut, -- 컬럼 삭제
add column is_married tinyint after age; -- 컬럼 추가
```

본격적으로 데이터를 테이블에 삽입해보자. `insert into`를 사용하여 데이터를 삽입할 수 있다. 아래처럼 컬럼명을 괄호안에 나열하고, `value()`안에 데이터를 입력하면 된다. 모든 컬럼에 값을 넣는다면 컬럼명을 생략해도 된다.

```sql
insert into player
	(player_id, player_name, age, debut)
    values(1, '박해민', 33, '2012-4-20');

insert into player
-- 컬럼명 생략 가능
	values
		(2, '김선빈', 34, '2008-5-1'), #여러 행 넣기도 가능
    (3, '나성범', 34, '2012-4-1'),
    (4, '강백호', 24, '2018-4-2')
```

테이블을 생성할 때 제약을 걸 수도 있다. 이 때 `primary key`는 기본키로 불리며 핵심적인 역할을 수행한다. 테이블당 하나만 있을 수 있고, 인덱스를 생성해준다. 보통 `auto_increment`와 함께 쓰이며 각 행을 고유하게 식별하게 해주는 역할이다.

| 제약 | 설명 |
| --- | --- |
| AUTO_INCREMENT | 새 행 생성시마다 자동으로 1씩 증가 |
| PRIMARY KEY | 중복 입력 불가, NULL(빈 값) 불가 |
| UNIQUE | 중복 입력 불가 |
| NOT NULL | NULL(빈 값) 입력 불가 |
| UNSIGNED | (숫자일시) 양수만 가능 |
| DEFAULT | 값 입력이 없을 시 기본값 |

```sql
create table player (
	player_id int auto_increment primary key, -- player_id는 기본키로 자동으로 1씩 증가하고 중복이나 null 불가
    player_name varchar(10) not null, -- person_name은 null 불가
    position_ varchar(10) unique not null, -- 중복, null 불가
    age tinyint unsigned, -- 양수만 가능
    mvp tinyint default 0);

insert into player
	(player_name, posiplayertion_, age)
    values('양의지', 'C', 35) 
    -- player_id는 자동으로 1이고, mvp는 기본값인 0이 나온다

insert into player
	(player_name, position_, age)
    values('양의지', 'C', -2);
    -- age는 양수만 가능하므로 out of range value for column 'age' 에러가 뜬다

insert into player
	(player_name, position_, age)
    values('김선빈', NULL, 30);
    -- position_은 null이 올 수 없으므로 column 'position_' cannot be null 에러가 뜬다

insert into player
	(player_name, position_, age)
    values('강민호', 'C', 38);
    -- position_은 중복이 불가한데 이미 포수가 있으니 duplicate entry 'C' for key~에러가 뜬다
```
<br>