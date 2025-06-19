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




Активность по сезонам. группа с детьми и без
```sql
WITH kids_groups_cte AS(
SELECT
    season,
    product_category,
    total_sales,
    CASE
        WHEN number_of_children = 0 THEN 'no_kids'
        ELSE 'with_kids'
    END AS kids_groups
FROM users_behavior_data),


category_count AS(
SELECT
    season, 
    kids_groups,
    product_category,
    COUNT(*) AS category_count
FROM kids_groups_cte
GROUP BY season, kids_groups, product_category),



top_categories AS(
SELECT
    season, 
    kids_groups,
    product_category,
    ROW_NUMBER() OVER (PARTITION BY season, kids_groups ORDER BY category_count DESC) AS rank
FROM category_count
WHERE category_count > 0)


SELECT
    k.season, 
    k.kids_groups,
    t.product_category AS top_category,
    COUNT(*) AS customers_number,
    ROUND(SUM(k.total_sales), 2) AS total_sales
FROM kids_groups_cte k
JOIN top_categories t
  ON k.season = t.season
 AND k.kids_groups = t.kids_groups
 AND k.product_category = t.product_category
WHERE t.rank = 1
GROUP BY k.season, k.kids_groups, t.product_category
ORDER BY k.kids_groups DESC, total_sales DESC;
```
<img width="528" alt="image" src="https://github.com/user-attachments/assets/2c5bbc0e-255f-4b24-8807-ce915c8b5375" />





Эффективность разных типов промоакций
```sql
SELECT
    promotion_type,
    COUNT(*) AS transactions_number,
    ROUND(SUM(online_purchases)::DECIMAL / (SUM(online_purchases) + SUM(in_store_purchases)), 2)*100 AS online_purchases_share,
    ROUND(SUM(in_store_purchases)::DECIMAL / (SUM(online_purchases) + SUM(in_store_purchases)), 2)*100 AS in_store_purchases_share,
    ROUND(COUNT(*)::DECIMAL / (SELECT COUNT(*) FROM users_behavior_data WHERE promotion_type IS NOT NULL),2)*100 AS transactions_share,
    ROUND(AVG(avg_transaction_value), 2) AS avg_cheque,
    ROUND(AVG(avg_discount_used), 2) AS avg_discount,
    ROUND(AVG(total_sales / total_transactions), 2) AS avg_ltv
FROM users_behavior_data
WHERE promotion_type IS NOT NULL
GROUP BY promotion_type
```
<img width="1002" alt="image" src="https://github.com/user-attachments/assets/b604a3b5-c17d-43cb-9a7b-1a1b0ceb32c8" />
