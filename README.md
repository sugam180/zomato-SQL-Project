# zomato-SQL-Project:zomato Data Analysis using SQL

# Creating database
Create Database zomato;

# Use of the database
use zomato;

# Overview of the tables:
SELECT * FROM zomato_delivery_partner;
SELECT * FROM zomato_food;
SELECT * FROM zomato_menu;
SELECT * FROM zomato_order_details;
SELECT * FROM zomato_orders;
SELECT * FROM zomato_restaurants;
SELECT * FROM zomato_users;

# Q. Find the customers who never ordered?
-- find the customers who never ordered
select u.name 
from zomato_users u
left join zomato_orders o
on u.user_id=o.user_id
where o.user_id is null;

# Q. What are the average prices of the dishes?
-- average price/dish
select f_name, avg(price) From
zomato_food f JOIN zomato_menu m
ON f.f_id = m.f_id 
GROUP BY f.f_name;

# Q. Find the top restaurants in terms of orders for a given month.
-- Find top restaurants in terms of number of orders for a given month
select r_name,total_order 
from (select r.r_name,count(o.order_id) as total_order,monthname(o.date) months
from zomato_restaurants r
join zomato_orders o 
on r.r_id=o.r_id
group by r.r_name,months) order_count
order by total_order desc
limit 4;

# Q. Find restaurants with monthly sales greater than x.
-- restaurants with monthly sales greater than x
select r.r_name, sum(o.amount) from zomato_restaurants r  join zomato_orders o  
on r.r_id = o.r_id
 where o.amount> 480  and monthname(o.date) like 'June'
 group by r.r_name;

 # Q. Show all orders with order details for a particular customers in a particular data range
 -- show all orders with order details for a particular customers in a particular date range
 Select u.name, f.f_name, u.email, sum(o.amount) total
FROM zomato_users u 
join zomato_orders o 
on u.user_id=o.user_id
join zomato_order_details od
on o.order_id=od.order_id
join zomato_food f
on od.f_id= f.f_id
where o.date between '2022-05-10' and '2022-07-21'
and u.name like 'Neha'
group by f.f_name,u.email,u.name;

# Q. Find restaurants with maximum repeated customers.
-- find restaurant with max repeated customers
select r.r_name, u.name, count(r.r_name) as freq from
zomato_restaurants r 
join zomato_orders o
on r.r_id = o.r_id
join zomato_users u
on u.user_id = o.user_id
GROUP BY r.r_name, u.name
ORDER BY  freq DESC LIMIT 3;

# Q. Show the month over month revenue growth of a restaurant.
-- month over month revenue growth of a restaurant
SELECT r.r_name, AVG(amount) total_rev,  MONTHNAME(date) as Months
FROM zomato_orders o JOIN 
zomato_restaurants r ON 
o.r_id = r.r_id
WHERE r.r_name = 'kfc'
GROUP BY r.r_name, Months;

# Q. List the name of restaurants who have received the orders along with the number of orders.
-- List Name of restaurants who have recieved the orders along with the number of orders
Select r_name , count(*) as order_recieved FROM
zomato_restaurants r JOIN zomato_orders o ON
r.r_id=o.r_id
GROUP BY r.r_name;

# Q. List the name of users with their food name
-- list the name of users with their food name
SELECT f_name, name
FROM zomato_users u
JOIN zomato_orders o ON u.user_id=o.user_id
JOIN zomato_order_details d ON o.order_id = d.order_id
JOIN zomato_food f ON d.f_id = f.f_id
GROUP BY f_name, name;

# Q. Which is the most ordered food?
-- Which is the Most Ordered Food
SELECT f.f_id,f.f_name, COUNT(f.f_name) as frequency
FROM zomato_food f JOIN
zomato_menu m ON
f.f_id = m.f_id
JOIN zomato_orders o ON
m.r_id = o.r_id
GROUP BY f.f_name, f.f_id
ORDER BY frequency DESC LIMIT 1;

# Q. Find the top restaurant in terms of number of orders for a given month?
-- Find top restaurants in terms of Number of orders for a given month
SELECT r.r_name, Count(o.order_id) as orders
FROM zomato_restaurants r JOIN
zomato_orders o ON
r.r_id = o.r_id 
GROUP BY r.r_name
ORDER BY orders DESC LIMIT 1;

# Q. Find each customer with their respective foods.
WITH customer_food_count AS (
    SELECT u.name AS customer_name, 
	COUNT(o.order_id) AS frequency, f.f_name AS food_name
    FROM zomato_users u JOIN zomato_orders o ON u.user_id = o.user_id
    JOIN zomato_order_details od ON o.order_id = od.order_id
    JOIN zomato_food f ON od.f_id = f.f_id
    GROUP BY u.name, f.f_name),
maximum_order_count AS (
    SELECT customer_name, MAX(frequency) AS max_freq
    FROM customer_food_count
    GROUP BY customer_name)
SELECT cfc.customer_name, cfc.frequency AS times_ordered, cfc.food_name
FROM customer_food_count cfc JOIN maximum_order_count moc 
ON cfc.customer_name = moc.customer_name AND cfc.frequency = moc.max_freq;

# Q. Which restaurants are most popular among customers who ordered vegetarian food?
-- Which restaurants are most popular among customers who ordered vegeterian food
WITH restaurant_selection as (
SELECT r.r_name as restaurant_name, count(o.order_id) as freq,f.type as food_type
FROM zomato_orders o JOIN zomato_restaurants r ON o.r_id = r.r_id
JOIN zomato_order_details od ON o.order_id = od.order_id
JOIN zomato_food f ON od.f_id = f.f_id
GROUP BY r.r_name, f.type), food_selection as (
SELECT restaurant_name, food_type,max(freq) as max_freq
FROM restaurant_selection WHERE food_type = 'veg'
GROUP BY restaurant_name)
SELECT rs.restaurant_name, rs.freq, fs.food_type
FROM restaurant_selection rs JOIN food_selection fs
ON rs.restaurant_name = fs.restaurant_name AND rs.freq = fs.max_freq 
ORDER BY freq DESC;

# Q. What is the average order value for each customers who order food from each restaurants with a rating above 4.5??
-- What is the average order value for each customers who order food from each restaurants with a rating above 4.5
SELECT u.name, AVG(o.amount), r.r_name
FROM zomato_users u JOIN
zomato_orders o ON u.user_id = o.user_id
JOIN zomato_restaurants r ON o.r_id = r.r_id
WHERE o.restaurant_rating > 4.5
GROUP BY u.name, r.r_name
ORDER BY r.r_name;













