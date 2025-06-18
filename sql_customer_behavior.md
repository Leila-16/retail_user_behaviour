Ушедшие клиенты. Сегментация по общей выручке:		
	- high-activity - total sales выше среднего
 	- low-activity - total sales ниже среднего
по результатам видно, что обе категории равны => ушло примерно равное количество пользователей с общими покупками выше и ниже среднего

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
    churned_activity,
    COUNT(*) AS churned_activity_count,
    ROUND(AVG(avg_transaction_value), 3) AS avg_cheque,
    ROUND(AVG(avg_discount_used), 3) AS avg_discount,
    ROUND(AVG(total_returned_value), 3) AS avg_returns
FROM categorised
GROUP BY churned_activity
```
<img width="594" alt="image" src="https://github.com/user-attachments/assets/4fc78a85-0942-4b7c-a4bb-cfa337af243e" />




Ушедшие клиенты. Сегментация по лояльности:
	- loyal: 
 		- membership years > 3
	 	- total_transactions выше среднего
	  - days_since_last_purchase < 90 - ушли менее 3х мес. назад

По результатам видно, что большая часть ушедших клиентов (92.5%) - не лояльны. Также видно, что возвраты или скидки не влияли на уход - вобеих категориях (лояльные/нет) средние возвраты и скидки примерно равны.

```sql
WITH avg_total_transactions AS(
SELECT
    AVG(total_transactions) AS avg_transactions
FROM users_behavior_data
),


churned_segments AS(
SELECT 
    CASE
        WHEN 
            membership_years > 3 AND
            days_since_last_purchase < 90 AND
            total_transactions > (SELECT avg_transactions FROM avg_total_transactions)
        THEN 'loyal_churned'
        ELSE 'disloyal_churned'
    END AS churned_loyalty,
    total_returned_value,
    avg_discount_used
FROM users_behavior_data
WHERE churned = TRUE)


SELECT
    churned_loyalty,
    COUNT(*) AS loyalty_segments,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS segment_share,
    ROUND(AVG(total_returned_value), 2) AS avg_returns,
    ROUND(AVG(avg_discount_used), 2) AS avg_discount
FROM churned_segments
GROUP BY churned_loyalty
```
<img width="594" alt="image" src="https://github.com/user-attachments/assets/289e30e8-0976-49c2-a767-b0d791add016" />

