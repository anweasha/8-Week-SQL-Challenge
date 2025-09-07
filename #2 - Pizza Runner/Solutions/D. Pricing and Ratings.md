## D. Pricing and Ratings

### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

```sql
SELECT SUM(CASE WHEN c.pizza_id = 1 THEN 12 ELSE 10 END) AS earnings
FROM orders_customer AS c
JOIN orders_runner AS r ON c.order_id = r.order_id AND r.cancellation = '';
```

| earnings |
| -------- |
| 138      |

---

### 2. What if there was an additional $1 charge for any pizza extras?
- Add cheese is $1 extra

```sql
SELECT SUM((CASE WHEN c.pizza_id = 1 THEN 12 ELSE 10 END)
  + COALESCE(cardinality(STRING_TO_ARRAY(c.extras, ', ')), 0)) AS earnings
FROM orders_customer AS c
JOIN orders_runner AS r ON c.order_id = r.order_id AND r.cancellation = '';
```

| earnings |
| -------- |
| 142      |

---

### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

```sql
DROP TABLE IF EXISTS runner_rating;
CREATE TABLE runner_rating (
  "order_id" INTEGER,
  "rating" INTEGER
);

INSERT INTO runner_rating
  ("order_id", "rating")
VALUES
  ('1', '1'),
  ('2', '3'),
  ('3', '2'),
  ('4', '1'),
  ('5', '2'),
  ('7', '4'),
  ('8', '5'),
  ('10', '4');

SELECT * FROM runner_rating;
```

| order_id | rating |
| -------- | ------ |
| 1        | 1      |
| 2        | 3      |
| 3        | 2      |
| 4        | 1      |
| 5        | 2      |
| 7        | 4      |
| 8        | 5      |
| 10       | 4      |

---

### 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
- `customer_id`
- `order_id`
- `runner_id`
- `rating`
- `order_time`
- `pickup_time`
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas

```sql
SELECT c.customer_id, c.order_id, r.runner_id, rr.rating, c.order_time, r.pickup_time, 
ROUND(EXTRACT(EPOCH FROM (r.pickup_time - c.order_time)/60)::numeric, 2) AS time_diff,
r.duration,
ROUND((r.distance/r.duration * 60)::numeric, 2) AS avg_speed,
COUNT(c.pizza_id) AS pizza_count
FROM orders_customer AS c
JOIN orders_runner AS r ON c.order_id = r.order_id
JOIN runner_rating AS rr ON c.order_id = rr.order_id
GROUP BY 1, 2, 3, 4, 5, 6, 8, r.distance
ORDER BY 1, 2;
```

| customer_id | order_id | runner_id | rating | order_time          | pickup_time         | time_diff | duration | avg_speed | pizza_count |
| ----------- | -------- | --------- | ------ | ------------------- | ------------------- | --------- | -------- | --------- | ----------- |
| 101         | 1        | 1         | 1      | 2020-01-01 18:05:02 | 2020-01-01 18:15:34 | 10.53     | 32       | 37.50     | 1           |
| 101         | 2        | 1         | 3      | 2020-01-01 19:00:52 | 2020-01-01 19:10:54 | 10.03     | 27       | 44.44     | 1           |
| 102         | 3        | 1         | 2      | 2020-01-02 23:51:23 | 2020-01-03 00:12:37 | 21.23     | 20       | 40.20     | 2           |
| 102         | 8        | 2         | 5      | 2020-01-09 23:54:33 | 2020-01-10 00:15:02 | 20.48     | 15       | 93.60     | 1           |
| 103         | 4        | 2         | 1      | 2020-01-04 13:23:46 | 2020-01-04 13:53:03 | 29.28     | 40       | 35.10     | 3           |
| 104         | 5        | 3         | 2      | 2020-01-08 21:00:29 | 2020-01-08 21:10:57 | 10.47     | 15       | 40.00     | 1           |
| 104         | 10       | 1         | 4      | 2020-01-11 18:34:49 | 2020-01-11 18:50:20 | 15.52     | 10       | 60.00     | 2           |
| 105         | 7        | 2         | 4      | 2020-01-08 21:20:29 | 2020-01-08 21:30:45 | 10.27     | 25       | 60.00     | 1           |

---

### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

```sql
WITH Temp AS (
 SELECT 0.30 * SUM(distance) AS runners_payment
  FROM orders_runner
  WHERE cancellation = ''
)

SELECT SUM(CASE WHEN c.pizza_id = 1 THEN 12 ELSE 10 END)
  - (SELECT runners_payment FROM Temp) AS leftover_earnings
FROM orders_customer AS c
JOIN orders_runner AS r ON c.order_id = r.order_id AND r.cancellation = '';
```

| leftover_earnings |
| ----------------- |
| 94.44             |

---

**Click [here](https://github.com/anweasha/8-Week-SQL-Challenge/blob/main/%232%20-%20Pizza%20Runner/Solutions/E.%20Bonus%20Questions.md) for solutions to E. Bonus Questions.**
