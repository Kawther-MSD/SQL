# Advanced SQL Queries Project Documentation

## Project Description

This project is designed to provide practice in advanced SQL querying techniques using a relational schema representing a commercial database. The main focus is on querying, filtering, aggregating, and manipulating data to simulate real-world business scenarios.

## Database Schema Overview

CATEGORIES: Contains product category data.

CUSTOMERS: Stores customer contact and location details.

ORDERS: Tracks customer orders.

ORDER_DETAILS: Contains details of each item in an order.

EMPLOYEES: Holds employee information including hierarchy and compensation.

SUPPLIERS: Records supplier contact and location data.

PRODUCTS: Lists all products with details such as supplier, category, pricing, and availability.

## SQL Queries 
You will need first to execute this [sql_script](https://github.com/Kawther-MSD/SQL/blob/main/SQL/SQL_project.sql) in your SQL Server .

### 1. Male employees whose net salary is ≥8000, ordered by seniority :

```
-- Select employee details where net salary is >= 8000 and gender is male (assuming gender data is inferred from title)
SELECT EMPLOYEE_int, FIRST_NAME, LAST_NAME,
       YEAR(GETDATE()) - YEAR(BIRTH_DATE) AS AGE, -- Calculate age from birth date
       YEAR(GETDATE()) - YEAR(HIRE_DATE) AS SENIORITY -- Calculate seniority from hire date
FROM EMPLOYEES
WHERE (SALARY + ISNULL(COMMISSION, 0)) >= 8000
  AND TITLE IN ('Mrs.', 'Mr.') -- Assuming TITLE holds gender-representative titles
ORDER BY SENIORITY DESC;
```
<img src="https://github.com/Kawther-MSD/SQL/blob/main/SQL/Capture%20d'%C3%A9cran%202025-04-24%20191328.png" heigth="800" width="750">
### 2. Products matching multiple criteria

```
-- Select products with specified packaging, name character, supplier, price range, and specified units on order
SELECT PRODUCT_REF, PRODUCT_NAME, SUPPLIER_int, UNITS_ON_ORDER, UNIT_PRICE
FROM PRODUCTS
WHERE QUANTITY LIKE '%bottle%' -- C1: Packaged in bottles
  AND SUBSTRING(PRODUCT_NAME, 3, 1) IN ('t', 'T') -- C2: 3rd char is 't' or 'T'
  AND SUPPLIER_int IN (1, 2, 3) -- C3: Supplier 1, 2, or 3
  AND UNIT_PRICE BETWEEN 70 AND 200 -- C4: Price between 70 and 200
  AND UNITS_ON_ORDER IS NOT NULL; -- C5: Units ordered is specified
```
<img src="https://github.com/Kawther-MSD/SQL/blob/main/SQL/Capture%20d'%C3%A9cran%202025-04-24%20195329.png" heigth="800" width="750">

### 3. Customers in same region as supplier 1

```
-- Select customers from same country, city, and last 3 postal code digits as supplier 1
SELECT *
FROM CUSTOMERS
WHERE (COUNTRY, CITY, RIGHT(POSTAL_CODE, 3)) = (
  SELECT COUNTRY, CITY, RIGHT(POSTAL_CODE, 3)
  FROM SUPPLIERS
  WHERE SUPPLIER_int = 1
);
```
<img src="https://github.com/Kawther-MSD/SQL/blob/main/SQL/Capture%20d'%C3%A9cran%202025-04-24%20195713.png" heigth="800" width="750">

### 4. Discount rate and note per order

```
-- Calculate discount based on total order value and annotate based on order number range
SELECT O.ORDER_int,
       CASE
         WHEN SUM(D.UNIT_PRICE * D.QUANTITY) BETWEEN 0 AND 2000 THEN '0%'
         WHEN SUM(D.UNIT_PRICE * D.QUANTITY) BETWEEN 2001 AND 10000 THEN '5%'
         WHEN SUM(D.UNIT_PRICE * D.QUANTITY) BETWEEN 10001 AND 40000 THEN '10%'
         WHEN SUM(D.UNIT_PRICE * D.QUANTITY) BETWEEN 40001 AND 80000 THEN '15%'
         ELSE '20%'
       END AS NEW_DISCOUNT_RATE,
       CASE
         WHEN O.ORDER_int BETWEEN 10000 AND 10999 THEN 'apply old discount rate'
         ELSE 'apply new discount rate'
       END AS DISCOUNT_NOTE
FROM ORDERS O
JOIN ORDER_DETAILS D ON O.ORDER_int = D.ORDER_int
WHERE O.ORDER_int BETWEEN 10998 AND 11003
GROUP BY O.ORDER_int;
```
<img src="https://github.com/Kawther-MSD/SQL/blob/main/SQL/Capture%20d'%C3%A9cran%202025-04-24%20211200.png" heigth="800" width="750">

### 5. Suppliers of beverage products

```
-- Select suppliers of products under the 'Beverages' category
SELECT DISTINCT S.SUPPLIER_int, S.COMPANY, S.ADDRESS, S.PHONE
FROM SUPPLIERS S
JOIN PRODUCTS P ON S.SUPPLIER_int = P.SUPPLIER_int
JOIN CATEGORIES C ON P.CATEGORY_CODE = C.CATEGORY_CODE
WHERE C.CATEGORY_NAME = 'Beverages';
```
<img src="https://github.com/Kawther-MSD/SQL/blob/main/SQL/Capture%20d'%C3%A9cran%202025-04-24%20213345.png" heigth="800" width="750">

### 6. Berlin customers who ordered at most 1 dessert

```
-- Identify customers in Berlin who ordered <=1 dessert product
SELECT C.CUSTOMER_CODE
FROM CUSTOMERS C
LEFT JOIN ORDERS O ON C.CUSTOMER_CODE = O.CUSTOMER_CODE
LEFT JOIN ORDER_DETAILS OD ON O.ORDER_int = OD.ORDER_int
LEFT JOIN PRODUCTS P ON OD.PRODUCT_REF = P.PRODUCT_REF
LEFT JOIN CATEGORIES CAT ON P.CATEGORY_CODE = CAT.CATEGORY_CODE
WHERE C.CITY = 'Berlin' AND (CAT.CATEGORY_NAME = 'Desserts' OR CAT.CATEGORY_NAME IS NULL)
GROUP BY C.CUSTOMER_CODE
HAVING COUNT(DISTINCT CASE WHEN CAT.CATEGORY_NAME = 'Desserts' THEN P.PRODUCT_REF ELSE NULL END) <= 1;
```
<img src="https://github.com/Kawther-MSD/SQL/blob/main/SQL/Capture%20d'%C3%A9cran%202025-04-24%20213417.png" heigth="800" width="750">

### 7. French customers' Monday orders in April 1998

```
-- Aggregate French customers' total orders placed on Mondays in April 1998
SELECT C.CUSTOMER_CODE, C.COMPANY, C.PHONE,
       ISNULL(SUM(D.UNIT_PRICE * D.QUANTITY * (1 - D.DISCOUNT)), 0) AS TOTAL_AMOUNT,
       C.COUNTRY
FROM CUSTOMERS C
LEFT JOIN ORDERS O ON C.CUSTOMER_CODE = O.CUSTOMER_CODE
LEFT JOIN ORDER_DETAILS D ON O.ORDER_int = D.ORDER_int
WHERE C.COUNTRY = 'France'
  AND MONTH(O.ORDER_DATE) = 4
  AND YEAR(O.ORDER_DATE) = 1998
  AND DATENAME(WEEKDAY, O.ORDER_DATE) = 'Monday'
GROUP BY C.CUSTOMER_CODE, C.COMPANY, C.PHONE, C.COUNTRY;
```
<img src="https://github.com/Kawther-MSD/SQL/blob/main/SQL/Capture%20d'%C3%A9cran%202025-04-24%20213439.png" heigth="800" width="750">

### 8. Customers who ordered all products

```
-- Find customers who have ordered every available product
SELECT C.CUSTOMER_CODE, C.COMPANY, C.PHONE
FROM CUSTOMERS C
JOIN ORDERS O ON C.CUSTOMER_CODE = O.CUSTOMER_CODE
JOIN ORDER_DETAILS D ON O.ORDER_int = D.ORDER_int
GROUP BY C.CUSTOMER_CODE, C.COMPANY, C.PHONE
HAVING COUNT(DISTINCT D.PRODUCT_REF) = (SELECT COUNT(*) FROM PRODUCTS);
```
<img src="https://github.com/Kawther-MSD/SQL/blob/main/SQL/Capture%20d'%C3%A9cran%202025-04-24%20213506.png" heigth="800" width="750">

### 9. Orders count per French customer

```
-- Count number of orders per French customer
SELECT C.CUSTOMER_CODE, COUNT(O.ORDER_int) AS NUM_ORDERS
FROM CUSTOMERS C
LEFT JOIN ORDERS O ON C.CUSTOMER_CODE = O.CUSTOMER_CODE
WHERE C.COUNTRY = 'France'
GROUP BY C.CUSTOMER_CODE;
```
<img src="https://github.com/Kawther-MSD/SQL/blob/main/SQL/Capture%20d'%C3%A9cran%202025-04-24%20213526.png" heigth="800" width="750">


### 10. Orders by year and difference

```
-- Count orders in 1996 and 1997 and show their difference
SELECT
  (SELECT COUNT(*) FROM ORDERS WHERE YEAR(ORDER_DATE) = 1996) AS ORDERS_1996,
  (SELECT COUNT(*) FROM ORDERS WHERE YEAR(ORDER_DATE) = 1997) AS ORDERS_1997,
  ((SELECT COUNT(*) FROM ORDERS WHERE YEAR(ORDER_DATE) = 1997) -
   (SELECT COUNT(*) FROM ORDERS WHERE YEAR(ORDER_DATE) = 1996)) AS DIFFERENCE;
```
<img src="https://github.com/Kawther-MSD/SQL/blob/main/SQL/Capture%20d'%C3%A9cran%202025-04-24%20213558.png" heigth="800" width="750">

## Conclusion
This project highlights practical applications of advanced SQL queries within a commercial database context. Each query was designed to simulate real-world data needs and analytical tasks, helping to reinforce understanding of complex SQL techniques like joins, subqueries, aggregation, filtering, and conditional logic.

These exercises not only serve to sharpen SQL skills but also to develop an analytical mindset essential for data-driven problem solving in real business scenarios.

Author
[Kaouther Messaoudi](https://www.linkedin.com/in/messaoudi-kaouther/) -LinkedIn


Advanced SQL Queries Project — 2025

For more projects, visit my GitHub [profile](https://github.com/Kawther-MSD/SQL/commits?author=Kawther-MSD).
