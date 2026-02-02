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

