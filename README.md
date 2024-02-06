# SQL-case-study
Contains solutions for #8WeekSQLChallenge case studies 

Q1. What is the total amount each customer spent at the restaurant?
  
     select 
         s.customer_id,sum(m.price) as spent
     from  sales s
     inner join menu m on s.product_id = m.product_id
     group by customer_id;

Q2. How many days has each customer visited the restaurant?
    
      select 
         customer_id,
	 count(distinct order_date) as visits
      from sales 
      group by customer_id;

Q3. What was the first item from the menu purchased by each customer?
 
    with item_cte as(
	select 
	    s.customer_id,
		m.product_name,
		s.order_date,
		s.product_id,
		row_number()over(partition by customer_id order by order_date)rnk
    from sales s
	inner join menu m on s.product_id = m.product_id)

	select customer_id,product_name
	from item_cte
	where rnk = 1;
    
Q4. What is the most purchased item on the menu and how many times was it purchased by all customers?
   
         select 
            top (1)
	   m.product_name,
	   count(s.product_id) as purchased
        from sales s
        inner join menu m on s.product_id = m.product_id
        group by m.product_name
        order by purchased desc;

Q5. Which item was the most popular for each customer?
    
        with popular_product_cte as(
	      select  
	           s.customer_id,
	           count(s.product_id) as ordered,
	           m.product_name,
	           dense_rank()over(partition by customer_id order by count(s.product_id) desc)rnk
      from sales s
	inner join menu m on s.product_id = m.product_id
	group by s.customer_id, m.product_name
	)
	select customer_id,
	       product_name,
		   ordered
	from popular_product_cte
	where rnk = 1;

Q6. Which item was purchased first by the customer after they became a member?
    
     with first_item_cte as(
	 select 
	    s.customer_id,
		s.order_date,
		s.product_id,
		m.join_date,
		row_number() over(partition by s.customer_id order by order_date)rnk
	from sales s
	left join members m on s.customer_id = m.customer_id 
	)
    select customer_id,order_date,join_date,product_id
    from first_item_cte
    where rnk = 1;

Q7. Which item was purchased just before the customer became a member?
   
     WITH before_memb_orders_cte AS(
	SELECT
		s.customer_id,
		order_date,
		join_date,
		product_id,
		row_number() over (partition by s.customer_id
		order by order_date desc) as rank
	FROM
		sales s
	INNER JOIN members m ON
		m.customer_id = s.customer_id
		AND
		order_date < join_date
	)
	SELECT 
	    customer_id,
	    product_name,
	    order_date, 
	    join_date
      FROM 
	before_memb_orders_cte mo
     INNER JOIN menu m ON
	m.product_id = mo.product_id
     WHERE
	rank = 1;


Q8. What is the total items and amount spent for each member before they became a member?

    select sales.customer_id,
		 sum(menu.price) as total_spend,
		 count(menu.product_id) as item
    from sales
    inner join menu on sales.product_id = menu.product_id
    inner join members on sales.customer_id = members.customer_id
    where members.join_date > sales.order_date
    group by sales.customer_id;

Q9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

    SELECT
	customer_id,
	sum(CASE
		WHEN s.product_id = 1 THEN price*20
		ELSE price*10 
	END) as total_points
    FROM
	sales s,
	menu m
    WHERE m.product_id = s.product_id
    GROUP BY customer_id;
   
Q10. In The first week after customer joins the program(including the join date) they earn 2x points on allthe items not just sushi how many points 
do customer A and B have at the end of january?

    WITH dates_cte AS(
	SELECT *, 
		DATEADD(DAY, 6, join_date) AS valid_date, 
		EOMONTH('2021-01-1') AS last_date
	FROM members
)

    SELECT
	s.customer_id,
	sum(CASE
		WHEN s.product_id = 1 THEN price*20
		WHEN s.order_date between d.join_date and d.valid_date THEN price*20
		ELSE price*10 
	END) as total_points
    FROM
	dates_cte d,
	sales s,
	menu m
    WHERE
	d.customer_id = s.customer_id
	AND
	m.product_id = s.product_id
	AND
	s.order_date <= d.last_date
    GROUP BY s.customer_id;




  



