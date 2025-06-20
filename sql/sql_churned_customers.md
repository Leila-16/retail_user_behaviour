<div id="user-content-toc">
  <ul style="list-style: none;">
    <summary>
      <h2>Сегментация клиентов: поведенческий профиль, причины оттока</h2>
    </summary>
  </ul>
</div>

---
### Результаты анализа
Проведён анализ клиентов, прекративших покупки, с фокусом на LTV, лояльность, recency и возвратность.

**Ушедшие покупатели**: Обе группы — активные и неактивные — представлены практически поровну, что свидетельствует о том, что объём покупок и размер скидок не являются причинами оттока. Тем не менее, 92.5% ушедших — это нелояльные клиенты, что подтверждает важность программы лояльности для удержания. Гипотеза, что высокий уровень возвратов влияет на отток, не подтвердилась: только каждый пятый ушедший клиент имел высокий показатель возвратов.

**Recency**: Наиболее частые покупки совершают клиенты, ориентированные на продукты питания. Те, кто покупает реже (раз в 2–3 месяца), преимущественно выбирают игрушки и составляют самую большую клиентскую группу. Уровень оттока везде стабилен. Расстояние до магазина не оказывает существенного влияния на частоту покупок — скорее решает тип предпочитаемой категории.

**LTV по возрасту**: Самый высокий LTV наблюдается у пользователей в возрасте 18–24, их меньше всего. Уровень оттока стабилен во всех возрастных группах — около 42%, что подчёркивает сложность удержания вне зависимости от возраста покупателя.

---
### Аналитические запросы
1. Ушедшие (churned) клиенты. Сегментация по общей выручке:
	 - high-activity - total sales выше среднего<br>
	 - low-activity - total sales ниже среднего<br>

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


---


2. Ушедшие (churned) клиенты. Сегментация по лояльности:
   - Loyal:
     - membership_years > 3
     - total_transactions выше среднего
     - days_since_last_purchase < 90 (ушли менее 3 месяцев назад)
   - Disloyal: остальные

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


---


3. Ушедшие (churned) клиенты с высокой долей возвратов:
   - return_rate выше среднего

```sql
WITH avg_values AS (
SELECT
    AVG(total_returned_value / NULLIF(total_sales, 0)) AS avg_return_rate
FROM users_behavior_data
WHERE churned = true),


categorised AS(
SELECT
    total_returned_value,
        total_sales,
        total_returned_value / NULLIF(total_sales, 0) AS return_rate,
	CASE
		WHEN total_returned_value / NULLIF(total_sales, 0) > (SELECT avg_return_rate FROM avg_values)
		THEN 'high_return_churned'
		ELSE 'other_churned'
	END AS return_type
FROM users_behavior_data
WHERE churned = true)


SELECT 
    return_type,
    COUNT(*) AS number_churned,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS segment_share
FROM categorised
GROUP BY return_type
ORDER BY return_type 
```
<img width="369" alt="image" src="https://github.com/user-attachments/assets/c4f35808-dad8-48ae-a32f-28280094c035" />


---


4. Сегментация покупателей по recency (количеству дней с последней покупки)

```sql
WITH recency_segments_cte AS(
SELECT
    distance_to_store,
    avg_transaction_value,
    total_transactions,
    churned,
    product_category,
    CASE
        WHEN days_since_last_purchase BETWEEN 0 AND 25 THEN '0-25'
        WHEN days_since_last_purchase BETWEEN 26 AND 50 THEN '26-50'
        WHEN days_since_last_purchase BETWEEN 51 AND 75 THEN '51-75'
        ELSE '76-99'
    END AS recency_segments
FROM users_behavior_data),


product_category_count AS(
SELECT
        recency_segments,
        product_category,
        COUNT(*) AS product_count,
        ROW_NUMBER() OVER (PARTITION BY recency_segments ORDER BY COUNT(*) DESC) AS rn
FROM recency_segments_cte
GROUP BY recency_segments, product_category),


top_categories AS (
    SELECT
        recency_segments,
        product_category AS favourite_category
    FROM product_category_count
    WHERE rn = 1)


SELECT
    recency_segments_cte.recency_segments,
    COUNT(*) AS customers_number,
    ROUND(COUNT(*)::DECIMAL / (SELECT COUNT(customer_id) FROM users_behavior_data), 2)*100 AS customers_share,
    ROUND(COUNT(*) FILTER(WHERE churned = TRUE)::DECIMAL / COUNT(*), 2)*100 AS churn_rate,
    favourite_category,
    ROUND(AVG(distance_to_store), 2) AS avg_distance_to_store,
    ROUND(AVG(avg_transaction_value), 2) AS avg_transaction,
    ROUND(AVG(total_transactions), 2) AS avg_total_transactions
FROM recency_segments_cte
    LEFT JOIN top_categories 
    ON recency_segments_cte.recency_segments = top_categories.recency_segments
GROUP BY recency_segments_cte.recency_segments, favourite_category
ORDER BY recency_segments_cte.recency_segments;
```
<img width="1060" alt="image" src="https://github.com/user-attachments/assets/fca7e0ac-7e16-41c4-b5dd-c5c9f432b94a" />


---


5. Средний LTV по возрастным группам

```sql
WITH age_buckets AS(
SELECT
    total_sales,
    total_transactions,
    churned,
    CASE
        WHEN age BETWEEN 18 AND 24 THEN '18-24'
        WHEN age BETWEEN 25 AND 34 THEN '25-34'
        WHEN age BETWEEN 35 AND 44 THEN '35-44'
        WHEN age BETWEEN 45 AND 54 THEN '45-54'
        WHEN age BETWEEN 55 AND 64 THEN '55-64'
        ELSE '65+'
    END AS age_groups
FROM users_behavior_data)


SELECT 
    age_groups,
    COUNT(*) AS total_users,
    ROUND(AVG(total_sales / total_transactions), 2) AS ltv,
    ROUND(COUNT(*) FILTER(WHERE churned = true)::DECIMAL / COUNT(*), 2)*100 AS churn_rate
FROM age_buckets
GROUP BY age_groups
```
<img width="370" alt="image" src="https://github.com/user-attachments/assets/acfbf8b9-91fd-4813-9be1-ce1c0e33c3fa" />




