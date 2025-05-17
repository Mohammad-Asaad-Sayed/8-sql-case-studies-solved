
---

# 📅 Week 1: Danny’s Diner

!\[Alt text: Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.]

Danny’s Diner is in need of your assistance to help the restaurant stay afloat. The restaurant has captured some very basic data from their few months of operation but has no idea how to use their data to help run the business.
**[Link to the challenge](#)**

---

## 🗃️ DATA

```sql
CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;

CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');

CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
```

---

## ❓ Questions

### 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT
  s.customer_id,
  SUM(m.price) as "Total ($)"
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
GROUP BY s.customer_id;
```

| customer\_id | Total (\$) |
| :----------: | :--------: |
|       B      |     74     |
|       C      |     36     |
|       A      |     76     |

---

### 2. How many days has each customer visited the restaurant?

```sql
SELECT 
  customer_id,
  COUNT(DISTINCT order_date) AS "Visited times"
FROM dannys_diner.sales
GROUP BY customer_id;
```

| customer\_id | Visited times |
| :----------: | :-----------: |
|       A      |       4       |
|       B      |       6       |
|       C      |       2       |

---

### 3. What was the first item from the menu purchased by each customer?

```sql
SELECT 
  DISTINCT sales.customer_id,
  menu.product_name
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id
WHERE sales.order_date = '2021-01-01'
LIMIT 3;
```

| customer\_id | product\_name |
| :----------: | :-----------: |
|       C      |     ramen     |
|       B      |     curry     |
|       A      |     sushi     |

---

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT 
  m.product_name,
  COUNT(s.product_id) as times
FROM dannys_diner.sales AS S
INNER JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
GROUP BY m.product_name, s.product_id
ORDER by times DESC
LIMIT 1;
```

| product\_name | times |
| :-----------: | :---: |
|     ramen     |   8   |

---

### 5. Which item was the most popular for each customer?

```sql
WITH temp AS (
  SELECT customer_id, product_name
  FROM dannys_diner.sales AS s
  JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
)
SELECT customer_id, product_name, COUNT(product_name) 
FROM temp
GROUP BY customer_id, product_name
ORDER BY count(product_name) DESC;
```

| customer\_id | product\_name | count |
| :----------: | :-----------: | :---: |
|       C      |     ramen     |   3   |
|       A      |     ramen     |   3   |
|       B      |     curry     |   2   |
|       B      |     sushi     |   2   |
|       B      |     ramen     |   2   |
|       A      |     curry     |   2   |
|       A      |     sushi     |   1   |

---

### 6. Which item was purchased first by the customer after they became a member?

```sql
SELECT DISTINCT ON (order_date) s.customer_id, order_date, product_name, join_date
FROM dannys_diner.sales AS s
JOIN dannys_diner.members AS m ON s.customer_id = m.customer_id
JOIN dannys_diner.menu AS men ON s.product_id = men.product_id
WHERE join_date <= order_date
ORDER BY order_date
LIMIT 2;
```

| customer\_id |        order\_date       | product\_name |        join\_date        |
| :----------: | :----------------------: | :-----------: | :----------------------: |
|       A      | 2021-01-07T00:00:00.000Z |     curry     | 2021-01-07T00:00:00.000Z |
|       A      | 2021-01-10T00:00:00.000Z |     ramen     | 2021-01-07T00:00:00.000Z |

---

### 7. Which item was purchased just before the customer became a member?

```sql
SELECT DISTINCT ON (s.customer_id) s.customer_id, order_date, product_name, join_date 
FROM dannys_diner.sales AS s
JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id
JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
WHERE join_date > order_date;
```

| customer\_id |        order\_date       | product\_name |        join\_date        |
| :----------: | :----------------------: | :-----------: | :----------------------: |
|       A      | 2021-01-01T00:00:00.000Z |     sushi     | 2021-01-07T00:00:00.000Z |
|       B      | 2021-01-04T00:00:00.000Z |     sushi     | 2021-01-09T00:00:00.000Z |

---

### 8. What is the total items and amount spent for each member before they became a member?

```sql
WITH temp AS (
  SELECT s.customer_id, s.product_id, price
  FROM dannys_diner.sales AS s
  JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
  JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id
  WHERE order_date < join_date
)
SELECT customer_id AS Members, COUNT(product_id) AS "Products bought", SUM(price)
FROM temp
GROUP BY customer_id;
```

| members | Products bought | sum |
| :-----: | :-------------: | :-: |
|    B    |        3        |  40 |
|    A    |        2        |  25 |

---

### 9. If each \$1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
WITH temp AS (
  SELECT customer_id, s.product_id, product_name, price
  FROM dannys_diner.sales AS s
  JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
)
SELECT DISTINCT ON (customer_id) customer_id, 
  SUM(CASE WHEN product_id = 1 THEN 20 * price ELSE 10 * price END) AS point
FROM temp
GROUP BY temp.customer_id;
```

| customer\_id | point |
| :----------: | :---: |
|       A      |  860  |
|       B      |  940  |
|       C      |  360  |

---

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
SELECT s.customer_id, 
  SUM(CASE WHEN ABS(order_date - join_date) <= 7 THEN price * 20 ELSE price * 10 END) AS "points earned"
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id
WHERE EXTRACT(MONTH FROM order_date) < 2
GROUP BY s.customer_id;
```

| customer\_id | points earned |
| :----------: | :-----------: |
|       A      |      1520     |
|       B      |      1090     |

---

