## B. Runner and Customer Experience

### 1. How many runners signed up for each 1 week period? (i.e. week starts `2021-01-01`)

```sql
SELECT 1 + FLOOR((registration_date - DATE '2021-01-01')/7) AS week, COUNT(*) AS runner_count
FROM pizza_runner.runners
GROUP BY week
ORDER BY week;
```

| week | runner_count |
| ---- | ------------ |
| 1    | 2            |
| 2    | 1            |
| 3    | 1            |

---

### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```sql
WITH delay AS (
  SELECT DISTINCT c.order_id, r.runner_id, (r.pickup_time - c.order_time) AS arrive_time
  FROM orders_customer AS c
  JOIN orders_runner AS r ON c.order_id = r.order_id AND r.cancellation = ''
)
SELECT runner_id, ROUND(AVG(EXTRACT(EPOCH FROM arrive_time)/60.0)::numeric, 2) AS avg_time_to_arrive
FROM delay
GROUP BY runner_id;
```

| runner_id | avg_time_to_arrive |
| --------- | ------------------ |
| 1         | 14.33              |
| 2         | 20.01              |
| 3         | 10.47              |

---

### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

```sql
WITH Temp AS (
  SELECT c.order_id, COUNT(c.pizza_id) AS pizza_count,
  EXTRACT(EPOCH FROM (r.pickup_time - c.order_time)/60.0) AS prep_time
  FROM orders_customer AS c
  JOIN orders_runner AS r ON c.order_id = r.order_id  AND r.cancellation = ''
  GROUP BY c.order_id, r.pickup_time, c.order_time
)
SELECT pizza_count, ROUND(AVG(prep_time)::numeric, 2) AS avg_prep_time
FROM Temp
GROUP BY pizza_count;
```

| pizza_count | avg_prep_time |
| ----------- | ------------- |
| 1           | 12.36         |
| 2           | 18.38         |
| 3           | 29.28         |

Preparation time increases as the number of pizzas in an order increases.

---

### 4. What was the average distance travelled for each customer?

```sql
SELECT c.customer_id, ROUND(AVG(r.distance)::numeric, 2) AS avg_dist_travelled
FROM orders_customer AS c
JOIN orders_runner AS r ON c.order_id = r.order_id AND r.cancellation = ''
GROUP BY c.customer_id;
```

| customer_id | avg_dist_travelled |
| ----------- | ------------------ |
| 101         | 20.00              |
| 102         | 16.73              |
| 103         | 23.40              |
| 104         | 10.00              |
| 105         | 25.00              |

---

### 5. What was the difference between the longest and shortest delivery times for all orders?

```sql
SELECT MAX(duration) - MIN(duration) AS delivery_time_diff
FROM orders_runner
WHERE cancellation = '';
```

| delivery_time_diff |
| ----------------- |
| 30                |

---

### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

```sql
SELECT runner_id, order_id, ROUND((distance/duration * 60.0)::numeric, 2) AS speed_km_hr
FROM orders_runner
WHERE cancellation = ''
ORDER BY runner_id, order_id;
```

| runner_id | order_id | speed_km_hr |
| --------- | -------- | ----------- |
| 1         | 1        | 37.50       |
| 1         | 2        | 44.44       |
| 1         | 3        | 40.20       |
| 1         | 10       | 60.00       |
| 2         | 4        | 35.10       |
| 2         | 7        | 60.00       |
| 2         | 8        | 93.60       |
| 3         | 5        | 40.00       |

With more delivery experience, each runner’s speed shows an upward trend.

---

### 7. What is the successful delivery percentage for each runner?

```sql
SELECT runner_id,
100 * SUM(CASE WHEN cancellation = '' THEN 1 ELSE 0 END)/COUNT(*) AS successful_delivery_percentage
FROM orders_runner 
GROUP BY runner_id
ORDER BY runner_id;
```

| runner_id | successful_delivery_percentage |
| --------- | ------------------------------ |
| 1         | 100                            |
| 2         | 75                             |
| 3         | 50                             |

---

**Click [here](https://github.com/anweasha/8-Week-SQL-Challenge/blob/main/%232%20-%20Pizza%20Runner/Solutions/C.%20Ingredient%20Optimisation.md) for solutions to C. Ingredient Optimisation.**
