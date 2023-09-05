# atliqhardware
1. Transactions Table
   
|Field Name|Data Type|
|----------|---------|
|product_code|String|
|customer_code|String|
|market_code|String|
|order_date|Date|
|sales_qty|Integer|
|sales_amount|Integer|
|curency|String|
|cost_price|Integer|

2. Customers Table

|Field Name|Data Type|
|----------|---------|
|customer_code|String|
|customer_name|String|
|customer_type|String|

3. Date Table

|Field Name|Data Type|
|----------|---------|
|date|Date|
|cy_date|Date|
|year|String|
|month_name|String|
|date_yy_mmm|Date|

4. Markets Table

|Field Name|Data Type|
|----------|---------|
|markets_code|String|
|markets_name|String|
|zone|String|

5. Products Table

|Field Name|Data Type|
|----------|---------|
|product_code|String|
|product_name|String|

- Analysis Data with looking all of column Table
  There is inconsistency record in currency column. There 2 type of curency USD and INR. So convert USD into INR

  `CREATE TEMPORARY TABLE transactions_clean SELECT *,CASE WHEN currency ='USD' THEN sales_amount*75
			ELSE sales_amount
            END AS normalized_amount FROM transactions;`
  
- Calculate profit and profit margin. Make temporary table
  
  `CREATE TEMPORARY TABLE transaction_cleaning SELECT *, round(normalized_amount-cost_price,2) as profit, round((normalized_amount-cost_price)/normalized_amount,2) as profit_margin
   FROM transactions_clean`

- Find market names that have highest revenue
  
  `SELECT markets_name, sum(normalized_amount) FROM transaction_cleaning tc JOIN market m ON tc.market_code=m.markets_code GROUP BY market_code ORDER BY 2 DESC LIMIT 1`
  
- Find total revenue from 2017-2020
  
  `SELECT year, sum(normalized_amount) as Total_Revenue FROM transaction_cleaning tc JOIN date d ON tc.order_date=d.date GROUP BY year`
  
- Find total revenue every month from 2017-2020
  
   `Select year, month_name, sum(normalized_amount) as Total_Revenue FROM transaction_cleaning tc JOIN date d ON tc.order_date=d.date GROUP BY 1,2 ORDER BY 1,2`
  
-  Find top 10 product code
  
  `with cte as(SELECT product_code, sum(normalized_amount) as 
  sales_qty, dense_rank()over(order by sum(normalized_amount)DESC) as Top_sales from transaction_cleaning GROUP BY product_code ORDER BY sum(normalized_amount) DESC )
   Select * from cte where top_sales <=10`

- Find top 10 customer_name
  `With cte as(SELECT custmer_name, sum(normalized_amount) as sales_qty, dense_rank()over(order by sum(normalized_amount)DESC) as Top_sales from transaction_cleaning tc join customers c ON 
  tc.customer_code=c.customer_code GROUP BY custmer_name ORDER BY sum(normalized_amount) DESC ) select * from cte where top_sales <=10`

- JOIN all table
  `SELECT tc.product_code, tc.customer_code, tc.market_code, order_date, sales_qty,sales_amount,currency,cost_price,
		normalized_amount,custmer_name,customer_type,date,cy_date,year, month_name,date_yy_mmm,
		markets_name,zone,product_type,profit,profit_margin
		FROM transaction_cleaning tc LEFT JOIN customers c ON tc.customer_code=c.customer_code
								   LEFT JOIN date d ON tc.order_date=d.date
								   LEFT JOIN market m ON tc.market_code=m.markets_code
								   LEFT JOIN products p ON tc.product_code=p.product_code;`
