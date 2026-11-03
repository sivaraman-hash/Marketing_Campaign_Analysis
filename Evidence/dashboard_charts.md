# Dashboard charts

## Page: Overview
| Chart title | Type | Dataset | Fields |
|---|---|---|---|
| Total Customers | Counter | KPI Summary | SUM(total_customers) |
| Total Spend | Counter | KPI Summary | total_spend |
| Response Rate (%) | Counter | KPI Summary | overall_response_pct (percent suffix added in the chart panel; aggregation shown as P50) |
| Higher income, higher spending | Bar | Income | X IncomeGroup, Y SUM(avg_spend) |
| Last campaign worked best | Horizontal bar | Campaign | X acceptance_pct, Y campaign |
| Store is the top channel | Bar | Channels | X channel, Y SUM(purchases) |
| Response rate by education | Bar | Education | X Education, Y response_rate_pct |

## Page: Customer Insights
| Chart title | Type | Dataset | Fields |
|---|---|---|---|
| Income vs spending | Scatter | Scatter | X Income, Y TotalSpend |
| Who spends the most | Heatmap | Headmap | X IncomeGroup, Y AgeGroup, color avg_spend |
| What each group buys | Stacked bar | ProductMix | X IncomeGroup, Y spend, color product |
| Offer acceptance funnel | Funnel | Funnel | stage, customers |
| Segment scorecard | Table | Scorecard | Education, Customers, Avg_Income, Avg_Spend, Response_Pct |

Note: the dataset "KPI" is not used by any chart. The KPI cards use "KPI Summary".