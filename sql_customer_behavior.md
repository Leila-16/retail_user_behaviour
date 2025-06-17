Ушедшие клиенты. Сегментация по общей выручке:		
	- high-activity - total sales выше среднего
 	- low-activity - total sales ниже среднего
```sql
WITH avg_total_sales AS(
SELECT
    AVG(total_sales) AS avg_sales
FROM users_behavior_data 
WHERE churned = true    
),

categorised AS(
SELECT
    total_sales,
    churned,
    avg_transaction_value,
    avg_discount_used,
    total_returned_value,
    CASE 
        WHEN total_sales > (SELECT avg_sales FROM avg_total_sales) THEN 'high_activity_churned'
        ELSE 'low_activity_churned'
    END AS churned_activity
FROM users_behavior_data
WHERE churned = true)

SELECT 
    COUNT(*) FILTER(WHERE churned_activity = 'high_activity_churned') AS high_activity_churned,
    COUNT(*) FILTER(WHERE churned_activity = 'low_activity_churned') AS low_activity_churned,
    ROUND(AVG(avg_transaction_value), 3) AS avg_cheque,
    ROUND(AVG(avg_discount_used), 3) AS avg_discount,
    ROUND(AVG(total_returned_value), 3) AS avg_returns
FROM categorised
GROUP BY churned_activity
```

Ушедшие клиенты. Сегментация по лояльности:
	- loyal: 
 		- membership years > 3
	 	- total_transactions выше среднего
	  - days_since_last_purchase < 90 - ушли менее 3х мес. назад
 

