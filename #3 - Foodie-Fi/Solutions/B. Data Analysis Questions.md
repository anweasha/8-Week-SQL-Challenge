## B. Data Analysis Questions

### 1. How many customers has Foodie-Fi ever had?

```sql
SELECT COUNT(DISTINCT(customer_id)) AS total_customers
FROM foodie_fi.subscriptions;
```

| total_customers |
| --------------- |
| 1000            |

---

### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

```sql
SELECT TO_CHAR(start_date, 'Month') AS month, COUNT(*) AS trial_plan_count
FROM foodie_fi.subscriptions
WHERE plan_id = 0
GROUP BY month
ORDER BY MIN(start_date);
```

| month     | trial_plan_count |
| --------- | ---------------- |
| January   | 88               |
| February  | 68               |
| March     | 94               |
| April     | 81               |
| May       | 88               |
| June      | 79               |
| July      | 89               |
| August    | 88               |
| September | 87               |
| October   | 79               |
| November  | 75               |
| December  | 84               |

---

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.

```sql
SELECT p.plan_name, COUNT(*) AS event_count
FROM foodie_fi.subscriptions AS s
JOIN foodie_fi.plans AS p ON s.plan_id = p.plan_id
WHERE EXTRACT(YEAR FROM s.start_date) > 2020
GROUP BY p.plan_name
ORDER BY event_count;
```

| plan_name     | event_count |
| ------------- | ----------- |
| basic monthly | 8           |
| pro monthly   | 60          |
| pro annual    | 63          |
| churn         | 71          |

---

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

```sql
SELECT SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END) AS churn_count,
ROUND(100.0 * SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END)/COUNT(DISTINCT customer_id), 1) AS churn_percentage
FROM foodie_fi.subscriptions;
```

| churn_count | churn_percentage |
| ----------- | ---------------- |
| 307         | 30.7             |

---

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

```sql
WITH Temp AS (
  SELECT customer_id, plan_id, start_date,
  LEAD(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_plan
  FROM foodie_fi.subscriptions
)

SELECT SUM(CASE WHEN plan_id = 0 AND next_plan = 4 THEN 1 ELSE 0 END) AS churn_count,
ROUND(100.0 * SUM(CASE WHEN plan_id = 0 AND next_plan = 4 THEN 1 ELSE 0 END)/COUNT(DISTINCT customer_id)) AS churn_percentage
FROM Temp;
```

| churn_count | churn_percentage |
| ----------- | ---------------- |
| 92          | 9                |

---

### 6. What is the number and percentage of customer plans after their initial free trial?

```sql
WITH Temp AS (
  SELECT customer_id, plan_id, start_date,
  LEAD(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_plan
  FROM foodie_fi.subscriptions
)

SELECT t.next_plan, p.plan_name, COUNT(*) AS customer_count,
ROUND(100.0 * COUNT(*)/(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions), 1) AS customer_percentage
FROM Temp AS t
JOIN foodie_fi.plans AS p ON t.next_plan = p.plan_id
WHERE t.plan_id = 0
GROUP BY t.next_plan, p.plan_name
ORDER BY next_plan;
```

| next_plan | plan_name     | customer_count | customer_percentage |
| --------- | ------------- | -------------- | ------------------- |
| 1         | basic monthly | 546            | 54.6                |
| 2         | pro monthly   | 325            | 32.5                |
| 3         | pro annual    | 37             | 3.7                 |
| 4         | churn         | 92             | 9.2                 |

---

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

```sql
WITH Temp AS (
  SELECT customer_id, plan_id, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date DESC) AS rn
  FROM foodie_fi.subscriptions
  WHERE start_date <= '2020-12-31'
)
SELECT t.plan_id, p.plan_name, COUNT(*) AS customer_count, 
ROUND(100.0 * COUNT(*)/(SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions), 1) AS customer_percentage
FROM Temp AS t
JOIN foodie_fi.plans AS p ON t.plan_id = p.plan_id
WHERE t.rn = 1
GROUP BY t.plan_id, p.plan_name;
```

| plan_id | plan_name     | customer_count | customer_percentage |
| ------- | ------------- | -------------- | ------------------- |
| 0       | trial         | 19             | 1.9                 |
| 1       | basic monthly | 224            | 22.4                |
| 2       | pro monthly   | 326            | 32.6                |
| 3       | pro annual    | 195            | 19.5                |
| 4       | churn         | 236            | 23.6                |

---

### 8. How many customers have upgraded to an annual plan in 2020?

```sql
SELECT COUNT(*) AS customer_count
FROM foodie_fi.subscriptions
WHERE plan_id = 3 AND EXTRACT(YEAR FROM start_date) = 2020;
```

| customer_count |
| -------------- |
| 195            |

---

### 9. How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?

```sql
SELECT ROUND(AVG(a.start_date - s.start_date), 2) AS avg_days_to_upgrade
FROM foodie_fi.subscriptions AS s
JOIN foodie_fi.subscriptions AS a ON a.customer_id = s.customer_id AND a.plan_id = 3 
WHERE s.plan_id = 0;
```

| avg_days_to_upgrade |
| ------------------- |
| 104.62              |

---
