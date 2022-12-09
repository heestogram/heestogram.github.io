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

> 실습 웹사이트 링크

[MySQL Tryit Editor v1.0](https://www.w3schools.com/mysql/trymysql.asp?filename=trysql_select_all) 


# Section2. SELECT 더 깊이 파보기

## 2-1 쿼리 안에 서브쿼리

### **비상관 서브쿼리**

**서브쿼리**란 쿼리 안에 있는 쿼리를 말한다. 아래 코드처럼 SELECT 안에 또 SELECT가 있는 경우가 서브쿼리이다.

```sql
SELECT * FROM Products
WHERE Price < (SELECT AVG(Price) FROM Products);

#만약 위 코드를 서브쿼리를 쓰지않고
SELECT * FROM Products
WHERE Price AVG(Price);
#라고만 쓰면 Invalid use of group function 오류가 뜬다.
```

서브쿼리를 사용하면 다른 테이블의 정보도 활용할 수 있다. 아래는 Categories 테이블의 컬럼을 뽑아내고 Products 테이블에서 조건을 걸어준 경우다.

```sql
SELECT
	CategoryID, CategoryName, Description
FROM Categories
WHERE
	CategoryID = (SELECT CategoryID FROM Products
    WHERE ProductName = 'Chais');
```

---


❓ **문제.** 
```
배송회사는 1960년 이후 출생한 직원들의 영어권 고객(UK, USA) 대상 배달실적을 조사해보려 한다. Orders 테이블에서 OrderID, CustomerID, EmployeeID를 불러오고, 서브쿼리로 조건을 걸어 데이터를 적절히 불러오자.
```

💡 **답.**

```sql
SELECT
	OrderID, CustomerID, EmployeeID
FROM Orders
WHERE
	EmployeeID IN
    (SELECT EmployeeID FROM Employees
    WHERE BirthDate > DATE('1960-01-01'))
    AND
    CustomerID IN
    (SELECT CustomerID FROM Customers
    WHERE Country = 'UK' or Country= 'USA');
```

---

| 연산자 | 의미 |
| --- | --- |
| ~ ALL | 서브쿼리의 모든 결과에 대해 ~하다 |
| ~ ANY | 서브쿼리의 하나 이상의 결과에 대해 ~하다 |

```sql
SELECT * FROM Products
WHERE Price > ALL(
	SELECT Price FROM Products
    WHERE CategoryID = 2);
-- CategoryID가 2인 모든 Product보다 가격이 높은 것만 추출한다는 것.

-- CategoryID가 2인 Product 중 가장 비싼 것은?
SELECT max(Price)
FROM Products
WHERE CategoryID=2; --43.9
-- 즉, 43.9보다 비싼 Products들을 추출한다는 의미.
```

```sql
SELECT
	CategoryID, CategoryName, Description
FROM Categories
WHERE
	CategoryID = ANY(SELECT CategoryID FROM Products WHERE Price>50);
-- 가격이 50만 넘으면 그 CategoryID는 출력됨. 
-- CategoryID 중 2가 출력이 안됐다면, ID가 2인 Product 중 그 어떤 것도 50보다 비싼 게 없다는 의미.
```

---

<br>

### **상관 서브쿼리**

비상관 서브쿼리란 건 원래 쿼리와 서브쿼리가 독자적으로 실행되는 경우다. 그러나 지금 다룰 **상관 서브쿼리**는 원래 쿼리와 서브쿼리가 맞물려서 실행된다. 비상관 서브쿼리는 서브쿼리를 써서 다른 테이블의 컬럼으로 조건을 걸 수 있었지만, 상관 서브쿼리는 다른 테이블의 컬럼을 직접 출력하는 것도 가능하다.

<img src="https://user-images.githubusercontent.com/115082062/206619591-e3b65c23-646f-426d-9f44-535f9b229da2.jpg">

보다시피 Products 테이블에 CategoryID는 있어도 CategoryName은 없다. CategoryName은 Categories 테이블에만 있다. 만약 Products 테이블의 컬럼을 가져오면서 CategoryName도 갖고오고 싶다면, 상관 서브쿼리를 써야 한다.

```sql
SELECT 
ProductID, ProductName,
(SELECT CategoryName FROM Categories C -- Categories 테이블을 C로 저장
WHERE C.CategoryID = P.CategoryID) AS CategoryName
FROM Products P; -- Products 테이블을 P로 저장
```

<img src="https://user-images.githubusercontent.com/115082062/206619816-ac8220c8-6386-4d18-bf72-abc0ee70dad1.jpg">

Products 테이블의 컬럼을 부르고 Categories 테이블(다른 테이블)의 컬럼까지도 불러왔다.

이번에는 Suppliers 테이블에서 Customers 테이블의 컬럼을 불러오려고 한다. Supplier마다 Country와 City가 같은 Customer의 수를 세는 것이다.

```sql
SELECT 
	SupplierName, Country, City,
    (SELECT COUNT(*) FROM Customers C
    WHERE C.Country = S.Country) AS CustomersInTheCountry,
    (SELECT COUNT(*) FROM Customers C
    WHERE C.City = S.City) AS CustomersInTheCity
FROM Suppliers S;
```

---

❓ **문제.**  
```
OrderID별로 총 주문금액이 얼마인지, 배송을 담당하는 Employee의 이름은 무엇인지를 알아내려고 한다. OrderDetails 테이블에서 쿼리를 만들고 Products 테이블과 Employees테이블, Orders 테이블을 적절히 서브쿼리에 활용하여 값을 추출하라
```

💡 **답.**

```sql
SELECT 
OrderID, ProductID, Quantity,
	SUM((SELECT Price FROM Products P -- Products 테이블에서 Price 정보를 갖고오는데
    WHERE P.ProductID = O.ProductID) -- ProductID가 서로 같은 정보만 갖고오고
		*Quantity) -- 이 Price를 수량과 곱하여
    AS total_price,
    
    (SELECT concat_ws(' ',LastName,FirstName) -- 이름을 조합하여
    FROM Employees E -- Employees 테이블에서 가져오는데
    WHERE (SELECT EmployeeID FROM Orders Ord -- Orders테이블과 OrderDetails테이블의 OrderID가 같은 경우의 EmployeeID랑 같은 경우에만 갖고온다.
    		WHERE O.OrderID = Ord.OrderID) = E.EmployeeID) 
            AS Employeename
FROM OrderDetails O
GROUP BY OrderID -- 각 OrderID별로 이 값의 합을 출력한다.
```

<img src="https://user-images.githubusercontent.com/115082062/206619927-92e21b90-0eca-4a65-b3ee-7f6d65ad2a1e.jpg">

---

`EXISTS`는 해당 값이 존재한다는 연산자이다.

```sql
SELECT
	CategoryID, CategoryName
FROM Categories C
WHERE EXISTS(SELECT * FROM Products P -- 존재하면 출력하라
			WHERE P.CategoryID=C.CategoryID -- CategoryID가 같은 것중에
            AND P.Price>80); -- 가격이 80을 넘어가는 것이
```

---