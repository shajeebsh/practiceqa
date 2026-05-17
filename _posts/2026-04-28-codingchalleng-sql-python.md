---
title: "challenge-sql-python3"
date: 2026-04-28 19:26:50 
author: shajeebhameed
layout: page
slug: codingchalleng-sql-python
status: publish
---

## 🐍 Python & 🗄️ SQL Practice Challenges

> Difficulty: **Intermediate** — interview & assessment style

* * *

## PART 1: PYTHON CHALLENGES

* * *

### Challenge 1 — Contains Duplicate

Given a list of integers called `input`, return `True` if any value appears **at least twice**. Return `False` if every element is distinct.

**Example:**
    
    
    Input:  [1, 3, 5, 7, 1]  →  True   (1 appears twice)
    Input:  [1, 3, 5, 7]     →  False  (all distinct)
    
    
    
    
    
    
    # Answer
    def contains_duplicate(input):
        return len(input) != len(set(input))
    
    print(contains_duplicate([1, 3, 5, 7, 1]))  # True
    print(contains_duplicate([1, 3, 5, 7]))     # False
    
    
    
    

**Why it works:** Converting the list to a `set` removes duplicates. If the lengths differ, a duplicate existed.

* * *

### Challenge 2 — Two Sum

Given a list of integers `nums` and a target integer `target`, return the **indices** of the two numbers that add up to `target`. You may assume exactly one solution exists.

**Example:**
    
    
    Input:  nums = [2, 7, 11, 15], target = 9
    Output: [0, 1]   (because nums[0] + nums[1] = 2 + 7 = 9)
    
    
    
    
    
    
    # Answer
    def two_sum(nums, target):
        seen = {}
        for i, num in enumerate(nums):
            complement = target - num
            if complement in seen:
                return [seen[complement], i]
            seen[num] = i
    
    print(two_sum([2, 7, 11, 15], 9))   # [0, 1]
    print(two_sum([3, 2, 4], 6))        # [1, 2]
    
    
    
    

**Why it works:** Use a dictionary to store each number and its index. For each number, check if its complement (`target - num`) has already been seen.

* * *

### Challenge 3 — Valid Anagram

Given two strings `s` and `t`, return `True` if `t` is an anagram of `s`, and `False` otherwise. An anagram uses the same characters the same number of times, just in a different order.

**Example:**
    
    
    Input:  s = "anagram", t = "nagaram"  →  True
    Input:  s = "rat",     t = "car"      →  False
    
    
    
    
    
    
    # Answer
    from collections import Counter
    
    def is_anagram(s, t):
        return Counter(s) == Counter(t)
    
    print(is_anagram("anagram", "nagaram"))  # True
    print(is_anagram("rat", "car"))          # False
    
    
    
    

**Alternative without Counter:**
    
    
    def is_anagram(s, t):
        return sorted(s) == sorted(t)
    
    
    
    

* * *

### Challenge 4 — Maximum Subarray Sum (Kadane's Algorithm)

Given a list of integers, find the **contiguous subarray** with the largest sum and return that sum.

**Example:**
    
    
    Input:  [-2, 1, -3, 4, -1, 2, 1, -5, 4]
    Output: 6   (subarray [4, -1, 2, 1] has the largest sum)
    
    
    
    
    
    
    # Answer
    def max_subarray(nums):
        max_sum = current_sum = nums[0]
        for num in nums[1:]:
            current_sum = max(num, current_sum + num)
            max_sum = max(max_sum, current_sum)
        return max_sum
    
    print(max_subarray([-2, 1, -3, 4, -1, 2, 1, -5, 4]))  # 6
    print(max_subarray([-1, -2, -3]))                       # -1
    
    
    
    

**Why it works:** At each step, decide whether to extend the current subarray or start fresh. Track the best seen so far.

* * *

### Challenge 5 — Group Anagrams

Given a list of strings, group the strings that are **anagrams of each other** together.

**Example:**
    
    
    Input:  ["eat", "tea", "tan", "ate", "nat", "bat"]
    Output: [["eat","tea","ate"], ["tan","nat"], ["bat"]]
    
    
    
    
    
    
    # Answer
    from collections import defaultdict
    
    def group_anagrams(strs):
        groups = defaultdict(list)
        for word in strs:
            key = tuple(sorted(word))
            groups[key].append(word)
        return list(groups.values())
    
    print(group_anagrams(["eat", "tea", "tan", "ate", "nat", "bat"]))
    # [['eat', 'tea', 'ate'], ['tan', 'nat'], ['bat']]
    
    
    
    

**Why it works:** Sorting the letters of an anagram always gives the same result. Use that sorted tuple as the dictionary key.

* * *

### Challenge 6 — Longest Consecutive Sequence

Given an unsorted list of integers, return the length of the **longest consecutive sequence**.

**Example:**
    
    
    Input:  [100, 4, 200, 1, 3, 2]
    Output: 4   (consecutive sequence: [1, 2, 3, 4])
    
    
    
    
    
    
    # Answer
    def longest_consecutive(nums):
        num_set = set(nums)
        longest = 0
    
        for num in num_set:
            # Only start counting from the beginning of a sequence
            if num - 1 not in num_set:
                current = num
                streak = 1
                while current + 1 in num_set:
                    current += 1
                    streak += 1
                longest = max(longest, streak)
    
        return longest
    
    print(longest_consecutive([100, 4, 200, 1, 3, 2]))  # 4
    print(longest_consecutive([0, 3, 7, 2, 5, 8, 4, 6, 0, 1]))  # 9
    
    
    
    

* * *

### Challenge 7 — Product of Array Except Self

Given a list `nums`, return a list `output` where `output[i]` is the product of all elements **except** `nums[i]`. You cannot use division.

**Example:**
    
    
    Input:  [1, 2, 3, 4]
    Output: [24, 12, 8, 6]
    
    
    
    
    
    
    # Answer
    def product_except_self(nums):
        n = len(nums)
        output = [1] * n
    
        # Left pass: output[i] = product of all elements to the LEFT of i
        prefix = 1
        for i in range(n):
            output[i] = prefix
            prefix *= nums[i]
    
        # Right pass: multiply by product of all elements to the RIGHT of i
        suffix = 1
        for i in range(n - 1, -1, -1):
            output[i] *= suffix
            suffix *= nums[i]
    
        return output
    
    print(product_except_self([1, 2, 3, 4]))  # [24, 12, 8, 6]
    
    
    
    

* * *

### Challenge 8 — Valid Parentheses

Given a string containing only `(`, `)`, `{`, `}`, `[`, `]`, determine if the input string is **valid**. A string is valid if every open bracket is closed by the same type of bracket in the correct order.

**Example:**
    
    
    Input:  "([]{})"  →  True
    Input:  "(]"      →  False
    Input:  "([)]"    →  False
    
    
    
    
    
    
    # Answer
    def is_valid(s):
        stack = []
        matching = {')': '(', '}': '{', ']': '['}
    
        for char in s:
            if char in matching:
                if not stack or stack[-1] != matching[char]:
                    return False
                stack.pop()
            else:
                stack.append(char)
    
        return len(stack) == 0
    
    print(is_valid("([]{})"))  # True
    print(is_valid("(]"))      # False
    print(is_valid("([)]"))    # False
    
    
    
    

* * *

### Challenge 9 — Top K Frequent Elements

Given a list of integers `nums` and an integer `k`, return the `k` **most frequently occurring** elements.

**Example:**
    
    
    Input:  nums = [1, 1, 1, 2, 2, 3], k = 2
    Output: [1, 2]
    
    
    
    
    
    
    # Answer
    from collections import Counter
    
    def top_k_frequent(nums, k):
        count = Counter(nums)
        return [item for item, _ in count.most_common(k)]
    
    print(top_k_frequent([1, 1, 1, 2, 2, 3], 2))  # [1, 2]
    print(top_k_frequent([1], 1))                   # [1]
    
    
    
    

* * *

### Challenge 10 — Merge Intervals

Given a list of intervals where `intervals[i] = [start, end]`, merge all **overlapping intervals** and return the resulting list.

**Example:**
    
    
    Input:  [[1,3],[2,6],[8,10],[15,18]]
    Output: [[1,6],[8,10],[15,18]]
    
    
    
    
    
    
    # Answer
    def merge_intervals(intervals):
        intervals.sort(key=lambda x: x[0])
        merged = [intervals[0]]
    
        for start, end in intervals[1:]:
            if start <= merged[-1][1]:  # Overlapping
                merged[-1][1] = max(merged[-1][1], end)
            else:
                merged.append([start, end])
    
        return merged
    
    print(merge_intervals([[1,3],[2,6],[8,10],[15,18]]))  # [[1,6],[8,10],[15,18]]
    print(merge_intervals([[1,4],[4,5]]))                 # [[1,5]]
    
    
    
    

* * *

### Challenge 11 — Climbing Stairs

You are climbing a staircase with `n` steps. Each time you can climb either **1 or 2 steps**. In how many distinct ways can you reach the top?

**Example:**
    
    
    Input:  n = 3
    Output: 3   (1+1+1 | 1+2 | 2+1)
    
    
    
    
    
    
    # Answer
    def climb_stairs(n):
        if n <= 2:
            return n
        a, b = 1, 2
        for _ in range(3, n + 1):
            a, b = b, a + b
        return b
    
    print(climb_stairs(2))  # 2
    print(climb_stairs(3))  # 3
    print(climb_stairs(5))  # 8
    
    
    
    

**Why it works:** The number of ways to reach step `n` = ways to reach `n-1` \+ ways to reach `n-2`. This is a Fibonacci pattern.

* * *

### Challenge 12 — Flatten a Nested List

Write a function that takes a **nested list** (of arbitrary depth) and returns a single flat list.

**Example:**
    
    
    Input:  [1, [2, [3, 4], 5], [6, 7]]
    Output: [1, 2, 3, 4, 5, 6, 7]
    
    
    
    
    
    
    # Answer
    def flatten(nested):
        result = []
        for item in nested:
            if isinstance(item, list):
                result.extend(flatten(item))
            else:
                result.append(item)
        return result
    
    print(flatten([1, [2, [3, 4], 5], [6, 7]]))  # [1, 2, 3, 4, 5, 6, 7]
    
    
    
    

* * *

### Challenge 13 — Decode a Run-Length Encoded String

Given a run-length encoded string like `"3a2b1c"`, decode it back to `"aaabbc"`.

**Example:**
    
    
    Input:  "3a2b1c4d"  →  "aaabbcdddd"
    Input:  "2x3y"      →  "xxyyy"
    
    
    
    
    
    
    # Answer
    import re
    
    def decode_rle(s):
        result = []
        i = 0
        while i < len(s):
            # Gather digits
            num_str = ""
            while i < len(s) and s[i].isdigit():
                num_str += s[i]
                i += 1
            # Gather the character
            char = s[i]
            i += 1
            result.append(char * int(num_str))
        return "".join(result)
    
    print(decode_rle("3a2b1c"))   # aaabbc
    print(decode_rle("2x3y1z"))   # xxyyyZ
    
    
    
    

* * *

### Challenge 14 — Find Missing Number

Given a list containing `n` distinct numbers taken from the range `0` to `n`, find the **one number that is missing**.

**Example:**
    
    
    Input:  [3, 0, 1]  →  2
    Input:  [0, 1]     →  2
    Input:  [9,6,4,2,3,5,7,0,1]  →  8
    
    
    
    
    
    
    # Answer
    def missing_number(nums):
        n = len(nums)
        expected_sum = n * (n + 1) // 2
        return expected_sum - sum(nums)
    
    print(missing_number([3, 0, 1]))           # 2
    print(missing_number([9,6,4,2,3,5,7,0,1]))  # 8
    
    
    
    

**Why it works:** The sum of 0 to n is `n*(n+1)/2`. Subtract the actual sum to find the gap.

* * *

### Challenge 15 — LRU Cache

Implement a **Least Recently Used (LRU) Cache** with a fixed capacity. It should support `get(key)` and `put(key, value)` operations in O(1) time. When capacity is exceeded, evict the least recently used item.

**Example:**
    
    
    cache = LRUCache(2)
    cache.put(1, 1)
    cache.put(2, 2)
    cache.get(1)     # returns 1
    cache.put(3, 3)  # evicts key 2
    cache.get(2)     # returns -1 (not found)
    
    
    
    
    
    
    # Answer
    from collections import OrderedDict
    
    class LRUCache:
        def __init__(self, capacity):
            self.capacity = capacity
            self.cache = OrderedDict()
    
        def get(self, key):
            if key not in self.cache:
                return -1
            self.cache.move_to_end(key)  # Mark as recently used
            return self.cache[key]
    
        def put(self, key, value):
            if key in self.cache:
                self.cache.move_to_end(key)
            self.cache[key] = value
            if len(self.cache) > self.capacity:
                self.cache.popitem(last=False)  # Evict the oldest (least recently used)
    
    # Test
    cache = LRUCache(2)
    cache.put(1, 1)
    cache.put(2, 2)
    print(cache.get(1))    # 1
    cache.put(3, 3)        # evicts key 2
    print(cache.get(2))    # -1
    print(cache.get(3))    # 3
    
    
    
    

* * *

## PART 2: SQL CHALLENGES

> All challenges use a consistent dataset defined below.

* * *

### Dataset Setup
    
    
    CREATE TABLE products (
        product_id   INTEGER PRIMARY KEY,
        product_name VARCHAR(100),
        category_name VARCHAR(50)
    );
    
    CREATE TABLE product_sales (
        product_id     INTEGER,
        sales_quantity INTEGER,
        rating         DECIMAL(2,1)
    );
    
    INSERT INTO products VALUES
    (3690, 'Game of Thrones', 'Books'),
    (5520, 'Refrigerator',    'Home Appliances'),
    (5952, 'Dishwasher',      'Home Appliances'),
    (3561, 'IKIGAI',          'Books'),
    (1010, 'Blender',         'Home Appliances'),
    (2020, 'Atomic Habits',   'Books'),
    (4040, 'Headphones',      'Electronics'),
    (4041, 'Earbuds',         'Electronics'),
    (5050, 'Office Chair',    'Furniture'),
    (6060, 'Standing Desk',   'Furniture');
    
    INSERT INTO product_sales VALUES
    (3690, 300, 4.9),
    (5520, 70,  3.8),
    (5952, 70,  4.0),
    (3561, 290, 4.5),
    (1010, 55,  4.2),
    (2020, 290, 4.8),
    (4040, 150, 4.3),
    (4041, 150, 4.6),
    (5050, 80,  3.9),
    (6060, 95,  4.7);
    
    CREATE TABLE employees (
        employee_id   INTEGER PRIMARY KEY,
        employee_name VARCHAR(50),
        department    VARCHAR(50),
        manager_id    INTEGER,
        salary        DECIMAL(10,2),
        hire_date     DATE
    );
    
    INSERT INTO employees VALUES
    (1, 'Alice',   'Engineering', NULL, 95000, '2018-03-01'),
    (2, 'Bob',     'Engineering', 1,    72000, '2020-06-15'),
    (3, 'Carol',   'Engineering', 1,    68000, '2021-01-10'),
    (4, 'Diana',   'Marketing',   NULL, 85000, '2017-09-20'),
    (5, 'Edward',  'Marketing',   4,    60000, '2022-04-05'),
    (6, 'Fiona',   'HR',          NULL, 78000, '2019-11-30'),
    (7, 'George',  'HR',          6,    55000, '2023-02-14'),
    (8, 'Hannah',  'Engineering', 1,    91000, '2019-07-22');
    
    CREATE TABLE orders (
        order_id    INTEGER PRIMARY KEY,
        customer_id INTEGER,
        product_id  INTEGER,
        amount      DECIMAL(10,2),
        order_date  DATE,
        status      VARCHAR(20)
    );
    
    INSERT INTO orders VALUES
    (1001, 1, 3690, 29.99,  '2024-01-05', 'completed'),
    (1002, 2, 5520, 899.00, '2024-01-10', 'completed'),
    (1003, 1, 4040, 199.99, '2024-01-15', 'completed'),
    (1004, 3, 3561, 14.99,  '2024-02-01', 'refunded'),
    (1005, 2, 6060, 549.00, '2024-02-14', 'completed'),
    (1006, 4, 4041, 89.99,  '2024-03-01', 'completed'),
    (1007, 1, 5952, 749.00, '2024-03-10', 'completed'),
    (1008, 3, 3690, 29.99,  '2024-03-22', 'completed'),
    (1009, 5, 1010, 79.99,  '2024-04-01', 'pending'),
    (1010, 4, 2020, 12.99,  '2024-04-10', 'completed');
    
    
    
    

* * *

### Challenge 1 — Best-Selling Product per Category

Write a query to find the **best-selling product in each category** by `sales_quantity`. If two products tie on quantity, prefer the one with a **higher rating**. Return `category_name` and `product_name` ordered alphabetically by category.

**Expected Output:**
    
    
    Books            | Game of Thrones
    Electronics      | Earbuds
    Furniture        | Standing Desk
    Home Appliances  | Dishwasher
    
    
    
    
    
    
    -- Answer
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
    
    
    
    

**Key concepts:** `ROW_NUMBER()` with `PARTITION BY` assigns a rank within each category. Ties are broken by `rating DESC` in the `ORDER BY`.

* * *

### Challenge 2 — Running Total of Sales

Write a query that shows each order's date, amount, and a **running total** of the `amount` over time (ordered by `order_date`) for `completed` orders only.

**Expected Output:**
    
    
    2024-01-05 |  29.99 |   29.99
    2024-01-10 | 899.00 |  928.99
    2024-01-15 | 199.99 | 1128.98
    ...
    
    
    
    
    
    
    -- Answer
    SELECT
        order_date,
        amount,
        SUM(amount) OVER (ORDER BY order_date) AS running_total
    FROM orders
    WHERE status = 'completed'
    ORDER BY order_date;
    
    
    
    

**Key concept:** `SUM(...) OVER (ORDER BY ...)` creates a window function that accumulates as rows are processed in order.

* * *

### Challenge 3 — Employees Earning More Than Their Manager

Find all employees who earn a **higher salary than their direct manager**.

**Expected Output:**
    
    
    Hannah | 91000 | Alice | 95000   ← Actually Alice earns more, so no result here
    
    
    
    

> With the given data, Hannah (91000) earns less than Alice (95000), so no rows return — which is the correct answer. Adjust salaries to test.
    
    
    -- Answer
    SELECT
        e.employee_name AS employee,
        e.salary        AS employee_salary,
        m.employee_name AS manager,
        m.salary        AS manager_salary
    FROM employees e
    JOIN employees m ON e.manager_id = m.employee_id
    WHERE e.salary > m.salary;
    
    
    
    

**Key concept:** Self-join — joining the `employees` table to itself to compare employee vs. manager salaries.

* * *

### Challenge 4 — Month-over-Month Revenue Change

Write a query that shows the **total completed revenue per month** and the **revenue from the previous month** , so you can calculate the change.
    
    
    -- Answer
    WITH monthly AS (
        SELECT
            DATE_FORMAT(order_date, '%Y-%m') AS month,
            SUM(amount) AS total_revenue
        FROM orders
        WHERE status = 'completed'
        GROUP BY DATE_FORMAT(order_date, '%Y-%m')
    )
    SELECT
        month,
        total_revenue,
        LAG(total_revenue) OVER (ORDER BY month) AS prev_month_revenue,
        ROUND(total_revenue - LAG(total_revenue) OVER (ORDER BY month), 2) AS revenue_change
    FROM monthly
    ORDER BY month;
    
    
    
    

**Key concept:** `LAG()` window function looks at the previous row's value, making month-over-month comparisons straightforward.

* * *

### Challenge 5 — Customers Who Ordered in Consecutive Months

Find `customer_id` values that placed **completed orders in at least 2 consecutive calendar months**.
    
    
    -- Answer
    WITH customer_months AS (
        SELECT DISTINCT
            customer_id,
            DATE_FORMAT(order_date, '%Y-%m') AS order_month,
            YEAR(order_date) * 12 + MONTH(order_date) AS month_num
        FROM orders
        WHERE status = 'completed'
    ),
    with_lag AS (
        SELECT
            customer_id,
            month_num,
            LAG(month_num) OVER (PARTITION BY customer_id ORDER BY month_num) AS prev_month
        FROM customer_months
    )
    SELECT DISTINCT customer_id
    FROM with_lag
    WHERE month_num - prev_month = 1;
    
    
    
    

* * *

### Challenge 6 — Percentage of Revenue by Category

For each product category, calculate the **total revenue from completed orders** and what **percentage** it represents of overall revenue.

**Expected Output:**
    
    
    Books           |  59.98 |  3.6%
    Electronics     | 289.98 | 17.6%
    Furniture       | 549.00 | 33.4%
    Home Appliances | 748.00 | 45.4%
    
    
    
    
    
    
    -- Answer
    WITH category_revenue AS (
        SELECT
            p.category_name,
            SUM(o.amount) AS revenue
        FROM orders o
        JOIN products p ON o.product_id = p.product_id
        WHERE o.status = 'completed'
        GROUP BY p.category_name
    )
    SELECT
        category_name,
        revenue,
        ROUND(revenue * 100.0 / SUM(revenue) OVER (), 1) AS pct_of_total
    FROM category_revenue
    ORDER BY revenue DESC;
    
    
    
    

**Key concept:** `SUM(...) OVER ()` with no partitioning computes the grand total across all rows, allowing a clean percentage calculation.

* * *

### Challenge 7 — Nth Highest Salary per Department

Write a query to find the **2nd highest salary** in each department. If a department doesn't have a 2nd distinct salary, exclude it.

**Expected Output:**
    
    
    Engineering | 91000
    HR          | 55000
    Marketing   | 60000
    
    
    
    
    
    
    -- Answer
    WITH ranked AS (
        SELECT
            department,
            salary,
            DENSE_RANK() OVER (
                PARTITION BY department
                ORDER BY salary DESC
            ) AS salary_rank
        FROM employees
    )
    SELECT department, salary
    FROM ranked
    WHERE salary_rank = 2
    ORDER BY department;
    
    
    
    

**Key concept:** `DENSE_RANK()` handles ties — if two people share the top salary, the next unique salary is rank 2. (`ROW_NUMBER()` would not work correctly here for ties.)

* * *

### Challenge 8 — Products Never Ordered

Find all products that have **never appeared in any order** , regardless of status.
    
    
    -- Answer
    
    -- Using LEFT JOIN
    SELECT p.product_id, p.product_name
    FROM products p
    LEFT JOIN orders o ON p.product_id = o.product_id
    WHERE o.order_id IS NULL;
    
    -- Alternative using NOT EXISTS
    SELECT product_id, product_name
    FROM products p
    WHERE NOT EXISTS (
        SELECT 1 FROM orders o WHERE o.product_id = p.product_id
    );
    
    
    
    

* * *

### Challenge 9 — Departments With All Employees Hired After 2019

Return departments where **every** employee was hired after January 1, 2020. A department with even one pre-2020 hire should be excluded.

**Expected Output:**
    
    
    Marketing
    
    
    
    
    
    
    -- Answer
    SELECT department
    FROM employees
    GROUP BY department
    HAVING MIN(hire_date) >= '2020-01-01'
    ORDER BY department;
    
    
    
    

**Why it works:** `MIN(hire_date)` gives the earliest hire date in the department. If even that is >= 2020, then all employees are post-2020.

* * *

### Challenge 10 — Customer Lifetime Value with Rank

Calculate the **total completed spend per customer** and rank them from highest to lowest spender. Include customers who spent nothing (show 0).
    
    
    -- Answer
    WITH customer_spend AS (
        SELECT
            o.customer_id,
            COALESCE(SUM(CASE WHEN o.status = 'completed' THEN o.amount ELSE 0 END), 0) AS total_spend
        FROM orders o
        GROUP BY o.customer_id
    )
    SELECT
        customer_id,
        total_spend,
        RANK() OVER (ORDER BY total_spend DESC) AS spend_rank
    FROM customer_spend
    ORDER BY spend_rank;
    
    
    
    

* * *

### Challenge 11 — Pivot: Sales Count by Status per Month

Show a **pivoted view** of orders where each row is a month and columns show the count for each status (`completed`, `refunded`, `pending`).
    
    
    -- Answer
    SELECT
        DATE_FORMAT(order_date, '%Y-%m') AS month,
        COUNT(CASE WHEN status = 'completed' THEN 1 END) AS completed,
        COUNT(CASE WHEN status = 'refunded'  THEN 1 END) AS refunded,
        COUNT(CASE WHEN status = 'pending'   THEN 1 END) AS pending
    FROM orders
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
    ORDER BY month;
    
    
    
    

**Key concept:** Conditional aggregation with `CASE WHEN` inside `COUNT()` is a standard SQL pivot technique.

* * *

### Challenge 12 — Find Gaps in a Sequence

The `order_id` column should be sequential. Write a query to find any **gaps** in the sequence (i.e., missing order IDs).
    
    
    -- Answer
    WITH RECURSIVE seq AS (
        SELECT MIN(order_id) AS id FROM orders
        UNION ALL
        SELECT id + 1 FROM seq WHERE id < (SELECT MAX(order_id) FROM orders)
    )
    SELECT seq.id AS missing_order_id
    FROM seq
    LEFT JOIN orders o ON seq.id = o.order_id
    WHERE o.order_id IS NULL;
    
    
    
    

**Key concept:** A recursive CTE generates every integer in the expected range; a `LEFT JOIN` then reveals which ones are absent in the actual table.

* * *

## Quick Reference

### Python Patterns

Pattern| Use case  
---|---  
`set(lst)`| Remove duplicates / O(1) lookup  
`Counter(lst)`| Frequency counting  
`defaultdict(list)`| Grouping by key  
`enumerate(lst)`| Index + value in loops  
`sorted(s)`| Sort a string's characters  
Window (`a, b = b, a+b`)| Space-efficient DP  
`stack.append / pop`| Bracket matching, DFS  
  
### SQL Patterns

Pattern| Use case  
---|---  
`ROW_NUMBER() OVER (PARTITION BY ...)`| Rank within group  
`DENSE_RANK()`| Rank with tie-handling  
`LAG() / LEAD()`| Previous/next row value  
`SUM() OVER (ORDER BY ...)`| Running total  
`SUM() OVER ()`| Grand total for % calculation  
`LEFT JOIN ... WHERE x IS NULL`| Anti-join (find missing)  
`HAVING MIN(...) >= value`| All-or-nothing group filter  
`CASE WHEN ... END` inside `COUNT`| Pivot / conditional count  
Recursive CTE| Generate sequences, find gaps  
  
* * *

_The dataset may differ in real assessments — always read the schema and sample data carefully before writing your query._ 🚀
