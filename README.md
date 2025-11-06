-- ##########################################################################
-- T-SQL CODE FOR E-COMMERCE DATA ANALYSIS (MS SQL SERVER)
-- Based on the 'Task 4: SQL for Data Analysis' Mini Guide.
-- ##########################################################################

-- =========================================================================
-- PART 1: DATABASE SETUP AND SCHEMA CREATION
-- =========================================================================

-- Create the Database (Ensure you have permissions)
IF DB_ID('Ecommerce_SQL_Database') IS NOT NULL
    DROP DATABASE Ecommerce_SQL_Database;
GO

CREATE DATABASE Ecommerce_SQL_Database;
GO

USE Ecommerce_SQL_Database;
GO

-- 1. Create the Customers Table
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY IDENTITY(1,1),
    FirstName VARCHAR(100) NOT NULL,
    LastName VARCHAR(100) NOT NULL,
    Email VARCHAR(100) UNIQUE NOT NULL,
    DateJoined DATE DEFAULT GETDATE()
);

-- 2. Create the Products Table
CREATE TABLE Products (
    ProductID INT PRIMARY KEY IDENTITY(101,1),
    ProductName VARCHAR(255) NOT NULL,
    Category VARCHAR(100),
    Price DECIMAL(10, 2) NOT NULL,
    StockQuantity INT NOT NULL
);

-- 3. Create the Orders Table (Transactional table)
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY IDENTITY(1000,1),
    CustomerID INT NOT NULL,
    OrderDate DATE DEFAULT GETDATE(),
    Status VARCHAR(50) DEFAULT 'Pending', -- e.g., 'Pending', 'Shipped', 'Delivered'
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

-- 4. Create the OrderItems Table (Detail table for many-to-many relationship)
CREATE TABLE OrderItems (
    OrderItemID INT PRIMARY KEY IDENTITY(1,1),
    OrderID INT NOT NULL,
    ProductID INT NOT NULL,
    Quantity INT NOT NULL,
    UnitPrice DECIMAL(10, 2) NOT NULL, -- Price at the moment of the order
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);
GO

-- =========================================================================
-- PART 2: DATA INSERTION (Sample Data)
-- =========================================================================

-- Insert Products
INSERT INTO Products (ProductName, Category, Price, StockQuantity) VALUES
('Laptop Pro X', 'Electronics', 1200.00, 50),
('Mechanical Keyboard', 'Accessories', 85.50, 150),
('Ergonomic Mouse', 'Accessories', 35.00, 200),
('4K Monitor 27"', 'Electronics', 450.00, 30),
('T-Shirt - Code Life', 'Apparel', 25.00, 500),
('SQL Cheatsheet Mug', 'Home Goods', 15.00, 80),
('Noise-Cancelling Headphones', 'Electronics', 250.00, 75);

-- Insert Customers
INSERT INTO Customers (FirstName, LastName, Email, DateJoined) VALUES
('Alice', 'Smith', 'alice@example.com', '2023-01-10'),
('Bob', 'Johnson', 'bob@example.com', '2023-02-15'),
('Charlie', 'Brown', 'charlie@example.com', '2023-03-01'),
('Diana', 'Prince', 'diana@example.com', '2023-05-20');

-- Insert Orders
-- Alice: Orders 1000, 1002
INSERT INTO Orders (CustomerID, OrderDate, Status) VALUES
(1, '2023-10-01', 'Shipped'),    -- OrderID 1000
(2, '2023-10-05', 'Delivered'),  -- OrderID 1001
(1, '2023-10-15', 'Processing'), -- OrderID 1002
(4, '2023-10-18', 'Delivered'),  -- OrderID 1003
(3, '2023-10-20', 'Pending');    -- OrderID 1004

-- Insert OrderItems (Mapping products to orders)
-- Order 1000 (Alice): Laptop (1) + Keyboard (1)
INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice) VALUES
(1000, 101, 1, 1200.00),
(1000, 102, 1, 85.50);

-- Order 1001 (Bob): Monitor (1) + T-Shirt (2)
INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice) VALUES
(1001, 104, 1, 450.00),
(1001, 105, 2, 25.00);

-- Order 1002 (Alice): Mouse (1) + Mug (3)
INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice) VALUES
(1002, 103, 1, 35.00),
(1002, 106, 3, 15.00);

-- Order 1003 (Diana): Headphones (1)
INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice) VALUES
(1003, 107, 1, 250.00);

-- Order 1004 (Charlie): T-Shirt (1)
INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice) VALUES
(1004, 105, 1, 25.00);
GO

-- =========================================================================
-- PART 3: DATA ANALYSIS QUERIES (Using all required commands)
-- =========================================================================

PRINT '--- 1. Simple SELECT, WHERE, ORDER BY Query ---';
-- Task: Find all products with a price greater than $100, ordered by price.
SELECT
    ProductName,
    Category,
    Price
FROM
    Products
WHERE -- Filter rows based on a condition
    Price > 100.00
ORDER BY -- Sort the result set
    Price DESC;
GO
----------------------------------------------------------------------------

PRINT '--- 2. Aggregate Functions (SUM) and GROUP BY ---';
-- Task: Calculate the total inventory value for each product category.
SELECT
    Category,
    SUM(Price * StockQuantity) AS TotalInventoryValue -- Use aggregate function SUM()
FROM
    Products
GROUP BY -- Summarize data by Category
    Category
ORDER BY
    TotalInventoryValue DESC;
GO
----------------------------------------------------------------------------

PRINT '--- 3. JOINS (INNER JOIN) and Aggregate Functions (AVG) ---';
-- Task: Calculate the total revenue and average order value (AOV) for each customer.
SELECT
    c.FirstName,
    c.LastName,
    COUNT(DISTINCT o.OrderID) AS TotalOrders,
    SUM(oi.Quantity * oi.UnitPrice) AS TotalRevenue,
    AVG(oi.Quantity * oi.UnitPrice) AS AverageItemValue -- AVG() on item level
FROM
    Customers c
INNER JOIN -- Standard join to get matching records from both tables
    Orders o ON c.CustomerID = o.CustomerID
INNER JOIN
    OrderItems oi ON o.OrderID = oi.OrderID
GROUP BY
    c.CustomerID, c.FirstName, c.LastName
ORDER BY
    TotalRevenue DESC;
GO
----------------------------------------------------------------------------

PRINT '--- 4. Subquery (IN Clause) ---';
-- Task: List all products that have been part of an order with 'Delivered' status.
SELECT
    ProductID,
    ProductName
FROM
    Products
WHERE
    ProductID IN ( -- Filter using a subquery
        SELECT DISTINCT
            ProductID
        FROM
            OrderItems
        WHERE
            OrderID IN (
                SELECT
                    OrderID
                FROM
                    Orders
                WHERE
                    Status = 'Delivered'
            )
    );
GO
----------------------------------------------------------------------------

PRINT '--- 5. JOIN Types (LEFT JOIN) to find non-matching records ---';
-- Task: Find customers who have NOT placed any orders (Inactive Users).
SELECT
    c.FirstName,
    c.LastName
FROM
    Customers c
LEFT JOIN -- Keep all records from the LEFT table (Customers)
    Orders o ON c.CustomerID = o.CustomerID
WHERE
    o.OrderID IS NULL; -- Filter for unmatched records in the RIGHT table (Orders)
GO
----------------------------------------------------------------------------

PRINT '--- 6. Using HAVING (Filtering on aggregated data) ---';
-- Task: Identify product categories that have generated more than $500 in total sales.
SELECT
    p.Category,
    SUM(oi.Quantity * oi.UnitPrice) AS TotalSales
FROM
    OrderItems oi
INNER JOIN
    Products p ON oi.ProductID = p.ProductID
GROUP BY
    p.Category
HAVING -- Filter the results of the GROUP BY
    SUM(oi.Quantity * oi.UnitPrice) > 500.00
ORDER BY
    TotalSales DESC;
GO
----------------------------------------------------------------------------

PRINT '--- 7. Interview Question: Calculate Average Revenue Per User (ARPU) ---';
-- ARPU = Total Revenue / Total Number of Customers
SELECT
    CAST(SUM(TotalRevenue) AS DECIMAL(10, 2)) AS TotalRevenue,
    COUNT(DISTINCT CustomerID) AS TotalCustomers,
    CAST(SUM(TotalRevenue) / COUNT(DISTINCT CustomerID) AS DECIMAL(10, 2)) AS ARPU -- Final Calculation
FROM
(
    -- Subquery: Calculates total revenue for *each* customer who has placed an order
    SELECT
        o.CustomerID,
        SUM(oi.Quantity * oi.UnitPrice) AS TotalRevenue
    FROM
        Orders o
    INNER JOIN
        OrderItems oi ON o.OrderID = oi.OrderID
    GROUP BY
        o.CustomerID
) AS CustomerSales;
GO
----------------------------------------------------------------------------

-- =========================================================================
-- PART 4: OPTIMIZATION AND VIEW CREATION
-- =========================================================================

PRINT '--- 8. Create a VIEW for Analysis ---';
-- Task: Create a VIEW to quickly access key order details (JOIN, Aggregation)
IF OBJECT_ID('OrderDetailsView') IS NOT NULL
    DROP VIEW OrderDetailsView;
GO

CREATE VIEW OrderDetailsView AS
SELECT
    o.OrderID,
    o.OrderDate,
    o.Status,
    c.FirstName + ' ' + c.LastName AS CustomerName,
    SUM(oi.Quantity * oi.UnitPrice) AS OrderTotal,
    COUNT(oi.OrderItemID) AS NumberOfItems
FROM
    Orders o
INNER JOIN
    Customers c ON o.CustomerID = c.CustomerID
INNER JOIN
    OrderItems oi ON o.OrderID = oi.OrderID
GROUP BY
    o.OrderID, o.OrderDate, o.Status, c.FirstName, c.LastName;
GO

-- Query the VIEW
SELECT * FROM OrderDetailsView WHERE OrderTotal > 50;
GO
----------------------------------------------------------------------------

PRINT '--- 9. Optimize Queries with a NONCLUSTERED INDEX ---';
-- Task: Create an index on the CustomerID column of the Orders table.
-- This is a Foreign Key frequently used in JOINs, making it a perfect candidate for an index.
CREATE NONCLUSTERED INDEX IX_Orders_CustomerID
ON Orders (CustomerID);
GO

-- Create an index on a frequently searched text column (ProductName).
CREATE NONCLUSTERED INDEX IX_Products_ProductName
ON Products (ProductName);
GO

PRINT 'Full SQL script execution complete. All tables, data, and analysis queries are ready.';
