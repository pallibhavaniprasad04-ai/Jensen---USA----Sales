# 7. Find the names of staff members who have not made any sales.

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
