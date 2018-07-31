# Capstone_sql

SELECT *
FROM subscriptions
Limit 100;

WITH months as (
  SELECT 
   '2017-01-01' AS first_day,
   '2017-01-31' AS last_day
  UNION 
  SELECT 
   '2017-02-01' AS first_day,
   '2017-02-28' AS last_day
  UNION
  SELECT
   '2017-03-01' AS first_day,
   '2017-03-31' AS last_day  
),

cross_join as (
SELECT *
FROM subscriptions
CROSS JOIN months
),

status as (
 SELECT 
  id, 
  first_day as month, 
 CASE 
   WHEN (subscription_start < first_day)
   AND (subscription_end > first_day
   OR subscription_end is NULL)
   AND segment = 87
   Then 1 
   ELSE 0
   END AS is_active_87,
  
 CASE 
   WHEN (subscription_end between first_day and last_day)
   AND segment = 87
   THEN 1
   ELSE 0
   END as is_canceled_87,
  
 CASE 
   WHEN (subscription_start < first_day)
   AND (subscription_end > first_day
   OR subscription_end is NULL)
   AND segment = 30
   THEN 1
   ELSE 0
   END as is_active_30,
       
 CASE 
   WHEN (subscription_end between first_day and last_day)
   AND segment = 30
   THEN 1
   ELSE 0
   END as is_canceled_30
 FROM cross_join
),

status_aggregate as 
(SELECT 
 month,
 SUM(is_active_87) as sum_active_87,
 SUM(is_active_30) as sum_active_30,
 SUM(is_canceled_87) as sum_canceled_87,
 SUM(is_canceled_30) as sum_canceled_30
 FROM status
 Group by month
)

Select
 month,
 1.0 * sum_canceled_30 / sum_active_30 as churn_30,
 1.0 * sum_canceled_87 / sum_active_87 as 
churn_87,
 1.0 * (sum_canceled_30 + sum_canceled_87)/(sum_active_30 + sum_active_87) as total_churn
FROM status_aggregate;
