## A. Pizza Metrics

### 1. How many pizzas were ordered?

```sql
SELECT COUNT(pizza_id) AS pizza_order_count
FROM orders_customer;
```

| pizza_order_count |
| ----------------- |
| 14                |

---

### 2. How many unique customer orders were made?

```sql
SELECT COUNT(DISTINCT customer_id) AS unique_customers
FROM orders_customer;
```

| unique_customers |
| ---------------- |
| 5                |

---

### 3. How many successful orders were delivered by each runner?

```sql
SELECT runner_id, COUNT(order_id) AS delivered_orders
FROM orders_runner
WHERE cancellation = ''
GROUP BY runner_id;
```

| runner_id | delivered_orders |
| --------- | ---------------- |
| 1         | 4                |
| 2         | 3                |
| 3         | 1                |

---

### 4. How many of each type of pizza was delivered?

```sql
SELECT p.pizza_name, COUNT(*) AS delivered_pizza_count
FROM orders_customer AS c
JOIN orders_runner AS r ON c.order_id = r.order_id AND r.cancellation = ''
JOIN pizza_runner.pizza_names AS p ON c.pizza_id = p.pizza_id
GROUP BY p.pizza_name
ORDER BY p.pizza_name;
```

| pizza_name | delivered_pizza_count |
| ---------- | --------------------- |
| Meatlovers | 9                     |
| Vegetarian | 3                     |

---

### 5. How many Vegetarian and Meatlovers were ordered by each customer?

```sql
SELECT c.customer_id, p.pizza_name, COUNT(*) AS pizza_count
FROM orders_customer AS c
JOIN pizza_runner.pizza_names AS p ON c.pizza_id = p.pizza_id
GROUP BY c.customer_id, p.pizza_name
ORDER BY c.customer_id, p.pizza_name;
```

| customer_id | pizza_name | pizza_count |
| ----------- | ---------- | ----------- |
| 101         | Meatlovers | 2           |
| 101         | Vegetarian | 1           |
| 102         | Meatlovers | 2           |
| 102         | Vegetarian | 1           |
| 103         | Meatlovers | 3           |
| 103         | Vegetarian | 1           |
| 104         | Meatlovers | 3           |
| 105         | Vegetarian | 1           |

---

### 6. What was the maximum number of pizzas delivered in a single order?

```sql
SELECT c.order_id, COUNT(c.pizza_id) AS pizza_count
FROM orders_customer AS c
JOIN orders_runner AS r ON c.order_id = r.order_id AND r.cancellation = ''
GROUP BY c.order_id
ORDER BY pizza_count DESC
LIMIT 1;
```

| order_id | pizza_count |
| -------- | ----------- |
| 4        | 3           |

---

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
SELECT customer_id,
SUM(CASE WHEN exclusions != '' OR extras != '' THEN 1 ELSE 0 END) AS at_least_1_change,
SUM(CASE WHEN exclusions = '' AND extras = '' THEN 1 ELSE 0 END) AS no_change
FROM orders_customer AS c
JOIN orders_runner AS r ON c.order_id = r.order_id AND r.cancellation = ''
GROUP BY customer_id;
```

| customer_id | at_least_1_change | no_change |
| ----------- | ----------------- | --------- |
| 101         | 0                 | 2         |
| 102         | 0                 | 3         |
| 103         | 3                 | 0         |
| 104         | 2                 | 1         |
| 105         | 1                 | 0         |

---

### 8. How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT COUNT(*) AS pizza_count
FROM orders_customer AS c
JOIN orders_runner AS r ON c.order_id = r.order_id AND r.cancellation = ''
WHERE c.exclusions != '' AND c.extras != '';
```

| pizza_count |
| ----------- |
| 1           |

---

### 9. What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT EXTRACT(HOUR FROM order_time) AS hour_of_the_day, COUNT(pizza_id) AS pizza_count
FROM orders_customer
GROUP BY hour_of_the_day
ORDER BY hour_of_the_day;
```

| hour_of_the_day | pizza_count |
| --------------- | ----------- |
| 11              | 1           |
| 13              | 3           |
| 18              | 3           |
| 19              | 1           |
| 21              | 3           |
| 23              | 3           |

---

### 10. What was the volume of orders for each day of the week?

```sql
SELECT TO_CHAR(order_time, 'Day') AS day_of_the_week, COUNT(pizza_id) AS pizza_order_count
FROM orders_customer
GROUP BY day_of_the_week;
```

| day_of_the_week | pizza_order_count |
| --------------- | ----------------- |
| Saturday        | 5                 |
| Thursday        | 3                 |
| Friday          | 1                 |
| Wednesday       | 5                 |

---
