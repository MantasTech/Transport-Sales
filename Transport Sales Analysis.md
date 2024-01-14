# Transport Sales Data Analysis

## Overview
This project is focused on data analysis using sql to identify: 
* most selling products
* sales by year
* sales by dealsize
* months that sells the most
* what products sells the best during those months
* RFM analysis and customer segmentation
* what products are often sold together

## Analysis
Below are the SQL queries that I wrote in MySQL for analysis

**1. Grouping sales by product line**
```sql
SELECT PRODUCTLINE, SUM(sales) AS Revenue
FROM sales_data_sample
GROUP BY PRODUCTLINE
ORDER BY Revenue DESC;
```

![](https://github.com/MantasTech/Transport-Sales/blob/main/source_data/image/productline.png)

**2. Grouping sales by year**
```sql
SELECT YEAR_ID, SUM(sales) AS Revenue
FROM sales_data_sample
GROUP BY YEAR_ID
ORDER BY Revenue DESC;
```

![](https://github.com/MantasTech/Transport-Sales/blob/main/source_data/image/year.png)

**3. Grouping sales by deal size**
```sql
SELECT DEALSIZE, SUM(sales) AS Revenue
FROM sales_data_sample
GROUP BY DEALSIZE
ORDER BY Revenue DESC;
```

![](https://github.com/MantasTech/Transport-Sales/blob/main/source_data/image/dealsize.png)

**4. Best month for sales in a specific year**
```sql
SELECT MONTH_ID, SUM(sales) AS Revenue, COUNT(ORDERNUMBER) AS Frequency
FROM sales_data_sample
WHERE YEAR_ID = 2004 -- Change to check diff years
GROUP BY MONTH_ID
ORDER BY Revenue DESC;
```

![](https://github.com/MantasTech/Transport-Sales/blob/main/source_data/image/month.png)

**5. November 2004 sold the most, lets see what product did best**
```sql
SELECT MONTH_ID, PRODUCTLINE, SUM(sales) AS Revenue, COUNT(ORDERNUMBER)
FROM sales_data_sample
WHERE YEAR_ID = 2004 AND MONTH_ID = 11
GROUP BY MONTH_ID, PRODUCTLINE
ORDER BY Revenue DESC;
```

![](https://github.com/MantasTech/Transport-Sales/blob/main/source_data/image/november_sales.png)

**6. RFM Analysis (rcency/frequency/monetary and customer segmentaion to identify best customers)**
```sql
DROP TEMPORARY TABLE IF EXISTS rfm;

CREATE TEMPORARY TABLE rfm AS 
SELECT 
    CUSTOMERNAME, 
    SUM(sales) AS MonetaryValue,
    AVG(sales) AS AvgMonetaryValue,
    COUNT(ORDERNUMBER) AS Frequency,
    MAX(STR_TO_DATE(ORDERDATE, '%m/%d/%Y %H:%i')) AS last_order_date,
    (SELECT MAX(STR_TO_DATE(ORDERDATE, '%m/%d/%Y %H:%i')) FROM sales_data_sample) AS max_order_date,
    DATEDIFF((SELECT MAX(STR_TO_DATE(ORDERDATE, '%m/%d/%Y %H:%i')) FROM sales_data_sample), MAX(STR_TO_DATE(ORDERDATE, '%m/%d/%Y %H:%i'))) AS Recency
FROM sales_data_sample
GROUP BY CUSTOMERNAME;

CREATE TEMPORARY TABLE rfm_calc AS 
SELECT 
    r.*,
    NTILE(4) OVER (ORDER BY Recency DESC) AS rfm_recency,
    NTILE(4) OVER (ORDER BY Frequency) AS rfm_frequency,
    NTILE(4) OVER (ORDER BY MonetaryValue) AS rfm_monetary
FROM rfm r;

SELECT 
    c.CUSTOMERNAME, 
    c.rfm_recency, 
    c.rfm_frequency, 
    c.rfm_monetary,
    CASE 
        WHEN CONCAT(c.rfm_recency, c.rfm_frequency, c.rfm_monetary) IN ('111', '112', '121', '122', '123', '132', '211', '212', '114', '141') THEN 'Lost Customer'
        WHEN CONCAT(c.rfm_recency, c.rfm_frequency, c.rfm_monetary) IN ('133', '134', '143', '244', '334', '343', '344', '144') THEN 'Cannot Lose!'
        WHEN CONCAT(c.rfm_recency, c.rfm_frequency, c.rfm_monetary) IN ('311', '411', '331') THEN 'New Customer'
        WHEN CONCAT(c.rfm_recency, c.rfm_frequency, c.rfm_monetary) IN ('222', '223', '233', '322', '234') THEN 'Potential Churn'
        WHEN CONCAT(c.rfm_recency, c.rfm_frequency, c.rfm_monetary) IN ('323', '333', '321', '422', '332', '432', '423') THEN 'Active Customer'
        WHEN CONCAT(c.rfm_recency, c.rfm_frequency, c.rfm_monetary) IN ('433', '434', '443', '444') THEN 'Top Customer'
    END AS rfm_segment
FROM rfm_calc c;
```

![](https://github.com/MantasTech/Transport-Sales/blob/main/source_data/image/rfm_analysis.png)

**7. Products that are often sold together**
```sql
SELECT s.OrderNumber, GROUP_CONCAT(p.PRODUCTCODE ORDER BY p.PRODUCTCODE SEPARATOR ',') AS ProductCodes
FROM sales_data_sample s
JOIN sales_data_sample p ON s.OrderNumber = p.OrderNumber
WHERE s.OrderNumber IN (
    SELECT ORDERNUMBER
    FROM (
        SELECT ORDERNUMBER, COUNT(*) as rn
        FROM sales_data_sample
        WHERE STATUS = 'Shipped'
        GROUP BY ORDERNUMBER
    ) m
    WHERE rn = 3
)
GROUP BY s.OrderNumber
ORDER BY ProductCodes DESC;
```

![](https://github.com/MantasTech/Transport-Sales/blob/main/source_data/image/sold_together.png)

## Visualization
#### Click [here](https://public.tableau.com/app/profile/mantastech/viz/TransportSalesDashboard_17050113469390/Dashboard3) to see interactive dashboard in tableau.
![](https://github.com/MantasTech/Transport-Sales/blob/main/source_data/image/sales_dashboard.png)




