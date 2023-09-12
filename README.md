# Problem Statement
  In this project performed India based AtliQ hardware company sales insights - A Data 
  Analysis project.
	
  AtliQ Hardware is a company which supplies computer hardware and peripherals to many of 
  clients. AtliQ Hardware head office is situated in Delhi, India and they have many 
  regional office through out the India.
	
  Sales director for this company is facing a lot of challenges. The market is growing 
  dynamically and sales director is facing issue in terms of tracking the sales in this 
  dynamical growth market and he is having issues with growth of this bussiness, as overall 
  sales was declining. He has regional manager for North India, South and Central India. 
  Whenever he wants to get insights of these region he would call them  on the phone and 
  regional manager give some insights to him.
	
  The problem was that all these thing happening is verbal and these was no proof with facts 
  that how his business is affected and which made him frustraed as he can see that overall 
  sales is declining but when he ask regional manager, he is not getting complete picture of 
  this bussiness. All he wants is a simple data visualization tool which he can access on 
  daily basis. By using such tools and technology, he can make data driven decisions which 
  helps to increase the sales of the company. So, In this projects we will help a company 
  make its own sales related dashboard using tableau.
  
# Data Analysis Using MySQL
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
  `SELECT * FROM transaction`
  `SELECT * FROM markets`
  `SELECT * FROM customers`
  `SELECT * FROM products`
  `SELECT * FROM date`
  
 - Check the Null Values
   There are some NULL value in market table, so we drop that rows and make a new table.
  
   `CREATE TABLE market as(SELECT * FROM markets WHERE zone IS NOT NULL)`

- Check duplicated Values
  We can check duplicated values with row_number function

  `WITH CTE AS (select *,row_number() over (partition by product_code,customer_code,market_code,order_date,
			sales_qty,sales_amount  order by product_code) as rownumber
			from transactions)
   SELECT * FROM CTE WHERE rownumber=1`
  
- Convert Currency
    There is inconsistency record in currency column in transactions table. We can see that certain transactions are in USD. Hence, filtration of that is also needed by converting into INR.

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

  # Build Dashboard Using Tableau
![key insight](https://github.com/sintiakn/atliqhardware/assets/115802103/cc6baefc-ae14-418c-bf81-8dd303aa9ae4)


![profit analysis](https://github.com/sintiakn/atliqhardware/assets/115802103/77b0ca00-97d1-488d-8ff8-09daaee1c861)

- Insight
  - There was a decrease in revenue in 2020, likely due to the impact of the pandemic.
  - From 2017 to 2020, there was a noticeable drop in revenue around the month of June, indicating a seasonal pattern in the business.
  - August marked an increase in revenue, mirroring the success of previous years, especially in 2018-2019.
  - The highest revenue was generated in Market Delhi, with a large quantity of items sold, but with a profit margin of only 2.3%.
  - The customer contributing the highest revenue is Electricalsara Store, accounting for 37.75% of the total profit.
  - Prod318 is the product that dominates in terms of revenue generation for the company.
  - Although sales are low, Surat boasts the highest profit margin among all sales regions.

- Recommendations
  - Focus on Market Delhi because, although Market Delhi has high sales, the profit margin is low. The company should explore ways to enhance the margin in this market. This may include better pricing strategies, cost reduction, or increased operational efficiency.
  - Electricalsara Store's customer significantly contributes to the company's profits. Consider strengthening the relationship with these customers, perhaps by offering incentives or loyalty programs to encourage them to continue shopping.
  - The decrease in revenue that occurred in June from 2017 to 2020 needs further examination. Are there any seasonal factors or market trends influencing this? This analysis can assist in developing strategies to address the revenue decline during those months.
  - If there was a significant increase in revenue in August in 2018-2019, consider identifying the contributing factors and attempting to replicate that success in the following years. This might involve special promotions or marketing campaigns aimed at boosting sales during August.
  - While sales in Surat may not be as substantial as in other areas, note that the profit margin there is high. Consider increasing sales in this region through a more focused marketing strategy or improved customer support.
  - The pandemic likely had a significant impact on the revenue decline in 2020. It's essential to continually monitor the pandemic situation and have a crisis plan ready if similar situations occur in the future.

