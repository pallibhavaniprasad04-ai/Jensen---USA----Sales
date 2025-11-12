# 2. Calculate the cumulative sum of quantities sold for each product over time.

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

 
 
