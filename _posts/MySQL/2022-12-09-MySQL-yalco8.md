---
title: "[MySQL] 3-3, 3-4 자료형 파악하고 데이터를 삭제, 변경하기"
excerpt: "문자, 숫자 자료형을 이해하고 데이터를 삭제 및 변경해보자"

toc: true
toc_label: "목차"
toc_sticky: true

published: true

categories:
  - MySQL
tags: [MySQL, yalco, Workbench]

date: 2022-12-09 13:40:00
last_modified_at: 2022-12-09 13:40:00
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

## 3-3 자료형(+예제데이터)

### **숫자 자료형**

정수형

| 자료형 | 바이트 | SIGNED | UNSIGNED |
| --- | --- | --- | --- |
| TINYINT | 1 | -128 ~ 127 | 0 ~ 255 |
| SMALLINT | 2 | -32,768 ~ 32,767 | 0 ~ 65,535 |
| MEDIUMINT | 3 | -8,388,608 ~ 8,388,607 | 0 ~ 16,777,215 |
| INT | 4 | -2,147,483,648 ~ 2,147,483,647 | 0 ~ 4,294,967,295 |
| BIGINT | 8 | -2^63 ~ 2^63 - 1 | 0 ~ 2^64 - 1 |

**고정 소수점(Fixed Point)**은 좁은 범위의 수를 표현하고 정확한 값을 나타낸다.

| 자료형 | 설명 | 범위 |
| --- | --- | --- |
| DECIMAL(s,d) | 실수 부분 총 자릿수(s) & 소수 부분 자릿수 (d) | s 최대 65 |

**부동소수점(Floating Point)**은 넓은 범위의 수를 표현하고 비교적 정확하지 않은 값을 나타낸다.(일반적으로 충분히 정확하긴 하다.)

| 자료형 | 표현 범위 |
| --- | --- |
| FLOAT | -3.402...E+38 ~ -1.175...E-38 , 0 , 1.175...E-38 ~ 3.402...E+38 |
| DOUBLE | -1.797...E+308 ~ -2.225E-308 , 0 , 2.225...E-308 ~ 1.797...E+308 |

---

### **문자 자료형**

언뜻보기엔 `VARCHAR`가 경제적으로 보일 수도 있지만, 입력할 글자가 n바이트로 동일하다면 `CHAR`를 쓰는 것이 더 낫다. `VARCHAR`는 n바이트를 입력하면 기본적으로 n+1바이트를 쓰기 때문이다. 또한 `WHERE`문 등 필터링을 할 때 검색은 `CHAR`가 더 빠르다.

| 자료형 | 설명 | 차지하는 바이트 | 최대 바이트 |
| --- | --- | --- | --- |
| CHAR(s) | 고정 사이즈 (남는 글자 스페이스로 채움) | s (고정값) | 255 |
| VARCHAR(s) | 가변 사이즈 | 실제 글자 수[최대 s] + 1 [글자수 정보] | 65,535 |

장문의 글은 `CHAR`이나 `VARCHAR`가 아닌 `TEXT` 자료형을 이용한다.

| 자료형 | 최대 바이트 크기 |
| --- | --- |
| TINYTEXT | 255 |
| TEXT | 65,535 |
| MEDIUMTEXT | 16,777,215 |
| LONGTEXT | 4,294,967,295 |

---

### **시간 자료형**

`DATETIME`은 어디에서 읽든 동일한 값이지만, `TIMESTAMP`는 시간을 자동기록한다거나 국제적인 서비스를 이용할 때 사용한다.

| 자료형 | 설명 | 비고 |
| --- | --- | --- |
| DATE | YYYY-MM-DD |  |
| TIME | HHH:MI:SS | HHH: -838 ~ 838까지의 시간 |
| DATETIME | YYYY-MM-DD HH:MI:SS | 입력된 시간을 그 값 자체로 저장 |
| TIMESTAMP | YYYY-MM-DD HH:MI:SS | MySQL이 설치된 컴퓨터의 시간대를 기준으로 저장 |

---

### 앞으로 예제에서 사용할 테이블

```sql
create table sections (
	section_id int auto_increment primary key,
    section_name char(3) not null,
    floor tinyint not null);

insert into sections(section_name, floor)
values 
	('한식', 2),
    ('분식', 2),
    ('중식', 3),
    ('일식', 3),
    ('양식', 3),
    ('카페', 1),
    ('디저트', 1);
```

```sql
create table businesses (
	business_id int auto_increment primary key,
    fk_section_id int not null, -- fk는 foreign key를 의미하는데, 다른 테이블의 키를 조인할 때 끌어쓰는 용도
    business_name varchar(10) not null,
    status char(3) default 'OPN' not null, -- OPN=OPEN, RMD=리모델링, VCT=휴가, CLS=CLOSED
    can_takeout tinyint default 1 not null);

INSERT INTO businesses (fk_section_id, business_name, status, can_takeout)
VALUES  (3, '화룡각', 'OPN', 1), 
        (2, '철구분식', 'OPN', 1),
        (5, '얄코렐라', 'RMD', 1),
        (2, '바른떡볶이', 'OPN', 1),
        (1, '북극냉면', 'OPN', 0),
        (1, '보쌈마니아', 'OPN', 1),
        (5, '에그사라다', 'VCT', 1),
        (6, '달다방', 'OPN', 1),
        (7, '마카오마카롱', 'OPN', 1),
        (2, '김밥마라', 'OPN', 1),
        (7, '소소스윗', 'OPN', 1),
        (4, '사사서셔소쇼스시', 'VCT', 1),
        (3, '린민짬뽕', 'CLS', 1),
        (7, '파시조아', 'OPN', 1),
        (1, '할매장국', 'CLS', 0),
        (5, '노선이탈리아', 'OPN', 1),
        (6, '커피앤코드', 'OPN', 1),
        (2, '신림동백순대', 'VCT', 1);
```

```sql
create table menus (
	menu_id int auto_increment primary key,
    fk_business_id int not null,
    menu_name varchar(20) not null,
    kilocalories decimal(7,2) not null,
    price int not null,
    likes int default 0 not null);
    

INSERT INTO menus (fk_business_id, menu_name, kilocalories, price, likes)
VALUES  (5, '물냉면', 480.23, 8000, 3),
        (8, '아메리카노', 16.44, 4500, 6),
        (17, '고르곤졸라피자', 1046.27, 12000, 12),
        (6, '보쌈', 1288.24, 14000, 2),
        (15, '장국', 387.36, 8500, -1),
        (17, '까르보나라', 619.11, 9000, 10),
        (9, '바닐라마카롱', 160.62, 1500, 4),
        (16, '백순대', 681.95, 11000, 24),
        (6, '마늘보쌈', 1320.49, 16000, 7),
        (16, '양념순대볶음', 729.17, 12000, 0),
        (14, '단팥빵', 225.88, 1500, 13),
        (1, '간짜장', 682.48, 7000, 3),
        (9, '뚱카롱', 247.62, 2000, 8),
        (5, '비빔냉면', 563.45, 8000, 4),
        (10, '참치김밥', 532.39, 3000, 0),
        (2, '치즈떡볶이', 638.42, 5000, 15),
        (11, '플레인와플', 299.31, 6500, 2),
        (2, '찹쌀순대', 312.76, 3000, -4),
        (15, '육개장', 423.18, 8500, 2),
        (4, '국물떡볶이', 483.29, 4500, 1),
        (10, '돈가스김밥', 562.72, 4000, 0),
        (1, '삼선짬뽕', 787.58, 8000, 32),
        (11, '수플레팬케익', 452.37, 9500, 5),
        (4, '라볶이', 423.16, 5500, 0),
        (8, '모카프라푸치노', 216.39, 6000, 8),
        (14, '옛날팥빙수', 382.35, 8000, 2);
```

```sql
create table ratings (
	ratings_id int auto_increment primary key,
    fk_business_id int not null,
    stars tinyint not null,
    comment varchar(200) null,
    created timestamp default current_timestamp not null);
    
INSERT INTO ratings (fk_business_id, stars, comment, created)
VALUES  (2, 4, '치떡이 진리. 순대는 별로', '2021-07-01 12:30:04'),
        (16, 3, '그냥저냥 먹을만해요', '2021-07-01 17:16:07'),
        (14, 5, '인생팥빵. 말이 필요없음', '2021-07-03 11:28:12'),
        (5, 3, '육수는 괜찮은데 면은 그냥 시판면 쓴 것 같네요.', '2021-07-04 19:03:50'),
        (11, 4, '나오는데 넘 오래걸림. 맛은 있어요', '2021-07-04 13:37:42'),
        (9, 2, '빵집에서 파는 마카롱이랑 비슷하거나 못합니다.', '2021-07-06 15:19:23'),
        (16, 5, '신림에서 먹던 맛 완벽재현', '2021-07-06 20:01:39');
```

---

<br>

## 3-4 데이터 변경, 삭제하기

### **DELETE - 주어진 조건의 행 삭제하기**

`where` 조건을 안 걸어주고 그냥 `delete from`을 쓰면 행이 전체 삭제가 되므로 주의하자.

```sql
delete from businesses
where status = 'CLS'; -- status가 'CLS'인 행만 삭제

DELETE FROM businesses; --businesses의 모든 행 삭제
```

만약에 `delete`문으로 행을 전체 삭제하고 다시 이전 데이터를 `insert`한다면, `auto_increment`된 수는 삭제 전의 마지막행에서 1씩 증가한다.

```sql
DELETE FROM businesses; --모든 행 삭제하고
-- 원래  19,20,21행이었던 데이터를  insert하자
INSERT INTO businesses (fk_section_id, business_name, status, can_takeout)
VALUES  (3, '화룡각', 'OPN', 1),
        (2, '철구분식', 'OPN', 1),
        (5, '얄코렐라', 'RMD', 1);
```

<img src="https://user-images.githubusercontent.com/115082062/206625037-6fe5be93-a0e9-4930-b29b-e7d9258d301f.jpg">

business_id(auto_increment)가 삭제 전 마지막행(18행)부터 1씩 증가한다.

하지만 `truncate`문은 테이블을 초기화시켜서 `auto_increment`된 값이라도 아예 초기화된다.

```sql
TRUNCATE businesses; --모든 행을 초기화하고
-- 원래 19,20,21행이었던 데이터를 insert하자
INSERT INTO businesses (fk_section_id, business_name, status, can_takeout)
VALUES  (3, '화룡각', 'OPN', 1),
        (2, '철구분식', 'OPN', 1),
        (5, '얄코렐라', 'RMD', 1);
```

<img src="https://user-images.githubusercontent.com/115082062/206625071-ffc84561-a073-4733-901f-40f5e9e46bf1.jpg">

business_id(auto_increment)가 초기화되어 다시 1부터 시작한다.

---

<br>

### **UPDATE - 주어진 조건의 행 수정하기**


❓ **문제.** 
```
철구분식은 좋아요가 음수인 찹쌀순대 메뉴를 ‘고른햇살 토종순대’로 바꾸려고 한다. 
이 때 가격은 1,000원 인상할 것이다. update문을 사용하여 수정하라.
```

💡 **답.**

```sql
update menus
set menu_name = '고른햇살 토종순대',
	price = price +1000
where fk_business_id =2 and menu_name='찹쌀순대';
```

---


❓ **문제.** 
```
베이징올림픽 오심 여파로 중식 보이콧이 일어나면서 중식 메뉴들의 좋아요가 5씩 내려갔다. 
때문에 section이 중식인 가게들은 전 메뉴 가격을 10% 할인하기로 했다. update문을 사용하여 수정하라.
```


💡 **답.**

```sql
update menus m
set price = price*0.9, -- 가격 인하
	likes = likes-5 -- 좋아요 -5개

where fk_business_id in ( -- business_id가 3에 포함된 것이야 함.
	select business_id  -- business_id를 갖고 온다
    from sections S -- sections 테이블에서
    left join businesses B on S.section_id = B.fk_section_id -- section_id가 갖은 것끼리 join
    where section_name = '중식'); -- section_name이 '중식'인 것만 추출
```


`where` 조건문 없이 update를 하면 모든 행이 변경된다.

<br>