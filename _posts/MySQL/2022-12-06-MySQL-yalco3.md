---
title: "[MySQL] 1-3,4 숫자, 문자열, 날짜 함수"
excerpt: "MySQL에서 숫자와 문자열을 다루는 함수들을 알아보고 날짜형에 익숙해져보자"

toc: true
toc_label: "목차"
toc_sticky: true

published: true

categories:
  - MySQL
tags: [MySQL, yalco]

date: 2022-12-06 16:00:00
last_modified_at: 2022-12-06 16:00:00
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

# 1. SELECT기초 - 원하는 정보 가져오기

실습 웹사이트 링크

[MySQL Tryit Editor v1.0](https://www.w3schools.com/mysql/trymysql.asp?filename=trysql_select_all) 

## 1-3 숫자와 문자열을 다루는 함수들

### **숫자 관련 함수들**

| 함수 | 설명 |
| --- | --- |
| ROUND | 반올림 |
| CEIL | 올림 |
| FLOOR | 내림 |
| ABS | 절댓값 |
| GREATEST | 괄호안에서 최댓값 |
| LEAST | 괄호안에서 최솟값 |

```sql
select round(0.5), --반올림
ceil(0.4), --올림
floor(0.6), --내림
abs(-1), -- 절댓값
greatest(1,2,3), --괄호 안에서 최댓값. 
least(1,2,3), --괄호 안에서 최솟값. 
```

이 때 헷갈려서는 안 될 것이 `greatest`와 `max` 그리고 `least`와 `min`의 차이이다. `greatest`, `least`는 괄호 안에 있는 것 중에서 최대최소를 찾는 것이라면, `max`와 `min`은 특정 컬럼 내에서 최댓값 혹은 최솟값을 갖는 열을 찾는 것이다.

| 함수 | 설명 |
| --- | --- |
| MAX | 가장 큰 값 |
| MIN | 가장 작은 값 |
| COUNT | 갯수 (NULL값 제외) |
| SUM | 총합 |
| AVG | 평균 값 |

```sql
select
max(quantity), -- 최댓값
min(quantity), -- 최솟값
count(quantity), -- NULL을 제외한 개수
sum(quantity), -- 총합
avg(quantity) -- 평균
from OrderDetails;
```

❓ **문제**. 
```
Products 테이블에서 Price의 평균, 최댓값, 최솟값을 출력하라. 
이 때 각 값들은 반올림하도록 한다.
```

💡 **답**.

```sql
SELECT 
round(avg(Price)) as '평균가격',
round(max(Price)) as '최대가격',
round(min(Price)) as '최소가격'
from Products;
```

<img src="(https://user-images.githubusercontent.com/115082062/205837510-8eac67f1-92c5-4888-ac64-b6881b75467b.jpg">

---

| 함수 | 설명 |
| --- | --- |
| POW(A, B), POWER(A, B) | A를 B만큼 제곱 |
| SQRT | 제곱근 |
| TRUNCATE(N, n) | N을 소숫점 n자리까지 선택 |

```sql
SELECT 
power(2,2), -- 2의 2제곱
pow(2,2), -- 2의 2제곱
sqrt(4), --4의 제곱근
truncate(123.4567, 1) --123.4 소수점 첫째짜리까지 출력(그 뒤는 버림)
truncate(123.4567, 1) --120 일의 자리에서 버림
```

---

<br>

### **문자열 관련 함수들**

| 함수 | 설명 |
| --- | --- |
| UCASE, UPPER | 모두 대문자로 |
| LCASE, LOWER | 모두 소문자로 |

```sql
SELECT
upper('abc'), -- 모두 대문자로
ucase('abc'), -- 모두 대문자로
lower('ABC'), -- 모두 소문자로
lcase('ABC'); -- 모두 소문자로
```

❓ **문제**. 
```
Suppliers 테이블에서 City는 대문자로, Address는 소문자로 출력하고, 
SupplierName을 내림차순으로 정렬하라.
```

💡 **답**.

```sql
SELECT 
SupplierName,
upper(City),
lower(Address)
FROM Suppliers
order by SupplierName desc;
```

---

| 함수 | 설명 |
| --- | --- |
| CONCAT(...) | 괄호 안의 내용 이어붙임 |
| CONCAT_WS(S, ...) | 괄호 안의 내용 S로 이어붙임 |

```sql
SELECT 
concat('hi', ' ', 'this is ', 2022), --괄호 안의 원소를 이어붙인다.
concat_ws('-', 2022, 02, 08); --괄호 안의 원소를 맨 처음 원소로 이어붙인다.
```

<img src="https://user-images.githubusercontent.com/115082062/205837647-ff2712da-cacd-4e09-ae8a-1c70b7928c03.jpg">

---


❓ **문제**. 
```
Customers 테이블에서 Country와 City를 컴마로 묶어서 불러오자(location으로 컬럼명 변경). 
이 때 Country를 오름차순으로 정렬하고 그 후에 City를 오름차순으로 정렬한다.
```

💡 **답**.

```sql
SELECT 
CustomerName,
concat_ws(', ', City, Country)
FROM Customers
order by Country, City;
```

---

<img src="https://user-images.githubusercontent.com/115082062/205837895-46ad0ea9-3060-4962-a7cc-7692374c8d77.jpg">

---

| 함수 | 설명 |
| --- | --- |
| SUBSTR, SUBSTRING | 주어진 값에 따라 문자열 자름 |
| LEFT | 왼쪽부터 N글자 |
| RIGHT | 오른쪽부터 N글자 |

```sql
SELECT 
substr('heejun', 3), -- ejun 3번째 단어부터만 출력
substr('heejun', 3, 2), -- ej 3번째 단어부터 2개만 출력
substr('heejun', -4), -- ejun 뒤에서 4번째 단어부터 출력
substr('heejun', -4, 2), -- ej 뒤에서 4번째 단어부터 2개만 출력
left('heejun', 3), -- 왼쪽부터 3글자
right('heejun', 3); -- 오른쪽부터 3글자
```

```sql
select
OrderDate,
LEFT(OrderDate, 4) AS YEAR,
SUBSTR(OrderDate, 6, 2) AS Month,
RIGHT(OrderDate, 2) AS Day
FROM Orders;
```

<img src="https://user-images.githubusercontent.com/115082062/205838023-7d012184-d64f-487a-b7f4-81ae119054a4.jpg">

| 함수 | 설명 |
| --- | --- |
| LENGTH | 문자열의 바이트 길이 |
| CHAR_LENGTH, CHARACTER_LEGNTH | 문자열의 문자 길이 |

문자열의 길이를 출력하려면 그냥 `LENGTH`가 아니라 `CHAR_LENGTH`를 써야 한다!

```sql
SELECT 
LENGTH('안녕하세요'), -- 15
CHAR_LENGTH('안녕하세요'); -- 5
```

| 함수 | 설명 |
| --- | --- |
| TRIM | 양쪽 공백 제거 |
| LTRIM | 왼쪽 공백 제거 |
| RTRIM | 오른쪽 공백 제거 |

| 함수 | 설명 |
| --- | --- |
| LPAD(S, N, P) | S가 N글자가 될 때까지 P를 왼쪽에 이어붙임 |
| RPAD(S, N, P) | S가 N글자가 될 때까지 P를 오른쪽에 이어붙임 |

```sql
SELECT 
LPAD('ABC', 5, '*'), -- **ABC
RPAD('ABC', 5, '-'); -- ABC**
```

| 함수 | 설명 |
| --- | --- |
| REPLACE(S, A, B) | S중 A를 B로 변경 |

---

❓ **문제**. 
```
Employees 테이블에서 생년월일을 yy.mm.dd. 형식으로 바꾸고,
LastName, FirstName과 함께 출력하라.
```

💡 **답**.

```sql
SELECT 
LastName,
FirstName,
concat(replace(substr(BirthDate, 3), '-', '.'),'.') as Birthdate
FROM Employees;
```

---

| 함수 | 설명 |
| --- | --- |
| INSTR(S, s) | S중 s의 첫 위치 반환, 없을 시 0 |

```sql
SELECT * FROM Customers 
WHERE INSTR(CustomerName, ' ') BETWEEN 1 AND 6; -- CustomerName의 첫글자가 6보다 짧은 것만 출력.
```

| 함수 | 설명 |
| --- | --- |
| Convert(A, T) | A를 T 자료형으로 변환 |

```sql
SELECT
'01' = '1', -- false(0)
convert('01', decimal) = convert('1', decimal); -- true(1)
--문자열을 decimal(숫자 자료형)으로 변환
```
---

<br>

## 1-4 시간/날짜 관련 및 기타 함수들

| 함수 | 설명 |
| --- | --- |
| CURRENT_DATE, CURDATE | 현재 날짜 반환 |
| CURRENT_TIME, CURTIME | 현재 시간 반환 |
| CURRENT_TIMESTAMP, NOW | 현재 시간과 날짜 반환 |

```sql
SELECT
CURDATE(), -- 2022-02-09
CURTIME(), -- 08:25:55
NOW(); -- 2022-02-09 08:25:55
```

| 함수 | 설명 |
| --- | --- |
| DATE | 문자열에 따라 날짜 생성 |
| TIME | 문자열에 따라 시간 생성 |

만약에 `DATE` 함수에 시간과 날짜를 모두 입력한다면, 날짜만 불러온다. 마찬가지로 `TIME`함수에 시간과 날짜를 모두 입력한다면, 시간만 불러온다. 이 때 함수에는 ‘문자열’형식으로 입력해줘야 한다.

```sql
SELECT
  '2021-6-1 1:2:3' = '2021-06-01 01:02:03', -- 0
  DATE('2021-6-1 1:2:3') = DATE('2021-06-01 01:02:03'), -- 1
  TIME('2021-6-1 1:2:3') = TIME('2021-06-01 01:02:03'), -- 1
  DATE('2021-6-1 1:2:3') = TIME('2021-06-01 01:02:03'), -- 0 DATE는 날짜만 가져오고, TIME은 시간만 가져온다.
  DATE('2021-6-1') = DATE('2021-06-01 01:02:03'), -- 1
  TIME('2021-6-1 1:2:3') = TIME('01:02:03'); -- 1
```

---

❓ **문제**. 
```
Orders 테이블에서 주문날짜가 1997년 3월부터 1998년 4월까지인 것 중 
ShipperID가 1인 것만을 불러오자.
```


💡 **답**.

```sql
SELECT *
FROM Orders
WHERE 
OrderDate BETWEEN DATE('1997-3-1') AND DATE('1998-4-30') and
ShipperID=1
```

---

| 함수 | 설명 |
| --- | --- |
| YEAR | 주어진 DATETIME값의 년도 반환 |
| MONTHNAME | 주어진 DATETIME값의 월(영문) 반환 |
| MONTH | 주어진 DATETIME값의 월 반환 |
| WEEKDAY | 주어진 DATETIME값의 요일값 반환(월요일: 0) |
| DAYNAME | 주어진 DATETIME값의 요일명 반환 |
| DAYOFMONTH, DAY | 주어진 DATETIME값의 날짜(일) 반환 |

```sql
SELECT
OrderDate,
CONCAT(CONCAT_WS('/', YEAR(OrderDate), MONTH(OrderDate), DAY(OrderDate)),
' ', UPPER(LEFT(DAYNAME(OrderDate),3))) AS orderdate
FROM Orders;
```

<img src="https://user-images.githubusercontent.com/115082062/205838430-c33ad3cd-a348-428f-9f19-348adcf2a8d2.jpg">

| 함수 | 설명 |
| --- | --- |
| HOUR | 주어진 DATETIME의 시 반환 |
| MINUTE | 주어진 DATETIME의 분 반환 |
| SECOND | 주어진 DATETIME의 초 반환 |

```sql
SELECT
HOUR(NOW()), MINUTE(NOW()), SECOND(NOW());
```

| 함수 | 설명 |
| --- | --- |
| ADDDATE, DATE_ADD | 시간/날짜 더하기 |
| SUBDATE, DATE_SUB | 시간/날짜 빼기 |

```sql
SELECT
ADDDATE('2022-02-09', INTERVAL 1 YEAR),
ADDDATE('2022-02-09', INTERVAL -2 MONTH),
ADDDATE('2021-02-09', INTERVAL 3 WEEK);
```

| 함수 | 설명 |
| --- | --- |
| DATE_DIFF | 두 시간/날짜 간 일수차 |
| TIME_DIFF | 두 시간/날짜 간 시간차 |

```sql
SELECT
OrderDate,
NOW(),
DATEDIFF(OrderDate, NOW())
FROM Orders;
```

---


❓ **문제**. 
```
1998년 2월 10일은 구정이다. 
Orders 테이블에서 구정 앞뒤 1주일의 기간동안 주문한 상품은 배송이 지연될 것으로 예상된다. 
주문이 지연되는 상품들만 불러오자.
```

💡 **답**.

```sql
SELECT * FROM Orders
WHERE abs(datediff(OrderDate, '1998-02-10')) <= 7;
```

---

| 함수 | 설명 |
| --- | --- |
| LAST_DAY | 해당 달의 마지막 날짜 |

`LAST_DAY` 함수는 해당 달의 마지막 날짜를 불러오는데, 만약 LAST

```sql
SELECT 
OrderDate,
LAST_DAY(OrderDate),
DAY(LAST_DAY(OrderDate)),
DATEDIFF(LAST_DAY(OrderDate), OrderDate)
FROM Orders;
```

| 함수 | 설명 |
| --- | --- |
| DATE_FORMAT | 시간/날짜를 지정한 형식으로 반환 |

| 형식 | 설명 |
| --- | --- |
| %Y | 년도 4자리 |
| %y | 년도 2자리 |
| %M | 월 영문 |
| %m | 월 숫자 |
| %D | 일 영문(1st, 2nd, 3rd...) |
| %d, %e | 일 숫자 (01 ~ 31) |
| %T | hh:mm:ss |
| %r | hh:mm:ss AM/PM |
| %H, %k | 시 (~23) |
| %h, %l | 시 (~12) |
| %i | 분 |
| %S, %s | 초 |
| %p | AM/PM |

```sql
SELECT
DATE_FORMAT(NOW(), '%M %d, %Y, %r') AS "1",
DATE_FORMAT(NOW(), '%Y년 %m월 %d일, %p %h시 %i분') AS "3",
DATE_FORMAT(NOW(), '%M %d, %y, %H:%i') AS "A";
```

| 함수 | 설명 |
| --- | --- |
| STR _ TO _ DATE(S, F) | S를 F형식으로 해석하여 시간/날짜 생성 |

```sql
SELECT
DATEDIFF(STR_TO_DATE('2022-10-11 07:48:52', '%Y-%m-%d %T'),
STR_TO_DATE('2022-02-10 22:10:52', '%Y-%m-%d %T')),
TIMEDIFF(STR_TO_DATE('2022-10-11 07:48:52', '%Y-%m-%d %T'),
STR_TO_DATE('2022-02-10 22:10:52', '%Y-%m-%d %T'));
```

| 형식 | 설명 |
| --- | --- |
| IF(조건, T, F) | 조건이 참이라면 T, 거짓이면 F 반환 |

```sql
SELECT IF (1 > 2, '1는 2보다 크다.', '1은 2보다 작다.');

#보다 복잡한 조건은 CASE문을 사용한다. 들여쓰기 할 필요가 없다.
SELECT
CASE
  WHEN -1 > 0 THEN '-1은 양수다.'
  WHEN -1 = 0 THEN '-1은 0이다.'
  ELSE '-1은 음수다.'
END;

```

---

❓ **문제**. 
```
OrderDetails 테이블에서 Quantity가 45 이상이면 ‘초대량주문’, 20 이상 45 미만이면 ‘대량주문’, 20 미만이면 ‘일반주문’으로 표시되도록 하자.
```


💡 **답**.

```sql
SELECT 
	OrderID,
	Quantity,
	CASE
		WHEN Quantity >= 45 THEN '초대량주문'
		WHEN Quantity BETWEEN 20 AND 45 THEN '대량주문'
		ELSE '일반주문'
		END
FROM OrderDetails;
```

| 형식 | 설명 |
| --- | --- |
| IFNULL(A,B) | A가 NULL일 시 B출력 |

```sql
SELECT
IFNULL('A', 'B'), -- A
IFNULL(NULL, 'B'); -- B
```

<br>
