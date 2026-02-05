# 📊 RFM Customer Segmentation Using MySQL

![RFM Analysis](https://github.com/Mdkamrulislam54/RFM-SEGMENTATION/blob/2a702bc8d28bf0c06e0957d468172498b9055928/rfm-analysis.png)

## 🎯 Project Overview

This project demonstrates **RFM (Recency, Frequency, Monetary) Analysis** using MySQL to segment customers based on their purchasing behavior. RFM segmentation is a powerful marketing analysis tool that helps businesses identify their most valuable customers and tailor marketing strategies accordingly.

## 📑 Table of Contents

- [What is RFM Analysis?](#what-is-rfm-analysis)
- [Dataset Overview](#dataset-overview)
- [Key Insights](#key-insights)
- [Customer Segments](#customer-segments)
- [SQL Implementation](#sql-implementation)
- [Repository Structure](#repository-structure)
- [How to Use](#how-to-use)
- [Results](#results)
- [Technologies Used](#technologies-used)

## 🔍 What is RFM Analysis?

RFM Analysis is a customer segmentation technique that uses three key metrics:

- **Recency (R)**: How recently did the customer make a purchase?
- **Frequency (F)**: How often do they purchase?
- **Monetary (M)**: How much do they spend?

Each customer is scored on a scale of 1-5 for each metric, and these scores are combined to create customer segments.

![Customer Segmentation](https://github.com/Mdkamrulislam54/RFM-SEGMENTATION/blob/2a702bc8d28bf0c06e0957d468172498b9055928/Customer-segmentation.png)

## 📊 Dataset Overview

### Business Metrics

| Metric | Value |
|--------|-------|
| **Total Customers** | 92 |
| **Total Orders** | 307 |
| **Total Quantity Ordered** | 99,067 units |
| **Total Sales** | $10,032,629 |

### Top 5 Performing Dates by Order Volume

| Order Date | Number of Orders |
|------------|------------------|
| 11/14/2003 | 38 |
| 11/24/2004 | 35 |
| 11/12/2003 | 34 |
| 11/17/2004 | 32 |
| 11/4/2004 | 29 |

## 👥 Customer Segments

Based on RFM scoring, customers are classified into 6 segments:

| Segment | RFM Score | Description | Count |
|---------|-----------|-------------|-------|
| 🏆 **Champions** | 15 | Best customers - bought recently, buy often, and spend the most | 7 |
| 💎 **Loyal Customers** | 12-14 | Spend good money with us often. Responsive to promotions | 18 |
| 🌟 **Potential Loyalists** | 9-11 | Recent customers, but spent a good amount and bought more than once | 25 |
| 🎯 **Promising** | 6-8 | Recent shoppers, but haven't spent much | 19 |
| 😴 **Hibernating** | 4-5 | Below average recency, frequency & monetary values | 15 |
| ⚠️ **Lost** | ≤3 | Lowest recency, frequency & monetary scores | 8 |

### Sample Champions (Top Customers)

- Danish Wholesale Imports
- Salzburg Collectables
- Souveniers And Things Co.
- The Sharp Gifts Warehouse
- La Rochelle Gifts
- Mini Gifts Distributors Ltd.
- Euro Shopping Channel

## 💻 SQL Implementation

### 1. Business Metrics Query

```sql
SELECT 
    COUNT(DISTINCT CUSTOMERNAME) AS NO_OF_CUSTOMERS,
    COUNT(DISTINCT ORDERNUMBER) AS NO_OF_ORDERS,
    SUM(QUANTITYORDERED) AS ORDER_QTY,
    ROUND(SUM(SALES),0) AS TOTAL_SALES
FROM SAMPLE_SALES_DATA;
```

### 2. RFM Segmentation (Complete View)

The RFM analysis is implemented using a multi-layered SQL view that:

1. **Summary Table**: Calculates Recency, Frequency, and Monetary values for each customer
2. **RFM Score**: Assigns 1-5 scores using NTILE function
3. **RFM Combination**: Combines scores and creates customer segments

```sql
CREATE OR REPLACE VIEW RFM AS
WITH SUMMARY_TABLE AS (
    SELECT
        CUSTOMERNAME,
        DATEDIFF((SELECT MAX(ORDERDATE) FROM SAMPLE_SALES_DATA), MAX(ORDERDATE)) AS RECENCY_VALUE,
        COUNT(DISTINCT ORDERNUMBER) AS FREQUENCY_VALUE,
        ROUND(SUM(SALES),0) AS MONETARY_VALUE
    FROM SAMPLE_SALES_DATA
    GROUP BY 1
),
RFM_SCORE AS (
    SELECT
        SUMMARY_TABLE.*,
        NTILE(5) OVER(ORDER BY RECENCY_VALUE DESC) AS R_SCORE,
        NTILE(5) OVER(ORDER BY FREQUENCY_VALUE ASC) AS F_SCORE,
        NTILE(5) OVER(ORDER BY MONETARY_VALUE ASC) AS M_SCORE 
    FROM SUMMARY_TABLE
),
RFM_COMBINATION AS (
    SELECT
        RFM_SCORE.*,
        (R_SCORE + F_SCORE + M_SCORE) AS TOTAL_RFM_SCORE,
        CONCAT_WS('', R_SCORE, F_SCORE, M_SCORE) AS RFM_COMBINATION_SCORE
    FROM RFM_SCORE
)
SELECT
    CUSTOMERNAME,
    R_SCORE AS RECENCY, 
    F_SCORE AS FREQUENCY, 
    M_SCORE AS MONETARY, 
    TOTAL_RFM_SCORE AS RFM_SCORE,
    CASE 
        WHEN TOTAL_RFM_SCORE = 15 THEN 'CHAMPIONS'
        WHEN TOTAL_RFM_SCORE >= 12 THEN 'LOYAL_CUSTOMERS'
        WHEN TOTAL_RFM_SCORE BETWEEN 9 AND 11 THEN 'POTENTIAL_LOYALISTS'
        WHEN TOTAL_RFM_SCORE BETWEEN 6 AND 8 THEN 'PROMISING_CUSTOMERS'
        WHEN TOTAL_RFM_SCORE BETWEEN 4 AND 5 THEN 'HIBERNATING'
        WHEN TOTAL_RFM_SCORE <= 3 THEN 'LOST'
        ELSE 'OTHERS'
    END AS CUSTOMER_SEGMENT
FROM RFM_COMBINATION;
```

### 3. Additional Analysis Queries

**Orders Per Date:**
```sql
SELECT 
    ORDERDATE,
    COUNT(ORDERNUMBER) AS NO_OF_ORDERS
FROM SAMPLE_SALES_DATA
GROUP BY 1
ORDER BY 2 DESC;
```

**Top Customers:**
```sql
SELECT
    CUSTOMERNAME,
    ORDERDATE,
    SUM(SALES) AS TOTAL_SALES
FROM SAMPLE_SALES_DATA
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 5;
```

## 📁 Repository Structure

```
RFM-SEGMENTATION/
│
├── README.md                                  # Project documentation
├── RFM SEGMENTATION.sql                      # Complete SQL code
├── Sales Data for RFM Segmentation.csv       # Raw sales data
├── RFM SEGMENTATION OUTPUT.csv               # Final customer segments
├── Customer Segment.xlsx                      # Excel analysis file
├── rfm-analysis.png                          # RFM diagram
└── Customer-segmentation.png                 # Segmentation visualization
```

## 🚀 How to Use

### Prerequisites
- MySQL Server (8.0 or higher recommended)
- MySQL Workbench or any SQL client

### Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/Mdkamrulislam54/RFM-SEGMENTATION.git
   cd RFM-SEGMENTATION
   ```

2. **Import the dataset**
   - Load `Sales Data for RFM Segmentation.csv` into your MySQL database
   - Name the table: `SAMPLE_SALES_DATA`

3. **Run the SQL scripts**
   - Open `RFM SEGMENTATION.sql` in your SQL client
   - Execute the queries sequentially
   - The RFM view will be created automatically

4. **View results**
   ```sql
   SELECT * FROM RFM;
   ```

5. **Export results** (optional)
   - Results can be exported to CSV for further analysis in Excel/Python

## 📈 Results

### Customer Distribution by Segment

| Segment | Count | Percentage |
|---------|-------|------------|
| Champions | 7 | 7.6% |
| Loyal Customers | 18 | 19.6% |
| Potential Loyalists | 25 | 27.2% |
| Promising | 19 | 20.7% |
| Hibernating | 15 | 16.3% |
| Lost | 8 | 8.7% |

### Key Findings

✅ **27.2%** of customers are Potential Loyalists - a great opportunity for targeted marketing  
✅ **27.2%** are Champions or Loyal Customers - the most valuable segment  
✅ **25%** are at risk (Hibernating or Lost) - need re-engagement campaigns  

## 🛠️ Technologies Used

- **Database**: MySQL 8.0
- **SQL Features**: 
  - Common Table Expressions (CTEs)
  - Window Functions (NTILE)
  - CASE statements
  - Date functions (DATEDIFF)
  - Aggregate functions
- **Visualization**: Excel
- **Version Control**: Git & GitHub

## 📚 Resources & References

- [Sales Data CSV](https://github.com/Mdkamrulislam54/RFM-SEGMENTATION/blob/2a702bc8d28bf0c06e0957d468172498b9055928/Sales%20Data%20for%20RFM%20Segmentation.csv)
- [RFM Output CSV](https://github.com/Mdkamrulislam54/RFM-SEGMENTATION/blob/2a702bc8d28bf0c06e0957d468172498b9055928/RFM%20SEGMENTATION%20OUTPUT.csv)
- [Customer Segment Analysis (Excel)](https://github.com/Mdkamrulislam54/RFM-SEGMENTATION/blob/2a702bc8d28bf0c06e0957d468172498b9055928/Customer%20Segment.xlsx)
- [Complete SQL Code](https://github.com/Mdkamrulislam54/RFM-SEGMENTATION/blob/2a702bc8d28bf0c06e0957d468172498b9055928/RFM%20SEGMENTATION.sql)

## 💡 Marketing Strategies by Segment

### 🏆 Champions
- Reward them - early access to new products
- Make them brand ambassadors
- Ask for reviews and testimonials

### 💎 Loyal Customers
- Upsell higher value products
- Exclusive loyalty programs
- Cross-sell opportunities

### 🌟 Potential Loyalists
- Personalized offers
- Membership programs
- Increase engagement frequency

### 🎯 Promising
- Create brand awareness
- Free shipping offers
- Onboarding campaigns

### 😴 Hibernating
- Win-back campaigns
- Special discounts
- "We miss you" emails

### ⚠️ Lost
- Last-ditch offers
- Survey to understand churn
- Aggressive discounts (if high-value)

## 📬 Contact

For questions or suggestions, feel free to reach out!

---

⭐ **If you found this project helpful, please give it a star!** ⭐

---

**License**: This project is open source and available under the MIT License.
