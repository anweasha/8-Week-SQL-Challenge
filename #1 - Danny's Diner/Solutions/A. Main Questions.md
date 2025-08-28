### 1. What is the total amount each customer spent at the restaurant?</b>

```sql
SELECT s.customer_id, SUM(m.price) AS total_spent
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```
| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

---

### 2. How many days has each customer visited the restaurant?

```sql
SELECT customer_id, COUNT(DISTINCT order_date) AS visit_count
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
```

| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4           |
| B           | 6           |
| C           | 2           |

---

### 3. What was the first item from the menu purchased by each customer?

```sql
WITH Temp AS (
  SELECT s.customer_id, s.order_date, m.product_name,
  DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS rn
  FROM dannys_diner.sales AS s
  JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
  )
SELECT DISTINCT customer_id, product_name
FROM Temp
WHERE rn = 1;
```

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

---

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT m.product_name, COUNT(*) AS purchase_freq
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY purchase_freq DESC
LIMIT 1;
```

| product_name | purchase_freq |
| ------------ | ------------- |
| ramen        | 8             |

---

### 5. Which item was the most popular for each customer?

```sql
WITH Temp AS (
  SELECT s.customer_id, m.product_name, COUNT(*) AS frequency,
  DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.customer_id, COUNT(*) DESC) AS rn
  FROM dannys_diner.sales AS s
  JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
  GROUP BY s.customer_id, m.product_name
)
SELECT customer_id, product_name, frequency
FROM Temp
WHERE rn = 1;
```

| customer_id | product_name | frequency |
| ----------- | ------------ | --------- |
| A           | ramen        | 3         |
| B           | ramen        | 2         |
| B           | curry        | 2         |
| B           | sushi        | 2         |
| C           | ramen        | 3         |

---

### 6. Which item was purchased first by the customer after they became a member?

```sql
WITH Temp AS (
  SELECT s.customer_id, s.order_date, m.product_name,
  DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS rn
  FROM dannys_diner.sales AS s
  JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
  JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id AND s.order_date >= mem.join_date
)
SELECT customer_id, product_name
FROM Temp
WHERE rn = 1;
```

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |

---

### 7. Which item was purchased just before the customer became a member?

```sql
WITH Temp AS (
  SELECT s.customer_id, s.order_date, m.product_name,
  DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rn
  FROM dannys_diner.sales AS s
  JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
  JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id AND s.order_date < mem.join_date
)
SELECT customer_id, product_name
FROM Temp
WHERE rn = 1;
```

| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | sushi        |

---

### 8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT s.customer_id, COUNT(*) AS total_items, SUM(m.price) AS total_spent
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id AND s.order_date < mem.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

| customer_id | total_items | total_spent |
| ----------- | ----------- | ----------- |
| A           | 2           | 25          |
| B           | 3           | 40          |

---

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
SELECT s.customer_id,
SUM(CASE WHEN s.product_id = 1 THEN 20 * m.price ELSE 10 * m.price END) AS points
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

| customer_id | points |
| ----------- | ------ |
| A           | 860    |
| B           | 940    |
| C           | 360    |

---

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
WITH dates AS (
  SELECT customer_id, join_date, join_date + 6 AS valid_date, '2021-01-31'::date AS end_date
  FROM dannys_diner.members
)

SELECT s.customer_id,
SUM(CASE WHEN s.order_date BETWEEN d.join_date AND d.valid_date THEN 20 * price
    WHEN s.product_id = 1 THEN 20 * price
    ELSE 10 * price END) AS points
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
JOIN dates AS d ON s.customer_id = d.customer_id AND s.order_date <= d.end_date
GROUP BY s.customer_id;
```

| customer_id | points |
| ----------- | ------ |
| A           | 1370   |
| B           | 820    |
