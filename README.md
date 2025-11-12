# 🛒 Jenson USA – SQL Analytics Project

# Project Overview

This project simulates the role of a Data Analyst at Jenson USA, where the goal is to extract actionable insights from retail data using SQL. The analysis covers customer behavior, staff performance, inventory management, and store operations, helping leadership make data-driven decisions to improve sales, optimize inventory, and enhance staff productivity.

The dataset consists of multiple relational tables that capture orders, customers, products, categories, staff, and stores, enabling a holistic view of operations.


# 📊 Questions & Business Problems

# Store Performance

## Q1. Find the total number of products sold by each store along with the store name.

Total reimbursed for inpatient claims, grouped by provider

```sql

select * from stores;

select * from orders;

select * from order_items; 

SELECT 
    s.store_id, s.store_name, SUM(oi.quantity) AS products_sold
FROM
    stores s
        JOIN
    orders o ON o.store_id = s.store_id
        JOIN
    order_items oi ON o.order_id = oi.order_id
GROUP BY 1 , 2;

```

# Product Trends

## Q2. Calculate the cumulative sum of quantities sold for each product over time.

```sql

select * from order_items;

# Method - 1
select p.product_id, p.product_name, oi.quantity, 
sum(oi.quantity) over(partition by p.product_name order by o.order_date) as cumulative_quantity
from products p
join order_items oi
on p.product_id = oi.product_id
join orders o
on o.order_id = oi.order_id;


# Method - 2
with a as 
	(select p.product_id, p.product_name, o.order_date,
	sum(oi.quantity) as total_quantity
	from products p
	join order_items oi
	on p.product_id = oi.product_id
	join orders o
	on o.order_id = oi.order_id
	group by 1,2,3)

select *, sum(total_quantity) 
over(partition by product_name 
order by order_date) as cummilative_quantity
from a;

```

## Q3. Find the product with the highest total sales (quantity * price) for each category.

```sql

select * from order_items;
select * from categories;
select * from products;

with a as
	(select c.category_name, p.product_name,
	sum(oi.quantity * oi.list_price) as sales
	from categories c
	join products p
	on c.category_id = p.category_id
	join order_items oi
	on p.product_id = oi.product_id
	group by 1,2)
    
select category_name, product_name, sales from     
	(select *, 
    dense_rank() 
    over(partition by category_name 
    order by sales desc) rnk from a) b
    where rnk = 1;

```

## Q4. Find the top 3 most sold products in terms of quantity.

```sql

select * from orders;
select * from order_items;
select * from products;

# METHOD - 1
select p.product_id, p.product_name, 
sum(oi.quantity) as quantity_sold 
from products p
join order_items oi
on p.product_id = oi.product_id
group by 1,2
order by 3 desc
limit 3;


# METHOD - 2

select * from 
	(select p.product_id, p.product_name, 
	sum(oi.quantity) as quantity_sold,
    rank() over(order by sum(oi.quantity) desc) as rnk
	from products p
	join order_items oi
	on p.product_id = oi.product_id
	group by 1,2) as product_rank
where rnk <= 3;

```


## Q5. Find the median value of the price list.

```sql

select * from order_items;

with a as 
	(select list_price,
	row_number() over(order by list_price) as pos,
	count(*) over() as n 
	from order_items)
select 
	case
		when n%2 = 0 then (select round(avg(list_price),0) from a where pos in ((n/2), (n/2+1))) -- For even pos
        else (select round(list_price,0) from a where pos = ((n+1)/2)) -- For odd pos
        end as median
from a 
limit 1;

```

## Q6. List all products that have never been ordered (use EXISTS).

```sql

select * from products;
select * from orders;
select * from order_items;

# METHOD - 1
select p.product_id, p.product_name, oi.order_id
from products p
left join order_items oi
on p.product_id = oi.product_id
group by 1,2,3
having order_id is null;

# METHOD - 2

SELECT 
    p.product_id, p.product_name
FROM
    products p
WHERE
    NOT EXISTS( SELECT 
            p.product_id
        FROM
            order_items oi
        WHERE
            p.product_id = oi.product_id);

```

# Customer Insights

## Q7. Find the customer who spent the most money on orders.

```sql

select * from customers;
select * from orders; 
select * from order_items;

# Method 1
select c.customer_id, concat(c.first_name, ' ' ,c.last_name) as customer_name,
round(sum(oi.quantity * oi.list_price -(oi.quantity * oi.list_price * (oi.discount/100))),2) as amount_spent
from customers c
join orders o
on c.customer_id = o.customer_id
join order_items oi
on o.order_id = oi.order_id
group by 1,2
order by amount_spent desc
limit 1;


# Method 2
with a as 
	(select c.customer_id, concat(c.first_name, ' ' ,c.last_name) as customer_name,
	round(sum(oi.quantity * oi.list_price -(oi.quantity * oi.list_price * (oi.discount/100))),2) as amount_spent
	from customers c
	join orders o
	on c.customer_id = o.customer_id
	join order_items oi
	on o.order_id = oi.order_id
	group by 1,2
    order by amount_spent desc)

select * from 
(select *, 
rank() 
over(order by amount_spent desc) 
rnk from a) b
where rnk = 1;

```

## Q8. Find the total number of orders placed by each customer per store.

```sql

select * from stores;
select * from customers;
select * from orders;

# METHOD - 1
select c.customer_id, concat(first_name, ' ', last_name) as customer_name, s.store_name, 
count(o.order_id) as product_count
from customers c
join orders o
on c.customer_id = o.customer_id
join stores s
on o.store_id = s.store_id
group by 1,2,3;

# METHOD - 2
select  distinct c.customer_id, 
concat(first_name, ' ', last_name) as customer_name, s.store_name, 
count(o.order_id) over(partition by c.customer_id, s.store_name) as product_count
from customers c
join orders o
on c.customer_id = o.customer_id
join stores s
on o.store_id = s.store_id;

```


## Q9. Identify customers who have ordered all types of products (i.e., from every category).

```sql

with customer_categories as	
    (select c.customer_id, concat(c.first_name, ' ',c.last_name) as customer_name, ct.category_id
	from customers c
	join orders o
	on c.customer_id = o.customer_id
	join order_items oi
	on o.order_id = oi.order_id
	join products p
	on oi.product_id = p.product_id
	join categories ct
	on p.category_id = ct.category_id),
    
category_counts as (
	select customer_id, customer_name, 
    count(distinct category_id) as customer_category_count
    from customer_categories
    group by 1,2),
    
total_categories as 
	(select count(*) as total_category_count
    from categories)
 
    select cc.customer_id, cc.customer_name
    from category_counts cc
    join total_categories tc
    on cc.customer_category_count = tc.total_category_count;


# METHOD - 2 - [Class]
select customers.customer_id, customers.first_name, count(order_items.product_id)
from customers join orders
on customers.customer_id = orders.customer_id
join order_items
on orders.order_id = order_items.order_id
join products
on order_items.product_id = products.product_id
group by customers.customer_id, customers.first_name
having count(distinct(category_id)) = (select count(category_id) from categories);

```

Staff Performance

## Q10. Find the names of staff members who have not made any sales.

```sql

select * from staffs;
select * from orders;
select * from order_items;

# METHOD - 1

select s.staff_id, 
concat(first_name, ' ', last_name) as staff_name,
count(o.order_id) as product_sold
from staffs s
left join orders o
on s.staff_id = o.staff_id
group by 1,2
having product_sold = 0;


# METHOD - 2
select s.staff_id, 
concat(first_name, ' ', last_name) as staff_name
from staffs s
left join orders o
on s.staff_id = o.staff_id
where o.order_id is null;

```

## Q11. List the names of staff members who have made more sales than the average number of sales by all staff members.

```sql

# METHOD - 1 --- Wrong Method
# Sale by each staff
	select s.staff_id, s.first_name,
	round(sum(oi.quantity * oi.list_price -(oi.quantity * oi.list_price * (oi.discount/100))),2) as total_sales  
	from staffs s
	left join orders o
	on s.staff_id = o.staff_id
	left join order_items oi
	on o.order_id = oi.order_id
	group by 1,2;

# Staff who sold more than avg sale 
   
# METHOD - 2

with a as 
	(select staffs.staff_id, staffs.first_name,
	coalesce(sum(order_items.list_price * order_items.quantity),0) as total_sales
	from staffs left join orders
	on orders.staff_id = staffs.staff_id
	left join order_items
	on orders.order_id = order_items.order_id
	group by staffs.staff_id, staffs.first_name)

select * from a
where total_sales > (select avg(total_sales) from a);

```

Inventory & Pricing

## Q12. Find the highest-priced product for each category name.


```sql

select * from products;

# METHOD -1 
select * from 
	(select c.category_name, p.product_name, p.list_price,
	max(p.list_price) over(partition by c.category_name) as max_price  
	from categories c
	join products p
	on c.category_id = p.category_id) b
    where list_price = max_price;
    

# Method - 2 
select * from 
	(select c.category_name, p.product_name, p.list_price,
	dense_rank() over(partition by c.category_id order by list_price desc) as rnk  
	from categories c
	join products p
	on c.category_id = p.category_id) b
where rnk = 1;

```
