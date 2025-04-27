# SQL-Queries
Using the AdventureWorks database this repository showcases all the SQL queries to try and query the database.

--1. Which shippers for we have?

SELECT *
FROM Shippers 

--2. Certain fields from categories

SELECT CategoryName, Description
FROM Categories

--3. Sales reps

SELECT *
FROM Employees


SELECT FirstName, LastName, HireDate
FROM Employees
WHERE Title='Sales Representative'

--4. Sales Reps in the US

SELECT FirstName, LastName, HireDate
FROM Employees
WHERE Title='Sales Representative' AND Country= 'USA'

--5. Orders placed by specific EmployeeID

SELECT OrderID,OrderDate
FROM Orders
WHERE EmployeeID=5

--6.Suppliers and ContactTitles

SELECT SupplierID,ContactName, ContactTitle
FROM Suppliers
WHERE ContactTitle != 'Marketing Manager'

--7.Products with queso in name

SELECT ProductID,ProductName
FROM Products
WHERE ProductName LIKE '%queso%'

--8.Orders shipping to France or Belgium

SELECT OrderID, CustomerID, ShipCountry
FROM Orders
WHERE ShipCountry IN ('France', 'Belgium')

--9--Orders shipping to any country in latin America

SELECT OrderID, CustomerID, ShipCountry
FROM Orders
WHERE ShipCountry IN ('Brazil', 'Mexico', 'Argentina','Venezuela')

--10-- Employees in order of age

SELECT FirstName,LastName, Title, BirthDate
FROM Employees
ORDER BY BirthDate 

--11 Showing only the date with datetime field


SELECT FirstName,LastName, Title, CONVERT(Date,BirthDate) AS DateOnlyBirthDate
FROM Employees
ORDER BY BirthDate 

--12 Employees Full Name

SELECT FirstName, LastName, (FirstName + ' '+ Lastname) AS FullName
FROM Employees

--13 Order Details amount per line item

SELECT OrderID, ProductID, UnitPrice, Quantity, (UnitPrice*Quantity) AS TotalPrice
FROM OrderDetails
ORDER BY OrderID,ProductID

--14 How many customers?

SELECT COUNT(DISTINCT(CustomerID)) as TotalCustomers
FROM Customers

--15 When was first order?

SELECT MIN(OrderDate) AS FirstOrder
FROM Orders

--16 Countries where there are customers?

SELECT Country
FROM Customers
GROUP BY Country

--17 Contact titles for customers

SELECT ContactTitle, COUNT(ContactTitle) AS TotalContactTitle
FROM Customers
GROUP BY ContactTitle
ORDER BY COUNT(ContactTitle) DESC

--18 Products with associated supplier names

SELECT p.ProductID, p.ProductName, s.CompanyName
FROM Products p
INNER JOIN Suppliers s
ON p.SupplierID = s.SupplierID
ORDER BY p.ProductID

--19 order and shipper that was used

SELECT o.OrderID, CONVERT(Date,o.OrderDate) AS OrderDate, s.CompanyName AS Shipper
FROM Orders o
INNER JOIN Shippers s
ON o.ShipVia=s.ShipperID
WHERE OrderID<10270

--20 Categories and total products in each category

SELECT c.CategoryName, COUNT(c.CategoryName) AS TotalProducts
FROM Categories c
INNER JOIN Products p
ON c.CategoryID=p.CategoryID
GROUP BY c.CategoryName
ORDER BY TotalProducts DESC

-- 21 total customers per country/city

SELECT Country, City, (COUNT(City)) AS TotalCustomers
FROM Customers
GROUP BY Country, City
ORDER BY TotalCustomers DESC

--22 Products that need re-ordering

SELECT ProductID, ProductName, UnitsInStock, ReorderLevel
FROM Products
WHERE UnitsInStock <= ReorderLevel
ORDER BY ProductID

--23 continued

SELECT ProductID, ProductName, UnitsInStock,UnitsOnOrder,ReorderLevel, Discontinued
FROM Products
WHERE UnitsInStock + UnitsOnOrder <= ReorderLevel AND Discontinued= 0
ORDER BY ProductID

--24 Customer List by Region

SELECT CustomerID
,CompanyName
,Region
FROM Customers
ORDER BY 
CASE WHEN Region IS NULL THEN 1 ELSE 0 END,REGION , CustomerID 

--25 High Freight charges

SELECT TOP 3 ShipCountry, AVG(Freight) AS AverageFreight
FROM Orders
GROUP BY ShipCountry
ORDER BY AverageFreight DESC



--26 High Freight charges -2015

SELECT TOP 3 ShipCountry, AVG(Freight) AS AverageFreight
FROM Orders
WHERE YEAR(OrderDate)='2015'
GROUP BY ShipCountry
ORDER BY AverageFreight DESC

--27 Very interesting error below which returns Sweden as third highest as its a datetime field,
--noticing the data for the 31st dec 2015 there is an order on the day
-- the 20151231 is equivalent to 2015-12-31 00:00:00 

SELECT TOP 3 ShipCountry, AVG(Freight) AS AverageFreight
FROM Orders
WHERE OrderDate between '20150101'and '20151231'
GROUP BY ShipCountry
ORDER BY AverageFreight DESC

--28 High Freight charges last year

SELECT TOP 3 ShipCountry, AVG(Freight) AS AverageFreight
FROM Orders
WHERE OrderDate >= DATEADD(yy,-1,(SELECT MAX(OrderDate) from Orders)) 
GROUP BY ShipCountry
ORDER BY AverageFreight DESC

--29 Employee/Order Detail Report

SELECT e.EmployeeID, e.LastName, o.OrderID,p.ProductName,od.Quantity
FROM Employees e
INNER JOIN Orders o
ON e.EmployeeID=o.EmployeeID
INNER JOIN OrderDetails od
ON od.OrderID=o.OrderID
INNER JOIN Products p
ON p.ProductID=od.ProductID
ORDER BY o.OrderID, od.ProductID

--30 Customers with no orders

SELECT c.CustomerID AS Customers_CustomerID, OrderID AS Orders_CustomerID
FROM Customers c
LEFT JOIN Orders o
ON c.CustomerID=o.CustomerID
WHERE O.OrderID is null

--31 Customers with no orders for EmployeeID 4

SELECT c.CustomerID , o.CustomerID
FROM Customers c
LEFT JOIN Orders o
ON c.CustomerID=o.CustomerID and o.EmployeeID=4
WHERE o.CustomerID is NULL
ORDER BY c.CustomerID

--32 High Value Customers

SELECT c.CustomerID,c.CompanyName,o.OrderID,SUM(od.Quantity*od.UnitPrice) AS TotalOrderAmount
FROM Orders o
INNER JOIN Customers c
ON c.CustomerID=o.CustomerID
INNER JOIN OrderDetails od
ON od.OrderID=o.OrderID
WHERE YEAR(OrderDate)='2016'
GROUP BY c.CustomerID, o.OrderID,c.CompanyName
HAVING SUM(od.Quantity*od.UnitPrice) > 10000
ORDER BY TotalOrderAmount DESC

--33 High Value customers- total orders

SELECT c.CustomerID,c.CompanyName,SUM(od.Quantity*od.UnitPrice) AS TotalOrderAmount
FROM Orders o
INNER JOIN Customers c
ON c.CustomerID=o.CustomerID
INNER JOIN OrderDetails od
ON od.OrderID=o.OrderID
WHERE YEAR(OrderDate)='2016'
GROUP BY c.CustomerID, c.CompanyName
HAVING SUM(od.Quantity*od.UnitPrice) > 15000
ORDER BY TotalOrderAmount DESC

--34 --33 High Value customers- with discount

SELECT c.CustomerID,c.CompanyName,SUM(od.Quantity*od.UnitPrice) AS TotalOrderAmount,(SUM(od.Quantity*od.UnitPrice)-SUM(od.Quantity*od.UnitPrice*od.Discount)) AS TotalOrderAmountwithDiscount
FROM Orders o
INNER JOIN Customers c
ON c.CustomerID=o.CustomerID
INNER JOIN OrderDetails od
ON od.OrderID=o.OrderID
WHERE YEAR(OrderDate)='2016'
GROUP BY c.CustomerID, c.CompanyName
HAVING (SUM(od.Quantity*od.UnitPrice)-SUM(od.Quantity*od.UnitPrice*od.Discount)) >= 15000
ORDER BY TotalOrderAmountwithDiscount DESC

--35 Month End Orders

SELECT e.EmployeeID, o.OrderID, o.OrderDate
FROM Employees e
INNER JOIN Orders o
ON e.EmployeeID=o.EmployeeID
WHERE OrderDate=EOMONTH(OrderDate)
GROUP BY e.EmployeeID, o.OrderID,o.OrderDate
ORDER BY e.EmployeeID

--36 Orders with many line items

SELECT TOP 10 OrderID, COUNT(Quantity) AS TotalOrderDetails
FROM OrderDetails
GROUP BY OrderID
ORDER BY TotalOrderDetails DESC

--37 Random Assortment
--This was new for me and I tried various things but didn't know how to do it.

SELECT TOP 2 percent OrderID
FROM Orders
Order by NewID()
