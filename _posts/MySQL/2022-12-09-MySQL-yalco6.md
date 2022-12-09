---
title: "[MySQL] 2-2, 2-3 JOIN, LEFT/RIGHT OUTER JOIN, UNION"
excerpt: "서로 다른 테이블을 합쳐주는 JOIN의 다양한 형태를 알고, UNION으로 집합을 구현해보자"

toc: true
toc_label: "목차"
toc_sticky: true

published: true

categories:
  - MySQL
tags: [MySQL, yalco, JOIN]

date: 2022-12-09 13:15:00
last_modified_at: 2022-12-09 13:15:00
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

<br>

# Section2. SELECT 더 깊이 파보기

## 2-2 JOIN - 여러 테이블 조립하기

### **JOIN(INNER JOIN) - 내부 조인**

JOIN은 테이블들을 합쳐 하나의 테이블로 만드는 기능을 한다. 그 중 내부 조인은 양쪽 모두에 값이 있는 행, 즉 NULL이 아닌 행을 반환한다. 기본 문법은 다음과 같다.

```sql
SELECT * FROM Categories C -- Categories 테이블과
JOIN Products P -- Products 테이블을 조인한다
ON C.CategoryID = P.CategoryID; -- 두 테이블의 CategoryID가 같은 것을 기준으로
```

---


❓ **문제.** 
```
배송회사에서 각 주문 건마다 OrderID, 고객명, 고객주소, 도시를 
한번에 볼 수 있게 하나의 테이블로 만들려고 한다. 
JOIN을 활용하여 이 작업을 수행하라.
```

💡 **답.**


```sql
SELECT OrderID, CustomerID
FROM Orders O
JOIN Customers C
ON C.OrderID = O.OrderID;
-- 이 경우 CustomerID가 ambiguous하다는 에러가 뜬다.
-- CustomerID는 두 테이블 모두에 있어서 어느 한 테이블을 앞에 지정해줘야 하기 때문.

--따라서 아래처럼 변수명 앞에 테이블 약자를 적어주는 습관을 들이자
SELECT O.OrderID, O.CustomerID, C.CustomerName, C.Address, C.City
FROM Orders O
JOIN Customers C
ON C.CustomerID = O.CustomerID;
```

<img src="https://user-images.githubusercontent.com/115082062/206622494-9ebf3530-50c5-4742-aa1e-2b7d412b4c08.jpg">


---

여러 테이블을 `JOIN`할 수도 있다. 

```sql
SELECT
	C.CategoryID, C.CategoryName,
    P.ProductName,
    O.OrderDate,
    D.Quantity
FROM Categories C
JOIN Products P
ON C.CategoryID = P.CategoryID
JOIN OrderDetails D
ON P.ProductID = D.ProductID
JOIN Orders O
ON O.OrderID = D.OrderID
```

이때 주의해야 할 것이 아래와 같이 join순서를 뒤죽박죽으로 하는 경우이다. 

```sql
SELECT
	C.CategoryID, C.CategoryName,
    P.ProductName,
    O.OrderDate,
    D.Quantity
FROM Categories C
JOIN OrderDetails D
ON P.ProductID = D.ProductID --아직 Products 테이블이 JOIN되기 전이라 
-- Unknown column 'P.ProductID' in 'on clause'이라는 에러가 뜸
JOIN Products P -- 따라서 이 행을 먼저 선언해줘야 함.
ON C.CategoryID = P.CategoryID

JOIN Orders O
ON O.OrderID = D.OrderID
```

`JOIN`한 테이블을 `GROUP`할 수도 있다.

```sql
SELECT
	C.CategoryName,
    MIN(O.OrderDate) AS Firstorder,
    MAX(O.OrderDate) AS LastOrder,
    SUM(D.Quantity) AS TotalQuantity
FROM Categories C
JOIN Products P
ON C.CategoryID = P.CategoryID
JOIN OrderDetails D
ON P.ProductID = D.ProductID
JOIN Orders O
ON O.OrderID = D.OrderID
GROUP BY C.CategoryID;
```

---


❓ **문제.** 
```
직원별 배송실적을 알아보려고 한다. 
직원의 이름(Last Name과 First Name 모두)과 직원별 총 판매 건 수(Total_Quantity)를 출력하라. 
또한 오늘이 2000년 1월 1일이라고 가정하고, 직원들의 나이도 함께 출력하라.
```

💡 **답.**

```sql
SELECT 
	CONCAT_WS(' ',E.LastName,E.FirstName) AS EmployeeName, -- 이름 조합
    YEAR('2000-1-1')-YEAR(E.BirthDate)+1 AS Age, -- 2000년1월1일의 년도에서 출생년도 빼고 1 더하기
	SUM(D.Quantity) AS Total_Quantity -- OrderDetails 테이블의 Quantity값을 EmployeeID 그룹 기준으로 합
FROM Employees E
JOIN Orders O
ON O.EmployeeID = E.EmployeeID
JOIN OrderDetails D
ON O.OrderID = D.OrderID

GROUP BY O.EmployeeID;
```

---

경우에 따라 같은 테이블끼리 SELF JOIN을 할 때도 있다. 아래 코드의 결과를 보면 E1 테이블에선 ID 9번이 없고, E2 테이블에선 ID 1번이 없다. 같은 행에 NULL이 있어서 그 행을 가져오지 않았기 때문이다. 이것이 바로 서두에서 언급한 INNER JOIN의 특성, 양쪽 모두 값이 있는 행만 가져온다는 특성이다.

```sql
select
	E1.EmployeeID, CONCAT_WS(' ', E1.FirstName, E1.LastName) AS Employee,
    E2.EmployeeID, CONCAT_WS(' ', E2.FirstName, E2.LastName) AS NextEmployee
FROM Employees E1
JOIN Employees E2
ON E1.EmployeeID+1 = E2.EmployeeID
```

<img src="https://user-images.githubusercontent.com/115082062/206622667-27842cbd-b5df-4425-b407-349bee4273dc.png">

왼쪽엔 ID9가, 오른쪽엔 ID1이 없다. 이는 INNER JOIN이기 때문이다.

---

<br>

### **LEFT/RIGHT OUTER JOIN - 외부 조인**

INNER JOIN과 달리 반대쪽에 데이터가 있든 없든 선택된 방향(LEFT/RIGHT)에 있으면 출력한다. 아까의 예시를 외부조인으로 출력해보자.

```sql
SELECT
  E1.EmployeeID, CONCAT_WS(' ', E1.FirstName, E1.LastName) AS Employee,
  E2.EmployeeID, CONCAT_WS(' ', E2.FirstName, E2.LastName) AS NextEmployee
FROM Employees E1
LEFT JOIN Employees E2
ON E1.EmployeeID + 1 = E2.EmployeeID
```

<img src="https://user-images.githubusercontent.com/115082062/206622728-091a8d04-60f9-47ed-a344-59843e729b1c.png">

LEFT JOIN을 했더니 왼쪽 열엔 오른쪽 값이 NULL이더라도 ID9가 마저 출력됐다.

```sql
SELECT
	C.CustomerName, S.SupplierName,
    C.City, C.Country
FROM Customers C
LEFT JOIN Suppliers S --LEFT인 경우엔 Suppliers엔 없지만 Customers엔 있는 것들 모조리 출력
--반대로, RIGHT인 경우엔 Customers엔 없지만 Suppliers엔 있는 것들 모조리 출력.
ON C.City = S.city AND C.Country = S.Country;
```

```sql
SELECT 
	IFNULL(C.CustomerName, '-- No Customer --'),
    IFNULL(S.SupplierName, '-- No Supplier --'),
    IFNULL(C.City, S.City),
    IFNULL(C.Country, S.Country),
FROM Customers C
right JOIN Suppliers S
ON C.City = S.City AND C.Country = S.Country;
-- 이처럼 코드를 작성하면 NULL 값에 적당한 값을 채워넣어서 빈칸을 메울 수 있다.
```

---

❓ **문제.** 
```
Alfreds Futterkiste 회사는 자신들이 주문한 상품 중 
자신의 국가(Germany)에 연고를 둔 공급업체가 다루는 상품이 있는지 알고 싶다. 
만약 그런 공급업체가 있다면 그 업체의 이름을 출력하고, 
상품의 공급업체가 Germany가 아니라면 업체 이름은 null, 국가명은’—discord—’를 출력하라
```

💡 **답.**
```sql
SELECT 
	D.OrderDetailID,
    C.CustomerName, C.Country as customer_country,
    S.SupplierName,
    ifnull(S.Country,'--discord--') as supplier_country
FROM Customers C
JOIN Orders O
ON O.CustomerID = C.CustomerID
JOIN OrderDetails D
ON D.OrderID = O.OrderID
JOIN Products P
ON D.ProductID = P.ProductID

LEFT JOIN Suppliers S
ON P.SupplierID = S.SupplierID and C.Country = S.Country
WHERE C.CustomerName = 'Alfreds Futterkiste'
GROUP BY D.OrderDetailID;
```

---

<br>

### **CROSS JOIN(교차조인)**

내부조인과 외부조인은 모두 조건(ON)을 걸어줬다. 하지만 **CROSS JOIN(교차조인)**은 조건 없이 모든 조합을 반환한다. 아래 코드를 보면 같은 Employees 테이블을 두 개 활용하여 CROSS JOIN을 했다. 이 때 Employees의 행은 9개이므로 이 두 테이블을 조건없이 조합하면 9*9=81개의 행을 가진 테이블이 완성된다.

```sql
SELECT
  E1.LastName, E2.FirstName
FROM Employees E1
CROSS JOIN Employees E2
ORDER BY E1.EmployeeID;
```

---

<br>

## 2-3 UNION - 집합으로 다루기

### **UNION**

`UNION`은 데이터를 가로로 추가한 것이다. `JOIN`이 서로 다른 테이블에서 컬럼을 갖고 왔다면, 이번엔 서로 다른 테이블에서 행을 갖고와 위아래로 이어 붙인 것이다.

```sql
SELECT CustomerName AS Name, City, Country, 'CUSTOMER'
FROM Customers
UNION
SELECT SupplierName AS Name, City, Country, 'SUPPLIER'
FROM Suppliers
ORDER BY Name;
```

위같은 코드는 Customers 테이블, Suppliers 테이블 각각의 행을 불러와 이어붙인 것이다.

이 때 UNION은 중복을 제거한 집합을 출력한다. 반면 UNION ALL은 중복을 포함한 모든 값을 출력한다. 아래 예시를 보자.

```sql
SELECT CategoryID AS ID FROM Categories
WHERE CategoryID > 4 -- 4보다 ID가 큰 것은 5,6,7,8뿐이다.
UNION
SELECT EmployeeID AS ID FROM Employees
WHERE EmployeeID%2 = 0; -- 짝수 아이디는 2,4,6,8 뿐이다.
-- 이 때 중복을 제거하므로 출력값은 5,6,7,8,2,4일 것이다.
-- 만약 UNION ALL을 썼다면, 출력값은 5,6,7,8,2,4,6,8일 것이다.
```

---

### 합집합

SupplierID와 ProductID로 합집합을 만들어보자. 12보다 작은 짝수와 20보다 작은 자연수의 합집합을 찾을 것이다.

```sql
SELECT SupplierID AS ID, 'SUPPLIER' FROM Suppliers
WHERE SupplierID%2 = 0 AND SupplierID < 12 -- 12보다 작은 짝수
UNION -- 합집합
SELECT ProductID AS ID, 'PRODUCT' FROM Products
WHERE ProductID < 15 -- 215보다 작은 자연수
;
-- 중복을 허용하지 않으므로 총 14개 행이 나올 것이다.
```

### 교집합

4보다 크고 8보다 작거나 같은 자연수와 8보다 작거나 같은 짝수의 교집합

```sql
SELECT CategoryID AS ID
FROM Categories C, Employees E
WHERE
C.CategoryID > 4 -- 4보다 크고 8보다 작거나 같은 자연수
AND E.EmployeeID % 2 = 0 -- 8보다 작거나 같은 짝수
AND C.CategoryID = E.EmployeeID; -- 이들의 교집합
```

### 차집합

4보다 크고 8보다 작거나 같은 자연수에서 9보다 작은 짝수를 뺀 차집합

```sql
SELECT CategoryID AS ID
FROM Categories
WHERE
	CategoryID > 4 -- 4보다 크고 8보다 작거나 같은 자연수
    AND CategoryID NOT IN ( -- 차집합
    	SELECT EmployeeID 
        FROM Employees
        WHERE EmployeeID %2 = 0); -- 9보다 작은 짝수
```

### 대칭차집합

```sql
SELECT ID FROM (
  SELECT CategoryID AS ID FROM Categories
  WHERE CategoryID > 4
  UNION ALL -- 두 집합의 중복을 허용한 집합.
  SELECT EmployeeID AS ID FROM Employees
  WHERE EmployeeID % 2 = 0
) AS Temp 
GROUP BY ID HAVING COUNT(*) = 1; -- 만약 2번 이상 나타난 아이디가 있다면 그 아이디는 교집합이므로, 없애고 나머지 부분만 출력
```