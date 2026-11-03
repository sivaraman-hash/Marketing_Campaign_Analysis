## KPI Summary
```sql
SELECT COUNT(*) AS total_customers,
       ROUND(SUM(TotalSpend), 0) AS total_spend,
       ROUND(AVG(Response) * 100, 2) AS overall_response_pct
FROM marketing.default.silver_campaign
```
Note: the KPI cards (Total Customers, Total Spend, Response Rate) use this dataset. The % sign on the Response Rate card is a chart-level format, not part of the SQL.

## KPI (not used by any chart)
```sql
SELECT COUNT(*) AS customers,
       ROUND(SUM(TotalSpend), 0) AS total_spend,
       CONCAT(ROUND(AVG(Response) * 100, 2), '%') AS response_pct
FROM marketing.default.silver_campaign
```
Note: this dataset is an unused leftover. No chart reads it.

## Income
```sql
SELECT IncomeGroup, avg_spend, customers
FROM marketing.default.gold_by_income
ORDER BY CASE IncomeGroup WHEN 'Under 30K' THEN 1 WHEN '30K-60K' THEN 2
                          WHEN '60K-90K' THEN 3 ELSE 4 END
```

## Campaign
```sql
SELECT * FROM marketing.default.gold_by_campaign ORDER BY campaign
```

## Channels
```sql
SELECT * FROM marketing.default.gold_by_channel ORDER BY purchases DESC
```

## Education
```sql
SELECT Education, customers, avg_spend, response_rate_pct
FROM marketing.default.gold_by_education
ORDER BY response_rate_pct DESC
```

## Scatter
```sql
SELECT Income, TotalSpend FROM marketing.default.silver_campaign
```

## Headmap
```sql
SELECT AgeGroup, IncomeGroup, ROUND(AVG(TotalSpend), 0) AS avg_spend
FROM marketing.default.silver_campaign
GROUP BY AgeGroup, IncomeGroup
```

## ProductMix
```sql
SELECT IncomeGroup, product, SUM(spend) AS spend
FROM (
  SELECT IncomeGroup,
    stack(6, 'Wine', MntWines, 'Fruits', MntFruits, 'Meat', MntMeatProducts,
             'Fish', MntFishProducts, 'Sweets', MntSweetProducts, 'Gold', MntGoldProds)
    AS (product, spend)
  FROM marketing.default.silver_campaign
)
GROUP BY IncomeGroup, product
```

## Funnel
```sql
WITH t AS (
  SELECT AcceptedCmp1 + AcceptedCmp2 + AcceptedCmp3 + AcceptedCmp4 + AcceptedCmp5 + Response AS n
  FROM marketing.default.silver_campaign
)
SELECT '1. All customers' AS stage, COUNT(*) AS customers FROM t
UNION ALL SELECT '2. Accepted 1+ offers', COUNT(*) FROM t WHERE n >= 1
UNION ALL SELECT '3. Accepted 2+ offers', COUNT(*) FROM t WHERE n >= 2
UNION ALL SELECT '4. Accepted 3+ offers', COUNT(*) FROM t WHERE n >= 3
```

## Scorecard
```sql
SELECT Education,
       COUNT(*) AS Customers,
       ROUND(AVG(Income), 0) AS Avg_Income,
       ROUND(AVG(TotalSpend), 0) AS Avg_Spend,
       ROUND(AVG(Response) * 100, 1) AS Response_Pct
FROM marketing.default.silver_campaign
GROUP BY Education
ORDER BY Avg_Spend DESC
```