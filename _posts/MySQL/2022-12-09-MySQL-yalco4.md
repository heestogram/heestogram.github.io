---
title: "[MySQL] 1-5 group by로 조건에 따라 그룹화시키기"
excerpt: "group by, having, distinct 등 함수를 통계 조건에 따른 그룹화를 해보자"

toc: true
toc_label: "목차"
toc_sticky: true

published: true

categories:
  - MySQL
tags: [MySQL, yalco, group by]

date: 2022-12-09 11:45:00
last_modified_at: 2022-12-09 11:45:00
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

## 1-5 조건에 따라 그룹으로 묶기

### **GROUP BY - 조건에 따라 집계된 값 가져오기**

`GROUP BY`는 조건에 따라 집계된 값을 가져와준다. `GROUP BY 컬럼명`을 입력하면 그 컬럼을 기준으로 집계된 값들을 보여준다.

```sql
SELECT 
Country,
COUNT(*) AS COUNT
FROM Customers
GROUP BY Country;
```

<img src="https://user-images.githubusercontent.com/115082062/206611831-e206d100-cf22-4cd3-9a26-98ffd787449c.jpg">

Country를 기준으로 빈도를 count하여 집계된 값을 보여준다.

```sql
SELECT
	CategoryID,
	MAX(Price) AS MAX,
	MIN(Price) AS MIN,
	TRUNCATE(AVG(Price), 2) AS AVG
FROM Products

GROUP BY CategoryID;
```

---


❓ **문제**. 
```
Products 테이블에서 CategoryID를 기준으로 상품의 최댓값, 최솟값, 평균, 아이디별 개수를 집계해보자. 그리고 마지막 행에 전체의 집계도 나타내보자.
```

💡 **답**.

```sql
select
	CategoryID,
	MAX(Price) AS MAX,
	MIN(Price) AS MIN,
	TRUNCATE(AVG(Price), 2) AS AVG,
	count(*) AS COUNT
FROM Products

GROUP BY CategoryID
WITH ROLLUP;
```

`WITH ROLLUP`은 마지막 행에 전체의 집계값을 나타내준다.

<img src="https://user-images.githubusercontent.com/115082062/206616236-9feb0183-e04e-47d3-b529-f45657ad5fd9.png">

마지막 행에 WITH ROLLUP이 실행되어 있다.

---

`HAVING`은 그룹화된 데이터에서 조건을 걸어준다. `WHERE`이 그룹하기 전에 조건을 걸어준다면, `HAVING`은 그룹화한 후에 조건을 걸어준다는 차이가 있다.

```sql
SELECT
	COUNT(*) AS Count, OrderDate
FROM Orders
WHERE OrderDate > DATE('1996-12-31') #그룹화 전에 조건 걸어주기
GROUP BY OrderDate
HAVING Count >2; #그룹화 후에 조건 걸어주기
```

```sql
SELECT
	CategoryID,
    MAX(Price) AS MaxPrice,
    MIN(Price) AS MinPrice,
    TRUNCATE((MAX(Price)+MIN(Price))/2, 2) AS MedianPrice,
    TRUNCATE(AVG(Price),2) AS AveragePrice
 FROM Products
 WHERE CategoryID > 2
 GROUP BY CategoryID
 HAVING
 	AveragePrice BETWEEN 15 AND 30
    AND MedianPrice < 40;
```

---

<br>

### **DISTINCT - 중복된 값들을 제거하기**

`GROUP BY`와 달리 집계함수가 사용되지 않고, 정렬되지 않기 때문에 더 빠르다.

```sql
SELECT DISTINCT CategoryID
FROM Products;

SELECT DISTINCT Country
FROM Customers
ORDER BY Country; #이렇게 정렬을 따로 해줘야 함.

#Country 기준으로 먼저 중복값 제거하고, 그 다음에 City 기준으로 중복값 제거
SELECT DISTINCT Country, City
FROM Customers
ORDER BY Country, City;
```

필요에 따라 `GROUP BY`와 `DISTINCT`를 함께 사용할 수도 있다. Country별로 City가 몇 종류씩 있는지 알고 싶은 경우를 떠올려보자.

```sql
SELECT 
	Country,
    COUNT(CITY) #이렇게 하면 도시의 종류 수가 아니라 전체 도시 수가 나온다.
FROM Customers
GROUP BY Country;

SELECT 
	Country,
    COUNT(DISTINCT CITY) #따라서 중복된 도시들을 제거해주고 오로지 종류 수만 count하게 바꿔준다.
FROM Customers
GROUP BY Country;
```

---