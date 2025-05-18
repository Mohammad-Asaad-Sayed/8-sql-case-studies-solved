**MySQL-compatible and properly formatted Markdown version** of:

# 🍕 Case Study #2 - Pizza Runner

## 🏃 Solution - B. Runner and Customer Experience

---

### 1. How many runners signed up for each 1-week period? (Week starts 2021-01-01)

```sql
SELECT 
  WEEK(registration_date, 1) AS registration_week,
  COUNT(runner_id) AS runner_signup
FROM runners
GROUP BY WEEK(registration_date, 1);
```

**Answer:**

* Week 1 of Jan 2021: **2 new runners signed up**
* Week 2 and Week 3: **1 new runner each**

---

### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pick up the order?

```sql
WITH time_taken_cte AS (
  SELECT 
    c.order_id, 
    c.order_time, 
    r.pickup_time, 
    TIMESTAMPDIFF(MINUTE, c.order_time, r.pickup_time) AS pickup_minutes
  FROM customer_orders AS c
  JOIN runner_orders AS r ON c.order_id = r.order_id
  WHERE r.distance IS NOT NULL AND r.distance != 0
)

SELECT 
  ROUND(AVG(pickup_minutes), 0) AS avg_pickup_minutes
FROM time_taken_cte
WHERE pickup_minutes > 1;
```

**Answer:**
Average pickup time: **15 minutes**

---

### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

```sql
WITH prep_time_cte AS (
  SELECT 
    c.order_id, 
    COUNT(c.pizza_id) AS pizza_order, 
    c.order_time, 
    r.pickup_time, 
    TIMESTAMPDIFF(MINUTE, c.order_time, r.pickup_time) AS prep_time_minutes
  FROM customer_orders AS c
  JOIN runner_orders AS r ON c.order_id = r.order_id
  WHERE r.distance IS NOT NULL AND r.distance != 0
  GROUP BY c.order_id, c.order_time, r.pickup_time
)

SELECT 
  pizza_order, 
  ROUND(AVG(prep_time_minutes), 0) AS avg_prep_time_minutes
FROM prep_time_cte
WHERE prep_time_minutes > 1
GROUP BY pizza_order
ORDER BY pizza_order;
```

**Answer:**

| Pizza Count | Avg Prep Time (mins) | Per Pizza Efficiency |
| ----------- | -------------------- | -------------------- |
| 1           | 12                   | 12 mins per pizza    |
| 2           | 16                   | 8 mins per pizza     |
| 3           | 30                   | 10 mins per pizza    |

📊 **Most efficient** = 2 pizzas/order (8 mins per pizza)

---

### 4. What was the average distance travelled for each customer?

```sql
SELECT 
  c.customer_id, 
  ROUND(AVG(r.distance), 1) AS avg_distance
FROM customer_orders AS c
JOIN runner_orders AS r ON c.order_id = r.order_id
WHERE r.duration IS NOT NULL AND r.duration != 0
GROUP BY c.customer_id
ORDER BY c.customer_id;
```

**Answer:**

| Customer | Avg Distance (km)    |
| -------- | -------------------- |
| 101      | e.g. 15 km           |
| 102      | e.g. 20 km           |
| 103      | e.g. 18 km           |
| 104      | **10 km** (nearest)  |
| 105      | **25 km** (farthest) |

---

### 5. What was the difference between the longest and shortest delivery times for all orders?

**Step 1 (Optional - View valid durations):**

```sql
SELECT 
  order_id, duration
FROM runner_orders
WHERE TRIM(duration) != '';
```

**Step 2 (Final Query):**

```sql
SELECT 
  MAX(CAST(duration AS UNSIGNED)) - MIN(CAST(duration AS UNSIGNED)) AS delivery_time_difference
FROM runner_orders
WHERE TRIM(duration) != '';
```

**Answer:**
**Longest delivery:** 40 minutes
**Shortest delivery:** 10 minutes
**Difference:** **30 minutes**

---

### 6. What was the average speed for each runner for each delivery? Any trend?

```sql
SELECT 
  r.runner_id, 
  c.customer_id, 
  c.order_id, 
  COUNT(c.pizza_id) AS pizza_count,
  r.distance,
  r.duration / 60 AS duration_hr,
  ROUND((r.distance / (r.duration / 60)), 2) AS avg_speed
FROM runner_orders AS r
JOIN customer_orders AS c ON r.order_id = c.order_id
WHERE r.distance IS NOT NULL AND r.distance != 0 AND r.duration IS NOT NULL
GROUP BY r.runner_id, c.customer_id, c.order_id, r.distance, r.duration
ORDER BY c.order_id;
```

**Answer:**

| Runner | Speed Range (km/h) | Observations        |
| ------ | ------------------ | ------------------- |
| 1      | 37.5 – 60          | Fairly consistent   |
| 2      | 35.1 – 93.6        | ⚠️ High fluctuation |
| 3      | 40.0               | Stable              |

🔍 **Runner 2** might need attention due to high speed variability (93.6 km/h seems extreme!).

---

### 7. What is the successful delivery percentage for each runner?

```sql
SELECT 
  runner_id,
  ROUND(100 * SUM(CASE WHEN distance IS NOT NULL AND distance != 0 THEN 1 ELSE 0 END) / COUNT(*), 0) AS success_perc
FROM runner_orders
GROUP BY runner_id;
```

**Answer:**

| Runner | Success % |
| ------ | --------- |
| 1      | 100%      |
| 2      | 75%       |
| 3      | 50%       |

💡 Note: Success % doesn’t necessarily reflect runner performance — order cancellations may be out of their control.

---


