# Exfloring_adventureworks_database_googlebigquery
# Project_StudySQL
# Bicycle Manufacturer Dataset Analysis

## I. Introduction
This project contains a bicycle manufacturer dataset that I will explore using SQL on Google BigQuery. The dataset is based on the Adventure Works public dataset.

## II. Requirements
- Google Cloud Platform account  
- Project on Google Cloud Platform  
- Google BigQuery API enabled  
- SQL query editor or IDE  

## III. Dataset Access
The bicycle manufacturer dataset is stored in a public Google BigQuery dataset. To access the dataset, follow these steps:
1. Log in to your Google Cloud Platform account and create a new project.  
2. Navigate to the BigQuery console and select your newly created project.  
3. In the navigation panel, select **"Add Data"** and then **"Star a project by name"**.  
4. Enter the project name `adventureworks2019` and click **"Star"**.  
5. Click on the `adventureworks2019` table to open it.

## IV. Exploring the Dataset
## Easy_Level
**How many items with ListPrice more than $1000 have been sold?**
```sql
SELECT COUNT(SalesOrderID) AS TOTAL
FROM adventureworks2019.Sales.SalesOrderDetail AS S 
JOIN adventureworks2019.Production.Product AS P ON P.ProductID = S.ProductID
WHERE ListPrice > 1000
```
- **Query Results**
- ![image](https://github.com/user-attachments/assets/746941a6-dd59-4aa0-8042-53a19be9d53c)

**Select the job title of all male employees who are not married.**
```sql
SELECT JobTitle
FROM adventureworks2019.HumanResources.Employee
WHERE Gender = 'M'
AND MaritalStatus != 'M'
```
- **Query Results**
- ![image](https://github.com/user-attachments/assets/18de073d-28cf-4fc6-80e0-dbe9cf801614)


**Show the total order value for each Customer Name.
    List by value with the highest first.**
  
```sql
SELECT CONCAT(CI.FirstName,CI.LastName) AS Full_Name,
SUM(SH.SubTotal) AS Total_Value
FROM adventureworks2019.Sales.SalesOrderHeader AS SH 
JOIN adventureworks2019.Person.Customerinfo AS CI 
ON CI.CustomerID = SH.CustomerID
GROUP BY Full_Name
ORDER BY Total_Value DESC
```
- **Query Results**
- ![image](https://github.com/user-attachments/assets/f6026737-0b11-48b2-b38d-04b471a56e07)


**Select information all employees who joined new departments in 2010**
```sql
SELECT E.BusinessEntityID, EDH.DepartmentID, EDH.StartDate,E.Gender, E.JobTitle, E.BirthDate
FROM adventureworks2019.HumanResources.EmployeeDepartmentHistory AS EDH 
JOIN adventureworks2019.HumanResources.Employee AS E ON EDH.BusinessEntityID = E.BusinessEntityID
WHERE EDH.StartDate BETWEEN '2008-01-01' AND '2008-12-31'
```
- **Query Results**
- ![image](https://github.com/user-attachments/assets/400515e3-adb7-4f02-9210-ffcf2fa77055)

**Select total number of orders and revenue with AWC Logo Cap product.**
```sql
SELECT SUM(SD.OrderQty) AS TOTAL,P.Name, SUM(SD.OrderQty * P.ListPrice) AS REVENUE
FROM adventureworks2019.Production.Product AS P 
JOIN adventureworks2019.Sales.SalesOrderDetail AS SD ON SD.ProductID = P.ProductID
WHERE P.Name = "AWC Logo Cap"
GROUP BY P.Name
```
- **Query Results**
- ![image](https://github.com/user-attachments/assets/8545c0bb-4378-4460-baa8-384999383514)

## Medium_Level
**Calculate total revenue, total orders, total number of items order by Subcategory within the last 12 months**
```sql
SELECT 
      FORMAT_DATETIME("%b %Y", SD.ModifiedDate) as Month_Year
      ,PS.name AS Name_SubCategory
      ,SUM(SD.OrderQty) AS Qty_item
      ,SUM(SD.LineTotal) AS Revenue
      ,COUNT(DISTINCT SD.SalesOrderID)AS ToTal_Order

FROM `adventureworks2019.Sales.SalesOrderDetail` AS SD
LEFT JOIN `adventureworks2019.Production.Product` AS P
ON SD.ProductID = P.ProductID
LEFT JOIN `adventureworks2019.Production.ProductSubcategory` AS PS
ON CAST(P.ProductSubcategoryID AS INT) = PS.ProductSubcategoryID

WHERE DATE(SD.ModifiedDate) >= (SELECT DATE_SUB(MAX(DATE(ModifiedDate)) , 
INTERVAL 12 month) FROM `adventureworks2019.Sales.SalesOrderDetail`)
GROUP BY Month_Year, Name_SubCategory
ORDER BY Month_Year DESC, Name_SubCategory
```
- **Query Results**
- ![image](https://github.com/user-attachments/assets/e27acea1-47b2-47bc-bd85-268ba99c1884)

  
**Calculate the growth rate of the number of orders by Subcategory and get the top 10**

```sql
WITH Qty_now AS (
    SELECT 
        PS.name AS Name,
        FORMAT_DATE("%Y", SD.ModifiedDate) AS Year,
        SUM(SD.OrderQty) AS qty_item_now
    FROM `adventureworks2019.Sales.SalesOrderDetail` AS SD
    LEFT JOIN `adventureworks2019.Production.Product` AS P
        ON SD.ProductID = P.ProductID
    LEFT JOIN `adventureworks2019.Production.ProductSubcategory` AS PS
        ON CAST(P.ProductSubcategoryID AS INT) = PS.ProductSubcategoryID
    GROUP BY PS.name, Year
    ORDER BY Year
),
Qty_before AS (
    SELECT 
        Name,
        Year, 
        qty_item_now,
        LAG(qty_item_now) 
            OVER(PARTITION BY Name ORDER BY Year) AS qty_item_before
    FROM Qty_now
)
SELECT
   Name,
   Year,  
   qty_item_now,
   qty_item_before,
   ROUND((qty_item_now - qty_item_before) / qty_item_before, 2) AS Ratio
FROM Qty_before
GROUP BY Name, Year, qty_item_now, qty_item_before
ORDER BY Ratio DESC
lIMIT 10;
```
- **Query Results**
- ![image](https://github.com/user-attachments/assets/734f94a1-1124-44d4-bbd8-3c7b9e745b7a)

  **Identify the three most important cities.
   Show the break down revenue of product category in top citys**

  ```sql
  WITH c1 AS(
  SELECT A.City, 
  SUM(SD.OrderQty * SD.UnitPrice) as Total_Sales
  FROM adventureworks2019.Sales.SalesOrderDetail AS SD
  JOIN adventureworks2019.Sales.SalesOrderHeader AS SH 
  ON SD.SalesOrderID = SH.SalesOrderID
  JOIN adventureworks2019.Person.Address AS A 
  ON A.AddressID = SH.ShipToAddressID 
  GROUP BY A.City
  ORDER BY Total_Sales DESC 
  LIMIT 5)
  SELECT 
      A.City, PC.Name, SUM(UnitPrice * OrderQty) AS Total_Sales
      FROM  adventureworks2019.Sales.SalesOrderDetail AS SD 
      JOIN  adventureworks2019.Sales.SalesOrderHeader AS SH 
      ON SD.SalesOrderId = SH.SalesOrderId
      JOIN adventureworks2019.Person.Address AS A 
      ON A.AddressID = SH.ShipToAddressID 
      JOIN adventureworks2019.Sales.Product AS P 
      ON SD.ProductId = P.ProductId
      JOIN adventureworks2019.Production.ProductCategory AS PC 
      ON P.ProductCategoryId = PC.ProductCategoryId
  WHERE A.City IN (SELECT City FROM c1)
  GROUP BY City, PC.Name
  ORDER BY City, Total_Sales DESC
  ```
- **Query Results**
- ![image](https://github.com/user-attachments/assets/eded535d-ace3-40cb-a7b4-7cca89fbb3cc)

**Show how many orders are in the following ranges (in $):
   RANGE      Num Orders      Total Value
   0-99
   100-999
   1000-9999
   10000-**


   ```sql
WITH c1 AS (
  SELECT SalesOrderID, SUM(OrderQty * UnitPrice) AS Order_Total
  FROM adventureworks2019.Sales.SalesOrderDetail
  GROUP BY SalesOrderID
), 
c2 AS (
  SELECT 
   SalesOrderID,
   Order_Total, 
   CASE
    WHEN Order_Total BETWEEN 0 AND 99 THEN '0-99'
    WHEN Order_Total BETWEEN 100 AND 999 THEN '100-999'
    WHEN Order_Total BETWEEN 1000 AND 9999 THEN '1000-9999'
    WHEN Order_Total >= 10000 THEN '10000-'
    ELSE 'Error'
    END AS rg
  FROM c1
)
SELECT 
rg as RANGES, 
COUNT(rg) AS Num_Orders, 
SUM(Order_Total) AS Total_Value
FROM c2
GROUP BY rg
ORDER BY rg ASC 
```
- **Query Results**
- ![image](https://github.com/user-attachments/assets/f24ace57-e114-4487-bb12-1aa550e880b5)

**Select Rranking Top 3 TeritoryID and Country with biggest Order quantity of every year**

```sql
WITH 
  c1 AS (
    SELECT 
      FORMAT_TIMESTAMP("%Y", SD.ModifiedDate) AS Year
      ,SH.TerritoryID
      ,L.Country
      ,SUM(SD.OrderQty) AS Sum_Qty
    FROM adventureworks2019.Sales.SalesOrderDetail AS SD
    LEFT JOIN adventureworks2019.Sales.SalesOrderHeader AS SH
    ON SD.SalesOrderID = SH.SalesOrderID
    JOIN adventureworks2019.Sales.Location AS L 
    ON L.TerritoryID = SH.TerritoryID
    GROUP BY Year, SH.TerritoryID,L.Country)
  ,rnk AS (
    SELECT 
        Year
        ,TerritoryID    
        ,Sum_Qty
        ,Country
        ,DENSE_RANK() OVER(PARTITION BY Year 
                          ORDER BY Sum_Qty DESC) AS ranks
    FROM c1
    GROUP BY Year, TerritoryID, Sum_Qty,Country)

  SELECT *
  FROM rnk
  WHERE ranks <=3
  ORDER BY Year DESC
```
- **Query Results**
- ![image](https://github.com/user-attachments/assets/8fe1397a-b0e9-4c2f-888c-76712fa10662)

**Trend of Stock level & month diff % by all name product in 2012**
```sql
WITH 
  c1_date AS ( 
    SELECT 
       P.Name
      ,EXTRACT(Month FROM WO.ModifiedDate) AS month
      ,EXTRACT(Year FROM WO.ModifiedDate) AS year
      ,WO.StockedQty                             
    FROM `adventureworks2019.Production.WorkOrder` AS WO
    LEFT JOIN `adventureworks2019.Production.Product` AS P
    ON WO.ProductID = P.ProductID
    GROUP BY WO.ModifiedDate, P.Name, StockedQty )

  ,c2_stocks AS (  
    SELECT
        Name
        ,month
        ,year
        ,SUM(StockedQty) AS Sum_stockqty
    FROM c1_date 
    WHERE year = 2012
    GROUP BY Name,month,year)

  ,c3_prvious AS (
  SELECT *
        ,LEAD(Sum_stockqty) 
        OVER(PARTITION BY Name ORDER BY Name, month DESC) AS Sum_stockqty_prv
  FROM c2_stocks)
  
  SELECT *
        ,COALESCE(ROUND((Sum_stockqty /Sum_stockqty_prv -1)*100,1),0) AS Rate
  FROM c3_prvious
  ORDER BY Name, month DESC;
```

- **Query Results**
- ![image](https://github.com/user-attachments/assets/6d38a579-4596-4f95-9418-dc705c729862)








  




