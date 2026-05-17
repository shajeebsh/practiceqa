---
title: "sql-codingchallengeHME"
date: 2026-04-30 15:50:26 
author: shajeebhameed
layout: page
slug: sql-codingchallengehme
status: publish
---

This is the complete, consolidated technical report of all **26** challenges. Each entry now includes the **Table Schema** , the **Synthetic Data (as a CTE)** , and the **Optimized Solution**.

* * *

## 1\. Best-selling Product

**Question:** Identify the best-selling product for each category based on total quantity sold.

**Table Schema:** `Sales (category VARCHAR, product_name VARCHAR, quantity INT)`

**Answer:**

SQL
    
    
    WITH Sales AS (
        SELECT * FROM (VALUES 
        ('Electronics', 'Phone', 100), ('Electronics', 'Laptop', 50), 
        ('Home', 'Lamp', 20), ('Home', 'Rug', 50)) 
        AS t(category, product_name, quantity)
    )
    SELECT category, product_name
    FROM (
        SELECT category, product_name, 
               RANK() OVER(PARTITION BY category ORDER BY SUM(quantity) DESC) as rnk
        FROM Sales
        GROUP BY category, product_name
    ) t
    WHERE rnk = 1;
    
    

* * *

## 2\. Shopping Sprees

**Question:** Find customers who made purchases on three or more consecutive days.

**Table Schema:** `Transactions (customer_id INT, transaction_date DATE)`

**Answer:**

SQL
    
    
    WITH Transactions AS (
        SELECT * FROM (VALUES 
        (1, '2023-01-01'), (1, '2023-01-02'), (1, '2023-01-03'), (2, '2023-01-01')) 
        AS t(customer_id, transaction_date)
    )
    SELECT DISTINCT s1.customer_id
    FROM Transactions s1, Transactions s2, Transactions s3
    WHERE s1.customer_id = s2.customer_id 
      AND s1.customer_id = s3.customer_id
      AND DATEDIFF(day, s1.transaction_date, s2.transaction_date) = 1
      AND DATEDIFF(day, s2.transaction_date, s3.transaction_date) = 1;
    
    

* * *

## 3\. Second Highest Salary

**Question:** Find the second highest salary. Return `null` if it doesn't exist.

**Table Schema:** `Employee (id INT, salary INT)`

**Answer:**

SQL
    
    
    WITH Employee AS (
        SELECT * FROM (VALUES (1, 100), (2, 200), (3, 300)) AS t(id, salary)
    )
    SELECT MAX(salary) AS SecondHighestSalary
    FROM Employee
    WHERE salary < (SELECT MAX(salary) FROM Employee);
    
    

* * *

## 4\. Rank Scores

**Question:** Rank scores. If there is a tie, both should have the same ranking (Dense Rank).

**Table Schema:** `Scores (score DECIMAL)`

**Answer:**

SQL
    
    
    WITH Scores AS (
        SELECT * FROM (VALUES (3.50), (4.00), (4.00), (3.85)) AS t(score)
    )
    SELECT score, DENSE_RANK() OVER(ORDER BY score DESC) as rank
    FROM Scores;
    
    

* * *

## 5\. Consecutive Numbers

**Question:** Find all numbers that appear at least three times consecutively.

**Table Schema:** `Logs (id INT, num INT)`

**Answer:**

SQL
    
    
    WITH Logs AS (
        SELECT * FROM (VALUES (1, 1), (2, 1), (3, 1), (4, 2), (5, 1)) AS t(id, num)
    )
    SELECT DISTINCT num AS ConsecutiveNums
    FROM (
        SELECT num, 
               LEAD(num) OVER(ORDER BY id) as next, 
               LAG(num) OVER(ORDER BY id) as prev
        FROM Logs
    ) t
    WHERE num = next AND num = prev;
    
    

* * *

## 6\. Managers vs. Employees

**Question:** Find employees who earn more than their direct managers.

**Table Schema:** `Employee (id INT, name VARCHAR, salary INT, managerId INT)`

**Answer:**

SQL
    
    
    WITH Employee AS (
        SELECT * FROM (VALUES 
        (1, 'Joe', 70000, 3), (2, 'Henry', 80000, 4), 
        (3, 'Sam', 60000, NULL), (4, 'Max', 90000, NULL)) 
        AS t(id, name, salary, managerId)
    )
    SELECT e1.name AS Employee
    FROM Employee e1
    JOIN Employee e2 ON e1.managerId = e2.id
    WHERE e1.salary > e2.salary;
    
    

* * *

## 7\. Highest Salary in Department

**Question:** Find the employee with the highest salary in each department.

**Table Schema:** `Employee (id, name, salary, deptId)`, `Department (id, name)`

**Answer:**

SQL
    
    
    WITH Employee AS (
        SELECT * FROM (VALUES (1, 'Joe', 70000, 1), (2, 'Jim', 90000, 1), (3, 'Henry', 80000, 2)) AS t(id, name, salary, deptId)
    ), Department AS (
        SELECT * FROM (VALUES (1, 'IT'), (2, 'Sales')) AS t(id, name)
    )
    SELECT d.name AS Department, e.name AS Employee, e.salary
    FROM Employee e
    JOIN Department d ON e.deptId = d.id
    WHERE e.salary = (SELECT MAX(salary) FROM Employee WHERE deptId = d.id);
    
    

* * *

## 8\. Delete Duplicate Emails

**Question:** Keep only the unique email with the smallest ID.

**Table Schema:** `Person (id INT, email VARCHAR)`

**Answer:**

SQL
    
    
    -- For SQL environments, we join the table to itself to find duplicates
    WITH Person AS (
        SELECT * FROM (VALUES (1, 'a@b.com'), (2, 'c@d.com'), (3, 'a@b.com')) AS t(id, email)
    )
    DELETE p1 FROM Person p1, Person p2
    WHERE p1.email = p2.email AND p1.id > p2.id;
    
    

* * *

## 9\. Rising Temperature

**Question:** Find IDs where the temperature was higher than the previous day.

**Table Schema:** `Weather (id INT, recordDate DATE, temperature INT)`

**Answer:**

SQL
    
    
    WITH Weather AS (
        SELECT * FROM (VALUES (1, '2015-01-01', 10), (2, '2015-01-02', 25), (3, '2015-01-03', 20)) AS t(id, recordDate, temperature)
    )
    SELECT w1.id
    FROM Weather w1
    JOIN Weather w2 ON DATEDIFF(day, w2.recordDate, w1.recordDate) = 1
    WHERE w1.temperature > w2.temperature;
    
    

* * *

## 10\. Primary Department

**Question:** Report the primary department for each employee.

**Table Schema:** `Employee (employee_id INT, department_id INT, primary_flag VARCHAR)`

**Answer:**

SQL
    
    
    WITH Employee AS (
        SELECT * FROM (VALUES (1, 1, 'N'), (1, 2, 'Y'), (2, 3, 'N')) AS t(employee_id, department_id, primary_flag)
    )
    SELECT employee_id, department_id FROM Employee WHERE primary_flag = 'Y'
    UNION
    SELECT employee_id, department_id FROM Employee 
    WHERE employee_id NOT IN (SELECT employee_id FROM Employee WHERE primary_flag = 'Y')
    GROUP BY employee_id, department_id;
    
    

* * *

## 11\. One String Swap (Python)

**Question:** Check if one string swap can make two strings equal.

**Answer:**

Python
    
    
    # Logic: Identifies index of mismatches. Must be 0 differences or 2 that match when swapped.
    def areAlmostEqual(s1: str, s2: str) -> bool:
        diff = [(a, b) for a, b in zip(s1, s2) if a != b]
        return not diff or (len(diff) == 2 and diff[0] == diff[1][::-1])
    
    

* * *

## 12\. Trips and Users (Hard)

**Question:** Calculate cancellation rate for non-banned users between 2013-10-01 and 2013-10-03.

**Table Schema:** `Trips (id, client_id, driver_id, status, request_at)`, `Users (users_id, banned, role)`

**Answer:**

SQL
    
    
    WITH Trips AS (
        SELECT * FROM (VALUES (1, 1, 10, 'completed', '2013-10-01'), (2, 2, 11, 'cancelled_by_driver', '2013-10-01')) AS t(id, client_id, driver_id, status, request_at)
    ), Users AS (
        SELECT * FROM (VALUES (1, 'No', 'client'), (10, 'No', 'driver'), (2, 'Yes', 'client')) AS t(users_id, banned, role)
    )
    SELECT request_at AS Day,
           ROUND(CAST(SUM(CASE WHEN status != 'completed' THEN 1 ELSE 0 END) AS FLOAT) / COUNT(*), 2) AS [Cancellation Rate]
    FROM Trips t
    JOIN Users d ON t.driver_id = d.users_id AND d.banned = 'No'
    JOIN Users c ON t.client_id = c.users_id AND c.banned = 'No'
    WHERE request_at BETWEEN '2013-10-01' AND '2013-10-03'
    GROUP BY request_at;
    
    

* * *

## 13\. Game Play Analysis I

**Question:** Find the first login date for each player.

**Table Schema:** `Activity (player_id INT, event_date DATE)`

**Answer:**

SQL
    
    
    WITH Activity AS (
        SELECT * FROM (VALUES (1, '2016-03-01'), (1, '2016-05-02'), (2, '2017-06-25')) AS t(player_id, event_date)
    )
    SELECT player_id, MIN(event_date) AS first_login
    FROM Activity
    GROUP BY player_id;
    
    

* * *

## 14\. Game Play Analysis IV (Medium)

**Question:** Fraction of players who logged in the day after their first login.

**Table Schema:** `Activity (player_id INT, event_date DATE)`

**Answer:**

SQL
    
    
    WITH Activity AS (
        SELECT * FROM (VALUES (1, '2016-03-01'), (1, '2016-03-02'), (2, '2017-06-25')) AS t(player_id, event_date)
    ), FirstLogin AS (
        SELECT player_id, MIN(event_date) as first_day FROM Activity GROUP BY player_id
    )
    SELECT ROUND(CAST(COUNT(a.player_id) AS FLOAT) / (SELECT COUNT(DISTINCT player_id) FROM Activity), 2) as fraction
    FROM FirstLogin f
    JOIN Activity a ON f.player_id = a.player_id AND DATEDIFF(day, f.first_day, a.event_date) = 1;
    
    

* * *

## 15\. Managers with 5+ Reports

**Question:** Find names of managers with 5 or more reports.

**Table Schema:** `Employee (id INT, name VARCHAR, managerId INT)`

**Answer:**

SQL
    
    
    WITH Employee AS (
        SELECT * FROM (VALUES (101, 'John', NULL), (102, 'Dan', 101), (103, 'James', 101), (104, 'Amy', 101), (105, 'Anne', 101), (106, 'Ron', 101)) AS t(id, name, managerId)
    )
    SELECT name FROM Employee
    WHERE id IN (SELECT managerId FROM Employee GROUP BY managerId HAVING COUNT(*) >= 5);
    
    

* * *

## 16\. Human Traffic of Stadium (Hard)

**Question:** Find 3+ consecutive rows where people count >= 100.

**Table Schema:** `Stadium (id INT, visit_date DATE, people INT)`

**Answer:**

SQL
    
    
    WITH Stadium AS (
        SELECT * FROM (VALUES (1, '2017-01-01', 10), (2, '2017-01-02', 150), (3, '2017-01-03', 120), (4, '2017-01-04', 130)) AS t(id, visit_date, people)
    ), Grouped AS (
        SELECT *, id - ROW_NUMBER() OVER(ORDER BY id) as grp FROM Stadium WHERE people >= 100
    )
    SELECT id, visit_date, people FROM Grouped
    WHERE grp IN (SELECT grp FROM Grouped GROUP BY grp HAVING COUNT(*) >= 3);
    
    

* * *

## 17\. Who Has the Most Friends

**Question:** Find the person with the most friendships.

**Table Schema:** `RequestAccepted (requester_id INT, accepter_id INT, accept_date DATE)`

**Answer:**

SQL
    
    
    WITH RequestAccepted AS (
        SELECT * FROM (VALUES (1, 2, '2016-06-03'), (1, 3, '2016-06-08'), (2, 3, '2016-06-08')) AS t(requester_id, accepter_id, accept_date)
    )
    SELECT TOP 1 id, SUM(cnt) as num FROM (
        SELECT requester_id AS id, COUNT(*) AS cnt FROM RequestAccepted GROUP BY requester_id
        UNION ALL
        SELECT accepter_id AS id, COUNT(*) AS cnt FROM RequestAccepted GROUP BY accepter_id
    ) t GROUP BY id ORDER BY num DESC;
    
    

* * *

## 18\. Capital Gain/Loss

**Question:** Total capital gain or loss for each stock.

**Table Schema:** `Stocks (stock_name VARCHAR, operation VARCHAR, day INT, price INT)`

**Answer:**

SQL
    
    
    WITH Stocks AS (
        SELECT * FROM (VALUES ('Leetcode', 'Buy', 1, 1000), ('Leetcode', 'Sell', 5, 9000)) AS t(stock_name, operation, day, price)
    )
    SELECT stock_name, SUM(CASE WHEN operation = 'Buy' THEN -price ELSE price END) AS capital_gain_loss
    FROM Stocks
    GROUP BY stock_name;
    
    

* * *

## 19\. Investments in 2016

**Question:** Sum of 2016 values where 2015 value is shared but location is unique.

**Table Schema:** `Insurance (pid INT, tiv_2015 FLOAT, tiv_2016 FLOAT, lat FLOAT, lon FLOAT)`

**Answer:**

SQL
    
    
    WITH Insurance AS (
        SELECT * FROM (VALUES (1, 10, 5, 10, 10), (2, 20, 20, 10, 10), (3, 10, 30, 20, 20)) AS t(pid, tiv_2015, tiv_2016, lat, lon)
    ), Stats AS (
        SELECT tiv_2016, COUNT(*) OVER(PARTITION BY tiv_2015) as t_cnt, COUNT(*) OVER(PARTITION BY lat, lon) as l_cnt FROM Insurance
    )
    SELECT CAST(SUM(tiv_2016) AS DECIMAL(18,2)) as tiv_2016 FROM Stats WHERE t_cnt > 1 AND l_cnt = 1;
    
    

* * *

## 20\. Tree Node

**Question:** Classify nodes as 'Root', 'Inner', or 'Leaf'.

**Table Schema:** `Tree (id INT, p_id INT)`

**Answer:**

SQL
    
    
    WITH Tree AS (
        SELECT * FROM (VALUES (1, NULL), (2, 1), (3, 1), (4, 2), (5, 2)) AS t(id, p_id)
    )
    SELECT id, CASE WHEN p_id IS NULL THEN 'Root'
                    WHEN id IN (SELECT p_id FROM Tree WHERE p_id IS NOT NULL) THEN 'Inner'
                    ELSE 'Leaf' END as type
    FROM Tree;
    
    

* * *

## 21\. Exchange Seats

**Question:** Swap seat IDs of every two consecutive students.

**Table Schema:** `Seat (id INT, student VARCHAR)`

**Answer:**

SQL
    
    
    WITH Seat AS (
        SELECT * FROM (VALUES (1, 'Abbot'), (2, 'Doris'), (3, 'Emerson')) AS t(id, student)
    )
    SELECT CASE WHEN id % 2 != 0 AND id = (SELECT MAX(id) FROM Seat) THEN id
                WHEN id % 2 != 0 THEN id + 1
                ELSE id - 1 END AS id, student
    FROM Seat ORDER BY id;
    
    

* * *

## 22\. Customers Who Bought All Products

**Question:** Find customers who bought every product in the company catalog.

**Table Schema:** `Customer (customer_id INT, product_key INT)`, `Product (product_key INT)`

**Answer:**

SQL
    
    
    WITH Customer AS (
        SELECT * FROM (VALUES (1, 5), (1, 6), (2, 6)) AS t(customer_id, product_key)
    ), Product AS (
        SELECT * FROM (VALUES (5), (6)) AS t(product_key)
    )
    SELECT customer_id FROM Customer GROUP BY customer_id
    HAVING COUNT(DISTINCT product_key) = (SELECT COUNT(*) FROM Product);
    
    

* * *

## 23\. Product Sales Analysis III

**Question:** Find all sales in the first year each product was sold.

**Table Schema:** `Sales (sale_id INT, product_id INT, year INT, quantity INT, price INT)`

**Answer:**

SQL
    
    
    WITH Sales AS (
        SELECT * FROM (VALUES (100, 1, 2008, 10, 5000), (100, 1, 2009, 12, 5000), (200, 2, 2011, 15, 9000)) AS t(sale_id, product_id, year, quantity, price)
    ), CTE AS (
        SELECT product_id, year as first_year, quantity, price, RANK() OVER(PARTITION BY product_id ORDER BY year) as rnk FROM Sales
    )
    SELECT product_id, first_year, quantity, price FROM CTE WHERE rnk = 1;
    
    

* * *

## 24\. Market Analysis I

**Question:** Find each user's join date and the number of buyer orders in 2019.

**Table Schema:** `Users (user_id INT, join_date DATE)`, `Orders (order_id INT, order_date DATE, buyer_id INT)`

**Answer:**

SQL
    
    
    WITH Users AS (
        SELECT * FROM (VALUES (1, '2018-01-01'), (2, '2019-02-01')) AS t(user_id, join_date)
    ), Orders AS (
        SELECT * FROM (VALUES (1, '2019-03-01', 1), (2, '2018-12-01', 1)) AS t(order_id, order_date, buyer_id)
    )
    SELECT u.user_id as buyer_id, u.join_date, COUNT(o.order_id) as orders_in_2019
    FROM Users u LEFT JOIN Orders o ON u.user_id = o.buyer_id AND YEAR(o.order_date) = 2019
    GROUP BY u.user_id, u.join_date;
    
    

* * *

## 25\. Product Price at a Given Date

**Question:** Find the price of all products on 2019-08-16 (Default is 10).

**Table Schema:** `Products (product_id INT, new_price INT, change_date DATE)`

**Answer:**

SQL
    
    
    WITH Products AS (
        SELECT * FROM (VALUES (1, 20, '2019-08-14'), (1, 30, '2019-08-15'), (2, 50, '2019-08-17')) AS t(product_id, new_price, change_date)
    ), Recent AS (
        SELECT product_id, new_price, RANK() OVER(PARTITION BY product_id ORDER BY change_date DESC) as rnk FROM Products WHERE change_date <= '2019-08-16'
    )
    SELECT p.product_id, ISNULL(r.new_price, 10) as price
    FROM (SELECT DISTINCT product_id FROM Products) p LEFT JOIN Recent r ON p.product_id = r.product_id AND r.rnk = 1;
    
    

* * *

## 26\. First Letter Capitalization II (Hard)

**Question:** Capitalize the first letter of words (including single-hyphenated ones) and lowercase the rest.

**Table Schema:** `user_content (content_id INT, content_text VARCHAR)`

**Answer:**

SQL
    
    
    WITH user_content AS (
        SELECT * FROM (VALUES (1, 'the QUICK-brown fox'), (2, 'foo--bar -baz')) AS t(content_id, content_text)
    ), RecursiveCap AS (
        SELECT content_id, content_text as original_text, CAST(LOWER(content_text) AS VARCHAR(MAX)) as work_text, 1 as pos FROM user_content
        UNION ALL
        SELECT content_id, original_text, CAST(STUFF(work_text, pos, 1, 
            CASE WHEN pos = 1 OR SUBSTRING(work_text, pos-1, 1) = ' ' 
                 OR (SUBSTRING(work_text, pos-1, 1) = '-' AND SUBSTRING(work_text, pos-2, 1) LIKE '[a-z0-9]' AND SUBSTRING(work_text, pos, 1) LIKE '[a-z0-9]')
            THEN UPPER(SUBSTRING(work_text, pos, 1)) ELSE SUBSTRING(work_text, pos, 1) END) AS VARCHAR(MAX)), pos + 1
        FROM RecursiveCap WHERE pos <= LEN(work_text)
    )
    SELECT content_id, original_text, work_text as converted_text 
    FROM RecursiveCap 
    WHERE pos = LEN(original_text) + 1 
    OPTION (MAXRECURSION 0);
