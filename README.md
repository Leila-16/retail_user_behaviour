<kbd>[RU Русская версия](#русская-версия)</kbd>  
<kbd>[ENG English Version](#english-version)</kbd>


<h3 align="right"><a name="русская-версия">Русская версия</a></h3>

# Анализ поведения потребителей в ритейле на основе данных из публичного датасета

- Использован публичный датасет [Retail Sales and Customer Behavior Analysis](https://www.kaggle.com/datasets/utkalk/large-retail-data-set-for-eda/data)
- Для симуляции разбалансировки удалена часть ушедших (churned) клиентов
- Составлены аналитические запросы на языке SQL:
	- [customers behavior](sql_customers_behavior.md)
	- [seasonal activity](sql_seasonal_activity.md)
---

### Результаты анализа:
Был проведен сводный анализ поведенческих паттернов клиентов, включая их активность и отток. По итогам анализа сделаны следующие выводы.

Покупательская активность не подвержена резкому влиянию дня недели, но заметно усиливается в вечернее время. Сезонность подчёркивает различия в предпочтениях: семьи с детьми чаще совершают крупные покупки весной и осенью, в то время как клиенты без детей стабильно ориентируются на электронику. Промоакции не оказывают существенного влияния на потребление, но "1+1 бесплатно" демонстрирует наибольшую вовлечённость.

В анализе ушедших клиентов выявлено, что отток слабо связан с объёмом покупок или возвратами, но чётко коррелирует с отсутствием лояльности: 92,5% ушедших — не участвовали в программе лояльности. Молодые клиенты (18–24) генерируют самый высокий LTV, хотя их численно меньше, а основная клиентская масса — это покупатели с низкой частотой покупок и ориентацией на нестандартные категории, такие как игрушки. Расстояние до магазина практически не влияет на поведение.

---

<h3 align="right"><a name="english-version">English Version</a></h3>

# Consumer Behavior Analysis in Retail Based on a Public Dataset

- Utilized public dataset [Retail Sales and Customer Behavior Analysis](https://www.kaggle.com/datasets/utkalk/large-retail-data-set-for-eda/data)
- To simulate class imbalance, a portion of churned customers was removed
- Developed analytical SQL queries:
	- [customers behavior](sql_customers_behavior.md)
	- [seasonal activity](sql_seasonal_activity.md)
---

### Analysis Findings:
A comprehensive analysis of customer behavioral patterns was conducted, focusing on engagement and churn. The key insights include.

Customer activity is not heavily influenced by the day of the week but significantly increases during evening hours. Seasonal trends reveal differing preferences: families with children tend to make larger purchases in spring and autumn, while child-free customers consistently focus on electronics. Promotions generally show little impact on consumption, though "Buy One Get One Free" campaigns generate the highest engagement.

The churn analysis revealed that customer attrition is weakly linked to purchase volume or returns but strongly correlates with loyalty program participation: 92.5% of churned users were not enrolled in the loyalty program. Young customers (ages 18–24) generate the highest LTV despite being a smaller group, while the core audience consists of infrequent buyers often interested in niche categories like toys. Distance to the store has little influence on behavior.
