📌 Project Overview

This project explores SQL-based analytics on a Pizza Retail dataset. The dataset consists of customer orders, pizza details, and sales transactions. By writing and executing SQL queries, we answer basic, intermediate, and advanced business questions to extract insights about sales trends, customer behavior, and revenue generation.

🗂 Dataset Description

The dataset contains 4 tables:

1️⃣ orders

Stores information about orders.

Column	Description
order_id	Unique ID for each order
date	Date of the order
time	Time of the order
2️⃣ order_details

Links orders with pizzas purchased.

Column	Description
order_details_id	Unique ID for each order detail
order_id	References orders table
pizza_id	References pizzas table
quantity	Number of pizzas ordered
3️⃣ pizzas

Pizza-specific details.

Column	Description
pizza_id	Unique ID for each pizza
pizza_type_id	References pizza_types table
size	Size of the pizza (S, M, L, XL, XXL)
price	Price of the pizza
4️⃣ pizza_types

Defines pizza categories and recipes.

Column	Description
pizza_type_id	Unique ID for each pizza type
name	Name of the pizza
category	Category (Classic, Supreme, etc.)
ingredients	List of ingredients used
📊 Analysis Questions
🔹 Basic Queries

Retrieve the total number of orders placed.
```sql
-- Retrieve the total number of orders placed
SELECT COUNT(order_id) AS total_Order
FROM orders;
```

Calculate the total revenue generated from pizza sales.
```sql
-- Calculate the total revenue generated from pizza sales.

-- solution
-- Now the problem is the data is not in the single table so u can make a table 
-- or else directly fetch the data from the both tables with the help of common thing(column) i.e. pizza_id

SELECT 
    ROUND(SUM(orders_details.quantity * pizzas.price),
            2) AS total_revenue
FROM
    orders_details
        JOIN
    pizzas ON orders_details.pizza_id = pizzas.pizza_id
    
    -- to Beautify the Query just use Ctrl + b 

```

Identify the highest-priced pizza.
```sql
-- Identify the highest-priced pizza.

-- In this we have to find the highest price pizza Name so take the name from the pizza_type table and price from the pizzas
-- after that just arracnge in desc order on the basis of prices and then limit by 1.

SELECT 
    pizza_types.name, pizzas.price
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;

```

Identify the most common pizza size ordered.
```sql
-- Identify the most common pizza size ordered.

-- group by size and take the pizzaid common from both table pizzas and from order_details
-- then count the order_details_id with size of pizzas
-- please make a point always most selling thing is not a most common thing here u need to find most common not most selling

SELECT 
    pizzas.size,
    COUNT(orders_details.order_details_id) AS order_count
FROM
    pizzas
        JOIN
    orders_details ON orders_details.pizza_id = pizzas.pizza_id
GROUP BY pizzas.size
ORDER BY order_count DESC
LIMIT 1;

```

List the top 5 most ordered pizza types along with their quantities.
```sql
-- List the top 5 most ordered pizza types along with their quantities.

-- Now here most ordered means most selling in quantity
-- just take quantity from order details, here u have to pick the name.
-- So grp the quantity to pizza id and join to the pizza's table with pizza's id and then take the name from pizza types having same pizza_type_id

-- Note* Here we join three tables one by one  with common columns in it ok 

SELECT 
    pizza_types.name, SUM(orders_details.quantity) AS Quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    orders_details ON pizzas.pizza_id = orders_details.pizza_id
GROUP BY pizza_types.name
ORDER BY quantity DESC
LIMIT 5;

```

Join the necessary tables to find the total quantity of each pizza category ordered.
```sql
-- Join the necessary tables to find the total quantity of each pizza category ordered.

SELECT 
    pizza_types.category,
    SUM(orders_details.quantity) AS total_Quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    orders_details ON orders_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY total_Quantity DESC;
```

Determine the distribution of orders by hour of the day.
```sql
-- Determine the distribution of orders by hour of the day.

select hour(order_time) as hours, count(order_id) as orders from orders
group by hours order by orders desc;
```

Join relevant tables to find the category-wise distribution of pizzas.
```sql
-- Join relevant tables to find the category-wise distribution of pizzas.

SELECT 
    category, COUNT(name)
FROM
    pizza_types
GROUP BY category
ORDER BY COUNT(name) DESC;
```

Group the orders by date and calculate the average number of pizzas ordered per day.
```sql
-- Group the orders by date and calculate the average number of pizzas ordered per day.

SELECT 
    ROUND(AVG(total_orders), 0) as Average_Pizza_perDay
FROM
    (SELECT 
        orders.order_date,
            SUM(orders_details.quantity) AS total_orders
    FROM
        orders
    JOIN orders_details ON orders.order_id = orders_details.order_id
    GROUP BY orders.order_date) AS averageNumber_perday;
```

Determine the top 3 most ordered pizza types based on revenue.
```sql
-- Determine the top 3 most ordered pizza types based on revenue.

SELECT 
    pizza_types.name,
    sum(orders_details.quantity * pizzas.price) AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    orders_details ON pizzas.pizza_id = orders_details.pizza_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 3;
```

Calculate the percentage contribution of each pizza type to total revenue.
```sql
-- Calculate the percentage contribution of each pizza type to total revenue.

SELECT 
    pizza_types.category,
    round((sum(orders_details.quantity * pizzas.price) / (SELECT 
    ROUND(SUM(orders_details.quantity * pizzas.price),
            2) AS total_revenue
FROM
    orders_details
        JOIN
    pizzas ON orders_details.pizza_id = pizzas.pizza_id))* 100, 0) As revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    orders_details ON pizzas.pizza_id = orders_details.pizza_id
GROUP BY pizza_types.category
ORDER BY revenue DESC;
```

Analyze the cumulative revenue generated over time.
```sql
-- Analyze the cumulative revenue generated over time.


select order_date, sum(revenue) over (order by order_date) as cum_revenue
from 
(select
orders.order_date, sum(orders_details.quantity * pizzas.price) as revenue
from orders_details join pizzas 
on orders_details.pizza_id = pizzas.pizza_id
join orders 
on orders.order_id = orders_details.order_id
group by orders.order_date) as sales;
```

Determine the top 3 most ordered pizza types based on revenue for each pizza category.
```sql
-- Determine the top 3 most ordered pizza types based on revenue for each pizza category.

select name, revenue from
(select category, name, revenue, 
rank() over (partition by category order by revenue desc ) as rn
from 
(select 
pizza_types.category, pizza_types.name, sum((orders_details.quantity)* pizzas.price) as revenue
from pizza_types join pizzas
on pizza_types.pizza_type_id = pizzas.pizza_type_id 
join orders_details
on orders_details.pizza_id = pizzas.pizza_id
group by  pizza_types.category, pizza_types.name) as a) as b
where rn <= 3;
	
```

⚙️ Tools & Technologies

SQL for data querying and analysis

MySQL  as the database

CSV files for raw dataset

📈 Sample Insights

Revenue trends highlight the top-selling pizza categories.

Large pizzas are the most commonly ordered size.

Sales peak during lunch (12–2 PM) and dinner (6–8 PM) hours.
