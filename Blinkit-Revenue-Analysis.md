# 🛒 Blinkit Retail Data Analysis Project (SQL)

## 📁 Overview

This project explores and analyzes a retail dataset (`blinkit`) using MySQL. It includes data cleaning, transformation, and detailed performance metrics grouped by various categories like item type, outlet type, and location. The aim is to gain insights into product sales, customer ratings, and outlet performance.

---

## 📌 Project Goals

- ✅ Clean inconsistent categorical data  
- 📊 Analyze sales metrics (total & average)  
- ⭐ Explore customer ratings  
- 📦 Segment performance by outlet characteristics  
- 🔄 Normalize and prepare data for dashboarding or reporting

---

## 🧼 Step 1: Data Cleaning – Normalize Item Fat Content

```sql
-- Check current distinct values before cleaning
SELECT DISTINCT(`Item Fat Content`) FROM blinkit;

-- Disable safe updates before running the update
SET SQL_SAFE_UPDATES = 0;

-- Start the transaction
START TRANSACTION;

-- Normalize the values
UPDATE blinkit 
SET `Item Fat Content` = 
    CASE 
        WHEN `Item Fat Content` = 'LF' THEN 'Low Fat'
        WHEN `Item Fat Content` LIKE 'r%' THEN 'Regular'
        ELSE `Item Fat Content`
    END;

-- Commit the changes
COMMIT;

-- Check distinct values after cleaning
SELECT DISTINCT(`Item Fat Content`) FROM blinkit;
```

---

## 📊 Step 2: Key Metrics

### 💰 Total Sales in Millions
```sql
SELECT CONCAT(CAST(SUM(`total sales`) / 1000000 AS DECIMAL(10,2)), " Millions") AS Total_sales_in_millions 
FROM blinkit;
```

### 📈 Average Sales
```sql
SELECT FORMAT(AVG(totalsales), 2) AS avg_sales 
FROM blinkit;
```

### 🔢 Unique Item Types
```sql
SELECT COUNT(DISTINCT(itemtype)) AS total_item_type 
FROM blinkit;
```

### ⭐ Average Rating
```sql
SELECT ROUND(AVG(rating), 2) AS comment
FROM blinkit;
```

---

## 📊 Step 3: Breakdown by Categories

### 🍽️ Sales & Ratings by ItemFatContent
```sql
SELECT 
  ItemFatContent,
  ROUND(SUM(totalsales), 2) AS total_sales,
  ROUND(AVG(totalsales), 2) AS avg_sales,
  COUNT(itemtype) AS item_count,
  ROUND(AVG(rating), 2) AS avg_rating
FROM blinkit
GROUP BY ItemFatContent;
```

### 📦 Sales by ItemType
```sql
SELECT 
  ItemType,
  ROUND(SUM(totalsales), 2) AS total_sales,
  ROUND(AVG(totalsales), 2) AS avg_sales,
  COUNT(itemtype) AS item_count,
  ROUND(AVG(rating), 2) AS avg_rating
FROM blinkit
GROUP BY ItemType;
```

---

## 📍 Step 4: Outlet-Based Analysis

### 🧾 Summary by OutletLocationType and ItemFatContent
```sql
SELECT 
  OutletLocationType,
  ROUND(SUM(CASE WHEN ItemFatContent = 'Low Fat' THEN totalsales ELSE 0 END), 2) AS total_sales_low_fat,
  ROUND(SUM(CASE WHEN ItemFatContent = 'Regular' THEN totalsales ELSE 0 END), 2) AS total_sales_regular,
  ROUND(AVG(CASE WHEN ItemFatContent = 'Low Fat' THEN totalsales ELSE NULL END), 2) AS avg_sales_low_fat,
  ROUND(AVG(CASE WHEN ItemFatContent = 'Regular' THEN totalsales ELSE NULL END), 2) AS avg_sales_regular,
  COUNT(CASE WHEN ItemFatContent = 'Low Fat' THEN itemtype ELSE NULL END) AS item_count_low_fat,
  COUNT(CASE WHEN ItemFatContent = 'Regular' THEN itemtype ELSE NULL END) AS item_count_regular,
  ROUND(AVG(CASE WHEN ItemFatContent = 'Low Fat' THEN rating ELSE NULL END), 2) AS avg_rating_low_fat,
  ROUND(AVG(CASE WHEN ItemFatContent = 'Regular' THEN rating ELSE NULL END), 2) AS avg_rating_regular
FROM blinkit
GROUP BY OutletLocationType
ORDER BY OutletLocationType;
```

### 📊 Summary by Outlet Type
```sql
SELECT 
  OutletLocationType,
  ItemFatContent,
  ROUND(SUM(totalsales), 2) AS total_sales,
  ROUND(AVG(totalsales), 2) AS avg_sales,
  COUNT(itemtype) AS item_count,
  ROUND(AVG(rating), 2) AS avg_rating
FROM blinkit
GROUP BY OutletLocationType, ItemFatContent
ORDER BY OutletLocationType, ItemFatContent;
```

### 🏗️ Sales by Outlet Establishment Year
```sql
SELECT 
  OutletEstablishmentYear,
  ROUND(SUM(totalsales), 2) AS total_sales,
  ROUND(AVG(totalsales), 2) AS avg_sales,
  COUNT(itemtype) AS item_count,
  ROUND(AVG(rating), 2) AS avg_rating
FROM blinkit
GROUP BY 1
ORDER BY 1;
```

---

## 📈 Step 5: Sales Contribution %

### 🏪 By Outlet Size
```sql
SELECT 
  outletsize,
  SUM(totalsales),
  ROUND(SUM(totalsales)/(SELECT SUM(totalsales)/100 FROM blinkit), 2) AS total_sales 
FROM blinkit
GROUP BY 1;
```

### 📍 By Outlet Location Type
```sql
SELECT 
  OutletLocationType,
  ROUND(SUM(totalsales)/(SELECT SUM(totalsales)/100 FROM blinkit), 2) AS total_sales_percentage,
  ROUND(AVG(totalsales), 2) AS avg_sales,
  COUNT(itemtype) AS item_count,
  ROUND(AVG(rating), 2) AS avg_rating
FROM blinkit
GROUP BY 1;
```

### 🏬 By Outlet Type
```sql
SELECT 
  Outlettype,
  ROUND(SUM(totalsales)/(SELECT SUM(totalsales)/100 FROM blinkit), 2) AS total_sales_percentage,
  ROUND(AVG(totalsales), 2) AS avg_sales,
  COUNT(itemtype) AS item_count,
  ROUND(AVG(rating), 2) AS avg_rating
FROM blinkit
GROUP BY 1;
```

---

## 📌 Conclusion

This SQL project cleans and analyzes sales, ratings, and product distribution across different outlets. Key takeaways:

- Standardizing data is essential before aggregation.
- Sales vary significantly by outlet type and item fat content.
- Location and establishment year impact performance.
- This forms a solid foundation for dashboards or business recommendations.
