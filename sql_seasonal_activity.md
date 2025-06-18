Активность по времени. топ 10 по кол-ву транзакций
```sql
WITH time_blocks_cte AS(
SELECT
    day_of_week,
    total_transactions,
    CASE
        WHEN transaction_hour BETWEEN 0 AND 5 THEN 'night'
        WHEN transaction_hour BETWEEN 6 AND 11 THEN 'morning'
        WHEN transaction_hour BETWEEN 12 AND 17 THEN 'afternoon'
        ELSE 'evening'
    END AS time_blocks
FROM users_behavior_data)
    


SELECT
    day_of_week,
    time_blocks,
    SUM(total_transactions) AS transactions_per_category
FROM time_blocks_cte
GROUP BY day_of_week, time_blocks
ORDER BY transactions_per_category DESC
LIMIT 10
```
<img width="372" alt="image" src="https://github.com/user-attachments/assets/c6f37814-8576-4de2-abc5-05f36c2f5151" />



Эффективность разных типов промоакций
