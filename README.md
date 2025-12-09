# Restaurant_Orders_SQL

## Objectives: 
A fictional international restaurant wants to analyze customer purchasing behavior to uncover actionable insights that will guide future menu expansion.

## **Objective 1: Explore the `menu_items` Table**

### **a. Number of items on the menu**

```sql
SELECT COUNT(*) FROM menu_items;
```

**Result:** `32 items`

### **b. Least & Most Expensive Items**

```sql
SELECT MAX(price) AS Most_expensive, MIN(price) AS Least_expensive 
FROM menu_items;
```

**Detailed listing**

```sql
SELECT 'Most Expensive' AS Category, item_name, price 
FROM menu_items
WHERE price = (SELECT MAX(price) FROM menu_items)
UNION ALL
SELECT 'Least Expensive', item_name, price 
FROM menu_items
WHERE price = (SELECT MIN(price) FROM menu_items);
```

**Result:**

| Category        | Item          | Price |
| --------------- | ------------- | ----- |
| Most Expensive  | Shrimp Scampi | 19.95 |
| Least Expensive | Edamame       | 5.00  |

### **c. Italian dishes count + least/most expensive Italian dish**

```sql
SELECT COUNT(*) 
FROM menu_items 
WHERE category = 'Italian';
```

**Least vs Most expensive (Italian only):**

```sql
(
  SELECT 'Least Expensive' AS price_type, item_name, price 
  FROM menu_items
  WHERE category = 'Italian'
  ORDER BY price ASC
  LIMIT 1
)
UNION ALL
(
  SELECT 'Most Expensive', item_name, price
  FROM menu_items
  WHERE category = 'Italian'
  ORDER BY price DESC
  LIMIT 1
);
```

### **d. Number of dishes & average price per category**

```sql
SELECT category, COUNT(*) AS number_of_dishes 
FROM menu_items
GROUP BY category;
```

```sql
SELECT category, AVG(price) AS avg_price 
FROM menu_items
GROUP BY category;
```

---

## **Objective 2: Explore the `order_details` Table**

### **a. Date range**

```sql
SELECT MIN(order_date), MAX(order_date) 
FROM order_details;
```

**Range:** `2023-01-01 to 2023-03-31`

### **b. Number of orders & number of items**

```sql
SELECT COUNT(DISTINCT order_id) FROM order_details;   -- 5370 orders
SELECT COUNT(*) FROM order_details;                   -- 12234 items ordered
```

### **c. Orders with the most items**

```sql
SELECT order_id, COUNT(item_id) AS number_of_items
FROM order_details
GROUP BY order_id
ORDER BY number_of_items DESC;
```

**Top orders:** Order IDs: `4305, 3473, 1957, 330` (each with 14 items)

### **d. Orders with more than 12 items**

```sql
WITH total_orders AS (
  SELECT order_id, COUNT(item_id) AS number_of_items
  FROM order_details
  GROUP BY order_id
  HAVING COUNT(item_id) > 12
)
SELECT COUNT(order_id) FROM total_orders;
```

**Result:** `20 orders`

---

## **Objective 3: Analyze Customer Behavior**

### **a. Join menu items with orders**

```sql
SELECT *
FROM order_details AS O
LEFT JOIN menu_items AS M 
  ON O.item_id = M.menu_item_id;
```

### **b. Least & Most Ordered Items**

```sql
SELECT item_name, category, COUNT(order_details_id) AS number_of_purchases
FROM order_details AS O
LEFT JOIN menu_items AS M ON O.item_id = M.menu_item_id
GROUP BY item_name, category
ORDER BY number_of_purchases DESC;
```

**Most ordered examples:**

* California Roll — 355
* Steak Burrito — 354
* Salmon Roll — 324

**Alternate list (item_name only):**

```sql
SELECT item_name, COUNT(order_details_id) 
FROM order_details AS O
LEFT JOIN menu_items AS M ON O.item_id = M.menu_item_id
GROUP BY item_name
ORDER BY 2 DESC;
```

### **c. Top 5 highest-spend orders**

```sql
SELECT order_id, SUM(price) AS total_spend
FROM order_details AS O
LEFT JOIN menu_items AS M ON O.item_id = M.menu_item_id
GROUP BY order_id
ORDER BY total_spend DESC
LIMIT 5;
```

**Top 5 Spend Orders:**

| Order ID | Total Spend |
| -------- | ----------- |
| 440      | 192.15      |
| 2075     | 191.05      |
| 1957     | 190.10      |
| 330      | 189.70      |
| 2675     | 185.10      |

### **d. Items inside the highest-spend order (Order 440)**

```sql
SELECT *
FROM order_details AS O
LEFT JOIN menu_items AS M ON O.item_id = M.menu_item_id
WHERE order_id = 440;
```

**Category breakdown**

```sql
SELECT category, COUNT(item_id) AS num_of_items
FROM order_details AS O
LEFT JOIN menu_items AS M ON O.item_id = M.menu_item_id
WHERE order_id = 440
GROUP BY category;
```

### **e. BONUS — Category breakdown of top 5 spend orders**

```sql
SELECT order_id, category, COUNT(item_id) AS num_of_items
FROM order_details AS O
LEFT JOIN menu_items AS M ON O.item_id = M.menu_item_id
WHERE order_id IN (440, 2075, 1957, 330, 2675)
GROUP BY order_id, category;
```
---

## **Insights**

1. **Italian dishes are the highest priced** and frequently appear in top-spend orders → customers are willing to pay more for Italian cuisine.
2. **American and Asian items are the most frequently ordered**, with rolls, burgers, and fries dominating volume.
3. Only **20 orders** had over 12 items, but these large orders drive significant revenue.
4. **High-spend orders include multiple cuisines**, showing customers prefer variety in larger/group orders.
5. The restaurant averages **2.27 items per order**, indicating many solo diners or small orders.

## **Recommendations**

1. **Expand Italian offerings** and introduce premium, high-margin dishes.
2. Add **new variations and combos** for top-selling American and Asian items (e.g., burger + fries sets, roll combos).
3. Create **group meal bundles** and multi-cuisine sampler boxes to target high-volume orders.
4. Introduce **mid-range priced items** to balance the menu and appeal to broader budgets.
5. Implement **upsell add-ons** (drinks, desserts, sides) to increase average order value.

---




