---
title: "coding-sql-python"
date: 2026-04-29 05:11:25 
author: shajeebhameed
layout: page
slug: coding-sql-python
status: publish
---

## DataLemur — Free Questions & Solutions

> All non-premium questions from the DataLemur list, with problem + solution side by side. Categories: SQL · Python · Statistics · Machine Learning

* * *

## Table of Contents

  * [SQL — Easy](<https://claude.ai/chat/18e57822-6dbb-4ef5-bad0-ee9c0946b899#sql--easy>)
  * [SQL — Medium](<https://claude.ai/chat/18e57822-6dbb-4ef5-bad0-ee9c0946b899#sql--medium>)
  * [SQL — Hard](<https://claude.ai/chat/18e57822-6dbb-4ef5-bad0-ee9c0946b899#sql--hard>)
  * [Python — Easy](<https://claude.ai/chat/18e57822-6dbb-4ef5-bad0-ee9c0946b899#python--easy>)
  * [Python — Medium](<https://claude.ai/chat/18e57822-6dbb-4ef5-bad0-ee9c0946b899#python--medium>)
  * [Python — Hard](<https://claude.ai/chat/18e57822-6dbb-4ef5-bad0-ee9c0946b899#python--hard>)
  * [Statistics & Probability — Easy](<https://claude.ai/chat/18e57822-6dbb-4ef5-bad0-ee9c0946b899#statistics--probability--easy>)
  * [Statistics & Probability — Medium](<https://claude.ai/chat/18e57822-6dbb-4ef5-bad0-ee9c0946b899#statistics--probability--medium>)
  * [Statistics & Probability — Hard](<https://claude.ai/chat/18e57822-6dbb-4ef5-bad0-ee9c0946b899#statistics--probability--hard>)
  * [Machine Learning — Easy](<https://claude.ai/chat/18e57822-6dbb-4ef5-bad0-ee9c0946b899#machine-learning--easy>)
  * [Machine Learning — Medium](<https://claude.ai/chat/18e57822-6dbb-4ef5-bad0-ee9c0946b899#machine-learning--medium>)



* * *

## SQL — Easy

* * *

### 🐦 Twitter — Histogram of Tweets

|   
---|---  
**Problem**|  Write a query to count the number of tweets each user posted in 2022, then output a histogram showing how many users posted that many tweets (i.e., tweet count → number of users). **Table:** `tweets(tweet_id, user_id, msg, tweet_date)`  
      
    
    -- Step 1: count tweets per user in 2022
    -- Step 2: group by that count to get the histogram
    WITH tweets_per_user AS (
      SELECT user_id, COUNT(*) AS tweet_count
      FROM tweets
      WHERE EXTRACT(YEAR FROM tweet_date) = 2022
      GROUP BY user_id
    )
    SELECT tweet_count, COUNT(*) AS number_of_users
    FROM tweets_per_user
    GROUP BY tweet_count
    ORDER BY tweet_count;
    
    

* * *

### 💼 LinkedIn — Data Science Skills

|   
---|---  
**Problem**|  Find candidates who are proficient in **Python, Tableau, and PostgreSQL**. Output their candidate IDs. **Tables:** `candidates(candidate_id, skill)`  
      
    
    SELECT candidate_id
    FROM candidates
    WHERE skill IN ('Python', 'Tableau', 'PostgreSQL')
    GROUP BY candidate_id
    HAVING COUNT(DISTINCT skill) = 3
    ORDER BY candidate_id;
    
    

* * *

### 📘 Facebook — Page With No Likes

|   
---|---  
**Problem**|  Find page IDs that have **zero likes** (never appeared in the likes table). **Tables:** `pages(page_id, page_name)`, `page_likes(user_id, page_id, liked_date)`  
      
    
    -- Anti-join approach
    SELECT p.page_id
    FROM pages p
    LEFT JOIN page_likes pl ON p.page_id = pl.page_id
    WHERE pl.page_id IS NULL
    ORDER BY p.page_id;
    
    

* * *

### 🚗 Tesla — Unfinished Parts

|   
---|---  
**Problem**|  Find parts that have begun assembly but are **not yet finished**. **Table:** `parts_assembly(part, finish_date, assembly_step)` — finish_date is NULL if unfinished.  
      
    
    SELECT part, assembly_step
    FROM parts_assembly
    WHERE finish_date IS NULL;
    
    

* * *

### 📰 NY Times — Laptop vs. Mobile Viewership

|   
---|---  
**Problem**|  Count the number of laptop viewers and mobile viewers (tablet + phone) separately. **Table:** `viewership(user_id, device_type, view_time)` — device_type ∈ {laptop, tablet, phone}  
      
    
    SELECT
      COUNT(*) FILTER (WHERE device_type = 'laptop')           AS laptop_views,
      COUNT(*) FILTER (WHERE device_type IN ('tablet','phone')) AS mobile_views
    FROM viewership;
    
    -- MySQL alternative:
    -- SUM(CASE WHEN device_type = 'laptop' THEN 1 ELSE 0 END)
    
    

* * *

### 📘 Facebook — Average Post Hiatus (Part 1)

|   
---|---  
**Problem**|  For users who posted **at least twice in 2021** , find the average number of days between their first and last post. **Table:** `posts(user_id, post_date, post_content)`  
      
    
    SELECT
      user_id,
      MAX(post_date::DATE) - MIN(post_date::DATE) AS days_between
    FROM posts
    WHERE EXTRACT(YEAR FROM post_date) = 2021
    GROUP BY user_id
    HAVING COUNT(*) >= 2;
    
    

* * *

### 💻 Microsoft — Teams Power Users

|   
---|---  
**Problem**|  Find the top 2 senders of messages in August 2022. Output sender ID and message count. **Table:** `messages(message_id, sender_id, receiver_id, content, sent_date)`  
      
    
    SELECT sender_id, COUNT(*) AS message_count
    FROM messages
    WHERE EXTRACT(MONTH FROM sent_date) = 8
      AND EXTRACT(YEAR  FROM sent_date) = 2022
    GROUP BY sender_id
    ORDER BY message_count DESC
    LIMIT 2;
    
    

* * *

### 💼 LinkedIn — Duplicate Job Listings

|   
---|---  
**Problem**|  Find companies that posted **duplicate job listings** (same title and description, different job_id). Return the company IDs. **Table:** `job_listings(job_id, company_id, title, description)`  
      
    
    SELECT company_id
    FROM job_listings
    GROUP BY company_id, title, description
    HAVING COUNT(*) > 1;
    
    

* * *

### 📈 Robinhood — Cities With Completed Trades

|   
---|---  
**Problem**|  Find the top 3 cities with the most completed trade orders. **Tables:** `trades(order_id, user_id, price, quantity, status, timestamp)`, `users(user_id, city, email, signup_date)`  
      
    
    SELECT u.city, COUNT(*) AS total_orders
    FROM trades t
    JOIN users u ON t.user_id = u.user_id
    WHERE t.status = 'Completed'
    GROUP BY u.city
    ORDER BY total_orders DESC
    LIMIT 3;
    
    

* * *

### 🛒 Amazon — Average Review Ratings

|   
---|---  
**Problem**|  Find the average star rating for each product per month. Output month (as a number), product_id, and average rating rounded to 2 decimal places. **Table:** `reviews(review_id, user_id, submit_date, product_id, stars)`  
      
    
    SELECT
      EXTRACT(MONTH FROM submit_date) AS mth,
      product_id,
      ROUND(AVG(stars), 2)            AS avg_stars
    FROM reviews
    GROUP BY mth, product_id
    ORDER BY mth, product_id;
    
    

* * *

### 💰 FAANG — Well Paid Employees

|   
---|---  
**Problem**|  Find employees who earn more than their direct manager. **Table:** `employee(employee_id, name, salary, department_id, manager_id)`  
      
    
    SELECT e.employee_id, e.name AS employee_name
    FROM employee e
    JOIN employee m ON e.manager_id = m.employee_id
    WHERE e.salary > m.salary;
    
    

* * *

### 💳 PayPal — Final Account Balance

|   
---|---  
**Problem**|  Given a transaction table with deposits and withdrawals, compute the final balance for each account. **Table:** `transactions(transaction_id, account_id, transaction_type, amount)`  
      
    
    SELECT
      account_id,
      SUM(CASE WHEN transaction_type = 'Deposit'    THEN  amount
               WHEN transaction_type = 'Withdrawal' THEN -amount
          END) AS final_balance
    FROM transactions
    GROUP BY account_id;
    
    

* * *

### 📘 Facebook — App Click-through Rate (CTR)

|   
---|---  
**Problem**|  Compute the CTR (clicks / impressions × 100) for each app in 2022. **Table:** `events(app_id, event_type, timestamp)` — event_type ∈ {impression, click}  
      
    
    SELECT
      app_id,
      ROUND(
        100.0 *
        SUM(CASE WHEN event_type = 'click'      THEN 1 ELSE 0 END) /
        SUM(CASE WHEN event_type = 'impression' THEN 1 ELSE 0 END),
      2) AS ctr
    FROM events
    WHERE EXTRACT(YEAR FROM timestamp) = 2022
    GROUP BY app_id;
    
    

* * *

### 🎵 TikTok — Second Day Confirmation

|   
---|---  
**Problem**|  Find users who confirmed their account **exactly the day after** they signed up. **Tables:** `emails(email_id, user_id, signup_date)`, `texts(text_id, email_id, signup_action, action_date)`  
      
    
    SELECT e.user_id
    FROM emails e
    JOIN texts t ON e.email_id = t.email_id
    WHERE t.signup_action = 'Confirmed'
      AND t.action_date = e.signup_date + INTERVAL '1 day';
    
    

* * *

### 🏥 IBM — IBM db2 Product Analytics

|   
---|---  
**Problem**|  Find the number of employees who did **not call** any product in Q2 2023 (April–June). **Tables:** `employees(employee_id, ...)`, `phone_calls(caller_id, receiver_id, call_date)`  
      
    
    SELECT COUNT(DISTINCT e.employee_id) AS total_employees
    FROM employees e
    WHERE e.employee_id NOT IN (
      SELECT caller_id
      FROM phone_calls
      WHERE call_date BETWEEN '2023-04-01' AND '2023-06-30'
    );
    
    

* * *

### 💳 JPMorgan — Cards Issued Difference

|   
---|---  
**Problem**|  For each card launch, find the difference between the max and min number of cards issued per month. **Table:** `monthly_cards_issued(issue_month, issue_year, card_name, issued_amount)`  
      
    
    SELECT
      card_name,
      MAX(issued_amount) - MIN(issued_amount) AS difference
    FROM monthly_cards_issued
    GROUP BY card_name
    ORDER BY difference DESC;
    
    

* * *

### 🏪 Alibaba — Compressed Mean

|   
---|---  
**Problem**|  A table stores (item_count, order_occurrences). Calculate the **mean number of items per order** , rounded to 1 decimal place. **Table:** `items_per_order(item_count, order_occurrences)`  
      
    
    SELECT
      ROUND(
        SUM(item_count * order_occurrences) * 1.0 / SUM(order_occurrences),
      1) AS mean
    FROM items_per_order;
    
    

* * *

### 💊 CVS Health — Pharmacy Analytics (Part 1)

|   
---|---  
**Problem**|  Find the top 3 drugs with the highest total profit (total_sales − cogs). **Table:** `pharmacy_sales(product_id, units_sold, total_sales, cogs, drug, ...)`  
      
    
    SELECT drug, total_sales - cogs AS total_profit
    FROM pharmacy_sales
    ORDER BY total_profit DESC
    LIMIT 3;
    
    

* * *

### 💊 CVS Health — Pharmacy Analytics (Part 2)

|   
---|---  
**Problem**|  Find drugs that **lost money** (cogs > total_sales). Return drug name and total loss (as a positive number), sorted by largest loss.  
      
    
    SELECT drug, cogs - total_sales AS total_loss
    FROM pharmacy_sales
    WHERE cogs > total_sales
    ORDER BY total_loss DESC;
    
    

* * *

### 💊 CVS Health — Pharmacy Analytics (Part 3)

|   
---|---  
**Problem**|  For each manufacturer, count drugs and compute total sales. For manufacturers with at least one drug losing money, also show total loss (else 0). Output sorted by total drugs descending.  
      
    
    SELECT
      manufacturer,
      COUNT(*)                                                        AS drug_count,
      SUM(total_sales)                                                AS total_sales,
      SUM(CASE WHEN cogs > total_sales THEN cogs - total_sales ELSE 0 END) AS total_loss
    FROM pharmacy_sales
    GROUP BY manufacturer
    ORDER BY drug_count DESC;
    
    

* * *

### 🏥 UnitedHealth — Patient Support Analysis (Part 1)

|   
---|---  
**Problem**|  Count the number of patients who called more than once per day (i.e., who had > 1 call on any single day). **Table:** `callers(policy_holder_id, case_id, call_category, call_date, call_duration_secs)`  
      
    
    SELECT COUNT(DISTINCT policy_holder_id) AS member_count
    FROM (
      SELECT policy_holder_id, call_date, COUNT(*) AS daily_calls
      FROM callers
      GROUP BY policy_holder_id, call_date
      HAVING COUNT(*) > 1
    ) sub;
    
    

* * *

## SQL — Medium

* * *

### 🚗 Uber — User's Third Transaction

|   
---|---  
**Problem**|  Return each user's **third transaction** (if they have one). **Table:** `transactions(user_id, spend, transaction_date)`  
      
    
    WITH ranked AS (
      SELECT *,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY transaction_date) AS rn
      FROM transactions
    )
    SELECT user_id, spend, transaction_date
    FROM ranked
    WHERE rn = 3;
    
    

* * *

### 💼 FAANG — Second Highest Salary

|   
---|---  
**Problem**|  Find the second-highest distinct salary. If it doesn't exist, return NULL. **Table:** `employee(employee_id, salary)`  
      
    
    SELECT MAX(salary) AS second_highest_salary
    FROM employee
    WHERE salary < (SELECT MAX(salary) FROM employee);
    
    

* * *

### 📸 Snapchat — Sending vs. Opening Snaps

|   
---|---  
**Problem**|  For each age bucket, compute the percentage of time spent sending vs. opening snaps (rounded to 2 dp). **Tables:** `activities(activity_id, user_id, type, time_spent, activity_date)`, `age_breakdown(user_id, age_bucket)`  
      
    
    SELECT
      ab.age_bucket,
      ROUND(100.0 * SUM(CASE WHEN a.type = 'send' THEN a.time_spent ELSE 0 END)
            / SUM(a.time_spent), 2) AS send_perc,
      ROUND(100.0 * SUM(CASE WHEN a.type = 'open' THEN a.time_spent ELSE 0 END)
            / SUM(a.time_spent), 2) AS open_perc
    FROM activities a
    JOIN age_breakdown ab ON a.user_id = ab.user_id
    WHERE a.type IN ('send', 'open')
    GROUP BY ab.age_bucket;
    
    

* * *

### 🐦 Twitter — Tweets' Rolling Averages

|   
---|---  
**Problem**|  For each user, compute the **3-day rolling average** of tweets. **Table:** `tweets(user_id, tweet_date, tweet_count)`  
      
    
    SELECT
      user_id,
      tweet_date,
      ROUND(AVG(tweet_count) OVER (
        PARTITION BY user_id
        ORDER BY tweet_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
      ), 2) AS rolling_avg_3d
    FROM tweets;
    
    

* * *

### 🛒 Amazon — Highest-Grossing Items

|   
---|---  
**Problem**|  For each category, find the **top 2 products** by total spend in 2022. **Table:** `product_spend(category, product, user_id, spend, transaction_date)`  
      
    
    WITH ranked AS (
      SELECT
        category, product,
        SUM(spend) AS total_spend,
        RANK() OVER (PARTITION BY category ORDER BY SUM(spend) DESC) AS rnk
      FROM product_spend
      WHERE EXTRACT(YEAR FROM transaction_date) = 2022
      GROUP BY category, product
    )
    SELECT category, product, total_spend
    FROM ranked
    WHERE rnk <= 2;
    
    

* * *

### 💰 FAANG — Top Three Salaries

|   
---|---  
**Problem**|  Find employees in each department who are among the **top 3 unique salaries**. **Table:** `employee(department_id, name, salary)`  
      
    
    WITH ranked AS (
      SELECT
        department_id, name, salary,
        DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rnk
      FROM employee
    )
    SELECT department_id, name, salary
    FROM ranked
    WHERE rnk <= 3;
    
    

* * *

### 🎵 TikTok — Signup Activation Rate

|   
---|---  
**Problem**|  Find the activation rate = users who confirmed signup / total users who were texted. **Tables:** `emails(email_id, user_id, signup_date)`, `texts(text_id, email_id, signup_action)`  
      
    
    SELECT
      ROUND(COUNT(t.email_id) FILTER (WHERE t.signup_action = 'Confirmed') * 1.0
            / COUNT(e.email_id), 2) AS activation_rate
    FROM emails e
    LEFT JOIN texts t ON e.email_id = t.email_id;
    
    

* * *

### 🎵 Spotify — Spotify Streaming History

|   
---|---  
**Problem**|  For each user, compute the total listen time and the cumulative listen time ordered by song listening date. **Table:** `songs_history(history_id, user_id, song_id, song_plays, listen_time)`  
      
    
    SELECT
      user_id,
      song_id,
      listen_time,
      SUM(listen_time) OVER (PARTITION BY user_id ORDER BY history_id) AS cumulative_listen_time
    FROM songs_history;
    
    

* * *

### 💻 Microsoft — Supercloud Customer

|   
---|---  
**Problem**|  A "supercloud" customer buys at least one product from **every** product category. Find all such customer IDs. **Tables:** `customer_contracts(customer_id, product_id, amount)`, `products(product_id, product_category, product_name)`  
      
    
    SELECT cc.customer_id
    FROM customer_contracts cc
    JOIN products p ON cc.product_id = p.product_id
    GROUP BY cc.customer_id
    HAVING COUNT(DISTINCT p.product_category) = (SELECT COUNT(DISTINCT product_category) FROM products);
    
    

* * *

### 📊 Google — Odd and Even Measurements

|   
---|---  
**Problem**|  For each day, sum the measurements at **odd-numbered** positions and even-numbered positions separately. **Table:** `measurements(measurement_id, measurement_value, measurement_time)`  
      
    
    WITH ranked AS (
      SELECT *,
        ROW_NUMBER() OVER (PARTITION BY measurement_time::DATE ORDER BY measurement_time) AS rn
      FROM measurements
    )
    SELECT
      measurement_time::DATE AS measurement_day,
      SUM(CASE WHEN rn % 2 = 1 THEN measurement_value ELSE 0 END) AS odd_sum,
      SUM(CASE WHEN rn % 2 = 0 THEN measurement_value ELSE 0 END) AS even_sum
    FROM ranked
    GROUP BY measurement_day
    ORDER BY measurement_day;
    
    

* * *

### 🍕 Zomato — Swapped Food Delivery

|   
---|---  
**Problem**|  Items were delivered in the wrong order. Swap item_id for every consecutive pair (1↔2, 3↔4, etc.). The last item if odd-count stays unchanged. **Table:** `items(item_id, item_name)`  
      
    
    SELECT
      CASE
        WHEN item_id % 2 = 1 AND item_id + 1 <= (SELECT MAX(item_id) FROM items) THEN item_id + 1
        WHEN item_id % 2 = 0 THEN item_id - 1
        ELSE item_id
      END AS corrected_item_id,
      item_name
    FROM items
    ORDER BY corrected_item_id;
    
    

* * *

### 📈 Bloomberg — FAANG Stock Min-Max (Part 1)

|   
---|---  
**Problem**|  Find the **lowest opening price** and the **highest opening price** for each FAANG stock in each month. **Table:** `stock_prices(date, ticker, open, high, low, close)`  
      
    
    SELECT
      ticker,
      DATE_TRUNC('month', date) AS month,
      MIN(open) AS lowest_open,
      MAX(open) AS highest_open
    FROM stock_prices
    GROUP BY ticker, DATE_TRUNC('month', date)
    ORDER BY ticker, month;
    
    

* * *

### 🛒 Amazon — Best-Selling Product

|   
---|---  
**Problem**|  Find the best-selling product in each category. On ties, pick the one with the higher review rating. **Tables:** `products(product_id, product_name, category_name)`, `product_sales(product_id, sales_quantity, rating)`  
      
    
    WITH ranked AS (
      SELECT
        p.category_name,
        p.product_name,
        ps.sales_quantity,
        ps.rating,
        ROW_NUMBER() OVER (
          PARTITION BY p.category_name
          ORDER BY ps.sales_quantity DESC, ps.rating DESC
        ) AS rn
      FROM products p
      JOIN product_sales ps ON p.product_id = ps.product_id
    )
    SELECT category_name, product_name
    FROM ranked
    WHERE rn = 1
    ORDER BY category_name;
    
    

* * *

### 🛒 Amazon — User Shopping Sprees

|   
---|---  
**Problem**|  Find users who purchased on **3 or more consecutive days**. **Table:** `transactions(user_id, amount, transaction_date)`  
      
    
    WITH dated AS (
      SELECT DISTINCT user_id, transaction_date
      FROM transactions
    ),
    lagged AS (
      SELECT *,
        LAG(transaction_date, 1) OVER (PARTITION BY user_id ORDER BY transaction_date) AS prev1,
        LAG(transaction_date, 2) OVER (PARTITION BY user_id ORDER BY transaction_date) AS prev2
      FROM dated
    )
    SELECT DISTINCT user_id
    FROM lagged
    WHERE transaction_date = prev1 + INTERVAL '1 day'
      AND prev1           = prev2 + INTERVAL '1 day';
    
    

* * *

### 🏪 Walmart — Histogram of Users and Purchases

|   
---|---  
**Problem**|  For each user's most recent transaction date, count how many products they purchased. **Table:** `user_transactions(product_id, user_id, spend, transaction_date)`  
      
    
    WITH latest AS (
      SELECT user_id, MAX(transaction_date) AS latest_date
      FROM user_transactions
      GROUP BY user_id
    )
    SELECT t.transaction_date, t.user_id, COUNT(*) AS purchase_count
    FROM user_transactions t
    JOIN latest l ON t.user_id = l.user_id AND t.transaction_date = l.latest_date
    GROUP BY t.transaction_date, t.user_id
    ORDER BY t.transaction_date;
    
    

* * *

### 🏪 Alibaba — Compressed Mode

|   
---|---  
**Problem**|  Find the most frequently occurring item count (the **mode**). If there's a tie, return all tied values ascending. **Table:** `items_per_order(item_count, order_occurrences)`  
      
    
    SELECT item_count AS mode
    FROM items_per_order
    WHERE order_occurrences = (SELECT MAX(order_occurrences) FROM items_per_order)
    ORDER BY item_count;
    
    

* * *

### 💳 JPMorgan — Card Launch Success

|   
---|---  
**Problem**|  Find the number of cards issued on the **launch month** for each card. Order by issued_amount descending. **Table:** `monthly_cards_issued(issue_month, issue_year, card_name, issued_amount)`  
      
    
    WITH launch AS (
      SELECT card_name, MIN(issue_year * 12 + issue_month) AS launch_ym
      FROM monthly_cards_issued
      GROUP BY card_name
    )
    SELECT m.card_name, m.issued_amount
    FROM monthly_cards_issued m
    JOIN launch l
      ON m.card_name = l.card_name
      AND (m.issue_year * 12 + m.issue_month) = l.launch_ym
    ORDER BY m.issued_amount DESC;
    
    

* * *

### 📞 Verizon — International Call Percentage

|   
---|---  
**Problem**|  What % of calls are international (caller and receiver in different countries)? Round to 1 dp. **Tables:** `phone_calls(caller_id, receiver_id, duration)`, `phone_info(caller_id, country_id, ...)`  
      
    
    SELECT
      ROUND(100.0 *
        SUM(CASE WHEN pi1.country_id <> pi2.country_id THEN 1 ELSE 0 END)
        / COUNT(*),
      1) AS international_call_pct
    FROM phone_calls pc
    JOIN phone_info pi1 ON pc.caller_id   = pi1.caller_id
    JOIN phone_info pi2 ON pc.receiver_id = pi2.caller_id;
    
    

* * *

### 🏥 UnitedHealth — Patient Support Analysis (Part 2)

|   
---|---  
**Problem**|  Find the % of patients with more than **one** call and the % with more than **five minutes** total call time. **Table:** `callers(policy_holder_id, case_id, call_category, call_date, call_duration_secs)`  
      
    
    WITH stats AS (
      SELECT
        policy_holder_id,
        COUNT(*)                  AS call_count,
        SUM(call_duration_secs)   AS total_duration
      FROM callers
      GROUP BY policy_holder_id
    )
    SELECT
      ROUND(100.0 * SUM(CASE WHEN call_count > 1 THEN 1 ELSE 0 END)     / COUNT(*), 1) AS multi_call_pct,
      ROUND(100.0 * SUM(CASE WHEN total_duration > 300 THEN 1 ELSE 0 END) / COUNT(*), 1) AS long_call_pct
    FROM stats;
    
    

* * *

## SQL — Hard

* * *

### 📘 Facebook — Active User Retention

|   
---|---  
**Problem**|  Count monthly active users who were also active in the **previous month** (retained users) for July 2022. **Table:** `user_actions(user_id, event_id, event_type, event_date)`  
      
    
    SELECT
      EXTRACT(MONTH FROM curr.event_date) AS month,
      COUNT(DISTINCT curr.user_id)        AS monthly_active_users
    FROM user_actions curr
    WHERE EXTRACT(MONTH FROM curr.event_date) = 7
      AND EXTRACT(YEAR FROM curr.event_date) = 2022
      AND EXISTS (
        SELECT 1 FROM user_actions prev
        WHERE prev.user_id = curr.user_id
          AND EXTRACT(MONTH FROM prev.event_date) = 6
          AND EXTRACT(YEAR FROM prev.event_date) = 2022
      )
    GROUP BY month;
    
    

* * *

### 🛋️ Wayfair — Y-on-Y Growth Rate

|   
---|---  
**Problem**|  For each product, compute total spend and the year-over-year growth rate (%). **Table:** `user_transactions(transaction_id, product_id, spend, transaction_date)`  
      
    
    WITH yearly AS (
      SELECT
        EXTRACT(YEAR FROM transaction_date) AS yr,
        product_id,
        SUM(spend) AS total_spend
      FROM user_transactions
      GROUP BY yr, product_id
    )
    SELECT
      yr,
      product_id,
      total_spend AS curr_year_spend,
      LAG(total_spend) OVER (PARTITION BY product_id ORDER BY yr) AS prev_year_spend,
      ROUND(
        100.0 * (total_spend - LAG(total_spend) OVER (PARTITION BY product_id ORDER BY yr))
        / LAG(total_spend) OVER (PARTITION BY product_id ORDER BY yr),
      2) AS yoy_rate
    FROM yearly
    ORDER BY product_id, yr;
    
    

* * *

### 🛒 Amazon — Maximize Prime Item Inventory

|   
---|---  
**Problem**|  A shipment has a fixed number of slots. Fill as many **prime** items as possible; fill remaining with non-prime. Return how many of each fit. **Table:** `inventory(item_id, item_type, item_category, square_footage)`  
      
    
    WITH summary AS (
      SELECT item_type,
        COUNT(*)        AS item_count,
        SUM(square_footage) AS total_sqft
      FROM inventory
      GROUP BY item_type
    ),
    prime AS (SELECT * FROM summary WHERE item_type = 'prime_eligible'),
    non   AS (SELECT * FROM summary WHERE item_type = 'not_prime')
    SELECT
      'prime_eligible'                                  AS item_type,
      FLOOR(500000.0 / prime.total_sqft) * prime.item_count AS item_count
    FROM prime
    UNION ALL
    SELECT
      'not_prime',
      FLOOR((500000.0 - FLOOR(500000.0 / prime.total_sqft) * prime.total_sqft)
            / non.total_sqft) * non.item_count
    FROM prime, non;
    
    

* * *

### 📊 Google — Median Google Search Frequency

|   
---|---  
**Problem**|  Find the **median** number of searches per user. **Table:** `search_frequency(searches, num_users)` — compressed format.  
      
    
    WITH expanded AS (
      SELECT searches
      FROM search_frequency
      -- generate one row per user
      JOIN generate_series(1, num_users) ON true
    )
    SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY searches) AS median
    FROM expanded;
    
    

* * *

### 📘 Facebook — Advertiser Status

|   
---|---  
**Problem**|  Update advertiser status based on current payment. Rules: existing + paid → existing; existing + not paid → churn; churned + paid → re-activated; new + paid → new; else → other. **Tables:** `advertiser(user_id, status)`, `daily_pay(user_id, paid)`  
      
    
    SELECT
      COALESCE(a.user_id, dp.user_id) AS user_id,
      CASE
        WHEN dp.paid IS NOT NULL AND a.status IN ('existing','new','re-activated') THEN 'existing'
        WHEN dp.paid IS NOT NULL AND a.status = 'churn'                            THEN 're-activated'
        WHEN dp.paid IS NOT NULL AND a.status IS NULL                              THEN 'new'
        WHEN dp.paid IS NULL     AND a.status IS NOT NULL                          THEN 'churn'
        ELSE 'churn'
      END AS new_status
    FROM advertiser a
    FULL OUTER JOIN daily_pay dp ON a.user_id = dp.user_id
    ORDER BY user_id;
    
    

* * *

### 🗓️ Intuit — Consecutive Filing Years

|   
---|---  
**Problem**|  Find users who filed taxes for **at least 3 consecutive years**. **Table:** `filed_taxes(filing_id, user_id, filing_date)`  
      
    
    WITH yearly AS (
      SELECT DISTINCT user_id, EXTRACT(YEAR FROM filing_date) AS yr
      FROM filed_taxes
    ),
    gaps AS (
      SELECT *,
        yr - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY yr) AS grp
      FROM yearly
    ),
    streaks AS (
      SELECT user_id, COUNT(*) AS streak_len
      FROM gaps
      GROUP BY user_id, grp
    )
    SELECT DISTINCT user_id
    FROM streaks
    WHERE streak_len >= 3
    ORDER BY user_id;
    
    

* * *

### ❄️ Snowflake — Marketing Touch Streak

|   
---|---  
**Problem**|  Find the longest streak of consecutive days each user had a marketing touch. **Table:** `user_touches(user_id, event_id, event_type, event_date)`  
      
    
    WITH daily AS (
      SELECT DISTINCT user_id, event_date::DATE AS d
      FROM user_touches
    ),
    grp AS (
      SELECT *,
        d - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY d) * INTERVAL '1 day' AS grp
      FROM daily
    ),
    streaks AS (
      SELECT user_id, COUNT(*) AS streak_len
      FROM grp
      GROUP BY user_id, grp
    )
    SELECT user_id, MAX(streak_len) AS longest_streak
    FROM streaks
    GROUP BY user_id
    ORDER BY user_id;
    
    

* * *

### 🍕 McKinsey — 3-Topping Pizzas

|   
---|---  
**Problem**|  Find all unique combinations of exactly 3 toppings. Output them as a comma-separated string, ordered alphabetically within each combo, with total cost. **Table:** `pizza_toppings(topping_name, ingredient_cost)`  
      
    
    SELECT
      CONCAT(p1.topping_name, ', ', p2.topping_name, ', ', p3.topping_name) AS pizza,
      p1.ingredient_cost + p2.ingredient_cost + p3.ingredient_cost           AS total_cost
    FROM pizza_toppings p1
    JOIN pizza_toppings p2 ON p1.topping_name < p2.topping_name
    JOIN pizza_toppings p3 ON p2.topping_name < p3.topping_name
    ORDER BY total_cost DESC, pizza;
    
    

* * *

### 📘 Facebook — Reactivated Users

|   
---|---  
**Problem**|  Find the number and % of users who were **inactive last month** but **active this month** (reactivated). **Table:** `user_actions(user_id, event_date, event_type)`  
      
    
    WITH monthly AS (
      SELECT DISTINCT user_id,
        DATE_TRUNC('month', event_date) AS activity_month
      FROM user_actions
    ),
    curr_month AS (SELECT MAX(activity_month) FROM monthly),
    prev_month AS (SELECT MAX(activity_month) FROM monthly WHERE activity_month < (SELECT * FROM curr_month))
    SELECT
      COUNT(*) AS reactivated_users,
      ROUND(100.0 * COUNT(*) / (SELECT COUNT(DISTINCT user_id) FROM monthly WHERE activity_month = (SELECT * FROM curr_month)), 2) AS pct
    FROM monthly m_curr
    WHERE m_curr.activity_month = (SELECT * FROM curr_month)
      AND NOT EXISTS (
        SELECT 1 FROM monthly m_prev
        WHERE m_prev.user_id = m_curr.user_id
          AND m_prev.activity_month = (SELECT * FROM prev_month)
      );
    
    

* * *

### 📊 Google — Senior Managers

|   
---|---  
**Problem**|  Find employees with the title "Senior" or "Manager" who have **no direct reports** themselves. **Table:** `employees(employee_id, name, title, reports_to)`  
      
    
    SELECT e.employee_id, e.name, e.title
    FROM employees e
    WHERE e.title ILIKE '%Senior%' OR e.title ILIKE '%Manager%'
      AND e.employee_id NOT IN (
        SELECT DISTINCT reports_to FROM employees WHERE reports_to IS NOT NULL
      );
    
    

* * *

### 💳 Stripe — Repeated Payments

|   
---|---  
**Problem**|  Find transactions that are **duplicate payments** — same merchant, same amount, same credit card, within 10 minutes of the previous one. **Table:** `transactions(transaction_id, merchant_id, credit_card_id, amount, transaction_timestamp)`  
      
    
    WITH lagged AS (
      SELECT *,
        LAG(transaction_timestamp) OVER (
          PARTITION BY merchant_id, credit_card_id, amount
          ORDER BY transaction_timestamp
        ) AS prev_ts
      FROM transactions
    )
    SELECT COUNT(*) AS payment_count
    FROM lagged
    WHERE transaction_timestamp - prev_ts <= INTERVAL '10 minutes';
    
    

* * *

### 🖥️ Amazon — Server Utilization Time

|   
---|---  
**Problem**|  Calculate total server **uptime** in full days across all servers. **Table:** `server_utilization(server_id, session_status, status_time)` — status ∈ {start, stop}  
      
    
    WITH sessions AS (
      SELECT
        server_id,
        status_time AS start_time,
        LEAD(status_time) OVER (PARTITION BY server_id ORDER BY status_time) AS stop_time
      FROM server_utilization
      WHERE session_status = 'start'
    )
    SELECT
      FLOOR(SUM(EXTRACT(EPOCH FROM (stop_time - start_time))) / 86400) AS total_uptime_days
    FROM sessions
    WHERE stop_time IS NOT NULL;
    
    

* * *

## Python — Easy

* * *

### 🏦 Capital One — Base 13 Conversion

|   
---|---  
**Problem**|  Convert a **base-10 integer** to its base-13 string representation. Digits 10, 11, 12 are represented as A, B, C.  
      
    
    def base_13(num: int) -> str:
        if num == 0:
            return "0"
        digits = "0123456789ABC"
        result = ""
        while num > 0:
            result = digits[num % 13] + result
            num //= 13
        return result
    
    print(base_13(13))   # "10"
    print(base_13(27))   # "21"
    print(base_13(155))  # "BC"
    
    

* * *

### 🐍 ServiceNow — Same Stripes

|   
---|---  
**Problem**|  Given two lists, return `True` if they have the **same pattern of repeated/non-repeated elements** at each index.  
      
    
    def same_stripes(a: list, b: list) -> bool:
        def pattern(lst):
            seen, result, mapping = {}, [], {}
            for x in lst:
                if x not in mapping:
                    mapping[x] = len(mapping)
                result.append(mapping[x])
            return result
        return pattern(a) == pattern(b)
    
    print(same_stripes([1,2,3,1],[4,5,6,4]))  # True
    print(same_stripes([1,2,3,1],[4,5,6,7]))  # False
    
    

* * *

### 💻 Microsoft — Factorial Formula

|   
---|---  
**Problem**|  Return the factorial of a non-negative integer `n`.  
      
    
    def factorial(n: int) -> int:
        result = 1
        for i in range(2, n + 1):
            result *= i
        return result
    
    # Or one-liner:
    import math
    factorial = math.factorial
    
    print(factorial(5))   # 120
    print(factorial(0))   # 1
    
    

* * *

### 🛒 Amazon — Intersection of Two Lists

|   
---|---  
**Problem**|  Return a **sorted list** of unique integers that appear in both input lists.  
      
    
    def intersection(a: list, b: list) -> list:
        return sorted(set(a) & set(b))
    
    print(intersection([1,2,3,4],[3,4,5,6]))  # [3, 4]
    print(intersection([1,2],[3,4]))           # []
    
    

* * *

### 🎵 Spotify — Another One

|   
---|---  
**Problem**|  Given a list of integers, replace each occurrence of the **maximum value** with the **second maximum value**.  
      
    
    def replace_max(nums: list) -> list:
        m1 = max(nums)
        m2 = max(x for x in nums if x != m1)
        return [m2 if x == m1 else x for x in nums]
    
    print(replace_max([1, 2, 3, 3]))  # [1, 2, 2, 2]
    
    

* * *

### 🧮 Intuit — Weakest Strong Link

|   
---|---  
**Problem**|  A "strong link" is a number greater than the **average** of its two neighbours. Find the minimum such value; return -1 if none.  
      
    
    def weakest_strong_link(nums: list) -> int:
        n = len(nums)
        strong = []
        for i in range(n):
            left  = nums[(i - 1) % n]
            right = nums[(i + 1) % n]
            if nums[i] > (left + right) / 2:
                strong.append(nums[i])
        return min(strong) if strong else -1
    
    print(weakest_strong_link([1, 3, 2]))   # 3
    print(weakest_strong_link([1, 1, 1]))   # -1
    
    

* * *

### 🔢 Google — Fizz Buzz Sum

|   
---|---  
**Problem**|  Return the **sum** of all integers from 1 to n that are divisible by 3 **or** 5.  
      
    
    def fizz_buzz_sum(n: int) -> int:
        return sum(x for x in range(1, n + 1) if x % 3 == 0 or x % 5 == 0)
    
    print(fizz_buzz_sum(10))  # 33  (3+5+6+9+10)
    print(fizz_buzz_sum(15))  # 60
    
    

* * *

### 💹 Fintech — Compound Interest

|   
---|---  
**Problem**|  Given principal `p`, annual rate `r` (as decimal), compounds per year `n`, and years `t`, return the final amount rounded to 2 dp. Formula: `A = p * (1 + r/n)^(n*t)`  
      
    
    def compound_interest(p: float, r: float, n: int, t: int) -> float:
        return round(p * (1 + r / n) ** (n * t), 2)
    
    print(compound_interest(1000, 0.05, 12, 10))  # 1647.01
    
    

* * *

### 🎵 TikTok — Triangular Sum

|   
---|---  
**Problem**|  Given `n`, return the sum of all triangular numbers from T(1) to T(n). T(k) = k*(k+1)/2.  
      
    
    def triangular_sum(n: int) -> int:
        return sum(k * (k + 1) // 2 for k in range(1, n + 1))
    
    print(triangular_sum(4))  # 1+3+6+10 = 20
    
    

* * *

### 🍎 Apple — Contains Duplicate

|   
---|---  
**Problem**|  Return `True` if any value in the list appears at least twice; `False` if all are distinct.  
      
    
    def contains_duplicate(nums: list) -> bool:
        return len(nums) != len(set(nums))
    
    print(contains_duplicate([1, 3, 5, 7, 1]))  # True
    print(contains_duplicate([1, 3, 5, 7]))     # False
    
    

* * *

### 🏛️ Palantir — Roman to Integer

|   
---|---  
**Problem**|  Convert a Roman numeral string to an integer.  
      
    
    def roman_to_int(s: str) -> int:
        vals = {'I':1,'V':5,'X':10,'L':50,'C':100,'D':500,'M':1000}
        total = 0
        for i in range(len(s)):
            if i + 1 < len(s) and vals[s[i]] < vals[s[i+1]]:
                total -= vals[s[i]]
            else:
                total += vals[s[i]]
        return total
    
    print(roman_to_int("III"))   # 3
    print(roman_to_int("MCMXC")) # 1990
    
    

* * *

### 🚗 Tesla — Counting Letters in Numbers

|   
---|---  
**Problem**|  Write out numbers from 1 to n in English words (e.g. "one", "two"). Count **only the letters** (no spaces or hyphens).  
      
    
    def count_letters(n: int) -> int:
        ones = ["","one","two","three","four","five","six","seven","eight","nine",
                "ten","eleven","twelve","thirteen","fourteen","fifteen","sixteen",
                "seventeen","eighteen","nineteen"]
        tens = ["","","twenty","thirty","forty","fifty","sixty","seventy","eighty","ninety"]
        def word(x):
            if x == 0:    return ""
            if x < 20:   return ones[x]
            if x < 100:  return tens[x//10] + ones[x%10]
            if x < 1000: return ones[x//100] + "hundred" + ("and" + word(x%100) if x%100 else "")
            return ones[x//1000] + "thousand" + word(x%1000)
        return sum(len(word(i)) for i in range(1, n+1))
    
    print(count_letters(5))   # 19  (one+two+three+four+five)
    
    

* * *

### ✅ Workday — Is Anagram?

|   
---|---  
**Problem**|  Return `True` if two strings are anagrams of each other (case-insensitive, ignoring spaces).  
      
    
    from collections import Counter
    
    def is_anagram(s: str, t: str) -> bool:
        clean = lambda x: x.lower().replace(" ", "")
        return Counter(clean(s)) == Counter(clean(t))
    
    print(is_anagram("Astronomer", "Moon starer"))  # True
    print(is_anagram("hello", "world"))             # False
    
    

* * *

### 🚗 Uber — Pascal's Triangle

|   
---|---  
**Problem**|  Return the first `n` rows of Pascal's Triangle as a list of lists.  
      
    
    def pascals_triangle(n: int) -> list:
        triangle = []
        for i in range(n):
            row = [1] * (i + 1)
            for j in range(1, i):
                row[j] = triangle[i-1][j-1] + triangle[i-1][j]
            triangle.append(row)
        return triangle
    
    print(pascals_triangle(4))
    # [[1],[1,1],[1,2,1],[1,3,3,1]]
    
    

* * *

### 🎵 Spotify — Is Palindrome

|   
---|---  
**Problem**|  Return `True` if the string is a palindrome, considering only alphanumeric characters and ignoring case.  
      
    
    def is_palindrome(s: str) -> bool:
        clean = [c.lower() for c in s if c.isalnum()]
        return clean == clean[::-1]
    
    print(is_palindrome("A man, a plan, a canal: Panama"))  # True
    print(is_palindrome("race a car"))                       # False
    
    

* * *

## Python — Medium

* * *

### 💰 FAANG — Longest Consecutive Sequence

|   
---|---  
**Problem**|  Return the length of the longest consecutive integer sequence in an unsorted list.  
      
    
    def longest_consecutive(nums: list) -> int:
        num_set = set(nums)
        best = 0
        for n in num_set:
            if n - 1 not in num_set:
                cur, streak = n, 1
                while cur + 1 in num_set:
                    cur += 1; streak += 1
                best = max(best, streak)
        return best
    
    print(longest_consecutive([100,4,200,1,3,2]))  # 4
    
    

* * *

### 🔢 Fintech — Looping Number

|   
---|---  
**Problem**|  Given integer `n`, apply: if even → n/2; if odd → 3n+1. Count the steps until reaching 1 (Collatz conjecture).  
      
    
    def collatz_steps(n: int) -> int:
        steps = 0
        while n != 1:
            n = n // 2 if n % 2 == 0 else 3 * n + 1
            steps += 1
        return steps
    
    print(collatz_steps(6))   # 8
    print(collatz_steps(27))  # 111
    
    

* * *

### 🎁 FAANG — Gift Card Satisfaction

|   
---|---  
**Problem**|  Given a list of survey scores (1–10), return the % of respondents who are "satisfied" (score >= 9), rounded to 2 dp.  
      
    
    def satisfaction_rate(scores: list) -> float:
        if not scores:
            return 0.0
        return round(100 * sum(1 for s in scores if s >= 9) / len(scores), 2)
    
    print(satisfaction_rate([10, 9, 8, 7, 10]))  # 60.0
    
    

* * *

### 📊 Salesforce — Matrix Rotation

|   
---|---  
**Problem**|  Rotate an N×N matrix **90 degrees clockwise** in-place.  
      
    
    def rotate(matrix: list) -> None:
        n = len(matrix)
        # Transpose
        for i in range(n):
            for j in range(i + 1, n):
                matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
        # Reverse each row
        for row in matrix:
            row.reverse()
    
    m = [[1,2,3],[4,5,6],[7,8,9]]
    rotate(m)
    print(m)  # [[7,4,1],[8,5,2],[9,6,3]]
    
    

* * *

### 📐 Google — Min Amplitude

|   
---|---  
**Problem**|  Given a list of integers, you can remove up to **3 elements**. Find the minimum possible amplitude (max − min) of the remaining list.  
      
    
    def min_amplitude(a: list) -> int:
        a.sort()
        n = len(a)
        if n <= 4:
            return 0
        # Remove up to 3: take from front, back, or split
        return min(a[n-1] - a[3], a[n-2] - a[2], a[n-3] - a[1], a[n-4] - a[0])
    
    print(min_amplitude([1, 2, 3, 4, 5]))  # 1  (remove 1 and 5)
    print(min_amplitude([-1, 3, -1, 8, 5, 4]))  # 2
    
    

* * *

### 📊 Walmart — Average Subarray

|   
---|---  
**Problem**|  Given a list of numbers and integer `k`, return a list of the **averages** of every contiguous subarray of length `k`, rounded to 2 dp.  
      
    
    def average_subarray(nums: list, k: int) -> list:
        result = []
        window_sum = sum(nums[:k])
        result.append(round(window_sum / k, 2))
        for i in range(k, len(nums)):
            window_sum += nums[i] - nums[i - k]
            result.append(round(window_sum / k, 2))
        return result
    
    print(average_subarray([1,2,3,4,5], 3))  # [2.0, 3.0, 4.0]
    
    

* * *

### 📊 Databricks — k-Radius Average

|   
---|---  
**Problem**|  For each index `i` in a list, compute the average of elements within radius `k` (i.e., indices i-k to i+k). If there aren't enough elements on either side, output -1.  
      
    
    def k_radius_average(nums: list, k: int) -> list:
        n = len(nums)
        result = [-1] * n
        window = sum(nums[:2*k+1]) if 2*k+1 <= n else None
        for i in range(k, n - k):
            if i == k:
                result[i] = round(sum(nums[0:2*k+1]) / (2*k+1), 2)
            else:
                result[i] = round(result[i-1] * (2*k+1) / (2*k+1) +
                    (nums[i+k] - nums[i-k-1]) / (2*k+1), 2)
        # Simpler clean version:
        result = [-1] * n
        for i in range(k, n - k):
            result[i] = round(sum(nums[i-k:i+k+1]) / (2*k+1), 2)
        return result
    
    print(k_radius_average([7,4,3,9,1,8,5,2,6], 3))
    # [-1,-1,-1,5.43,4.14,4.86,-1,-1,-1]
    
    

* * *

### 🎮 FAANG — Idle GPU Days

|   
---|---  
**Problem**|  Given a list of `(gpu_id, date)` usage records, find dates where a GPU had **no jobs** running (i.e., gaps between usage dates per GPU).  
      
    
    from datetime import timedelta
    
    def idle_gpu_days(records: list) -> list:
        from collections import defaultdict
        gpu_dates = defaultdict(list)
        for gpu_id, date in records:
            gpu_dates[gpu_id].append(date)
        idle = []
        for gpu_id, dates in gpu_dates.items():
            dates.sort()
            for i in range(1, len(dates)):
                gap = (dates[i] - dates[i-1]).days
                for d in range(1, gap):
                    idle.append((gpu_id, dates[i-1] + timedelta(days=d)))
        return sorted(idle)
    
    

* * *

### 📊 Google — Most Popular Integers

|   
---|---  
**Problem**|  Return the most frequently occurring integer(s) in a list. If there's a tie, return all tied values sorted ascending.  
      
    
    from collections import Counter
    
    def most_popular(nums: list) -> list:
        count = Counter(nums)
        max_freq = max(count.values())
        return sorted(k for k, v in count.items() if v == max_freq)
    
    print(most_popular([1,2,2,3,3]))  # [2, 3]
    print(most_popular([5,5,5]))      # [5]
    
    

* * *

### 🔢 Microsoft — Factorial Trailing Zeroes

|   
---|---  
**Problem**|  Return the number of **trailing zeros** in n! (without computing the factorial).  
      
    
    def trailing_zeroes(n: int) -> int:
        # Count factors of 5 (each pairs with a factor of 2)
        count = 0
        while n >= 5:
            n //= 5
            count += n
        return count
    
    print(trailing_zeroes(5))    # 1
    print(trailing_zeroes(25))   # 6
    print(trailing_zeroes(100))  # 24
    
    

* * *

### 📊 Blackstone — Generate Fractions

|   
---|---  
**Problem**|  Generate all unique fractions p/q where p < q <= n, in ascending order of their decimal value.  
      
    
    from math import gcd
    from fractions import Fraction
    
    def generate_fractions(n: int) -> list:
        fracs = set()
        for q in range(2, n + 1):
            for p in range(1, q):
                fracs.add(Fraction(p, q))
        return sorted(fracs)
    
    print(generate_fractions(3))
    # [Fraction(1,3), Fraction(1,2), Fraction(2,3)]
    
    

* * *

### 📊 D.E. Shaw — Max Product of Three Numbers

|   
---|---  
**Problem**|  Return the **maximum product** of any 3 numbers in a list (which may contain negatives).  
      
    
    def max_product_three(nums: list) -> int:
        nums.sort()
        n = len(nums)
        # Either top 3, or bottom 2 (most negative) × top 1
        return max(nums[-1] * nums[-2] * nums[-3],
                   nums[0]  * nums[1]  * nums[-1])
    
    print(max_product_three([-10, -10, 1, 3, 2]))  # 300
    print(max_product_three([1, 2, 3]))             # 6
    
    

* * *

### 🛒 Amazon — Two Sum

|   
---|---  
**Problem**|  Return the **indices** of two numbers that sum to the target. Exactly one solution guaranteed.  
      
    
    def two_sum(nums: list, target: int) -> list:
        seen = {}
        for i, n in enumerate(nums):
            comp = target - n
            if comp in seen:
                return [seen[comp], i]
            seen[n] = i
    
    print(two_sum([2,7,11,15], 9))  # [0, 1]
    print(two_sum([3,2,4], 6))      # [1, 2]
    
    

* * *

### 🛒 Amazon — Two Sum (Part 2)

|   
---|---  
**Problem**|  The list is now **sorted**. Return the 1-indexed positions of the two numbers that sum to target using O(1) extra space.  
      
    
    def two_sum_sorted(nums: list, target: int) -> list:
        left, right = 0, len(nums) - 1
        while left < right:
            s = nums[left] + nums[right]
            if s == target:   return [left + 1, right + 1]
            elif s < target:  left += 1
            else:             right -= 1
    
    print(two_sum_sorted([2,7,11,15], 9))   # [1, 2]
    print(two_sum_sorted([1,3,4,5,7], 9))   # [3, 4]  (4+5)
    
    

* * *

### 🛒 Amazon — Two Sum (Part 3)

|   
---|---  
**Problem**|  Return **all unique pairs** (as sorted tuples) that sum to the target. No index needed.  
      
    
    def two_sum_all(nums: list, target: int) -> list:
        seen, result = set(), set()
        for n in nums:
            comp = target - n
            if comp in seen:
                result.add(tuple(sorted((n, comp))))
            seen.add(n)
        return sorted(result)
    
    print(two_sum_all([1,3,2,4,3,2,1], 4))  # [(1,3),(2,2)]
    
    

* * *

### 📊 AQR — Pearson Correlation Coefficient

|   
---|---  
**Problem**|  Compute the **Pearson correlation coefficient** between two lists of numbers, rounded to 2 dp.  
      
    
    def pearson_correlation(x: list, y: list) -> float:
        n = len(x)
        mean_x, mean_y = sum(x)/n, sum(y)/n
        num = sum((xi - mean_x)*(yi - mean_y) for xi, yi in zip(x, y))
        den = (sum((xi - mean_x)**2 for xi in x) * sum((yi - mean_y)**2 for yi in y)) ** 0.5
        return round(num / den, 2) if den else 0.0
    
    print(pearson_correlation([1,2,3],[4,5,6]))   # 1.0
    print(pearson_correlation([1,2,3],[6,5,4]))   # -1.0
    
    

* * *

### 💰 Akuna Capital — Largest Contiguous Subarray Sum

|   
---|---  
**Problem**|  Return the largest sum of a contiguous subarray (Kadane's algorithm).  
      
    
    def max_subarray_sum(nums: list) -> int:
        best = cur = nums[0]
        for n in nums[1:]:
            cur  = max(n, cur + n)
            best = max(best, cur)
        return best
    
    print(max_subarray_sum([-2,1,-3,4,-1,2,1,-5,4]))  # 6
    print(max_subarray_sum([-1,-2,-3]))                # -1
    
    

* * *

### 🪙 Amazon — Coin Change

|   
---|---  
**Problem**|  Given coin denominations and a target amount, return the **minimum number of coins** needed. Return -1 if impossible.  
      
    
    def coin_change(coins: list, amount: int) -> int:
        dp = [float('inf')] * (amount + 1)
        dp[0] = 0
        for i in range(1, amount + 1):
            for c in coins:
                if c <= i:
                    dp[i] = min(dp[i], dp[i - c] + 1)
        return dp[amount] if dp[amount] != float('inf') else -1
    
    print(coin_change([1,5,11], 15))  # 3  (5+5+5)
    print(coin_change([2], 3))        # -1
    
    

* * *

## Python — Hard

* * *

### 🍎 Apple — Word Search

|   
---|---  
**Problem**|  Given an m×n grid of characters and a word, return `True` if the word can be found by traversing adjacent cells (no reuse).  
      
    
    def word_search(board: list, word: str) -> bool:
        R, C = len(board), len(board[0])
    
        def dfs(r, c, idx):
            if idx == len(word):   return True
            if not (0<=r<R and 0<=c<C): return False
            if board[r][c] != word[idx]: return False
            temp, board[r][c] = board[r][c], '#'
            found = any(dfs(r+dr, c+dc, idx+1) for dr,dc in [(0,1),(0,-1),(1,0),(-1,0)])
            board[r][c] = temp
            return found
    
        return any(dfs(r, c, 0) for r in range(R) for c in range(C))
    
    board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]]
    print(word_search(board, "ABCCED"))  # True
    print(word_search(board, "ABCB"))   # False
    
    

* * *

## Statistics & Probability — Easy

* * *

### 📘 Facebook — Coin Fairness Test

|   
---|---  
**Problem**|  You flip a coin 30 times and get 22 heads. How do you test if the coin is fair? What test would you use and why?  
      
    
    Use a two-sided binomial test (or z-test for proportions):
    
    H₀: p = 0.5  (coin is fair)
    H₁: p ≠ 0.5
    
    Under H₀, X ~ Binomial(n=30, p=0.5)
    Expected heads = 15, observed = 22
    
    p-value = P(X ≥ 22) + P(X ≤ 8)  [two-sided]
    
    Using Python:
      from scipy.stats import binom_test
      p_val = binom_test(22, 30, 0.5, alternative='two-sided')
      # p_val ≈ 0.043 → reject H₀ at α=0.05
    
    Conclusion: There is statistical evidence the coin is unfair.
    For large n (>30), approximate with z = (x - np) / √(np(1-p)).
    
    

* * *

### 🎲 Visa — 4 Rolls to 4

|   
---|---  
**Problem**|  What is the probability that you roll a fair 6-sided die exactly **4 times** before getting your first 4?  
      
    
    This is a geometric distribution problem.
    
    P(roll = 4) = 1/6
    P(roll ≠ 4) = 5/6
    
    "Exactly 4 rolls before first 4" means:
      - First 3 rolls: NOT 4
      - 4th roll: IS 4
    
    P = (5/6)³ × (1/6)
      = 125/216
      ≈ 0.0965 or about 9.65%
    
    General formula (Geometric): P(X = k) = (1-p)^(k-1) × p
    Here k=4, p=1/6: P = (5/6)^3 × (1/6)
    
    

* * *

### 🤖 BCG Gamma — Precision/Recall Tradeoff

|   
---|---  
**Problem**|  Explain the precision-recall tradeoff. When would you prioritise precision vs. recall?  
      
    
    Precision = TP / (TP + FP)  — of all predicted positives, how many are correct?
    Recall    = TP / (TP + FN)  — of all actual positives, how many did we catch?
    
    Tradeoff: Lowering the classification threshold increases recall
    (catch more positives) but decreases precision (more false alarms).
    
    Prioritise PRECISION when:
      - False positives are costly
      - e.g. spam filter (don't block valid emails)
      - e.g. recommending a product (don't annoy users)
    
    Prioritise RECALL when:
      - False negatives are costly
      - e.g. cancer screening (don't miss cases)
      - e.g. fraud detection (don't miss fraud)
    
    Use F1-score = 2 × (P × R)/(P + R) when both matter equally.
    Use Fβ-score to weight one more than the other.
    
    

* * *

## Statistics & Probability — Medium

* * *

### 🎲 Akuna Capital — Consecutive Fives

|   
---|---  
**Problem**|  What is the probability of rolling **two consecutive 5s** in a series of 3 fair die rolls?  
      
    
    We need P(at least one "55" in positions (1,2) or (2,3)).
    
    Let A = rolls 1,2 are both 5
    Let B = rolls 2,3 are both 5
    
    P(A) = (1/6)² = 1/36
    P(B) = (1/6)² = 1/36
    P(A∩B) = P(all three are 5) = (1/6)³ = 1/216
    
    By inclusion-exclusion:
    P(A∪B) = P(A) + P(B) - P(A∩B)
            = 1/36 + 1/36 - 1/216
            = 6/216 + 6/216 - 1/216
            = 11/216
            ≈ 0.051 or about 5.1%
    
    

* * *

### 🎬 Netflix — Lazy Movie Raters

|   
---|---  
**Problem**|  Users rate movies 1–5. A "lazy" rater always gives 3. How would you detect if a rating system is being gamed by lazy raters?  
      
    
    Statistical approach:
    
    1. Distribution test: Compare the distribution of a suspected user's ratings
       to the platform average using a chi-squared goodness-of-fit test.
       H₀: user distribution = platform distribution.
    
    2. Variance analysis: Lazy raters have very low variance. Flag users
       with rating variance < threshold (e.g., var < 0.5 across 50+ ratings).
    
    3. Z-score: For each user, compute z = (mean_rating - 3) / (σ / √n).
       Lazy raters cluster at mean=3 with near-zero σ.
    
    4. Entropy measure: A uniform rater has high entropy; a lazy rater
       has near-zero entropy (all mass at one value).
    
    Action: Down-weight or exclude ratings from users whose distribution
    is statistically indistinguishable from "always 3".
    
    

* * *

### 🪙 D.E. Shaw — Biased Coin?

|   
---|---  
**Problem**|  You suspect a coin is biased toward heads. You flip it 100 times and get 60 heads. Is this statistically significant?  
      
    
    One-sided hypothesis test:
    H₀: p = 0.5  (fair coin)
    H₁: p > 0.5  (biased toward heads)
    
    Under H₀: X ~ Binomial(100, 0.5)
      Mean = 50,  SD = √(100 × 0.5 × 0.5) = 5
    
    z = (observed - expected) / SD
      = (60 - 50) / 5
      = 2.0
    
    p-value = P(Z > 2.0) ≈ 0.0228
    
    At α = 0.05: p-value (0.023) < 0.05 → Reject H₀.
    
    Conclusion: Yes, statistically significant evidence the coin
    is biased toward heads (p ≈ 0.023).
    
    

* * *

## Statistics & Probability — Hard

* * *

### 📊 Facebook — Product vs. Square

|   
---|---  
**Problem**|  For two independent uniform random variables X, Y ~ U(0,1), is E[XY] equal to E[X]·E[Y]? What does this imply?  
      
    
    E[X] = E[Y] = 1/2  (for U(0,1))
    
    E[XY] = ∫₀¹ ∫₀¹ xy dx dy
          = (∫₀¹ x dx)(∫₀¹ y dy)   [by independence]
          = (1/2)(1/2)
          = 1/4
    
    E[X]·E[Y] = (1/2)(1/2) = 1/4  ✓
    
    Yes — E[XY] = E[X]·E[Y] = 1/4.
    
    This holds because X and Y are INDEPENDENT.
    For independent random variables: E[XY] = E[X]·E[Y].
    This is the definition of uncorrelated variables (and independence implies it).
    
    Cov(X,Y) = E[XY] - E[X]E[Y] = 0  → uncorrelated.
    Independence → uncorrelated, but uncorrelated ≠ independent (in general).
    
    

* * *

### 📊 Google — Minimum of Two Uniform Variables

|   
---|---  
**Problem**|  X, Y ~ U(0,1) independently. Find E[min(X,Y)].  
      
    
    Let M = min(X, Y).
    
    CDF of M:
    P(M > t) = P(X > t AND Y > t) = (1-t)²  for t ∈ [0,1]
    
    So P(M ≤ t) = 1 - (1-t)²
    PDF of M: f_M(t) = 2(1-t)
    
    E[M] = ∫₀¹ t · 2(1-t) dt
         = 2 ∫₀¹ (t - t²) dt
         = 2 [t²/2 - t³/3]₀¹
         = 2 (1/2 - 1/3)
         = 2 × 1/6
         = 1/3
    
    E[min(X,Y)] = 1/3
    
    General rule: E[min of n U(0,1) variables] = 1/(n+1)
    
    

* * *

### 📊 Microsoft — Frequentist vs. Bayesian

|   
---|---  
**Problem**|  Explain the key differences between frequentist and Bayesian statistics. When would you use each?  
      
    
    FREQUENTIST:
    - Probability = long-run frequency of events
    - Parameters are fixed (unknown constants), data is random
    - No prior beliefs incorporated
    - Tools: p-values, confidence intervals, MLE
    - Confidence interval: "95% of such intervals contain the true parameter"
    - Use when: large data, no strong prior, regulatory settings (clinical trials)
    
    BAYESIAN:
    - Probability = degree of belief
    - Parameters are random variables with distributions
    - Incorporates prior knowledge via Bayes' theorem:
        P(θ|data) ∝ P(data|θ) × P(θ)
        Posterior  ∝  Likelihood × Prior
    - Tools: posterior distributions, credible intervals, MAP estimation
    - Credible interval: "95% probability the parameter lies in this range"
    - Use when: small data, strong domain prior, sequential updating,
      natural probabilistic predictions needed (e.g., A/B testing at Airbnb)
    
    Key tension: Bayesian requires choosing a prior (subjective);
    Frequentist avoids priors but can't make direct probability
    statements about parameters.
    
    

* * *

## Machine Learning — Easy

* * *

### 🤖 BCG Gamma — Precision/Recall Tradeoff

_(Covered above in Statistics section)_

* * *

### 📊 McKinsey — Assumptions of Linear Regression

|   
---|---  
**Problem**|  What are the key assumptions of linear regression? How do you check them?  
      
    
    The 5 key assumptions (LINE):
    
    1. LINEARITY: The relationship between X and Y is linear.
       Check: Plot residuals vs. fitted values — should show no pattern.
    
    2. INDEPENDENCE: Observations are independent of each other.
       Check: Study design; Durbin-Watson test for autocorrelation
       (important for time-series data).
    
    3. NORMALITY: Residuals are normally distributed.
       Check: Q-Q plot of residuals; Shapiro-Wilk test.
       Note: Less critical for large samples (CLT).
    
    4. EQUAL VARIANCE (Homoscedasticity): Residual variance is constant.
       Check: Residuals vs. fitted plot — no funnel shape.
       Fix: Log-transform Y, or use weighted least squares.
    
    5. NO MULTICOLLINEARITY: Predictors are not highly correlated.
       Check: Variance Inflation Factor (VIF > 10 is problematic).
       Fix: Remove correlated features, use PCA, or regularisation (Ridge).
    
    Bonus: No influential outliers — check Cook's Distance.
    
    

* * *

### 🤖 Blackstone — Real Estate Modelling

|   
---|---  
**Problem**|  You want to predict real estate prices. Walk through your model selection process.  
      
    
    Step-by-step approach:
    
    1. DEFINE THE PROBLEM
       - Regression task: predict continuous price
       - Metric: RMSE, MAE, or MAPE depending on business need
    
    2. FEATURE ENGINEERING
       - Location (lat/lon, neighbourhood encoding)
       - Size (sqft, rooms, floors)
       - Age and renovation year
       - Proximity to amenities (schools, transit)
       - Market conditions (month, year, interest rates)
    
    3. BASELINE MODEL
       - Linear Regression with regularisation (Ridge/Lasso)
       - Interpretable, fast to train, good benchmark
    
    4. CANDIDATE MODELS
       - Gradient Boosted Trees (XGBoost/LightGBM): handles non-linearity,
         missing values, feature interactions — often wins on tabular data
       - Random Forest: robust, less tuning needed
       - Neural network: only if very large dataset
    
    5. VALIDATION
       - Time-based split (not random) to prevent data leakage
       - Cross-validation within training period
    
    6. EXPLAINABILITY
       - SHAP values to explain individual predictions
       - Important for regulatory and client-facing use
    
    7. DEPLOYMENT
       - Monitor for data drift (prices change with market conditions)
       - Retrain periodically
    
    

* * *

### 🔗 LinkedIn — Connection Request Spam

|   
---|---  
**Problem**|  How would you build a system to detect connection request spam on LinkedIn?  
      
    
    Frame as a BINARY CLASSIFICATION problem:
      - Label: 1 = spam request, 0 = legitimate
    
    FEATURES (per sender):
      - Request rate (requests per hour/day)
      - Accept rate (% of requests accepted)
      - Message similarity score (copy-paste detection)
      - Profile completeness score
      - Account age
      - Mutual connections count
      - Geographic mismatch between sender/receiver
      - Prior spam flags
    
    MODEL:
      - Start with Logistic Regression (interpretable, fast)
      - Graduate to Gradient Boosted Trees (XGBoost) for better accuracy
      - Use class weighting (spam is rare — imbalanced dataset)
    
    EVALUATION:
      - Prioritise RECALL to catch more spam (cost of missing spam > false block)
      - Use PR-AUC (precision-recall curve) over ROC-AUC for imbalanced data
    
    DEPLOYMENT:
      - Real-time scoring pipeline
      - Cooldown: rate-limit flagged accounts before blocking
      - Human review queue for edge cases
      - Feedback loop: use accepted/declined signals to retrain
    
    

* * *

_Solutions are provided for educational purposes. Problem statements are paraphrased from DataLemur._
