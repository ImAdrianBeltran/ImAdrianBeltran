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
