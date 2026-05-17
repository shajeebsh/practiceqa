---
title: "SQL"
date: 2026-04-26 15:22:55 
author: shajeebhameed
layout: page
slug: sql__trashed
status: trash
---

Challenge 1: The Running Total

Write a query to show each `transaction_id`, its `sale_date`, and a **running total**  of the `amount` ordered by the date.

Challenge 2: The Top Performer

Within **each region** , assign a rank to each salesperson based on their `amount` (highest amount = rank 1). If two sales are the same, they should have the same rank, and the next rank should be skipped (e.g., 1, 2, 2, 4).

Challenge 3: Growth Analysis

For each transaction, use a window function to show the `amount` of the **immediately preceding transaction**  (based on `transaction_id`).
    
    
    CREATE TABLE sales (
        transaction_id INT,
        sale_date DATE,
        salesperson_id INT,
        region VARCHAR(10),
        amount INT
    );
    
    INSERT INTO sales VALUES 
    (101, '2023-01-01', 1, 'North', 500),
    (102, '2023-01-02', 2, 'South', 300),
    (103, '2023-01-02', 1, 'North', 700),
    (104, '2023-01-03', 3, 'North', 400),
    (105, '2023-01-03', 2, 'South', 600),
    (106, '2023-01-04', 1, 'North', 800),
    (107, '2023-01-04', 3, 'North', 200),
    (108, '2023-01-05', 2, 'South', 500);
    
    with cte as (
    select 
    *,
    rank() over( order by sale_date desc) as rnk,
    sum(amount) over(
      partition by salesperson_id 
      order by sale_date
      rows between unbounded preceding and current row
    ) as runningtotal,
    lag(transaction_id) over(partition by salesperson_id order by sale_date) as prev_trans_id,
    lag(amount) over(partition by salesperson_id order by sale_date) as prev_trans_amount
    from sales where amount is not null)
    select *, 
    (CASE WHEN amount-prev_trans_amount>0 THEn 'Growth' else 'No Growth' END) as Growthstatus
    from cte where rnk = 1
    
    
    
    
    
    import pandas as pd;
    import numpy as np;
    
    data = {
        'transaction_id': [101, 102, 103, 104, 105, 106, 107, 108],
        'sale_date': pd.to_datetime(['2023-01-01', '2023-01-02', '2023-01-02', '2023-01-03', '2023-01-03', '2023-01-04', '2023-01-04', '2023-01-05']),
        'salesperson_id': [1, 2, 1, 3, 2, 1, 3, 2],
        'region': ['North', 'South', 'North', 'North', 'South', 'North', 'North', 'South'],
        'amount': [500, 300, 700, 400, 600, 800, 200, 500]
    }
    df = pd.DataFrame(data);
    df_sales = df[df['amount'].notna()].copy()
    #df_sales = df_sales[df_sales['amount'].notna()].copy()
    df_sales = df_sales.sort_values(by=['salesperson_id', 'sale_date']).reset_index();
    df_sales['runningtotal'] = df_sales.groupby('salesperson_id')['amount'].cumsum();
    df_sales['prev_transaction_id'] = df_sales.groupby('salesperson_id')['transaction_id'].shift(1);
    df_sales['prev_transaction_amount'] = df_sales.groupby('salesperson_id')['amount'].shift(1);
    df_sales['rnk'] = df_sales.groupby('salesperson_id')['sale_date'].rank(method='first', ascending=False);
    df_sales['growthstatus'] = np.where(
    	df_sales['amount'].isna(), 'First Sale',
        np.where(
    	df_sales['amount'] > df_sales['prev_transaction_amount'], 'Growth', 'No Growth'
    )
    )
    print(df_sales)
