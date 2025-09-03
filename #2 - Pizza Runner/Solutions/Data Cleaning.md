## Data Cleaning

### Table 2: `customer_orders`

``` sql
DROP TABLE IF EXISTS orders_customer;

CREATE TEMP TABLE orders_customer AS
SELECT order_id, customer_id, pizza_id, 
CASE WHEN exclusions = 'null' THEN '' ELSE exclusions END AS exclusions,
CASE WHEN extras IS NULL OR extras = 'null' THEN '' ELSE extras END AS extras,
order_time
FROM pizza_runner.customer_orders;

SELECT * FROM orders_customer;
```

Result set:

| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
| -------- | ----------- | -------- | ---------- | ------ | ------------------- |
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02 |
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52 |
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23 |
| 3        | 102         | 2        |            |        | 2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46 |
| 5        | 104         | 1        |            | 1      | 2020-01-08 21:00:29 |
| 6        | 101         | 2        |            |        | 2020-01-08 21:03:13 |
| 7        | 105         | 2        |            | 1      | 2020-01-08 21:20:29 |
| 8        | 102         | 1        |            |        | 2020-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10       | 104         | 1        |            |        | 2020-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |


---

### Table 3: `runner_orders`


``` sql
DROP TABLE IF EXISTS runner_orders;

CREATE TEMP TABLE orders_runner AS
SELECT order_id, runner_id,
CAST(CASE WHEN pickup_time = 'null' THEN NULL ELSE pickup_time END AS timestamp) AS pickup_time,
CAST(CASE WHEN distance = 'null' THEN NULL ELSE TRIM ('km' FROM distance) END AS float) AS distance,
CAST(CASE WHEN duration = 'null' THEN NULL ELSE SUBSTRING(duration, 1, 2) END AS int) AS duration,
CASE WHEN cancellation IS NULL OR cancellation = 'null' THEN '' ELSE cancellation END AS cancellation
FROM pizza_runner.runner_orders;
 
SELECT * FROM orders_runner;
```

Result set:

| order_id | runner_id | pickup_time         | distance | duration | cancellation            |
| -------- | --------- | ------------------- | -------- | -------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4     | 20       |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       |                         |
| 6        | 3         |       null          |  null    |    null  | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       |                         |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       |                         |
| 9        | 2         |           null      |    null  |  null    | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       |                         |
