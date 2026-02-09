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

### 1. Finding Number of Customers, Orders, Quantity and Sales

**Query:**
```sql
SELECT 
    COUNT(DISTINCT CUSTOMERNAME) AS NO_OF_CUSTOMERS,
    COUNT(DISTINCT ORDERNUMBER) AS NO_OF_ORDERS,
    SUM(QUANTITYORDERED) AS ORDER_QTY,
    ROUND(SUM(SALES),0) AS TOTAL_SALES
FROM SAMPLE_SALES_DATA;
```

**Output:**
| NO_OF_CUSTOMERS | NO_OF_ORDERS | ORDER_QTY | TOTAL_SALES |
|-----------------|--------------|-----------|-------------|
| 92 | 307 | 99067 | 10032629 |

---

### 2. Number of Orders Per Date

**Query:**
```sql
SELECT 
    ORDERDATE,
    COUNT(ORDERNUMBER) AS NO_OF_ORDERS
FROM SAMPLE_SALES_DATA
GROUP BY 1
ORDER BY 2 DESC;
```

**Output (Top 20):**
| ORDERDATE | NO_OF_ORDERS |
|-----------|--------------|
| 11/14/2003 | 38 |
| 11/24/2004 | 35 |
| 11/12/2003 | 34 |
| 11/17/2004 | 32 |
| 11/4/2004 | 29 |
| 11/5/2003 | 28 |
| 12/2/2003 | 28 |
| 10/16/2004 | 28 |
| 11/6/2003 | 27 |
| 8/20/2004 | 27 |
| 9/8/2004 | 26 |
| 10/14/2004 | 26 |

---

### 3. Quantities Ordered Per Date

**Query:**
```sql
SELECT 
    ORDERDATE,
    SUM(QUANTITYORDERED) AS ORDERS_QTY
FROM SAMPLE_SALES_DATA
GROUP BY 1
ORDER BY 2 DESC;
```

**Output (Top 20):**
| ORDERDATE | ORDERS_QTY |
|-----------|------------|
| 11/24/2004 | 1365 |
| 11/14/2003 | 1306 |
| 11/12/2003 | 1093 |
| 11/17/2004 | 1058 |
| 12/2/2003 | 1010 |
| 10/16/2004 | 1002 |
| 11/4/2004 | 1002 |
| 11/6/2003 | 977 |
| 8/20/2004 | 930 |
| 11/1/2004 | 928 |
| 9/8/2004 | 909 |
| 11/20/2003 | 902 |

---

### 4. Top 5 Highest Paid Customers

**Query:**
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

**Output:**
| CUSTOMERNAME | ORDERDATE | TOTAL_SALES |
|--------------|-----------|-------------|
| Euro Shopping Channel | 11/24/2004 | 259657.14 |
| Mini Gifts Distributors Ltd. | 11/14/2003 | 167130.88 |
| Australian Collectors, Co. | 11/12/2003 | 82261.22 |
| La Rochelle Gifts | 11/17/2004 | 80438.48 |
| Muscle Machine Inc | 11/24/2004 | 75754.54 |

---

### 5. RFM Segmentation (Complete View)

The RFM analysis is implemented using a multi-layered SQL view that:

1. **Summary Table**: Calculates Recency, Frequency, and Monetary values for each customer
2. **RFM Score**: Assigns 1-5 scores using NTILE function
3. **RFM Combination**: Combines scores and creates customer segments

**Query:**
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

SELECT * FROM RFM;
```

**Complete Output (All 92 Customers):**

| CUSTOMERNAME | RECENCY | FREQUENCY | MONETARY | RFM_SCORE | CUSTOMER_SEGMENT |
|--------------|---------|-----------|----------|-----------|------------------|
| Danish Wholesale Imports | 5 | 5 | 5 | 15 | CHAMPIONS |
| Salzburg Collectables | 5 | 5 | 5 | 15 | CHAMPIONS |
| Souveniers And Things Co. | 5 | 5 | 5 | 15 | CHAMPIONS |
| The Sharp Gifts Warehouse | 5 | 5 | 5 | 15 | CHAMPIONS |
| La Rochelle Gifts | 5 | 5 | 5 | 15 | CHAMPIONS |
| Mini Gifts Distributors Ltd. | 5 | 5 | 5 | 15 | CHAMPIONS |
| Euro Shopping Channel | 5 | 5 | 5 | 15 | CHAMPIONS |
| Handji Gifts& Co | 5 | 5 | 4 | 14 | LOYAL_CUSTOMERS |
| Tokyo Collectables, Ltd | 5 | 5 | 4 | 14 | LOYAL_CUSTOMERS |
| Diecast Classics Inc. | 5 | 5 | 4 | 14 | LOYAL_CUSTOMERS |
| Reims Collectables | 4 | 5 | 5 | 14 | LOYAL_CUSTOMERS |
| Corporate Gift Ideas Co. | 4 | 5 | 5 | 14 | LOYAL_CUSTOMERS |
| Anna's Decorations, Ltd | 4 | 5 | 5 | 14 | LOYAL_CUSTOMERS |
| Dragon Souveniers, Ltd. | 4 | 5 | 5 | 14 | LOYAL_CUSTOMERS |
| Gift Depot Inc. | 5 | 4 | 4 | 13 | LOYAL_CUSTOMERS |
| UK Collectables, Ltd. | 5 | 4 | 4 | 13 | LOYAL_CUSTOMERS |
| Muscle Machine Inc | 3 | 5 | 5 | 13 | LOYAL_CUSTOMERS |
| Australian Collectors, Co. | 3 | 5 | 5 | 13 | LOYAL_CUSTOMERS |
| Mini Caravy | 5 | 4 | 3 | 12 | LOYAL_CUSTOMERS |
| Gifts4AllAges.com | 5 | 4 | 3 | 12 | LOYAL_CUSTOMERS |
| Toys of Finland, Co. | 4 | 4 | 4 | 12 | LOYAL_CUSTOMERS |
| Scandinavian Gift Ideas | 4 | 4 | 4 | 12 | LOYAL_CUSTOMERS |
| L'ordine Souveniers | 5 | 2 | 5 | 12 | LOYAL_CUSTOMERS |
| Quebec Home Shopping Network | 5 | 4 | 2 | 11 | POTENTIAL_LOYALISTS |
| Tekni Collectables Inc. | 4 | 4 | 3 | 11 | POTENTIAL_LOYALISTS |
| Auto Canal Petit | 4 | 4 | 3 | 11 | POTENTIAL_LOYALISTS |
| FunGiftIdeas.com | 4 | 4 | 3 | 11 | POTENTIAL_LOYALISTS |
| Oulu Toy Supplies, Inc. | 4 | 3 | 4 | 11 | POTENTIAL_LOYALISTS |
| Toys4GrownUps.com | 4 | 3 | 4 | 11 | POTENTIAL_LOYALISTS |
| Technics Stores Inc. | 3 | 4 | 4 | 11 | POTENTIAL_LOYALISTS |
| AV Stores, Co. | 3 | 3 | 5 | 11 | POTENTIAL_LOYALISTS |
| Land of Toys Inc. | 2 | 4 | 5 | 11 | POTENTIAL_LOYALISTS |
| Royale Belge | 4 | 5 | 1 | 10 | POTENTIAL_LOYALISTS |
| Alpha Cognac | 4 | 4 | 2 | 10 | POTENTIAL_LOYALISTS |
| Volvo Model Replicas, Co | 3 | 5 | 2 | 10 | POTENTIAL_LOYALISTS |
| Lyon Souveniers | 4 | 4 | 2 | 10 | POTENTIAL_LOYALISTS |
| Collectables For Less Inc. | 4 | 3 | 3 | 10 | POTENTIAL_LOYALISTS |
| Mini Creations Ltd. | 3 | 3 | 4 | 10 | POTENTIAL_LOYALISTS |
| Suominen Souveniers | 3 | 3 | 4 | 10 | POTENTIAL_LOYALISTS |
| Baane Mini Imports | 2 | 4 | 4 | 10 | POTENTIAL_LOYALISTS |
| Mini Auto Werke | 4 | 4 | 1 | 9 | POTENTIAL_LOYALISTS |
| Australian Gift Network, Co | 4 | 4 | 1 | 9 | POTENTIAL_LOYALISTS |
| Petit Auto | 5 | 2 | 2 | 9 | POTENTIAL_LOYALISTS |
| Signal Gift Stores | 3 | 3 | 3 | 9 | POTENTIAL_LOYALISTS |
| Motor Mint Distributors Inc. | 3 | 3 | 3 | 9 | POTENTIAL_LOYALISTS |
| Blauer See Auto, Co. | 2 | 4 | 3 | 9 | POTENTIAL_LOYALISTS |
| Stylish Desk Decors, Co. | 3 | 3 | 3 | 9 | POTENTIAL_LOYALISTS |
| La Corne D'abondance, Co. | 3 | 3 | 3 | 9 | POTENTIAL_LOYALISTS |
| Rovelli Gifts | 2 | 2 | 5 | 9 | POTENTIAL_LOYALISTS |
| Australian Collectables, Ltd | 5 | 2 | 1 | 8 | PROMISING_CUSTOMERS |
| Mini Wheels Co. | 3 | 3 | 2 | 8 | PROMISING_CUSTOMERS |
| Marseille Mini Autos | 3 | 3 | 2 | 8 | PROMISING_CUSTOMERS |
| Classic Legends Inc. | 3 | 3 | 2 | 8 | PROMISING_CUSTOMERS |
| Enaco Distributors | 3 | 3 | 2 | 8 | PROMISING_CUSTOMERS |
| Cruz & Sons Co. | 2 | 3 | 3 | 8 | PROMISING_CUSTOMERS |
| Marta's Replicas Co. | 2 | 2 | 4 | 8 | PROMISING_CUSTOMERS |
| Corrida Auto Replicas, Ltd | 2 | 2 | 4 | 8 | PROMISING_CUSTOMERS |
| Online Diecast Creations Co. | 2 | 2 | 4 | 8 | PROMISING_CUSTOMERS |
| Saveley & Henriot, Co. | 1 | 2 | 5 | 8 | PROMISING_CUSTOMERS |
| Boards & Toys Co. | 4 | 2 | 1 | 7 | PROMISING_CUSTOMERS |
| Atelier graphique | 3 | 3 | 1 | 7 | PROMISING_CUSTOMERS |
| Auto-Moto Classics Inc. | 3 | 3 | 1 | 7 | PROMISING_CUSTOMERS |
| Gift Ideas Corp. | 3 | 3 | 1 | 7 | PROMISING_CUSTOMERS |
| Mini Classics | 2 | 2 | 3 | 7 | PROMISING_CUSTOMERS |
| Vitachrome Inc. | 2 | 2 | 3 | 7 | PROMISING_CUSTOMERS |
| Toms Spezialitten, Ltd | 2 | 2 | 3 | 7 | PROMISING_CUSTOMERS |
| Heintze Collectables | 2 | 2 | 3 | 7 | PROMISING_CUSTOMERS |
| Herkku Gifts | 1 | 2 | 4 | 7 | PROMISING_CUSTOMERS |
| Auto Assoc. & Cie. | 2 | 2 | 2 | 6 | PROMISING_CUSTOMERS |
| Classic Gift Ideas, Inc | 2 | 2 | 2 | 6 | PROMISING_CUSTOMERS |
| Canadian Gift Exchange Network | 2 | 2 | 2 | 6 | PROMISING_CUSTOMERS |
| Vida Sport, Ltd | 1 | 1 | 4 | 6 | PROMISING_CUSTOMERS |

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


