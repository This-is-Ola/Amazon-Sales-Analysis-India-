# Amazon-Sales-Analysis-India-
Amazon Sales Dashboard – Data cleaning, analysis, and visualization using SQL and Power BI
Dataset: For full dataset, please download from https://www.kaggle.com/datasets/thedevastator/unlock-profits-with-e-commerce-sales-data/data
SQL  Steps SQL Scripts
 create_table.sql – Create table schema
 load_data.sql – Load CSV into SQL
 data_profiling.sql – Explore and profile data
 cleaning_formatting.sql – Format and clean data
 remove_duplicates.sql – Remove duplicate rows
 
sql-CREATE TABLE AmazonSalesTemp (
    [index] NVARCHAR(MAX) NULL,
    order_id NVARCHAR(MAX) NULL,
    order_date NVARCHAR(MAX) NULL,
    status NVARCHAR(MAX) NULL,
    fulfilment NVARCHAR(MAX) NULL,
    sales_channel NVARCHAR(MAX) NULL,
    ship_service_level NVARCHAR(MAX) NULL,
    style NVARCHAR(MAX) NULL,
    sku NVARCHAR(MAX) NULL,
    category NVARCHAR(MAX) NULL,
    size NVARCHAR(MAX) NULL,
    asin NVARCHAR(MAX) NULL,
    courier_status NVARCHAR(MAX) NULL,
    qty NVARCHAR(MAX) NULL,
    currency NVARCHAR(MAX) NULL,
    amount NVARCHAR(MAX) NULL,
    ship_city NVARCHAR(MAX) NULL,
    ship_state NVARCHAR(MAX) NULL,
    ship_postal_code NVARCHAR(MAX) NULL,
    ship_country NVARCHAR(MAX) NULL,
    promotion_ids NVARCHAR(MAX) NULL,
    b2b NVARCHAR(MAX) NULL,
    fulfilled_by NVARCHAR(MAX) NULL,
    [unnamed] NVARCHAR(MAX) NULL
);


BULK INSERT AmazonSalesTemp
FROM 'C:\Users\admin\Desktop\kaggle\Amazon Sale Report.csv'
WITH
(
    FIRSTROW = 2,              
    FIELDTERMINATOR = ',',      
    ROWTERMINATOR = '\n',       
    CODEPAGE = '65001',        
    TABLOCK
);
select * from AmazonSalesTemp

SELECT TOP (1000) [index]
      ,[order_id]
      ,[order_date]
      ,[status]
      ,[fulfilment]
      ,[sales_channel]
      ,[ship_service_level]
      ,[style]
      ,[sku]
      ,[category]
      ,[size]
      ,[asin]
      ,[courier_status]
      ,[qty]
      ,[currency]
      ,[amount]
      ,[ship_city]
      ,[ship_state]
      ,[ship_postal_code]
      ,[ship_country]
      ,[promotion_ids]
      ,[b2b]
      ,[fulfilled_by]
      ,[unnamed]
  FROM [kaggle].[dbo].[AmazonSalesTemp]

select  * from dbo.AmazonSalestemp
---dtata profilling 
SELECT 
  COUNT(*) AS total_rows,
  COUNT(order_id) AS non_null_orderid,
  SUM(CASE WHEN qty IS NULL OR qty = '' THEN 1 ELSE 0 END) AS qty_missing,
  SUM(CASE WHEN amount IS NULL OR amount = '' THEN 1 ELSE 0 END) AS amount_missing
FROM dbo.AmazonSalestemp

--further checks on missing amount
SELECT *
FROM dbo.AmazonSalesTemp
WHERE amount IS NULL;

--most nulls in amount comes from cancelled order, further checks 

SELECT status, COUNT(*) AS null_amount_rows
FROM dbo.AmazonSalesTemp
WHERE amount IS NULL
GROUP BY status
ORDER BY null_amount_rows DESC;

--checking the status of the null amount and corresponding them with courier status to determine wther to delete or save as backup 
SELECT *
FROM dbo.AmazonSalesTemp
WHERE amount IS NULL
  AND (status = 'Shipped' or status = 'shipping' or status = 'Shipped - Delivered to Buyer' or  status = 'Shipped - Returned to Seller');

 -- all bad datas to a backup table
SELECT *
INTO Amazon_BadRows
FROM dbo.AmazonSalesTemp
WHERE amount IS NULL;
select * from Amazon_BadRows

-- Delete rows with NULL amount that were not delivered or cancelled
DELETE FROM dbo.AmazonSalesTemp
WHERE amount IS NULL
  AND courier_status IN ('unshipped', 'Cancelled')
 
 DELETE FROM dbo.AmazonSalesTemp
WHERE amount IS NULL
  AND status = 'Cancelled'

--the remaining 11 datas with NULLs as courier status
SELECT *
FROM dbo.AmazonSalesTemp
WHERE amount IS NULL;


--checking order status consistency
SELECT [status], COUNT(*) as status_count
FROM dbo.amazonsalestemp
GROUP BY status 
ORDER BY [status] DESC;

--rename the null in courier status
UPDATE dbo.AmazonSalesTemp
SET courier_status = 'Unknown'
WHERE courier_status IS NULL;

--formatting
select ship_city, ship_state, ship_postal_code 
from dbo.AmazonSalesTemp

-- Converting Ship City to uppercase
UPDATE dbo.AmazonSalesTemp
SET ship_city = UPPER(ship_city)
WHERE ship_city IS NOT NULL;

-- Converting Ship State to uppercase
UPDATE dbo.AmazonSalesTemp
SET ship_state = UPPER(ship_state)
WHERE ship_state IS NOT NULL;

--some postal code are wrong 
UPDATE dbo.AmazonSalesTemp
SET ship_postal_code = NULL
WHERE ISNUMERIC(ship_postal_code) = 0;

--unwanted charracter abd space (formatting)

UPDATE dbo.AmazonSalesTemp
SET ship_city = LTRIM(RTRIM(REPLACE(ship_city, '"', ''))),
    ship_state = LTRIM(RTRIM(REPLACE(ship_state, '"', '')));

    UPDATE dbo.AmazonSalesTemp
SET Category = UPPER(Category);


    --looking for duplicate
  SELECT order_id, order_date, sku, amount, ship_city, ship_state, ship_postal_code, courier_status,
       COUNT(*) AS cnt
FROM dbo.AmazonSalesTemp
GROUP BY order_id, order_date, sku, amount, ship_city, ship_state, ship_postal_code, courier_status
HAVING COUNT(*) > 1;

--deleting one duplicate out of the 3 found
;WITH CTE AS (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY order_id, order_date, sku, amount, ship_city, ship_state, ship_postal_code, courier_status
               ORDER BY order_id
           ) AS rn
    FROM dbo.AmazonSalesTemp
)
DELETE FROM CTE

WHERE rn > 1;

select * from AmazonSalesTemp

Key Insights-

Power BI -https://app.powerbi.com/groups/me/reports/6d27e6d0-07f5-44a6-9aae-3c3ab4f236ba/d2120115e970eabd065a?experience=power-bie
<img width="1207" height="857" alt="Screenshot 2025-12-02 160735" src="https://github.com/user-attachments/assets/c24a1d57-eb5c-430f-8dff-d468c6cba46d" />

Key Insights 


- Total Revenue: £78.6M, Total Quantity: 116K units, Total Orders: 113K.
- Monthly Sales Trend:April recorded the highest sales at £29M.
- Top Sales City: Bengaluru, leading in total revenue.
- Product Category Performance: Highest sales for categories Sales are Set, Kurta, and Western Dress, respectively.
- Order Fulfillment Status: Shipped orders generated £50M, Delivered £18M, Cancelled £6M, indicating over half of the products were successfully shipped.
- Order Status Distribution: 64% shipped, 23% delivered, 9% cancelled (based on total orders).
- Size & Category Insights: Medium size was most ordered, followed by L, XL, XXL, and S. Among categories, SET had the highest orders consistently.
