---
title: "[MySQL] 1-2 각종 연산자들"
excerpt: "MySQL에서 쓰이는 각종연산자들을 알아보자"

toc: true
toc_label: "목차"
toc_sticky: true

published: true

categories:
  - MySQL
tags: [MySQL, yalco]

date: 2022-11-29 16:30:00
last_modified_at: 2022-11-29 16:30:00
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

## 1-2 각종 연산자들

### **사칙연산**

이 때 컬럼명은 계산수식이 된다.

```sql
SELECT 1 + 2, -- 덧셈
6 - 3.2 AS DIFFERENCE, -- 계산값의 컬럼명을 DIFFERENCE로 지정
3 * (2 + 4) / 2, 'MATH', 
10%3;
```

만약 문자열에 사칙연산을 가하면 그 문자열을 0으로 인식하고 계산한다. 다만, 숫자를 문자열로 만들면 이것을 숫자로 자동인식한다.

```sql
SELECT 'HEEJUN' + 3; -- 'HEEJUN'을 0으로 인식하고 3만 출력.
SELECT '003' * 3; -- '003'을 숫자 3으로 인식하고 9를 출력.
```

이와 마찬가지로 컬럼들을 합해서 새로운 컬럼을 만들 수 있다.

```sql
SELECT 
OrderID, ProductID,
OrderID + ProductID
FROM OrderDetails;
```

---


❓ **문제.**
```
Products 테이블에서 상품들의 가격을 30% 올리고 increased_price라는 컬럼명으로 ProductName과 함께 출력해보자. 
이 때 인상된 가격이 30 이상인 것들을 오름차순으로 출력한다.
```


💡 **답.**
```sql
SELECT 
ProductName,
Price*1.3 AS increased_Price
FROM Products
WHERE Price*1.3 >= 30
ORDER BY Price;
```

<img src="https://user-images.githubusercontent.com/115082062/204468138-13d3cbc5-3474-4d5a-a11e-1391f0483741.jpg">

---

<br>

### **참/거짓 관련 연산자**

MySQL에서 `TRUE`는 1, `FALSE`는 0으로 저장된다.

```sql
SELECT True, False, !True, NOT 1, NOT False; --앞에 !나 NOT을 붙이면 그 역을 출력
```

<img src="https://user-images.githubusercontent.com/115082062/204468313-7fdbbc64-057a-48b6-a606-74fe35b7b82e.jpg">

`AND`는 양쪽이 모두 TRUE일 때만 TRUE를 반환하고, `OR`는 한쪽만 TRUE여도 TRUE를 반환한다.

```sql
SELECT TRUE IS TRUE, -- 1
TRUE IS NOT TRUE, -- 0
(TRUE IS FALSE) IS NOT TRUE, --1
TRUE AND FALSE; --0
```

---

❓ **문제**. 
```
Customers 테이블에서 거주 국가가 ‘Mexico’이거나 ‘UK’인 고객들을 불러오자.
```

💡 **답**.

```sql
SELECT * FROM Customers
where Country = 'UK' or Country= 'Mexico';
```

---
<br>

MySQL의 기본 사칙연산자는 대소문자 구분을 하지 않는다. 또한 문자열도 대소비교가 가능한데, 알파벳 순이 늦을수록 더 큰 값이다.

```sql
SELECT 'A' = 'a'; -- TRUE 출력.
SELECT 'A' > 'B'; -- FALSE 출력.
```

테이블의 컬럼이 아닌 값으로 선택할 수도 있다.

```sql
SELECT ProductName, Price, not Price > 20 AS cheap -- 20보다 가격이 작거나 같으면 cheap 열이 true로 출력
FROM Products;
```

`BETWEEN {MIN} AND {MAX}`는 두 값 사이에 있으면 TRUE를 반환한다.

```sql
SELECT 5 BETWEEN 1 AND 10; -- 5가 1과 10 사이에 있다:TRUE
```


❓ **문제**. 
```
Customers 테이블에서 CustomerName이 c~g로 시작하는 행들 중 Country가 UK, Mexico, Brazil, Spain인 행들을 불러오자.
```


💡 **답**.
```sql
SELECT * FROM Customers
where CustomerName between 'c' and 'h' # c부터 h직전, 즉 g까지를 출력함
and Country in ('UK', 'Brazil', 'Mexico', 'Spain');괄호 안에 있다면 true
```

`LIKE`는 문자열의 패턴에 대해 설명한다. `%`는 그 자리에 0~N개의 문자가 위치함을 의미한다.

```sql
SELECT
  'HELLO' LIKE 'hel%', -- 1
  'HELLO' LIKE 'H%', -- 1
  'HELLO' LIKE 'H%O', -- 1
  'HELLO' LIKE '%O', -- 1
  'HELLO' LIKE '%HELLO%', -- 1 %은 문자가 없어도 무방함.
  'HELLO' LIKE '%H', -- 0 H로 끝나지 않으니까 FALSE 
  'HELLO' LIKE 'L%'; -- 0 L로 시작하지 않으니까 FALSE
```

`_`는 그 자리에 _의 개수만큼의 문자가 위치함을 의미한다.

```sql
SELECT
  'HELLO' LIKE 'HEL__', --1
  'HELLO' LIKE 'h___O', --1
  'HELLO' LIKE '_____', --1
  'HELLO' LIKE '_HELLO', --1
  'HELLO' LIKE 'HEL_', --0 개수가 안 맞음
  'HELLO' LIKE 'H_O'; --0 개수가 안 맞음
```

<br>

### **연산자 정리**

| 연산자 | 의미 |
| --- | --- |
| +, -, *, / | 각각 더하기, 빼기, 곱하기, 나누기 |
| %, MOD | 나머지 |
| IS | 양쪽이 모두 TRUE 또는 FALSE |
| IS NOT | 한쪽은 TRUE, 한쪽은 FALSE |
| AND, && | 양쪽이 모두 TRUE일 때만 TRUE |
| OR | 한쪽은 TRUE면 TRUE |
| = | 양쪽 값이 같음 |
| !=, <> | 양쪽 값이 다름 |
| >, < | (왼쪽, 오른쪽) 값이 더 큼 |
| >=, <= | (왼쪽, 오른쪽) 값이 같거나 더 큼 |
| BETWEEN {MIN} AND {MAX} | 두 값 사이에 있음 |
| NOT BETWEEN {MIN} AND {MAX} | 두 값 사이가 아닌 곳에 있음 |
| IN (...) | 괄호 안의 값들 가운데 있음 |
| NOT IN (...) | 괄호 안의 값들 가운데 없음 |
| LIKE '... % ...' | 0~N개 문자를 가진 패턴 |
| LIKE '... _ ...' | _ 갯수만큼의 문자를 가진 패턴 |

