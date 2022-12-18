## Here I am showing more intermediate and advanced examples of SQL ##

### Show what the products cost on 2014-04-15 ###
```sql
SELECT ProductID, StandardCost
FROM ProductCostHistory
WHERE '2014-04-15' between startdate and ifnull(enddate, now())
Order By ProductID;
```
![image](https://user-images.githubusercontent.com/119534892/208212585-76af672a-c03a-42f8-b5e4-73ae434dcf07.png)

### Show how many price changes happened per month ###
```sql
SELECT c.CalendarMonth , count(h.StartDate) AS TotalRows
FROM Calendar c
LEFT JOIN ProductListPriceHistory h ON c.CalendarDate = h.StartDate
WHERE c.CalendarDate >= (Select min(Startdate) FROM ProductListPriceHistory)
AND C.CalendarDate <= (select max(StartDate) FROM ProductListPriceHistory)
GROUP BY CalendarMonth
ORDER BY CalendarMonth;
```
![image](https://user-images.githubusercontent.com/119534892/208213033-fe525eef-d212-4cd0-902e-bacfae0f4e26.png)

### Show all products that accidentally have multiple current list price records ###
```sql
SELECT ProductID
FROM ProductListPriceHistory
WHERE EndDate IS NULL
GROUP BY ProductID
HAVING count(*) > 1;
```
![image](https://user-images.githubusercontent.com/119534892/208213246-f0537d21-9f33-477a-91a8-ae134246df4d.png)

### Show the first and last order date, including name and subcategory for all products. Include both the products that have and have not been ordered ##
```sql
SELECT p.ProductID, p.ProductName, ps.ProductSubCategoryName, date(min(h.OrderDate)) AS FirstOrder, date(max(h.OrderDate)) AS LastOrder
FROM Product p
LEFT JOIN SalesOrderdetail d ON p.ProductID = d.ProductID
LEFT JOIN SalesOrderHeader h ON h.SalesOrderID = d.SalesOrderID
LEFT JOIN ProductSubCategory ps ON p.ProductSubCategoryID = ps.ProductSubCategoryID
GROUP BY p.ProductName
ORDER BY p.ProductName;
```
![image](https://user-images.githubusercontent.com/119534892/208214197-a2e79cc0-bc30-40b0-bf1f-d0ea1a607a6c.png)

### Show all products where the current price on the Price List History table doesn't match the actual list price on the Product table ###
```sql
SELECT p.ProductID, p.ProductName, p.ListPrice AS Product_ListPrice, h.ListPrice AS PriceHist_LatestListPrice, p.ListPrice - h.ListPrice AS Difference
FROM Product p
JOIN ProductListPriceHistory h ON p.ProductID = h.ProductID
WHERE p.listprice <> h.listPrice
AND h.EndDate IS NULL
GROUP BY p.ProductID
ORDER BY p.ProductID;
```
![image](https://user-images.githubusercontent.com/119534892/208214754-e26336bd-195d-4a56-acf5-34a262460b60.png)

### Show all products that were ordered before the sell start date of after the sell end date ###
```sql
SELECT p.ProductID, date(h.OrderDate) AS OrderDate, d.OrderQty AS Qty, date(p.SellStartDate) AS SellStartDate, date(p.SellEndDate) AS SellEndDate, case
when h.OrderDate < p.SellStartDate then 'Sold Before Start Date'
when h.OrderDate > p.SellEndDate then 'Sold After End Date'
end AS ProblemType
FROM Product p
JOIN SalesOrderDetail d ON p.ProductID = d.ProductID
JOIN SalesOrderHeader h ON d.SalesOrderID = h.SalesOrderID
WHERE OrderDate NOT BETWEEN p.SellStartDate AND ifnull(p.SellEndDate, h.OrderDate)
GROUP BY p.ProductID , h.OrderDate
ORDER BY p.ProductID;
```
![image](https://user-images.githubusercontent.com/119534892/208216024-a51886d3-056e-4625-9ceb-ce10f70b3d74.png)

### How many cost changes do products generally have? ###
```sql
WITH MAIN AS(
SELECT ProductID, count(StartDate) AS TotalPriceChanges
FROM ProductCostHistory
GROUP BY ProductID)
SELECT TotalPriceChanges, count(*) AS TotalProducts
FROM Main
GROUP BY TotalPriceChanges
ORDER BY TotalPriceChanges;
```
![image](https://user-images.githubusercontent.com/119534892/208217062-e190edda-48b6-4b5c-b433-03ceb0f24a44.png)

### Show the size and base ProductNumber for products whose product ID is greater than 533 ###
```sql
WITH Main AS(
SELECT ProductId, ProductNumber, locate('-', ProductNumber) AS HyphenLocation
FROM Product
WHERE ProductID > 533)
SELECT *, case
when HyphenLocation = 0 then ProductNumber
else substring(ProductNumber, 1, HyphenLocation -1) end AS BaseProductNumber
, case
when HyphenLocation = 0 then Null
else substring(ProductNumber, HyphenLocation +1, 2) end AS Size
FROM Main
ORDER BY PRoductID;
```
![image](https://user-images.githubusercontent.com/119534892/208217148-5964dae0-173a-46a5-8745-4c73db74a3ad.png)

### What's the running total of order in the last year ###
```sql
WITH Main AS(
SELECT c.CalendarMonth, count(h.SalesOrderID) AS TotalOrders
FROM Calendar c
JOIN SalesOrderHeader h ON c.CalendarDate = date(h.OrderDate)
WHERE h.OrderDate >=
date_add((SELECT max(OrderDate) FROm SalesOrderHeader), interval -1 year)
GROUP BY c.CalendarMonth)
SELECT CalendarMonth, TotalOrders, SUM(TotalOrders) OVER(ORDER BY CalendarMonth) AS RunningTotal
FROM Main
GROUP BY CalendarMonth
ORDER BY CalendarMonth;
```
![image](https://user-images.githubusercontent.com/119534892/208217282-681207ea-04f7-401f-b092-693f6bce4ce3.png)

### What is the total late order count by territory? ###
```sql
WITH Main AS(
SELECT SalesOrderID, TerritoryID, date(DueDate) AS DueDate, date(ShipDate) AS ShipDate, case
when DueDate < ShipDate then 1 else 0 end AS OrderArrivedLAte
FROM SalesOrderHeader)
SELECT t.TerritoryID, t.TerritoryName, t.CountryCode, count(Main.SalesOrderID) AS TotalOrders, Ifnull(sum(Main.OrderArrivedLate),0) AS TotalLateOrders
FROM SalesTerritory t
JOIN MAin ON Main.TerritoryID = t.TerritoryID
GROUP BY TerritoryID
ORDER BY TerritoryID;
```
![image](https://user-images.githubusercontent.com/119534892/208217339-2f7f09c6-f4d6-49c7-aa42-04bc8f0ea081.png)

