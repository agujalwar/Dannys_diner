# # 🍽 Case Study #1 - Danny's Diner

<img src="https://user-images.githubusercontent.com/103854541/211899169-ff412900-ec3d-45d2-b375-4660632c6ef1.png" width="500" height="500">



1. What is the total amount each customer spent at the restaurant? **
```SQL
select s.customer_id, sum(m.price)
from dannys_diner.sales s right join
dannys_diner.menu m on
s.product_id = m.product_id
group by s.customer_id
order by s.customer_id
```
**Output :

![image](https://user-images.githubusercontent.com/125154280/226230677-9efb1fe0-48a1-4194-958f-1f6956850b76.png)


2.How many days has each customer visited the restaurant?

```sql
select customer_id, count(distinct order_date) as 'Days'
from dannys_diner.sales
group by customer_id
```
**Outout

![image](https://user-images.githubusercontent.com/125154280/226232420-7f8716e5-f777-41b5-9130-d83a0e7ee5ef.png)


3.What was the first item from the menu purchased by each customer?
```sql
with temp as (
select s.customer_id,  m.product_name, order_date,
dense_rank() over( partition by s.customer_id order by s.order_date ) as Row_count1
from dannys_diner.sales s right join 
dannys_diner.menu m on
s.product_id = m.product_id
group by s.customer_id,  m.product_name, order_date
order by customer_id
)

select * from temp
where Row_count1 = 1
```
**Output

![image](https://user-images.githubusercontent.com/125154280/226232707-1dae24ba-1952-4ff5-af47-f52219113494.png)


4.What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
select m.product_name as 'Product name',count(s.customer_id) as 'Purchase count'
from dannys_diner.sales s join 
dannys_diner.menu m on
s.product_id = m.product_id
group by m.product_name
order by count(s.customer_id) desc
limit 1
```
**Output

![image](https://user-images.githubusercontent.com/125154280/226234028-aadc2523-2410-43a7-8624-a531eeaa42a1.png)


5.Which item was the most popular for each customer?
```SQL
with temp as (
select s.customer_id, m.product_name, m.product_id, count(s.product_id) as order_count , 
dense_rank() over( partition by s.customer_id order by count(s.product_id)desc) as Row_count1
from dannys_diner.sales s 
join dannys_diner.menu m
on s.product_id=m.product_id
group by customer_id, product_name, product_id
)

select   
t.customer_id, me.product_name,  order_count
from temp t
join dannys_diner.menu me on
t.product_id=me.product_id
where t.Row_count1 = 1
group by t.customer_id, me.product_name, order_count
order by t.customer_id

```
**Output

![image](https://user-images.githubusercontent.com/125154280/226234358-afdabd86-791d-4067-9a96-97e5f916b1e7.png)


6.Which item was purchased first by the customer after they became a member?
```sql
with added_row_count as (
select s.customer_id as customer_id, s.product_id as product_id, s.order_date as order_date, 
dense_rank() over( partition by s.customer_id order by s.order_date ) as Row_count1
from dannys_diner.sales s join dannys_diner.members m
on s.customer_id=m.customer_id
where order_date >= join_date
)
select   
rc.customer_id,rc.order_date, me.product_name
-- , rc.Row_count1
from added_row_count rc
join dannys_diner.menu me on
rc.product_id=me.product_id
group by rc.customer_id,rc.order_date, me.product_name, rc.Row_count1
having rc.Row_count1 = 1
order by rc.customer_id
```
**Output

![image](https://user-images.githubusercontent.com/125154280/226234463-7ba2161f-88fb-4412-b633-51ae9cab7530.png)


7.Which item was purchased just before the customer became a member?
```sql
with temp as (
select s.customer_id,  m2.product_name, order_date, m.join_date,
rank() over( partition by s.customer_id order by s.order_date desc) as Row_count1
from sales s
join members m 
on s.customer_id = m.customer_id
join menu m2
on s.product_id=m2.product_id
where order_date<m.join_date
group by s.customer_id,  m2.product_name, order_date, m.join_date
order by customer_id
)

select customer_id, product_name, order_date 
from temp
where Row_count1 = 1
```
**Output

![image](https://user-images.githubusercontent.com/125154280/226234573-d7678a21-2bb0-434a-bcd5-5e2699f64d19.png)


8.What is the total items and amount spent for each member before they became a member?
```sql
select s.customer_id, count(m2.product_id) as "Total items", sum(m2.price) as "Total Amount"
from sales s
join members m 
on s.customer_id = m.customer_id
join menu m2
on s.product_id=m2.product_id
where s.order_date < m.join_date
group by customer_id
```
**Output

![image](https://user-images.githubusercontent.com/125154280/226234678-b758658e-a83b-49cb-b5d6-4f5f04459297.png)


9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
select customer_id,sum( if(product_name != 'sushi', 10*price, 20*price) ) as "Points"
from sales s 
join menu m
on s.product_id = m.product_id
group by customer_id
```
**Output

![image](https://user-images.githubusercontent.com/125154280/226234776-6be8e78e-5907-4013-acda-b4dcee9ead26.png)


10.In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql

with temp as (
select s.customer_id, s.product_id, product_name, order_date, join_date, price
from sales s
join members m 
on s.customer_id = m.customer_id
join menu m2
on s.product_id=m2.product_id
)

select customer_id, sum(
case 
	when (order_date >= join_date and order_date <= date_add(join_date, interval 1 week)) then price*20 
	when (order_date > join_date and product_name = 'sushi' ) then price*20 
	else price*10
end) as "Points"
from temp t 
where order_date <= "2021-01-31"
group by customer_id
```
**Output

![image](https://user-images.githubusercontent.com/125154280/226235032-b3f222fe-0314-4b06-9bad-5205004be189.png)
