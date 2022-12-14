## Here I am showing a few basic coding examples in SQL. These are not complex. They are to show understanding of basic knowledge of using SQL in order to answer simple questions with the data. 


**Show all order details**
```sql
SELECT * FROM OrderDetails
```
![image](https://user-images.githubusercontent.com/119534892/205470919-026cbc64-40c0-4973-b9db-b751be8198d7.png)

**Show all orders placed by employee ID 2**
```sql
SELECT OrderID, OrderDate
FROM Orders
WHERE employeeID = 2;
```
![image](https://user-images.githubusercontent.com/119534892/205471085-3d42cb45-f26b-44e2-959f-00e94ed57736.png)

**Show all employees with the title Sales Representative and live in the UK**
```sql
SELECT concat(FirstName,' ',LastName) AS EmployeeName, HireDate
FROM Employees
WHERE Title = 'Sales Representative' and Country = 'UK';
```
![image](https://user-images.githubusercontent.com/119534892/205471204-b44a6cd4-a5e5-435e-99e3-06d4814aa37f.png)

**Show all products that contatin the word 'tofu'**
```sql
SELECT ProductID, ProductName
FROM Products
WHERE ProductName like '%tofu%'
```
![image](https://user-images.githubusercontent.com/119534892/205471391-358cdbbf-43a7-4921-8aec-9ca77dee7c18.png)

**Show all orders in the countries Brazil, France, and USA**
```sql
SELECT OrderID, CustomerId, ShipCountry
FROM Orders
WHERE ShipCountry in ('Brazil','France','USA');
```
![image](https://user-images.githubusercontent.com/119534892/205471753-d5aa74bb-c76d-40cf-abb1-6e96d3a086c5.png)

**Show all employees in order of oldest to youngest**
```sql
SELECT FirstName, LastName, Title, BirthDate
FROM Employees
ORDER BY BirthDate;
```
![image](https://user-images.githubusercontent.com/119534892/205471892-c22b3baf-3efb-4e4f-b312-7c3517134a2d.png)

**Show the total price per product by order ID**
```sql
SELECT OrderID, ProductID, UnitPrice, Quantity, UnitPrice * Quantity AS TotalPrice
FROM OrderDetails
Order By OrderID;
```
![image](https://user-images.githubusercontent.com/119534892/205472051-27686b00-9e09-4d27-a9bf-ff7b5ad6346f.png)

**Count how many products are on the Product table**
```sql
SELECT count(ProductID) AS TotalProducts
FROM Products;
```
![image](https://user-images.githubusercontent.com/119534892/205472182-fabbd723-c459-490d-b94a-39ca37b51705.png)

**Join the Orders table and the Order Details table to show total price per order**
```sql
SELECT o.CustomerID, o.OrderID, od.UnitPrice * od.Quantity AS TotalPrice
FROM Orders o
JOIN OrderDetails od ON o.OrderID = od.OrderID
GROUP BY o.CustomerID, o.OrderID;
```
![image](https://user-images.githubusercontent.com/119534892/205472410-73e0de7a-21d9-4e37-b3f5-4caaa2b983cc.png)

**Show the total number of products per category in descending order**
```sql
SELECT c.CategoryName, count(p.ProductID) AS TotalProducts
FROM Categories c
JOIN Products p ON c.CategoryID = p.CategoryID
GROUP BY c.CategoryName
ORDER BY TotalProducts DESC;
```
![image](https://user-images.githubusercontent.com/119534892/205472485-13879386-afaf-48b5-bedc-a36ebb3dd095.png)

