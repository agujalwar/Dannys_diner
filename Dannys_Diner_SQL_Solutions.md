# # 🍽 Case Study #1 - Danny's Diner
![image](https://user-images.githubusercontent.com/125154280/226223335-b2c6baa8-6264-443e-b35e-f6f7bb70df9b.png)


**1. What is the total amount each customer spent at the restaurant? **

select s.customer_id, sum(m.price)

from dannys_diner.sales s right join

dannys_diner.menu m on

s.product_id = m.product_id

group by s.customer_id

order by s.customer_id


select s.customer_id, sum(m.price) as "Total Amount"
from dannys_diner.sales s right join 
dannys_diner.menu m on
s.product_id = m.product_id
group by s.customer_id
order by s.customer_id

Output :

![image](https://user-images.githubusercontent.com/125154280/226230677-9efb1fe0-48a1-4194-958f-1f6956850b76.png)


How many days has each customer visited the restaurant?

What was the first item from the menu purchased by each customer?

What is the most purchased item on the menu and how many times was it purchased by all customers?

Which item was the most popular for each customer?

Which item was purchased first by the customer after they became a member?

Which item was purchased just before the customer became a member?

What is the total items and amount spent for each member before they became a member?

If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
