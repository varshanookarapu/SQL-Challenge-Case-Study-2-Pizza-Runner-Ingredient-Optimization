# Pizza Runner :  C . Ingredient Optimization

## Prequsite

```sql
--Cleaning Customer Orders Table Exclustions and Extras Columns - updating blank vlaues to nulls
-- Checking for null string value, blank values and actual null and replacing them all with nulls

UPDATE customer_orders
SET exclusions = null
WHERE exclusions IS null OR TRIM(exclusions) = '' OR exclusions = 'null';
   

UPDATE customer_orders
SET extras = null
WHERe extras IS NULL  OR TRIM(extras)='' OR extras  = 'null';

-- Checking Duplicates 
SELECT order_id,customer_id,pizza_id,exclusions,extras,order_time, COUNT(*)
FROM customer_orders
GROUP BY order_id,customer_id,order_time,pizza_id,exclusions,extras
HAVING COUNT(*) > 1;


--Deleting the duplicate records
--ctid → a unique physical identifier for each row  in PostgresSQL

DELETE FROM customer_orders 
WHERE ctid NOT IN ( SELECT MIN(ctid) FROM customer_orders GROUP BY order_id,customer_id,order_time,pizza_id,exclusions,extras ) ;
            
-- Final cleaned customer_orders Table
SELECT * FROM customer_orders ORDER BY order_id;
```


**Question 1:** What are the standard ingredients for each pizza?

---

## SQL Code

```sql
WITH normalized_pizza_recipes AS
(
SELECT pizza_id, CAST (TRIM(topping) AS INTEGER) as topping_id
FROM pizza_recipes ,
  UNNEST(STRING_TO_ARRAY (toppings, ',')) AS  topping
) 


--Normalizing the Table

SELECT pizza_id,npr.topping_id,topping_name 
 FROM normalized_pizza_recipes npr 
LEFT JOIN pizza_toppings pt ON
 npr.topping_id = pt.topping_id
 ORDER BY pizza_id,topping_id;


-- De-normalizing the table
SELECT pizza_id, 
STRING_AGG(npr.topping_id :: TEXT , ',' ORDER BY npr.topping_id ) as topping_id ,
STRING_AGG(topping_name , ' , ' ORDER BY npr.topping_id) as topping_name
FROM normalized_pizza_recipes npr 
LEFT JOIN pizza_toppings pt ON
npr.topping_id = pt.topping_id
GROUP BY npr.pizza_id
ORDER BY npr.pizza_id

```
**Normalized Table** 

| pizza_id | topping_id | topping_name |
| -------- | ---------- | ------------ |
| 1        | 1          | Bacon        |
| 1        | 2          | BBQ Sauce    |
| 1        | 3          | Beef         |
| 1        | 4          | Cheese       |
| 1        | 5          | Chicken      |
| 1        | 6          | Mushrooms    |
| 1        | 8          | Pepperoni    |
| 1        | 10         | Salami       |
| 2        | 4          | Cheese       |
| 2        | 6          | Mushrooms    |
| 2        | 7          | Onions       |
| 2        | 9          | Peppers      |
| 2        | 11         | Tomatoes     |
| 2        | 12         | Tomato Sauce |

---
**Denormalized Table**

| pizza_id | topping_id       | topping_name                                                                 |
| -------- | ---------------- | ---------------------------------------------------------------------------- |
| 1        | 1,2,3,4,5,6,8,10 | Bacon , BBQ Sauce , Beef , Cheese , Chicken , Mushrooms , Pepperoni , Salami |
| 2        | 4,6,7,9,11,12    | Cheese , Mushrooms , Onions , Peppers , Tomatoes , Tomato Sauce              |

---


**Question 2:** What was the most commonly added extra?

---

## SQL Code

```sql
WITH normalized_pizza_recipes AS
(
SELECT pizza_id, CAST (TRIM(topping) AS INTEGER) as topping_id
FROM pizza_recipes ,
  UNNEST(STRING_TO_ARRAY (toppings, ',')) AS  topping
), 


npr_pt AS
(
SELECT pizza_id,npr.topping_id,topping_name 
FROM normalized_pizza_recipes npr 
LEFT JOIN pizza_toppings pt ON
npr.topping_id = pt.topping_id
ORDER BY pizza_id,topping_id
)
,

nco AS 
(  
SELECT order_id,customer_id,pizza_id, CAST(TRIM(extra) AS INTEGER) as extras
FROM customer_orders ,
UNNEST(STRING_TO_ARRAY(extras,',')) AS extra
ORDER BY order_id
)


SELECT extras,COUNT(*)  as max_extras,topping_name  FROM nco 
LEFT JOIN npr_pt ON
nco.extras = npr_pt.topping_id
GROUP BY extras,npr_pt.topping_name
ORDER BY max_extras DESC 
LIMIT 1
```

