## Here I am showing more complex SQL coding. This is to show that I know how to use things beyond basic SQL statements. 

**Show all orders made on the last day of each month**
```sql
SELECT EmployeeID, OrderID, date(OrderDate) AS OrderDate
FROM Orders
WHERE OrderDate = Last_day(OrderDate)
ORDER BY EmployeeID;
```
![image](https://user-images.githubusercontent.com/119534892/207497045-86f77d9e-4992-4cee-9f64-7fe34d6d89f7.png)


**Which employees had late orders and what was their late order count?**
```sql
SELECT e.EmployeeID, e.LastName, count(*) AS TotalLateOrders
FROM employees e
JOIN orders o ON o.employeeID = e.employeeID
WHERE o.RequiredDate <= o.ShippedDate
GROUP BY e.EmployeeID
ORDER BY TotalLateOrders DESC;
```
![image](https://user-images.githubusercontent.com/119534892/205400711-8dd5ca4a-d638-4c1b-a733-8ab81d8fd2f1.png)



**What was the customer order count for years 2015 and 2016?**
```sql
SELECT CustomerID, 
count(distinct case when orderdate >= '2015-01-01' and Orderdate < '2016-01-01' then orderID else null end) AS OrderCount2015,
count(distinct case when OrderDate >= '2016-01-01' and orderdate < '2017-01-01' then OrderID else null end) AS OrderCount2016
FROM Orders
GROUP BY CustomerID
ORDER BY CustomerID;
```
![image](https://user-images.githubusercontent.com/119534892/205209860-000396be-c31b-4e3f-99ef-9238e1ce1ffd.png)



**What was the order ID and date of the second order made in each country?**
```sql
WITH Main AS(
SELECT ShipCountry, CustomerID, OrderId, date(OrderDate) AS OrderDate, 
  row_number() OVER(Partition By ShipCountry ORDER BY OrderDate) AS RowNumber
FROM Orders)
SELECT ShipCountry, CustomerID, OrderID, OrderDate
FROM Main
WHERE RowNumber = 2
ORDER BY ShipCountry;
```
![image](https://user-images.githubusercontent.com/119534892/205209973-1e7a9db4-3139-4931-940f-907da1d0fec5.png)



**What is the percent of the total product cost per order?**
```sql
WITH MAIN AS(
SELECT OrderID, Quantity, UnitPrice, ProductID, sum(Quantity * UnitPrice) AS TotalProductCost
FROM OrderDetails
GROUP BY OrderID, ProductID),
MAIN2 AS(
SELECT OrderID, sum(Quantity * UnitPrice) AS TotalOrderCost
FROM OrderDetails
GROUP BY OrderID)
SELECT Main.OrderId, Main.ProductID, Main.Quantity, Main.UnitPrice, Main.TotalProductCost, MAIN2.TotalOrderCost, 
cast(Main.TotalProductCost / Main2.TotalOrderCost as decimal(4,2)) AS Percent, o.ShipCountry
FROM Main
JOIN Main2 ON Main.OrderID = Main2.OrderID
JOIN Orders o ON o.OrderID = Main.OrderID
ORDER BY Main.OrderID;
```
![image](https://user-images.githubusercontent.com/119534892/205210109-58100a39-8655-4a4a-a069-78021db6bd0b.png)


**What was the average freight amount for the top 3 countries over the past 12 months?**
```sql
SELECT ShipCountry, avg(Freight) AS AverageFreight
FROM Orders
WHERE OrderDate >=
date_add((SELECT max(OrderDate) FROM Orders), interval -1 year)
GROUP BY ShipCountry
ORDER BY AverageFreight DESC
LIMIT 3;
```
![image](https://user-images.githubusercontent.com/119534892/205210985-0ff24c79-5344-4a7a-9a7b-92fa517cea69.png)

**Show the customers who have made more than 1 order in a 5 day period in order to reduce freight cost.**
```sql
SELECT Order1.CustomerID, Order1.OrderID AS InitialOrderID, date(Order1.OrderDate) AS InitialOrderDate, Order2.OrderID AS NextOrderID, 
date(Order2.OrderDate) AS NextOrderDate, datediff(Order2.OrderDate, Order1.OrderDate) AS DaysBetweenOrders
FROM Orders Order1
JOIN Orders Order2 ON Order1.CustomerID = Order2.CustomerID
WHERE Order1.OrderID < Order2.OrderID
AND datediff(Order2.OrderDate, Order1.OrderDate) <= 5
ORDER BY Order1.CustomerID;
```
![image](https://user-images.githubusercontent.com/119534892/207471449-438b96f0-9f31-4889-8729-3ac5d282cdd3.png)

**Group customers based on how much they ordered in 2016**

```sql
With Orders2016 AS(
SELECT c.CustomerId, c.CompanyName, sum(od.Quantity * od.UnitPrice) AS TotalOrderAmount
FROM Customers c
JOIN Orders o ON o.CustomerID = c.CustomerId
JOIN OrderDetails od ON od.OrderID = o.OrderID
WHERE o.OrderDate >= '2016-01-01' AND o.OrderDate < '2017-01-01'
GROUP BY CustomerID, CompanyName)
SELECT *, case
When TotalOrderAmount >= 0 AND TotalOrderAmount < 1000 then 'Low'
when TotalOrderAmount >= 1000 AND TotalOrderAmount < 5000 then 'Medium'
when TotalOrderAmount >= 5000 AND TotalOrderAmount < 10000 then 'High'
when TotalOrderAmount >= 10000 then 'Very High'
end as CustomerGroup
FROM Orders2016
ORDER BY CustomerID;
```
![image](https://user-images.githubusercontent.com/119534892/207497944-911beb6c-6b83-48ae-8d21-11b9aff364f4.png)

**Show all customer groups and the percentage in each.**
```sql
With Orders2016 AS(
SELECT c.CustomerId, c.CompanyName, sum(od.Quantity * od.UnitPrice) AS TotalOrderAmount
FROM Customers c
JOIN Orders o ON o.CustomerID = c.CustomerId
JOIN OrderDetails od ON od.OrderID = o.OrderID
WHERE o.OrderDate >= '2016-01-01' AND o.OrderDate < '2017-01-01'
GROUP BY CustomerID, CompanyName),
CustomerGrouping AS(
SELECT *, case
When TotalOrderAmount >= 0 AND TotalOrderAmount < 1000 then 'Low'
when TotalOrderAmount >= 1000 AND TotalOrderAmount < 5000 then 'Medium'
when TotalOrderAmount >= 5000 AND TotalOrderAmount < 10000 then 'High'
when TotalOrderAmount >= 10000 then 'Very High'
end as CustomerGroup
FROM Orders2016)
SELECT CustomerGroup, count(*) AS TotalInGroup, 
cast(count(*)/(SELECT count(*) FROM CustomerGrouping) AS Decimal (4,2)) AS PercentageInGRoup
FROM CustomerGrouping
GROUP BY CustomerGroup
ORDER BY TotalInGroup DESC;
```
![image](https://user-images.githubusercontent.com/119534892/207498148-91adc28b-3dfa-4b4d-a227-e419b6d4a114.png)
