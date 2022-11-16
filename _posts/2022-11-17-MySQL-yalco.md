# 갖고 노는 MySQL 데이터베이스 강좌


💡 유튜브 채널 ‘얄팍한 코딩사전’에서 제공하는 강좌입니다. SQL의 기초적인 내용을 웹에서 실습해볼 수 있는 강좌입니다. 
강좌에서 배운 개념을 바탕으로 직접 문제를 만들고, 이를 풀어보는 식으로 독학했습니다.

<br>

> **강좌 링크**

[MySQL](https://www.yalco.kr/lectures/sql/)

<br> 

---

### 오류 발생시

```sql
Error Code: 1046. No database selected Select the default DB to be used by double-clicking its name in the SCHEMAS list in the sidebar.
```

위와 같은 오류가 발생하면, schema탭에서 쿼리를 실행할 데이터베이스를 더블클릭하고 코드를 실행해주면 해결된다.

<br>

---

## Section1. SELECT기초 - 원하는 정보 가져오기
<br>

실습 웹사이트 링크

[MySQL Tryit Editor v1.0](https://www.w3schools.com/mysql/trymysql.asp?filename=trysql_select_all)

### 1-1 SELECT 전반 기능 훑어보기

본 강좌의 초반부는 SQL을 설치하지 않고 웹 상에서 SQL 코드를 실행하게 된다. 해당 웹에서는 아래와 같은 database를 제공하고 있다.

tip: MySQL의 구문들은 대소문자 구분 없이 사용 가능하고, 줄바꿈도 문제 없다.
<div align=center>
<img src="https://user-images.githubusercontent.com/115082062/202318292-1fa1efed-385f-4fa3-90e7-c05aa2235259.png">
</div>

<br>



- **테이블의 모든 내용 보기.**

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

- **원하는 column(열)만 골라서 보기.** 

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

- **원하는 조건의 행(row)만 걸러서 보기**

`WHERE` 구문 뒤에 조건을 붙여 원하는 데이터만 가져올 수 있다. 조건에 부합하는 행만 가져온다.

```sql
SELECT * FROM Orders
WHERE EmployeeID = 3;
-- Orders 테이블 중에서 EmployeeID 열이 3인 행들만 가져오기

SELECT * FROM OrderDetails
WHERE Quantity < 5;
-- OrderDetails 테이블 중에서 Quantity가 5보다 작은 행들만 가져오기
```



- **원하는 순서로 데이터 가져오기**

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

1. ****원하는 만큼만 데이터 가져오기****

`LIMIT 가져올 개수`  또는 `LIMIT 건너뛸 갯수, 가져올 갯수` 를 사용하여 원하는 만큼만 데이터 가져올 수 있다.

---

**문제.** Orders 테이블에서 CustomerID, OrderDate 열을 처음 20개를 건너뛰고 50개만큼만 불러오자.

**답.**

```sql
SELECT CustomerID, OrderDate FROM Orders
LIMIT 20, 50;
```

---

1. ****원하는 별명(alias)으로 데이터 가져오기****

`AS`를 사용해서 컬럼명을 임의로 변경할 수 있다.

---

<aside>
❓ **문제.** Orders 테이블에서 CustomerID 컬럼명을 ‘ID’로 바꾸고 OrderDate 컬럼명을 ‘주문날짜’로 바꿔 불러오자.

</aside>

<aside>
💡 **답.**

</aside>

```sql
SELECT 
CustomerID AS ID,
OrderDate AS '주문날짜'    -- 한글 문자열은 따옴표로 써줘야 함.
FROM Orders
```

---

<aside>
❓ **문제.**  OrderDetails 테이블에서 OrderID 컬럼과 Quantity 컬럼을 각각 ‘주문아이디’, ‘주문수량’으로 변경하고, 이 때 주문수량이 20 이상인 행만 불러오자. 순서는 ProductID를 오름차순으로 정렬한고, 이중에서 처음 10개의 행만 불러온다.

</aside>

<aside>
💡 **답.**

</aside>

```sql
SELECT OrderID AS '주문아이디',
Quantity AS '주문수량'
FROM OrderDetails
WHERE Quantity >= 20
ORDER BY ProductID
LIMIT 10;
```

---

### 1-2 각종 연산자들

1. **사칙연산**

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

<aside>
❓ **문제**. Products 테이블에서 상품들의 가격을 30% 올리고 increased_price라는 컬럼명으로 ProductName과 함께 출력해보자. 이 때 인상된 가격이 30 이상인 것들을 오름차순으로 출력한다.

</aside>

<aside>
💡 **답.**

</aside>

```sql
SELECT 
ProductName,
Price*1.3 AS increased_Price
FROM Products
WHERE Price*1.3 >= 30
ORDER BY Price;
```

![1.jpg](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/1.jpg)

---

1. **참/거짓 관련 연산자**

MySQL에서 `TRUE`는 1, `FALSE`는 0으로 저장된다.

```sql
SELECT True, False, !True, NOT 1, NOT False; --앞에 !나 NOT을 붙이면 그 역을 출력
```

![2.jpg](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/2%201.jpg)

`AND`는 양쪽이 모두 TRUE일 때만 TRUE를 반환하고, `OR`는 한쪽만 TRUE여도 TRUE를 반환한다.

```sql
SELECT TRUE IS TRUE, -- 1
TRUE IS NOT TRUE, -- 0
(TRUE IS FALSE) IS NOT TRUE, --1
TRUE AND FALSE; --0
```

---

<aside>
❓ **문제**. Customers 테이블에서 거주 국가가 ‘Mexico’이거나 ‘UK’인 고객들을 불러오자.

</aside>

<aside>
💡 **답**.

</aside>

```sql
SELECT * FROM Customers
where Country = 'UK' or Country= 'Mexico';
```

---

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

<aside>
❓ **문제**. Customers 테이블에서 CustomerName이 c~g로 시작하는 행들 중 Country가 UK, Mexico, Brazil, Spain인 행들을 불러오자.

</aside>

<aside>
💡 **답**.

</aside>

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

**연산자 정리**

| 연산자 | 의미 |
| --- | --- |
| +, -, *, / | 각각 더하기, 빼기, 곱하기, 나누기 |
| %, MOD | 나머지 |
| IS | 양쪽이 모두 TRUE 또는 FALSE |
| IS NOT | 한쪽은 TRUE, 한쪽은 FALSE |
| AND, && | 양쪽이 모두 TRUE일 때만 TRUE |
| OR, || | 한쪽은 TRUE면 TRUE |
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

---

### 1-3 숫자와 문자열을 다루는 함수들

1. **숫자 관련 함수들**

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

<aside>
❓ **문제**. Products 테이블에서 Price의 평균, 최댓값, 최솟값을 출력하라. 이 때 각 값들은 반올림하도록 한다.

</aside>

<aside>
💡 **답**.

</aside>

```sql
SELECT 
round(avg(Price)) as '평균가격',
round(max(Price)) as '최대가격',
round(min(Price)) as '최소가격'
from Products;
```

![1.jpg](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/1%201.jpg)

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

1. **문자열 관련 함수들**

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

<aside>
❓ **문제**. Suppliers 테이블에서 City는 대문자로, Address는 소문자로 출력하고, SupplierName을 내림차순으로 정렬하라.

</aside>

<aside>
💡 **답**.

</aside>

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

![3.jpg](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/3.jpg)

---

<aside>
❓ **문제**. Customers 테이블에서 Country와 City를 컴마로 묶어서 불러오자(location으로 컬럼명 변경). 이 때 Country를 오름차순으로 정렬하고 그 후에 City를 오름차순으로 정렬한다.

</aside>

<aside>
💡 **답**.

</aside>

```sql
SELECT 
CustomerName,
concat_ws(', ', City, Country)
FROM Customers
order by Country, City;
```

---

![4.jpg](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/4.jpg)

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

![5.jpg](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/5.jpg)

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

<aside>
❓ **문제**. Employees 테이블에서 생년월일을 yy.mm.dd. 형식으로 바꾸고, LastName, FirstName과 함께 출력하라.

</aside>

<aside>
💡 **답**.

</aside>

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

### 1-4 시간/날짜 관련 및 기타 함수들

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

<aside>
❓ **문제**. Orders 테이블에서 주문날짜가 1997년 3월부터 1998년 4월까지인 것 중 ShipperID가 1인 것만을 불러오자.

</aside>

<aside>
💡 **답**.

</aside>

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

![1.jpg](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/1%202.jpg)

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

<aside>
❓ **문제**. 1998년 2월 10일은 구정이다. Orders 테이블에서 구정 앞뒤 1주일의 기간동안 주문한 상품은 배송이 지연될 것으로 예상된다. 주문이 지연되는 상품들만 불러오자.

</aside>

<aside>
💡 **답**.

</aside>

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

<aside>
❓ **문제**. OrderDetails 테이블에서 Quantity가 45 이상이면 ‘초대량주문’, 20 이상 45 미만이면 ‘대량주문’, 20 미만이면 ‘일반주문’으로 표시되도록 하자.

</aside>

<aside>
💡 **답**.

</aside>

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

---

### 1-5 조건에 따라 그룹으로 묶기

1. **GROUP BY - 조건에 따라 집계된 값 가져오기**

`GROUP BY`는 조건에 따라 집계된 값을 가져와준다. `GROUP BY 컬럼명`을 입력하면 그 컬럼을 기준으로 집계된 값들을 보여준다.

```sql
SELECT 
Country,
COUNT(*) AS COUNT
FROM Customers
GROUP BY Country;
```

![Country를 기준으로 빈도를 count하여 집계된 값을 보여준다.](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/1%203.jpg)

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

<aside>
❓ **문제**. Products 테이블에서 CategoryID를 기준으로 상품의 최댓값, 최솟값, 평균, 아이디별 개수를 집계해보자. 그리고 마지막 행에 전체의 집계도 나타내보자.

</aside>

<aside>
💡 **답**.

</aside>

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

![마지막 행에 WITH ROLLUP이 실행되어 있다.](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/1%201.png)

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

1. **DISTINCT - 중복된 값들을 제거하기**

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

## Section2. SELECT 더 깊이 파보기

### 2-1 쿼리 안에 서브쿼리

1. **비상관 서브쿼리**

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

<aside>
❓ **문제.** 배송회사는 1960년 이후 출생한 직원들의 영어권 고객(UK, USA) 대상 배달실적을 조사해보려 한다. Orders 테이블에서 OrderID, CustomerID, EmployeeID를 불러오고, 서브쿼리로 조건을 걸어 데이터를 적절히 불러오자.

</aside>

<aside>
💡 **답.**

</aside>

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

1. **상관 서브쿼리**

비상관 서브쿼리란 건 원래 쿼리와 서브쿼리가 독자적으로 실행되는 경우다. 그러나 지금 다룰 **상관 서브쿼리**는 원래 쿼리와 서브쿼리가 맞물려서 실행된다. 비상관 서브쿼리는 서브쿼리를 써서 다른 테이블의 컬럼으로 조건을 걸 수 있었지만, 상관 서브쿼리는 다른 테이블의 컬럼을 직접 출력하는 것도 가능하다.

![1.jpg](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/1%204.jpg)

보다시피 Products 테이블에 CategoryID는 있어도 CategoryName은 없다. CategoryName은 Categories 테이블에만 있다. 만약 Products 테이블의 컬럼을 가져오면서 CategoryName도 갖고오고 싶다면, 상관 서브쿼리를 써야 한다.

```sql
SELECT 
ProductID, ProductName,
(SELECT CategoryName FROM Categories C -- Categories 테이블을 C로 저장
WHERE C.CategoryID = P.CategoryID) AS CategoryName
FROM Products P; -- Products 테이블을 P로 저장
```

![Products 테이블의 컬럼을 부르고 Categories 테이블(다른 테이블)의 컬럼까지도 불러왔다.](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/3%201.jpg)

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

<aside>
💡 **문제.**  OrderID별로 총 주문금액이 얼마인지, 배송을 담당하는 Employee의 이름은 무엇인지를 알아내려고 한다. OrderDetails 테이블에서 쿼리를 만들고 Products 테이블과 Employees테이블, Orders 테이블을 적절히 서브쿼리에 활용하여 값을 추출하라

</aside>

<aside>
💡 **답.**

</aside>

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

![4.jpg](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/4%201.jpg)

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

### 2-2 JOIN - 여러 테이블 조립하기

1. **JOIN(INNER JOIN) - 내부 조인**

JOIN은 테이블들을 합쳐 하나의 테이블로 만드는 기능을 한다. 그 중 내부 조인은 양쪽 모두에 값이 있는 행, 즉 NULL이 아닌 행을 반환한다. 기본 문법은 다음과 같다.

```sql
SELECT * FROM Categories C -- Categories 테이블과
JOIN Products P -- Products 테이블을 조인한다
ON C.CategoryID = P.CategoryID; -- 두 테이블의 CategoryID가 같은 것을 기준으로
```

---

<aside>
❓ **문제.** 배송회사에서 각 주문 건마다 OrderID, 고객명, 고객주소, 도시를 한번에 볼 수 있게 하나의 테이블로 만들려고 한다. JOIN을 활용하여 이 작업을 수행하라.

</aside>

<aside>
💡 **답.**

</aside>

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

![1.jpg](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/1%205.jpg)

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

<aside>
❓ **문제.** 직원별 배송실적을 알아보려고 한다. 직원의 이름(Last Name과 First Name 모두)과 직원별 총 판매 건 수(Total_Quantity)를 출력하라. 또한 오늘이 2000년 1월 1일이라고 가정하고, 직원들의 나이도 함께 출력하라.

</aside>

<aside>
💡 **답.**

</aside>

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

![왼쪽엔 ID9가, 오른쪽엔 ID1이 없다. 이는 INNER JOIN이기 때문이다.](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/1%202.png)

왼쪽엔 ID9가, 오른쪽엔 ID1이 없다. 이는 INNER JOIN이기 때문이다.

---

1. **LEFT/RIGHT OUTER JOIN - 외부 조인**

INNER JOIN과 달리 반대쪽에 데이터가 있든 없든 선택된 방향(LEFT/RIGHT)에 있으면 출력한다. 아까의 예시를 외부조인으로 출력해보자.

```sql
SELECT
  E1.EmployeeID, CONCAT_WS(' ', E1.FirstName, E1.LastName) AS Employee,
  E2.EmployeeID, CONCAT_WS(' ', E2.FirstName, E2.LastName) AS NextEmployee
FROM Employees E1
LEFT JOIN Employees E2
ON E1.EmployeeID + 1 = E2.EmployeeID
```

![LEFT JOIN을 했더니 왼쪽 열엔 오른쪽 값이 NULL이더라도 ID9가 마저 출력됐다.](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/2.png)

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

<aside>
❓ **문제.** Alfreds Futterkiste 회사는 자신들이 주문한 상품 중 자신의 국가(Germany)에 연고를 둔 공급업체가 다루는 상품이 있는지 알고 싶다. 만약 그런 공급업체가 있다면 그 업체의 이름을 출력하고, 상품의 공급업체가 Germany가 아니라면 업체 이름은 null, 국가명은’—discord—’를 출력하라

</aside>

<aside>
💡 **답.**

</aside>

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

1. **CROSS JOIN(교차조인)**

내부조인과 외부조인은 모두 조건(ON)을 걸어줬다. 하지만 **CROSS JOIN(교차조인)**은 조건 없이 모든 조합을 반환한다. 아래 코드를 보면 같은 Employees 테이블을 두 개 활용하여 CROSS JOIN을 했다. 이 때 Employees의 행은 9개이므로 이 두 테이블을 조건없이 조합하면 9*9=81개의 행을 가진 테이블이 완성된다.

```sql
SELECT
  E1.LastName, E2.FirstName
FROM Employees E1
CROSS JOIN Employees E2
ORDER BY E1.EmployeeID;
```

---

### 2-3 UNION - 집합으로 다루기

1. **UNION**

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

1. 합집합

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

1. 교집합

4보다 크고 8보다 작거나 같은 자연수와 8보다 작거나 같은 짝수의 교집합

```sql
SELECT CategoryID AS ID
FROM Categories C, Employees E
WHERE
C.CategoryID > 4 -- 4보다 크고 8보다 작거나 같은 자연수
AND E.EmployeeID % 2 = 0 -- 8보다 작거나 같은 짝수
AND C.CategoryID = E.EmployeeID; -- 이들의 교집합
```

1. 차집합

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

1. 대칭차집합

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

---

## Section3. 데이터 조작하기

### 3-1 MySQL 설치하기

[MySQL :: Begin Your Download](https://dev.mysql.com/downloads/file/?id=510039)

윈도우의 경우 Community Server와 Workbench, Sample Datebase를 한번에 설치할 수 있다. 설치가 끝나면 Wokrbench를 실행한다.

![1.jpg](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/1%206.jpg)

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

<aside>
❓ **문제.** 주어진 sakila 데이터베이스로 문제를 풀어보자. acotr_id에 따라서 그 actor가 출연한 영화를 보여주는 코드를 작성하라.

</aside>

<aside>
💡 **답.**

</aside>

```sql
SELECT concat_ws(' ', A.first_name, A.last_name) AS actor_name, F.title FROM actor A
JOIN film_actor FA on A.actor_id = FA.actor_id
JOIN film F on F.film_id = FA.film_id
WHERE A.actor_id = 1;
```

---

Workbench는 엄밀히 말하면 MySQL이 아니라, 쉽게 작동시키기 위한 클라이언트 툴이다. 진짜 MySQL은 **C:\Program Files\MySQL\MySQL Server 8.0\bin** 에서 MySQL shell을 실행해야 한다.

---

### 3-2 테이블 만들고 데이터 입력하기

`CREATE TABLE` 명령어로 테이블을 만들 수 있다. player이란 이름의 테이블을 만들어보자.

```sql
CREATE TABLE player (
	player_id INT, -- 정수 자료형
    player_name VARCHAR(10), -- 문자열(최대10글자)
    age TINYINT, -- 작은 숫자 자료형
    debut DATE); -- 날짜 자료형
```

물론 왼쪽 탭에서 우클릭 후 create table을 누르면 아래처럼 명령어 없이 테이블을 만들 수도 있다.

![2.jpg](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/2%202.jpg)

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

---

### 3-3 자료형(+예제데이터)

1. **숫자 자료형**

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

1. **문자 자료형**

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

1. **시간 자료형**

`DATETIME`은 어디에서 읽든 동일한 값이지만, `TIMESTAMP`는 시간을 자동기록한다거나 국제적인 서비스를 이용할 때 사용한다.

| 자료형 | 설명 | 비고 |
| --- | --- | --- |
| DATE | YYYY-MM-DD |  |
| TIME | HHH:MI:SS | HHH: -838 ~ 838까지의 시간 |
| DATETIME | YYYY-MM-DD HH:MI:SS | 입력된 시간을 그 값 자체로 저장 |
| TIMESTAMP | YYYY-MM-DD HH:MI:SS | MySQL이 설치된 컴퓨터의 시간대를 기준으로 저장 |

---

1. 앞으로 예제에서 사용할 테이블

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

### 3-4 데이터 변경, 삭제하기

1. **DELETE - 주어진 조건의 행 삭제하기**

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

![business_id(auto_increment)가 삭제 전 마지막행(18행)부터 1씩 증가한다.](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/1%207.jpg)

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

![business_id(auto_increment)가 초기화되어 다시 1부터 시작한다.](%E1%84%80%E1%85%A1%E1%86%BD%E1%84%80%E1%85%A9%20%E1%84%82%E1%85%A9%E1%84%82%E1%85%B3%E1%86%AB%20MySQL%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%80%E1%85%A1%E1%86%BC%E1%84%8C%E1%85%AA%20597b00ff80c14ee0b0a19d6541f718ec/2%203.jpg)

business_id(auto_increment)가 초기화되어 다시 1부터 시작한다.

---

1. **UPDATE - 주어진 조건의 행 수정하기**

<aside>
❓ **문제.** 철구분식은 좋아요가 음수인 찹쌀순대 메뉴를 ‘고른햇살 토종순대’로 바꾸려고 한다. 이 때 가격은 1,000원 인상할 것이다. update문을 사용하여 수정하라.

</aside>

<aside>
💡 **답.**

</aside>

```sql
update menus
set menu_name = '고른햇살 토종순대',
	price = price +1000
where fk_business_id =2 and menu_name='찹쌀순대';
```

---

<aside>
❓ **문제.** 베이징올림픽 오심 여파로 중식 보이콧이 일어나면서 중식 메뉴들의 좋아요가 5씩 내려갔다. 때문에 section이 중식인 가게들은 전 메뉴 가격을 10% 할인하기로 했다. update문을 사용하여 수정하라.

</aside>

<aside>
💡 **답.**

</aside>

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

---

`where` 조건문 없이 update를 하면 모든 행이 변경된다.

---