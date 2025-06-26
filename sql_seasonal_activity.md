<kbd>[RU Русская версия](#русская-версия)</kbd>  
<kbd>[ENG English Version](#english-version)</kbd>


<h3 align="right"><a name="русская-версия">Русская версия</a></h3>


# Анализ поведенческих паттернов покупателей: время, сезонность и ответ на промоакции
Проведён анализ поведения пользователей в зависимости от временных факторов, сезонности и промоакций.

---

### Результаты анализа

**Временные паттерны:** Явных лидеров среди дней недели не выявлено, однако в четверг фиксируется минимальное количество транзакций. Гипотеза о повышении покупательской активности в выходные не подтвердилась. Наиболее активное время покупок — вечер, наименее — утро и день. 

**Сезонность:** Гипотеза о том, что семьи с детьми совершают больше покупок летом и осенью в рамках подготовки к школе, не подтвердилась — сезонный всплеск активности практически отсутствует. Однако, у обеих категорий заметно повышение количества покупок зимой, что может быть связано с праздничными покупками. 

**Промоакции:** В целом, промоакции не оказали выраженного влияния на потребительские паттерны. Однако, предложение «Buy one get one free» вызвало наибольший отклик и привело к росту числа транзакций по сравнению с другими видами акций.

---
### Аналитические запросы
1. Активность пользователей по дню недели и времени. Топ-10 временных категорий по кол-ву транзакций
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




2. Активность пользователей по сезонам в разрезе наличия детей
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
    ROW_NUMBER() OVER (
			PARTITION BY season, kids_groups
			ORDER BY category_count DESC) AS rank
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





3. Эффективность разных типов промоакций
```sql
SELECT
    promotion_type,
    COUNT(*) AS transactions_number,
    ROUND(SUM(online_purchases)::DECIMAL
			/ (SUM(online_purchases) + SUM(in_store_purchases)), 2)*100 AS online_purchases_share,
    ROUND(SUM(in_store_purchases)::DECIMAL
			/ (SUM(online_purchases) + SUM(in_store_purchases)), 2)*100 AS in_store_purchases_share,
    ROUND(COUNT(*)::DECIMAL
			/ (SELECT COUNT(*) FROM users_behavior_data WHERE promotion_type IS NOT NULL),2)*100 AS transactions_share,
    ROUND(AVG(avg_transaction_value), 2) AS avg_cheque,
    ROUND(AVG(avg_discount_used), 2) AS avg_discount,
    ROUND(AVG(total_sales / total_transactions), 2) AS avg_ltv
FROM users_behavior_data
WHERE promotion_type IS NOT NULL
GROUP BY promotion_type
```
<img width="1002" alt="image" src="https://github.com/user-attachments/assets/b604a3b5-c17d-43cb-9a7b-1a1b0ceb32c8" />



<h3 align="right"><a name="english-version">English Version</a></h3>

# Customer Behavioral Pattern Analysis: Timing, Seasonality, and Response to Promotions
User behavior was analyzed in relation to temporal factors, seasonal trends, and promotional campaign effectiveness.

---

### Key Findings

**Time-based Patterns:** No clear leaders among days of the week were identified, although Thursday recorded the lowest number of transactions. The hypothesis that weekends drive higher shopping activity was not confirmed. Peak purchasing occurs in the evening, with the lowest activity in the morning and afternoon.

**Seasonality:** The hypothesis that families with children shop more in summer and fall during back-to-school season was not supported — seasonal spikes were nearly absent. However, both groups showed a notable increase in purchases during winter, likely tied to holiday shopping.

**Promotions:** Overall, promotions had limited impact on consumer patterns. However, the “Buy one get one free” offer elicited the strongest response, leading to a higher transaction volume compared to other promotion types.

---

### Analytical Queries
1. User Activity by Day of the Week and Time of Day: Top 10 Time Categories by Transactions Number
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




2. User Activity by Season, Segmented by Presence of Children
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
    ROW_NUMBER() OVER (
			PARTITION BY season, kids_groups
			ORDER BY category_count DESC) AS rank
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





3. Effectiveness of Different Types of Promotions
```sql
SELECT
    promotion_type,
    COUNT(*) AS transactions_number,
    ROUND(SUM(online_purchases)::DECIMAL
			/ (SUM(online_purchases) + SUM(in_store_purchases)), 2)*100 AS online_purchases_share,
    ROUND(SUM(in_store_purchases)::DECIMAL
			/ (SUM(online_purchases) + SUM(in_store_purchases)), 2)*100 AS in_store_purchases_share,
    ROUND(COUNT(*)::DECIMAL
			/ (SELECT COUNT(*) FROM users_behavior_data WHERE promotion_type IS NOT NULL),2)*100 AS transactions_share,
    ROUND(AVG(avg_transaction_value), 2) AS avg_cheque,
    ROUND(AVG(avg_discount_used), 2) AS avg_discount,
    ROUND(AVG(total_sales / total_transactions), 2) AS avg_ltv
FROM users_behavior_data
WHERE promotion_type IS NOT NULL
GROUP BY promotion_type
```
<img width="1002" alt="image" src="https://github.com/user-attachments/assets/b604a3b5-c17d-43cb-9a7b-1a1b0ceb32c8" />
