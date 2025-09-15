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
