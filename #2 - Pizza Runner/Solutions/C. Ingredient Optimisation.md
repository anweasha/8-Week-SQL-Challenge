## C. Ingredient Optimisation

### 1. What are the standard ingredients for each pizza?

```sql
SELECT pn.pizza_name,
STRING_AGG(pt.topping_name, ', ' ORDER BY pt.topping_id) AS standard_ingredients
FROM pizza_runner.pizza_recipes AS pr
JOIN pizza_runner.pizza_names AS pn ON pr.pizza_id = pn.pizza_id
JOIN LATERAL UNNEST(STRING_TO_ARRAY(pr.toppings, ', ')) AS t(tid) ON TRUE
JOIN pizza_runner.pizza_toppings AS pt ON pt.topping_id = t.tid::int
GROUP BY pn.pizza_name;
```

| pizza_name | standard_ingredients                                                                                           |
| ---------- | -------------------------------------------------------------------------------------------------------------- |
| Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami                                          |
| Supreme    | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Onions, Pepperoni, Peppers, Salami, Tomatoes, Tomato Sauce |
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce                                                     |

---

### 2. What was the most commonly added extra?

```sql
SELECT pt.topping_name, COUNT(*) AS extras_count
FROM orders_customer AS c
JOIN LATERAL UNNEST(STRING_TO_ARRAY(c.extras, ', ')) AS t(tid) ON TRUE
JOIN pizza_runner.pizza_toppings AS pt ON pt.topping_id = t.tid::int
GROUP BY pt.topping_name
ORDER BY extras_count DESC
LIMIT 1;
```

| topping_name | extras_count |
| ------------ | ------------ |
| Bacon        | 4            |

---

### 3. What was the most common exclusion?

```sql
SELECT pt.topping_name, COUNT(*) AS exclusions_count
FROM orders_customer AS c
JOIN LATERAL UNNEST(STRING_TO_ARRAY(c.exclusions, ', ')) AS t(tid) ON TRUE
JOIN pizza_runner.pizza_toppings AS pt ON pt.topping_id = t.tid::int
GROUP BY pt.topping_name
ORDER BY exclusions_count DESC
LIMIT 1;
```

| topping_name | exclusions_count |
| ------------ | ---------------- |
| Cheese       | 4                |

---

### 4. Generate an order item for each record in the `customers_orders` table in the format of one of the following:
- `Meat Lovers`
- `Meat Lovers - Exclude Beef`
- `Meat Lovers - Extra Bacon`
- `Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers`

```sql
SELECT c.order_id,
pn.pizza_name || COALESCE(' - Exclude ' || e.excl_list, '') || COALESCE(' - Extra ' || x.extra_list, '') AS order_item
FROM orders_customer AS c
JOIN pizza_runner.pizza_names AS pn ON c.pizza_id = pn.pizza_id

LEFT JOIN LATERAL (
SELECT STRING_AGG(pt.topping_name, ', ' ORDER BY pt.topping_id) AS excl_list
FROM UNNEST(STRING_TO_ARRAY(c.exclusions, ', ')) AS t(tid)
JOIN pizza_runner.pizza_toppings AS pt ON pt.topping_id = t.tid::int
) AS e ON TRUE

LEFT JOIN LATERAL (
SELECT STRING_AGG(pt.topping_name, ', ' ORDER BY pt.topping_id) AS extra_list
FROM UNNEST(STRING_TO_ARRAY(c.extras, ', ')) AS t(tid)
JOIN pizza_runner.pizza_toppings AS pt ON pt.topping_id = t.tid::int
) AS x ON TRUE

ORDER BY c.order_id, pn.pizza_name, (c.exclusions = '') DESC;
-- added (c.exclusions = '') DESC for item no. 1 in order 10 to appear first
```

| order_id | order_item                                                      |
| -------- | --------------------------------------------------------------- |
| 1        | Meatlovers                                                      |
| 2        | Meatlovers                                                      |
| 3        | Meatlovers                                                      |
| 3        | Vegetarian                                                      |
| 4        | Meatlovers - Exclude Cheese                                     |
| 4        | Meatlovers - Exclude Cheese                                     |
| 4        | Vegetarian - Exclude Cheese                                     |
| 5        | Meatlovers - Extra Bacon                                        |
| 6        | Vegetarian                                                      |
| 7        | Vegetarian - Extra Bacon                                        |
| 8        | Meatlovers                                                      |
| 9        | Meatlovers - Exclude Cheese - Extra Bacon, Chicken              |
| 10       | Meatlovers                                                      |
| 10       | Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |

---

### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the `customer_orders` table and add a `2x` in front of any relevant ingredients
- For example: `"Meat Lovers: 2xBacon, Beef, ... , Salami"`

```sql
WITH c AS (
  SELECT order_id, pizza_id, exclusions, extras,
  ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY order_time, pizza_id) AS item_no
  FROM orders_customer
),

Base AS (
  SELECT c.order_id, c.item_no, c.pizza_id, t.tid
  FROM c
  JOIN pizza_runner.pizza_recipes AS pr ON c.pizza_id = pr.pizza_id
  CROSS JOIN LATERAL UNNEST(STRING_TO_ARRAY(pr.toppings, ', ')::int[]) AS t(tid)
  LEFT JOIN LATERAL (SELECT STRING_TO_ARRAY(c.exclusions, ', ')::int[] AS arr) e ON TRUE
  WHERE e.arr IS NULL OR NOT t.tid = ANY (e.arr)
  
  UNION ALL
  
  SELECT c.order_id, c.item_no, c.pizza_id, x.tid
  FROM c
  CROSS JOIN LATERAL UNNEST(STRING_TO_ARRAY(c.extras, ', ')::int[]) AS x(tid)
),

Final AS (
  SELECT order_id, item_no, pizza_id, tid, COUNT(*) AS qty
  FROM Base
  GROUP BY order_id, item_no, pizza_id, tid
)

SELECT f.order_id,
pn.pizza_name || ': ' || STRING_AGG(CASE WHEN f.qty = 2 THEN '2x' || pt.topping_name ELSE pt.topping_name END, ', ' ORDER BY pt.topping_id) AS ingredient_list
FROM Final AS f
JOIN pizza_runner.pizza_names AS pn ON f.pizza_id = pn.pizza_id
JOIN pizza_runner.pizza_toppings AS pt ON pt.topping_id = f.tid
GROUP BY f.order_id, f.item_no, pn.pizza_name
ORDER BY f.order_id, f.item_no;
```

| order_id | ingredient_list                                                                     |
| -------- | ----------------------------------------------------------------------------------- |
| 1        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 2        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 3        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 3        | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce              |
| 4        | Meatlovers: Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami           |
| 4        | Meatlovers: Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami           |
| 4        | Vegetarian: Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce                      |
| 5        | Meatlovers: 2xBacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 6        | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce              |
| 7        | Vegetarian: Bacon, Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce       |
| 8        | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 9        | Meatlovers: 2xBacon, BBQ Sauce, Beef, 2xChicken, Mushrooms, Pepperoni, Salami       |
| 10       | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 10       | Meatlovers: 2xBacon, Beef, 2xCheese, Chicken, Pepperoni, Salami                     |

---

### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

```sql
WITH c AS (
  SELECT c.order_id, c.pizza_id, c.exclusions, c.extras
  FROM orders_customer AS c
  JOIN orders_runner AS r ON c.order_id = r.order_id AND r.cancellation = ''
),

Base AS (
  SELECT c.order_id, c.pizza_id, t.tid
  FROM c
  JOIN pizza_runner.pizza_recipes AS pr ON c.pizza_id = pr.pizza_id
  CROSS JOIN LATERAL UNNEST(STRING_TO_ARRAY(pr.toppings, ', ')::int[]) AS t(tid)
  LEFT JOIN LATERAL (SELECT STRING_TO_ARRAY(c.exclusions, ', ')::int[] AS arr) e ON TRUE
  WHERE e.arr IS NULL OR NOT t.tid = ANY (e.arr)
  
  UNION ALL
  
  SELECT c.order_id, c.pizza_id, x.tid
  FROM c
  CROSS JOIN LATERAL UNNEST(STRING_TO_ARRAY(c.extras, ', ')::int[]) AS x(tid)
)

SELECT pt.topping_name, COUNT(*) AS quantity
FROM Base AS b
JOIN pizza_runner.pizza_toppings AS pt ON pt.topping_id = b.tid
GROUP BY pt.topping_name
ORDER BY quantity DESC, pt.topping_name;
```

| topping_name | quantity |
| ------------ | -------- |
| Bacon        | 12       |
| Mushrooms    | 11       |
| Cheese       | 10       |
| Beef         | 9        |
| Chicken      | 9        |
| Pepperoni    | 9        |
| Salami       | 9        |
| BBQ Sauce    | 8        |
| Onions       | 3        |
| Peppers      | 3        |
| Tomato Sauce | 3        |
| Tomatoes     | 3        |

---

**Click [here](https://github.com/anweasha/8-Week-SQL-Challenge/blob/main/%232%20-%20Pizza%20Runner/Solutions/D.%20Pricing%20and%20Ratings.md) for solutions to D. Pricing and Ratings.**
