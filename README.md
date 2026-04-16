# pizza-sales-sql-analysis
I just completed an end-to-end SQL Data Analysis project on a pizza restaurant's sales database using MySQL Workbench.


PIZZA SALES DATA ANALYSIS USING SQL

-- 1. Retrieve the total number of orders placed
SELECT COUNT(order_id) AS total_orders
FROM orders;

-- 2. Calculate the total revenue generated from pizza sales
SELECT 
    SUM(order_details.quantity * pizzas.price) AS total_revenue
FROM order_details
JOIN pizzas 
ON order_details.pizza_id = pizzas.pizza_id;

-- 3. Identify the highest-priced pizza
SELECT 
    pizza_types.name,
    pizzas.price
FROM pizzas
JOIN pizza_types
ON pizzas.pizza_type_id = pizza_types.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;
-- 4. Identify the most common pizza size ordered
SELECT 
    pizzas.size,
    COUNT(order_details.order_details_id) AS order_count
FROM order_details
JOIN pizzas
ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizzas.size
ORDER BY order_count DESC
LIMIT 1;

-- 5. List the top 5 most ordered pizza types along with their quantities
SELECT 
    pizza_types.name,
    SUM(order_details.quantity) AS total_quantity
FROM order_details
JOIN pizzas
ON order_details.pizza_id = pizzas.pizza_id
JOIN pizza_types
ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY pizza_types.name
ORDER BY total_quantity DESC
LIMIT 5;


/* =====================================================
   INTERMEDIATE ANALYSIS
===================================================== */

-- 6. Join tables to find the total quantity of each pizza category ordered
SELECT 
    pizza_types.category,
    SUM(order_details.quantity) AS total_quantity
FROM order_details
JOIN pizzas
ON order_details.pizza_id = pizzas.pizza_id
JOIN pizza_types
ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY pizza_types.category
ORDER BY total_quantity DESC;

-- 7. Determine the distribution of orders by hour of the day
SELECT 
    EXTRACT(HOUR FROM time) AS order_hour,
    COUNT(order_id) AS total_orders
FROM orders
GROUP BY order_hour
ORDER BY order_hour;

-- 8. Find category-wise distribution of pizzas
SELECT 
    pizza_types.category,
    COUNT(pizza_types.name) AS pizza_count
FROM pizza_types
GROUP BY pizza_types.category
ORDER BY pizza_count DESC;


-- 9. Calculate the average number of pizzas ordered per day
SELECT 
    ROUND(AVG(daily_pizzas),2) AS avg_pizzas_per_day
FROM
(
    SELECT 
        orders.date,
        SUM(order_details.quantity) AS daily_pizzas
    FROM orders
    JOIN order_details
    ON orders.order_id = order_details.order_id
    GROUP BY orders.date
) AS daily_orders;

-- 10. Determine the top 3 most ordered pizza types based on revenue
SELECT 
    pizza_types.name,
    SUM(order_details.quantity * pizzas.price) AS revenue
FROM order_details
JOIN pizzas
ON order_details.pizza_id = pizzas.pizza_id
JOIN pizza_types
ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 3;

/* =====================================================
   ADVANCED ANALYSIS
===================================================== */

-- 11. Calculate the percentage contribution of each pizza type to total revenue
SELECT 
    pizza_types.name,
    SUM(order_details.quantity * pizzas.price) AS revenue,
    ROUND(
        SUM(order_details.quantity * pizzas.price) * 100 /
        (SELECT SUM(order_details.quantity * pizzas.price)
         FROM order_details
         JOIN pizzas
         ON order_details.pizza_id = pizzas.pizza_id),
    2) AS revenue_percentage
FROM order_details
JOIN pizzas
ON order_details.pizza_id = pizzas.pizza_id
JOIN pizza_types
ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY pizza_types.name
ORDER BY revenue DESC;


-- 12. Analyze the cumulative revenue generated over time
SELECT 
    order_date,
    SUM(revenue) OVER (ORDER BY order_date) AS cumulative_revenue
FROM
(
    SELECT 
        orders.date AS order_date,
        SUM(order_details.quantity * pizzas.price) AS revenue
    FROM orders
    JOIN order_details
    ON orders.order_id = order_details.order_id
    JOIN pizzas
    ON order_details.pizza_id = pizzas.pizza_id
    GROUP BY orders.date
) AS daily_revenue;


-- 13. Determine the top 3 most ordered pizza types based on revenue for each category
SELECT *
FROM
(
    SELECT 
        pizza_types.category,
        pizza_types.name,
        SUM(order_details.quantity * pizzas.price) AS revenue,
        RANK() OVER(
            PARTITION BY pizza_types.category
            ORDER BY SUM(order_details.quantity * pizzas.price) DESC
        ) AS rank
    FROM order_details
    JOIN pizzas
    ON order_details.pizza_id = pizzas.pizza_id
    JOIN pizza_types
    ON pizzas.pizza_type_id = pizza_types.pizza_type_id
    GROUP BY pizza_types.category, pizza_types.name
) ranked_pizzas
WHERE rank <= 3;
