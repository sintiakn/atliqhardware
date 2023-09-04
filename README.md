# atliqhardware
1. Transactions Table
   
|Field Name|Data Type|
|----------|---------|
|product_code|String|



`CREATE TEMPORARY TABLE transactions_clean SELECT *,CASE WHEN 
			currency ='USD' THEN sales_amount*75
			ELSE sales_amount
            END AS normalized_amount FROM transactions;`

`SELECT *, round(normalized_amount-cost_price,2) as profit, round((normalized_amount-cost_price)/normalized_amount,2) as profit_margin
FROM transactions_clean`
