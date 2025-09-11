## A. Customer Journey

### Based off the 8 sample customers provided in the sample from the `subscriptions` table, write a brief description about each customer’s onboarding journey.

### Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

```sql
SELECT s.customer_id, s.plan_id, p.plan_name, s.start_date
FROM foodie_fi.plans AS p
JOIN foodie_fi.subscriptions AS s ON p.plan_id = s.plan_id
WHERE s.customer_id IN ('1', '2', '11', '13', '15', '16', '18', '19')
ORDER BY s.customer_id, s.plan_id;
```

| customer_id | plan_id | plan_name     | start_date |
| ----------- | ------- | ------------- | ---------- |
| 1           | 0       | trial         | 2020-08-01 |
| 1           | 1       | basic monthly | 2020-08-08 |
| 2           | 0       | trial         | 2020-09-20 |
| 2           | 3       | pro annual    | 2020-09-27 |
| 11          | 0       | trial         | 2020-11-19 |
| 11          | 4       | churn         | 2020-11-26 |
| 13          | 0       | trial         | 2020-12-15 |
| 13          | 1       | basic monthly | 2020-12-22 |
| 13          | 2       | pro monthly   | 2021-03-29 |
| 15          | 0       | trial         | 2020-03-17 |
| 15          | 2       | pro monthly   | 2020-03-24 |
| 15          | 4       | churn         | 2020-04-29 |
| 16          | 0       | trial         | 2020-05-31 |
| 16          | 1       | basic monthly | 2020-06-07 |
| 16          | 3       | pro annual    | 2020-10-21 |
| 18          | 0       | trial         | 2020-07-06 |
| 18          | 2       | pro monthly   | 2020-07-13 |
| 19          | 0       | trial         | 2020-06-22 |
| 19          | 2       | pro monthly   | 2020-06-29 |
| 19          | 3       | pro annual    | 2020-08-29 |

- **Customer 1** signed up for the free trial on 2020-08-01 and downgraded to the basic monthly plan on the 7th day of the trial (2020-08-08).
- **Customer 2** signed up for the free trial on 2020-09-20 and upgraded to the pro annual plan on the 7th day of the trial (2020-09-27).
- **Customer 11** signed up for the free trial on 2020-11-19 and cancelled the service on the 7th day of the trial (2020-11-26).
- **Customer 13** signed up for the free trial on 2020-12-15 and downgraded to the basic monthly plan on the 7th day of the trial (2020-12-22). After about 3 months, they upgraded to the pro monthly plan on 2021-03-29.
- **Customer 15** signed up for the free trial on 2020-03-17 and automatically continued with the pro monthly plan at the end of the trial on 2020-03-24. After about one month, they cancelled the service on 2020-04-29.
- **Customer 16** signed up for the free trial on 2020-05-31 and downgraded to the basic monthly plan on the 7th day of the trial (2020-06-07). After about 4 months, they upgraded to the pro annual plan on 2020-10-21.
- **Customer 18** signed up for the free trial on 2020-07-06 and automatically continued with the pro monthly plan at the end of the trial on 2020-07-13.
- **Customer 19** signed up for the free trial on 2020-06-22 and automatically continued with the pro monthly plan at the end of the trial on 2020-06-29. After 2 months, they upgraded to the pro annual plan on 2020-08-29.

---
