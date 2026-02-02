# Pizza Runner :  C . Ingredient Optimization

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

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
