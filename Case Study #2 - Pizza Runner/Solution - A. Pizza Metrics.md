Here's a **complete and properly formatted Markdown document** for **Case Study #2 - Pizza Runner 🍕**, with **MySQL-compatible SQL syntax** wherever applicable.

---

# 🍕 Case Study #2 - Pizza Runner

## 🍝 Solution - A. Pizza Metrics

---

### 1. How many pizzas were ordered?

```sql
SELECT COUNT(*) AS pizza_order_count
FROM customer_orders;
```

**Answer:**
**Total of 14 pizzas** were ordered.

---

### 2. How many unique customer orders were made?

```sql
SELECT COUNT(DISTINCT order_id) AS unique_order_count
FROM customer_orders;
```

**Answer:**
There are **10 unique customer orders**.

---

### 3. How many successful orders were delivered by each runner?

```sql
SELECT 
  runner_id, 
  COUNT(order_id) AS successful_orders
FROM runner_orders
WHERE distance IS NOT NULL AND distance != 0
GROUP BY runner_id;
```

**Answer:**

| runner\_id | successful\_orders |
| :--------: | :----------------: |
|      1     |          4         |
|      2     |          3         |
|      3     |          1         |

---

### 4. How many of each type of pizza was delivered?

```sql
SELECT 
  p.pizza_name, 
  COUNT(c.pizza_id) AS delivered_pizza_count
FROM customer_orders AS c
JOIN runner_orders AS r ON c.order_id = r.order_id
JOIN pizza_names AS p ON c.pizza_id = p.pizza_id
WHERE r.distance IS NOT NULL AND r.distance != 0
GROUP BY p.pizza_name;
```

**Answer:**

| pizza\_name | delivered\_pizza\_count |
| :---------: | :---------------------: |
|  Meatlovers |            9            |
|  Vegetarian |            3            |

---

### 5. How many Vegetarian and Meatlovers pizzas were ordered by each customer?

```sql
SELECT 
  c.customer_id, 
  p.pizza_name, 
  COUNT(p.pizza_name) AS order_count
FROM customer_orders AS c
JOIN pizza_names AS p ON c.pizza_id = p.pizza_id
GROUP BY c.customer_id, p.pizza_name
ORDER BY c.customer_id;
```

**Answer:**

| customer\_id | pizza\_name | order\_count |
| :----------: | :---------: | :----------: |
|      101     |  Meatlovers |       2      |
|      101     |  Vegetarian |       1      |
|      102     |  Meatlovers |       2      |
|      102     |  Vegetarian |       2      |
|      103     |  Meatlovers |       3      |
|      103     |  Vegetarian |       1      |
|      104     |  Meatlovers |       1      |
|      105     |  Vegetarian |       1      |

---

### 6. What was the maximum number of pizzas delivered in a single order?

```sql
WITH pizza_count_cte AS (
  SELECT 
    c.order_id, 
    COUNT(c.pizza_id) AS pizza_per_order
  FROM customer_orders AS c
  JOIN runner_orders AS r ON c.order_id = r.order_id
  WHERE r.distance IS NOT NULL AND r.distance != 0
  GROUP BY c.order_id
)
SELECT 
  MAX(pizza_per_order) AS pizza_count
FROM pizza_count_cte;
```

**Answer:**
Maximum number of pizzas delivered in a single order is **3**.

---

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
SELECT 
  c.customer_id,
  SUM(CASE WHEN TRIM(c.exclusions) <> '' OR TRIM(c.extras) <> '' THEN 1 ELSE 0 END) AS at_least_1_change,
  SUM(CASE WHEN TRIM(c.exclusions) = '' AND TRIM(c.extras) = '' THEN 1 ELSE 0 END) AS no_change
FROM customer_orders AS c
JOIN runner_orders AS r ON c.order_id = r.order_id
WHERE r.distance IS NOT NULL AND r.distance != 0
GROUP BY c.customer_id
ORDER BY c.customer_id;
```

**Answer:**

| customer\_id | at\_least\_1\_change | no\_change |
| :----------: | :------------------: | :--------: |
|      101     |           0          |      3     |
|      102     |           0          |      4     |
|      103     |           3          |      1     |
|      104     |           1          |      0     |
|      105     |           1          |      0     |

---

### 8. How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT  
  COUNT(*) AS pizza_count_w_exclusions_extras
FROM customer_orders AS c
JOIN runner_orders AS r ON c.order_id = r.order_id
WHERE r.distance IS NOT NULL 
  AND r.distance >= 1 
  AND TRIM(exclusions) <> '' 
  AND TRIM(extras) <> '';
```

**Answer:**
Only **1 pizza** was delivered that had **both exclusions and extras**.

---

### 9. What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT 
  HOUR(order_time) AS hour_of_day, 
  COUNT(order_id) AS pizza_count
FROM customer_orders
GROUP BY HOUR(order_time)
ORDER BY hour_of_day;
```

**Answer:**
Highest order volumes were at **13:00, 18:00, and 21:00**.
Lowest volumes were at **11:00, 19:00, and 23:00**.

---

### 10. What was the volume of orders for each day of the week?

```sql
SELECT 
  DAYNAME(order_time) AS day_of_week,
  COUNT(order_id) AS total_pizzas_ordered
FROM customer_orders
GROUP BY DAYNAME(order_time);
```

**Answer:**

| day\_of\_week | total\_pizzas\_ordered |
| :-----------: | :--------------------: |
|     Friday    |            5           |
|     Monday    |            5           |
|    Saturday   |            3           |
|     Sunday    |            1           |

---


