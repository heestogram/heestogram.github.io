---
title: "[MySQL] 1-1 SELECT 전반 기능 훑어보기"

toc: true
toc_label: "목차"
toc_sticky: true

published: true

categories:
  - MySQL

date: 2022-11-17 09:15:30 +09:00
last_modified_at: 2022-11-17 00:15:30 +09:00
---

<br>

💡 유튜브 채널 ‘얄팍한 코딩사전’에서 제공하는 강좌를 보고 학습했습니다. <br>
SQL의 기초적인 내용을 웹에서 실습해볼 수 있는 강좌입니다. <br>
강좌에서 배운 개념을 바탕으로 직접 문제를 만들고, 이를 풀어보는 식으로 독학했습니다.

<br>

> **강좌 링크**

[얄코의 갖고 노는 MySQL 데이터베이스 강좌](https://www.yalco.kr/lectures/sql/)

<br> 


>목차

1. SELECT기초 - 원하는 정보 가져오기
    1) 💡**SELECT 전반 기능 훑어보기**💡
    2) 각종 연산자들
    3) 숫자와 문자열을 다루는 함수들
    4) 시간/날짜 관련 및 기타 함수들
    5) 조건에 따라 그룹으로 묶기
2. SELECT 더 깊이 파보기
    1) 쿼리 안에 서브쿼리
    2) JOIN - 여러 테이블 조립하기
    3) UNION - 집합으로 다루기
3. 데이터 조작하기
    1) MySQL 설치하기
    2) 테이블 만들고 데이터 입력하기
    3) 자료형(+예제데이터)
    4) 데이터 변경, 삭제하기

<br>

>오류 발생시

```sql
Error Code: 1046. No database selected Select the default DB to be used by double-clicking its name in the SCHEMAS list in the sidebar.
```

위와 같은 오류가 발생하면, schema탭에서 쿼리를 실행할 데이터베이스를 더블클릭하고 코드를 실행해주면 해결된다.

<br>

---

# 1. SELECT기초 - 원하는 정보 가져오기


실습 웹사이트 링크

[MySQL Tryit Editor v1.0](https://www.w3schools.com/mysql/trymysql.asp?filename=trysql_select_all) 

<br>

## 1-1. SELECT 전반 기능 훑어보기

본 강좌의 초반부는 SQL을 설치하지 않고 웹 상에서 SQL 코드를 실행하게 된다. 해당 웹에서는 아래와 같은 database를 제공하고 있다.

tip: MySQL의 구문들은 대소문자 구분 없이 사용 가능하고, 줄바꿈도 문제 없다.
<div align=center>
<img src="https://user-images.githubusercontent.com/115082062/202318292-1fa1efed-385f-4fa3-90e7-c05aa2235259.png">
</div>

<br>



### **테이블의 모든 내용 보기.**

*(seterisk)는 테이블의 모든 컬럼을 뜻한다. `SELECT 갖고 올 대상 FROM 테이블`



```
❓ 문제. Customers 테이블의 모든 데이터를 불러오자.
```

```sql
💡 답.
SELECT * FROM Customers;
-- Customers 테이블의 모든 데이터를 갖고 오기.
-- 주석다는 방식은 파이썬과 달리 #이 아니라 -- 임.
```

<img src = "https://user-images.githubusercontent.com/115082062/202318907-8633b3ac-e429-4627-a8ec-d70dabb95293.jpg">

출력 결과. 웹에서 코드를 실행했을 때 이러한 인터페이스로 나온다.

<br>

### **원하는 column(열)만 골라서 보기.** 

여러 컬럼을 콤마로 `select`해주면 된다.

```sql
SELECT CustomerName, ContactName FROM Customers;
-- Customers 테이블에서 CustomerName과 ContactName 열 가져오기
```

만약 테이블의 컬럼이 아닌 값을 선택하면, 그 값들을 포함하여 출력됨. 이 때 컬럼이 아닌 단순 문자열(string)은 따옴표로 묶어준다. NULL은 아무 값도 없는 상태를 뜻한다.

<br>

```
❓ 문제. 
Customers 테이블에서 CustomerName 컬럼을 숫자 23, 문자 ‘married’, NULL과 함께 불러오자.
```
```sql
💡 답.
SELECT CustomerName, 23, 'married', NULL FROM Customers;
```

<br>

### **원하는 조건의 행(row)만 걸러서 보기**

`WHERE` 구문 뒤에 조건을 붙여 원하는 데이터만 가져올 수 있다. 조건에 부합하는 행만 가져온다.

```sql
SELECT * FROM Orders
WHERE EmployeeID = 3;
-- Orders 테이블 중에서 EmployeeID 열이 3인 행들만 가져오기

SELECT * FROM OrderDetails
WHERE Quantity < 5;
-- OrderDetails 테이블 중에서 Quantity가 5보다 작은 행들만 가져오기
```

<br>

### **원하는 순서로 데이터 가져오기**

`ORDER BY` 구문을 사용하면 특정 컬럼을 기준으로 데이터를 정렬할 수 있다. 기본값은 `ASC(오름차순)`이며, `DESC(내림차순)`으로 바꿀 수 있다.


```
❓ 문제. 
Customers 테이블에서 거주 도시를 오름차순으로 먼저 정렬하고 거주 국가를 내림차순으로 정렬하라.
```

```sql
💡 답.
SELECT * FROM Customers
ORDER BY City ASC, Country DESC;
```
<br>

### **원하는 만큼만 데이터 가져오기**

`LIMIT 가져올 개수`  또는 `LIMIT 건너뛸 갯수, 가져올 갯수` 를 사용하여 원하는 만큼만 데이터 가져올 수 있다.


```
❓ 문제. 
Orders 테이블에서 CustomerID, OrderDate 열을 처음 20개를 건너뛰고 50개만큼만 불러오자.
```

```sql
💡 답.
SELECT CustomerID, OrderDate FROM Orders
LIMIT 20, 50;
```
<br>

### **원하는 별명(alias)으로 데이터 가져오기**

`AS`를 사용해서 컬럼명을 임의로 변경할 수 있다.


```
❓ 문제.
Orders 테이블에서 CustomerID 컬럼명을 ‘ID’로 바꾸고 OrderDate 컬럼명을 ‘주문날짜’로 바꿔 불러오자.
```

```sql
💡 답.
SELECT 
CustomerID AS ID,
OrderDate AS '주문날짜'    -- 한글 문자열은 따옴표로 써줘야 함.
FROM Orders
```


```
❓ 문제.
OrderDetails 테이블에서 OrderID 컬럼과 Quantity 컬럼을 각각 ‘주문아이디’,
‘주문수량’으로 변경하고, 이 때 주문수량이 20 이상인 행만 불러오자. 
순서는 ProductID를 오름차순으로 정렬한고, 이중에서 처음 10개의 행만 불러온다.
```

```sql
💡 답.
SELECT OrderID AS '주문아이디',
Quantity AS '주문수량'
FROM OrderDetails
WHERE Quantity >= 20
ORDER BY ProductID
LIMIT 10;
```


