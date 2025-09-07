## E. Bonus Questions

### If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an `INSERT` statement to demonstrate what would happen if a new `Supreme` pizza with all the toppings was added to the Pizza Runner menu?

```sql
INSERT INTO pizza_names
  ("pizza_id", "pizza_name")
VALUES
  (3, 'Supreme');

SELECT * FROM pizza_names;
```

| pizza_id | pizza_name |
| -------- | ---------- |
| 1        | Meatlovers |
| 2        | Vegetarian |
| 3        | Supreme    |


```sql
INSERT INTO pizza_recipes
  ("pizza_id", "toppings")
VALUES
  (3, '1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12');

SELECT * FROM pizza_recipes;
```

| pizza_id | toppings                              |
| -------- | ------------------------------------- |
| 1        | 1, 2, 3, 4, 5, 6, 8, 10               |
| 2        | 4, 6, 7, 9, 11, 12                    |
| 3        | 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12 |

---
