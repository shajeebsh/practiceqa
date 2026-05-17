---
title: "Python Data Analysis"
date: 2026-03-15 17:34:12 
author: shajeebhameed
layout: page
slug: python-data-analysis
status: publish
---

# PYTHON STACK INTERVIEW QUESTIONS FOR DATA ARCHITECT

## 📋 Question Categories

Pandas - <https://www.interviewbit.com/pandas-interview-questions/amp/>

Category| Focus Areas  
---|---  
**Core Python Concepts**|  Data types, memory management, OOP, functional programming  
**Pandas & Data Manipulation**| DataFrame operations, missing data, performance optimization  
**NumPy & Scientific Computing**| Arrays, vectorization, broadcasting  
**ETL & Data Pipeline Development**| File handling, automation, error handling  
**Performance Optimization**|  Generators, vectorization, profiling  
**Libraries & Tools**| Python ecosystem for data engineering  
  
* * *

# 1\. CORE PYTHON CONCEPTS

## Q1: Explain the difference between lists and tuples. When would you use each in your data projects?

**Model Answer (Based on your experience):**

> "Lists and tuples are both sequence data types, but they have key differences:
> 
> **Lists are mutable**  (can be modified after creation):
> 
> python
>     
>     
>     my_list = [1, 2, 3]
>     my_list[0] = 10  _# Works fine_
>     my_list.append(4)  _# Works fine_
> 
> **Tuples are immutable**  (cannot be changed):
> 
> python
>     
>     
>     my_tuple = (1, 2, 3)
>     my_tuple[0] = 10  _# TypeError! Tuples don't support item assignment_
> 
> **When I use them in my projects:**
> 
> **Lists for:**  Data that changes—like intermediate processing steps in ETL pipelines. In my Affinion project, I used lists to accumulate batches of records during migration before writing to Azure.
> 
> **Tuples for:**  Fixed data structures—like database connection configurations (host, port, credentials) that shouldn't change. Also for dictionary keys and set elements, where immutability is required.
> 
> **Performance consideration:**  Tuples are slightly more memory-efficient and faster to iterate over, which matters when processing millions of records. In my Refinitiv project, we used tuples for lookup tables that never changed."

* * *

## Q2: How does Python handle memory management? What's the GIL and how does it affect your data pipelines?

**Model Answer:**

> "Python manages memory through **reference counting and garbage collection** :
> 
> **Reference counting:**  Each object keeps track of how many references point to it. When count reaches zero, memory is freed immediately.
> 
> **Garbage collection:**  The `gc` module handles circular references (objects referencing each other) that reference counting can't resolve.
> 
> **The Global Interpreter Lock (GIL):**
> 
>   * The GIL allows only one thread to execute Python bytecode at a time
>   * This simplifies memory management but limits true parallel execution in CPU-bound tasks
> 

> 
> **Impact on my data pipelines:**
> 
> **For I/O-bound operations**  (like the API calls in my Google Street View project), threading still provides benefits because the GIL is released during I/O waits.
> 
> **For CPU-bound operations**  (like the complex transformations at Travelers), I use:
> 
>   1. **Multiprocessing**  (separate processes, each with its own GIL)
>   2. **Vectorized operations**  with NumPy/Pandas (C-based, releases GIL)
>   3. **PySpark**  for distributed processing (as I did at Travelers)
> 

> 
> **Real example:**  At Refinitiv, when processing streaming market data, we used multiprocessing to parallelize validation across CPU cores, achieving the 99.9% consistency with 50+ automated checks."

* * *

## Q3: Explain decorators in Python. Give an example from your experience.

**Model Answer:**

> "Decorators are functions that modify the behavior of other functions without changing their code. They use the `@decorator_name` syntax.
> 
> **Basic structure:**
> 
> python
>     
>     
>     def my_decorator(func):
>         def wrapper(*args, **kwargs):
>             _# Do something before_
>             result = func(*args, **kwargs)
>             _# Do something after_
>             return result
>         return wrapper
>     
>     @my_decorator
>     def my_function():
>         pass
> 
> **Real example from my work:**
> 
> In my Google Street View project, we used decorators for:
> 
>   1. **Timing functions**  to monitor pipeline performance
>   2. **Retry logic**  for API calls that occasionally failed
> 

> 
> python
>     
>     
>     import time
>     from functools import wraps
>     
>     def retry(max_attempts=3, delay=1):
>         def decorator(func):
>             @wraps(func)
>             def wrapper(*args, **kwargs):
>                 for attempt in range(max_attempts):
>                     try:
>                         return func(*args, **kwargs)
>                     except Exception as e:
>                         if attempt == max_attempts - 1:
>                             raise
>                         time.sleep(delay * (attempt + 1))
>                 return None
>             return wrapper
>         return decorator
>     
>     @retry(max_attempts=3, delay=2)
>     def call_bug_tracking_api():
>         _# API call that might fail occasionally_
>         pass
> 
> This pattern helped us achieve the 18-hour reduction in bug-to-resolution time by making our pipeline more resilient."

* * *

## Q4: What are generators and why are they useful for data processing?

**Model Answer:**

> "Generators are functions that yield values one at a time using the `yield` keyword, maintaining state between iterations. They enable **lazy evaluation** —values are generated only when needed.
> 
> **Example:**
> 
> python
>     
>     
>     def read_large_file(file_path):
>         with open(file_path, 'r') as file:
>             for line in file:
>                 yield line.strip()
>     
>     _# Memory-efficient: processes one line at a time_
>     for line in read_large_file('huge_dataset.csv'):
>         process(line)
> 
> **Why they're critical in data engineering:**
> 
> **1\. Memory efficiency:**  When processing the 20+ million records at Affinion, we couldn't load everything into memory. Generators allowed streaming processing.
> 
> **2\. Infinite sequences:**  Generate data on-the-fly for testing.
> 
> **3\. Pipeline composition:**  Chain generators for clean, modular ETL.
> 
> **Real example from Travelers:**
> 
> python
>     
>     
>     def stream_claims_from_source():
>         _# Connect to source system_
>         _# Yield claims one at a time_
>     
>     def validate_claim(claims_stream):
>         for claim in claims_stream:
>             if is_valid(claim):
>                 yield claim
>     
>     def transform_claim(valid_claims):
>         for claim in valid_claims:
>             yield apply_business_rules(claim)
>     
>     _# Pipeline: claims processed without loading all into memory_
>     pipeline = transform_claim(validate_claim(stream_claims_from_source()))
>     for processed_claim in pipeline:
>         write_to_snowflake(processed_claim)
> 
> This pattern enabled us to process millions of claims with minimal memory footprint."

* * *

## Q5: Explain `*args` and `**kwargs`. How have you used them?

**Model Answer:**

> "`*args` and `**kwargs` allow functions to accept variable numbers of arguments:
> 
>   * **`*args`**  captures positional arguments as a tuple
>   * **`**kwargs`**  captures keyword arguments as a dictionary
> 

> 
> **Example:**
> 
> python
>     
>     
>     def flexible_function(*args, **kwargs):
>         print(f"Positional: {args}")
>         print(f"Keyword: {kwargs}")
>     
>     flexible_function(1, 2, 3, name="Alice", age=30)
>     _# Output:_
>     _# Positional: (1, 2, 3)_
>     _# Keyword: {'name': 'Alice', 'age': 30}_
> 
> **How I've used them:**
> 
> **1\. Wrapper functions/decorators:**  
>  In my Google Street View project, I used `*args` and `**kwargs` in decorators to pass through any arguments to wrapped functions:
> 
> python
>     
>     
>     def log_execution(func):
>         @wraps(func)
>         def wrapper(*args, **kwargs):
>             logging.info(f"Calling {func.__name__}")
>             result = func(*args, **kwargs)
>             logging.info(f"Completed {func.__name__}")
>             return result
>         return wrapper
> 
> **2\. Database connection functions:**  
>  At Affinion, we created flexible ETL functions that could accept varying connection parameters:
> 
> python
>     
>     
>     def connect_to_database(**kwargs):
>         _# Handle different database types with different params_
>         db_type = kwargs.get('db_type', 'sqlserver')
>         if db_type == 'sqlserver':
>             return sqlserver_connect(kwargs.get('server'), kwargs.get('database'))
>         elif db_type == 'oracle':
>             return oracle_connect(kwargs.get('dsn'), kwargs.get('service_name'))

* * *

# 2\. PANDAS & DATA MANIPULATION

## Q6: How do you handle missing data in Pandas? Give examples from your projects.

**Model Answer:**

> "Missing data is inevitable in real-world datasets. In my projects, I've used several approaches:
> 
> **1\. Detection:**
> 
> python
>     
>     
>     df.isnull().sum()  _# Count missing per column_
>     df.info()  _# Overview of non-null counts_
> 
> **2\. Removal (when appropriate):**
> 
> python
>     
>     
>     _# Drop rows with any missing values_
>     df_clean = df.dropna()
>     
>     _# Drop rows where specific columns are missing_
>     df_clean = df.dropna(subset=['critical_field1', 'critical_field2'])
>     
>     _# Drop columns with too many missing values ( >40%)_
>     df_clean = df.dropna(thresh=len(df)*0.6, axis=1)
> 
> **3\. Imputation:**
> 
> python
>     
>     
>     _# Fill with constant value_
>     df['column'].fillna('Unknown', inplace=True)
>     
>     _# Fill with mean/median (for numerical)_
>     df['amount'].fillna(df['amount'].median(), inplace=True)
>     
>     _# Forward fill for time series_
>     df['sensor_reading'].fillna(method='ffill', inplace=True)
>     
>     _# Interpolation_
>     df['temperature'].interpolate(method='linear', inplace=True)
> 
> **Real example from Travelers:**
> 
> When processing claims data, we couldn't simply drop records with missing provider IDs—that would lose billing data. Instead:
> 
> python
>     
>     
>     def handle_missing_provider_data(claims_df):
>         _# Flag records for review_
>         claims_df['provider_id_missing'] = claims_df['provider_id'].isnull()
>     
>         _# Try to infer from other fields_
>         claims_df['provider_id'] = claims_df['provider_id'].fillna(
>             claims_df.groupby('facility_code')['provider_id'].transform('first')
>         )
>     
>         _# For remaining missing, flag for manual review_
>         claims_df['requires_review'] = claims_df['provider_id'].isnull()
>     
>         return claims_df
> 
> This approach maintained data integrity while ensuring compliance with Medicare/Medicaid requirements."

* * *

## Q7: Explain the difference between `apply()`, `map()`, and `applymap()` in Pandas.

**Model Answer:**

> "These three functions serve different purposes:
> 
> **`apply()`**  \- Works on DataFrames (row/column-wise) or Series:
> 
> python
>     
>     
>     _# Apply to each column (axis=0, default)_
>     df.apply(lambda col: col.mean())
>     
>     _# Apply to each row_
>     df.apply(lambda row: row['amount'] * row['quantity'], axis=1)
>     
>     _# In my Travelers project, used to apply complex business rules_
>     df['risk_score'] = df.apply(calculate_risk_score, axis=1)
> 
> **`map()`**  \- Works on Series only, for element-wise transformation:
> 
> python
>     
>     
>     _# Replace values based on dictionary_
>     df['status_code'].map({1: 'Active', 2: 'Pending', 3: 'Closed'})
>     
>     _# Apply function to each element_
>     df['clean_text'] = df['raw_text'].map(lambda x: x.strip().lower())
> 
> **`applymap()`**  \- Works on DataFrames, element-wise:
> 
> python
>     
>     
>     _# Format all numeric columns_
>     df_numeric = df.select_dtypes(include=['number'])
>     df_numeric = df_numeric.applymap(lambda x: f"${x:,.2f}")
> 
> **Performance tip:**  Vectorized operations are faster than `apply()` for simple transformations:
> 
> python
>     
>     
>     _# Slow_
>     df['new_col'] = df.apply(lambda row: row['col1'] + row['col2'], axis=1)
>     
>     _# Fast (vectorized)_
>     df['new_col'] = df['col1'] + df['col2']
> 
> In my Affinion migration, we profiled performance and used vectorized operations wherever possible to process 20M+ records efficiently."

* * *

## Q8: How do you merge/join DataFrames in Pandas? Explain the different types.

**Model Answer:**

> "Pandas provides `merge()` for SQL-like joins:
> 
> python
>     
>     
>     pd.merge(df1, df2, on='key_column', how='inner')
> 
> **Join types:**
> 
>   * **`how='inner'`** : Only matching keys (intersection)
>   * **`how='left'`** : All keys from left, matching from right
>   * **`how='right'`** : All keys from right, matching from left
>   * **`how='outer'`** : All keys from both (union)
> 

> 
> **Real example from JNJ project:**
> 
> We consolidated 4 legacy systems—each had customer data with different keys:
> 
> python
>     
>     
>     _# Start with primary source (HR system)_
>     customer_master = hr_customers.copy()
>     
>     _# Left join with payroll data_
>     customer_master = customer_master.merge(
>         payroll_data[['employee_id', 'salary', 'bank_details']],
>         on='employee_id',
>         how='left'
>     )
>     
>     _# Left join with procurement data_
>     customer_master = customer_master.merge(
>         procurement_data[['employee_id', 'vendor_access', 'approval_limit']],
>         on='employee_id',
>         how='left'
>     )
>     
>     _# Fill missing values with business rules_
>     customer_master['approval_limit'] = customer_master['approval_limit'].fillna(0)
> 
> **Key considerations:**
> 
>   * **Duplicate keys:**  Use `validate` parameter to check assumptions
>   * **Performance:**  For large datasets (like our 20M records), ensure keys are indexed
>   * **Memory:**  `merge()` creates new DataFrames—be mindful of memory with big data"
> 


* * *

## Q9: How do you handle large datasets that don't fit in memory? (Chunking, Dask, etc.)

**Model Answer:**

> "When datasets exceed available memory, several strategies apply:
> 
> **1\. Chunking with Pandas:**
> 
> python
>     
>     
>     _# Process in chunks_
>     chunk_size = 100000
>     chunks = []
>     
>     for chunk in pd.read_csv('huge_file.csv', chunksize=chunk_size):
>         _# Process each chunk_
>         chunk = chunk[chunk['status'] == 'active']
>         chunk['processed_date'] = pd.Timestamp.now()
>         chunks.append(chunk)
>     
>     _# Combine results_
>     result = pd.concat(chunks)
> 
> **2\. Use of generators (as mentioned earlier) for streaming processing**
> 
> **3\. Dask for parallel, out-of-core computing:**
> 
> python
>     
>     
>     import dask.dataframe as dd
>     
>     _# Lazy evaluation—doesn't load into memory_
>     ddf = dd.read_csv('huge_file.csv')
>     
>     _# Operations are lazy_
>     ddf = ddf[ddf['amount'] > 1000]
>     ddf['processed'] = ddf['amount'] * 0.95
>     
>     _# Compute triggers execution (result fits in memory)_
>     result = ddf.compute()
> 
> **4\. Database integration:**
> 
> python
>     
>     
>     import sqlalchemy
>     
>     _# Query database directly—let DB handle filtering_
>     query = "SELECT * FROM large_table WHERE date >= '2024-01-01'"
>     df = pd.read_sql(query, engine, chunksize=50000)
> 
> **Real example from Travelers:**
> 
> When processing claims data for actuarial analysis, we used a combination approach:
> 
> python
>     
>     
>     def process_claims_in_batches():
>         offset = 0
>         batch_size = 100000
>         all_results = []
>     
>         while True:
>             _# Fetch batch from Snowflake_
>             query = f"""
>             SELECT * FROM claims
>             WHERE claim_date >= '2020-01-01'
>             ORDER BY claim_id
>             LIMIT {batch_size} OFFSET {offset}
>             """
>     
>             batch_df = pd.read_sql(query, snowflake_conn)
>     
>             if len(batch_df) == 0:
>                 break
>     
>             _# Process batch_
>             batch_df = apply_transformations(batch_df)
>             all_results.append(batch_df)
>     
>             offset += batch_size
>     
>         return pd.concat(all_results)
> 
> This approach allowed us to process billions of claims without memory issues."

* * *

## Q10: Explain groupby operations in Pandas. Give an example.

**Model Answer:**

> "`groupby()` implements the split-apply-combine pattern:
> 
> **Basic syntax:**
> 
> python
>     
>     
>     df.groupby('category')['value'].agg(['sum', 'mean', 'count'])
> 
> **Real example from Google Street View project:**
> 
> We analyzed processing latency by region and image type:
> 
> python
>     
>     
>     import pandas as pd
>     
>     _# Load image processing logs_
>     df = pd.read_csv('image_latency_logs.csv')
>     
>     _# Group by region and image type_
>     latency_analysis = df.groupby(['region', 'image_type']).agg({
>         'collection_to_processing_hrs': ['mean', 'median', 'max'],
>         'processing_to_publish_hrs': ['mean', 'median'],
>         'image_id': 'count'
>     }).round(2)
>     
>     _# Rename columns for clarity_
>     latency_analysis.columns = [
>         'avg_collect_to_proc', 'median_collect_to_proc', 'max_collect_to_proc',
>         'avg_proc_to_publish', 'median_proc_to_publish',
>         'image_count'
>     ]
>     
>     _# Identify bottleneck regions_
>     bottlenecks = latency_analysis[
>         latency_analysis['avg_collect_to_proc'] > latency_analysis['avg_collect_to_proc'].quantile(0.9)
>     ]
> 
> This analysis identified the key bottleneck that led to 25% process optimization.
> 
> **Multiple aggregations with custom functions:**
> 
> python
>     
>     
>     def range_calc(x):
>         return x.max() - x.min()
>     
>     result = df.groupby('region').agg({
>         'latency_days': ['mean', range_calc],
>         'image_count': 'sum'
>     })
> 
> **Transformation with groupby:**
> 
> python
>     
>     
>     _# Add column with regional average_
>     df['regional_avg_latency'] = df.groupby('region')['latency_days'].transform('mean')

* * *

# 3\. NUMPY & SCIENTIFIC COMPUTING

## Q11: What is NumPy and why is it important for data engineering?

**Model Answer:**

> "NumPy is Python's fundamental library for scientific computing, providing **n-dimensional arrays**  and **vectorized operations**.
> 
> **Key features:**
> 
>   * Fast, memory-efficient arrays (contiguous memory, homogeneous types)
>   * Vectorized operations (C-level loops, no Python overhead)
>   * Broadcasting for efficient array arithmetic
>   * Linear algebra, random number generation, FFT
> 

> 
> **Why it matters for data engineering:**
> 
> **1\. Performance:**  NumPy operations are 10-100x faster than pure Python loops
> 
> python
>     
>     
>     _# Slow Python loop_
>     result = [x**2 for x in range(1000000)]  _# ~50ms_
>     
>     _# Fast NumPy_
>     import numpy as np
>     arr = np.arange(1000000)
>     result = arr**2  _# ~5ms (10x faster)_
> 
> **2\. Foundation for Pandas:**  Pandas Series/DataFrames are built on NumPy arrays
> 
> **3\. Memory efficiency:**  NumPy arrays store data in contiguous memory blocks
> 
> **Real example from Refinitiv:**
> 
> We used NumPy for real-time price calculations:
> 
> python
>     
>     
>     import numpy as np
>     
>     _# Price arrays from multiple venues_
>     venue1_prices = np.array([101.2, 101.5, 101.3, ...])
>     venue2_prices = np.array([101.1, 101.4, 101.6, ...])
>     
>     _# Vectorized calculations (millisecond-level)_
>     avg_prices = (venue1_prices + venue2_prices) / 2
>     spreads = np.abs(venue1_prices - venue2_prices)
>     best_prices = np.maximum(venue1_prices, venue2_prices)
>     
>     _# Statistical analysis_
>     mean_price = np.mean(avg_prices)
>     volatility = np.std(avg_prices)
> 
> These vectorized operations were critical for achieving millisecond-level latency."

* * *

## Q12: Explain broadcasting in NumPy with an example.

**Model Answer:**

> "Broadcasting allows NumPy to perform operations on arrays of different shapes by **virtually expanding**  the smaller array to match the larger one—without actually copying data.
> 
> **Rules of broadcasting:**
> 
>   1. Starting from the trailing dimensions, dimensions must be equal or one of them must be 1
>   2. Arrays with fewer dimensions are padded with ones on the left
> 

> 
> **Examples:**
> 
> python
>     
>     
>     import numpy as np
>     
>     _# Scalar broadcasting_
>     arr = np.array([1, 2, 3, 4])
>     result = arr * 2  _# [2, 4, 6, 8] — scalar 2 broadcast to all elements_
>     
>     _# Row-wise operation_
>     matrix = np.array([[1, 2, 3],
>                        [4, 5, 6],
>                        [7, 8, 9]])
>     row_mean = np.mean(matrix, axis=1)  _# [2, 5, 8]_
>     
>     _# Subtract row mean from each row_
>     centered = matrix - row_mean.reshape(-1, 1)
>     _# row_mean reshaped to (3,1) broadcasts across columns_
> 
> **Real example from Travelers:**
> 
> We used broadcasting to normalize claims amounts by category:
> 
> python
>     
>     
>     import numpy as np
>     import pandas as pd
>     
>     _# Claims data_
>     claims = np.array([1000, 2500, 5000, 3000, 8000])
>     categories = np.array([1, 1, 2, 2, 3])  _# claim_type_
>     
>     _# Category statistics_
>     unique_cats = np.unique(categories)  _# [1, 2, 3]_
>     cat_means = np.array([np.mean(claims[categories == cat]) for cat in unique_cats])
>     
>     _# Map each claim to its category mean (broadcasting with advanced indexing)_
>     claim_cat_means = cat_means[categories - 1]  _# categories-1 gives indices 0,1,2_
>     
>     _# Normalize: claim - category_mean_
>     normalized_claims = claims - claim_cat_means
> 
> This vectorized approach processed millions of claims much faster than looping."

* * *

## Q13: How do you optimize Python code for performance? Share specific techniques you've used.

**Model Answer:**

> "Based on my experience with high-volume data processing, here are key optimization techniques:
> 
> **1\. Use vectorized operations instead of loops:**
> 
> python
>     
>     
>     _# Slow_
>     result = []
>     for i in range(len(df)):
>         result.append(df['col1'][i] + df['col2'][i])
>     
>     _# Fast_
>     result = df['col1'] + df['col2']
> 
> **2\. Use built-in functions and libraries (C-optimized):**
> 
> python
>     
>     
>     import numpy as np
>     
>     _# Slow Python sum_
>     total = sum(large_list)  _# Python loop_
>     
>     _# Fast NumPy_
>     total = np.sum(np_array)  _# C-optimized_
> 
> **3\. Profile first, optimize second:**
> 
> python
>     
>     
>     import cProfile
>     import pstats
>     
>     _# Profile your code_
>     cProfile.run('my_function()', 'profile_results')
>     
>     _# Analyze results_
>     p = pstats.Stats('profile_results')
>     p.sort_stats('cumulative').print_stats(10)  _# Top 10 bottlenecks_
> 
> **4\. Use generators for memory efficiency:**
> 
> python
>     
>     
>     _# Bad: loads all into memory_
>     all_lines = file.readlines()
>     for line in all_lines:
>         process(line)
>     
>     _# Good: streams one line at a time_
>     for line in file:
>         process(line)
> 
> **5\. Leverage caching for repeated computations:**
> 
> python
>     
>     
>     from functools import lru_cache
>     
>     @lru_cache(maxsize=128)
>     def expensive_calculation(input_value):
>         _# Computationally intensive_
>         return result
> 
> **6\. Use appropriate data structures:**
> 
>   * Sets for membership testing (O(1) vs O(n) for lists)
>   * Dictionaries for lookups
>   * Deques for queue operations
> 

> 
> **Real example from Refinitiv:**
> 
> We needed to validate 50+ data quality checks on streaming market data. Instead of Python loops:
> 
> python
>     
>     
>     _# Before optimization (Python loops)_
>     def validate_prices(prices):
>         valid = []
>         for price in prices:
>             if price > 0 and price < 10000:
>                 if abs(price - previous_price) < 100:
>                     valid.append(price)
>         return valid
>     
>     _# After optimization (NumPy vectorized)_
>     def validate_prices_vectorized(prices):
>         prices_arr = np.array(prices)
>         valid_mask = (prices_arr > 0) & (prices_arr < 10000)
>         if len(prices_arr) > 1:
>             diff_mask = np.abs(np.diff(prices_arr, prepend=prices_arr[0])) < 100
>             valid_mask = valid_mask & diff_mask
>         return prices_arr[valid_mask]
> 
> This vectorized approach was 40x faster and helped achieve the 99.9% consistency target."

* * *

# 4\. ETL & DATA PIPELINE DEVELOPMENT

## Q14: How do you read large CSV files efficiently in Python?

**Model Answer:**

> "Reading large CSV files efficiently requires different strategies depending on the use case:
> 
> **1\. Use appropriate parameters in  `pd.read_csv()`:**
> 
> python
>     
>     
>     _# Specify only needed columns_
>     df = pd.read_csv('huge_file.csv', usecols=['col1', 'col2', 'col5'])
>     
>     _# Set data types to avoid inference overhead_
>     dtype_dict = {'col1': 'int32', 'col2': 'category', 'col5': 'float32'}
>     df = pd.read_csv('huge_file.csv', dtype=dtype_dict)
>     
>     _# Parse dates efficiently_
>     df = pd.read_csv('huge_file.csv', parse_dates=['date_column'])
>     
>     _# Skip rows if needed_
>     df = pd.read_csv('huge_file.csv', skiprows=1000000, nrows=100000)
> 
> **2\. Chunking for memory management:**
> 
> python
>     
>     
>     chunk_size = 100000
>     chunks = []
>     
>     for chunk in pd.read_csv('huge_file.csv', chunksize=chunk_size):
>         _# Process each chunk_
>         chunk = chunk[chunk['status'] == 'active']
>         chunks.append(chunk)
>     
>     result = pd.concat(chunks)
> 
> **3\. Use alternative formats for better performance:**
> 
> python
>     
>     
>     _# Once converted, Parquet is much faster to read_
>     df.to_parquet('data.parquet', compression='snappy')
>     
>     _# Reading is significantly faster_
>     df = pd.read_parquet('data.parquet')
> 
> **4\. Consider Dask for out-of-core processing:**
> 
> python
>     
>     
>     import dask.dataframe as dd
>     
>     _# Lazy loading—doesn't load all data_
>     ddf = dd.read_csv('huge_file.csv')
>     
>     _# Operations build a task graph_
>     result = ddf.groupby('category')['value'].mean().compute()
> 
> **Real example from Affinion:**
> 
> When migrating 20M customer records, we used chunking with progress tracking:
> 
> python
>     
>     
>     def migrate_customers_in_batches():
>         chunk_size = 50000
>         total_records = 0
>     
>         for i, chunk in enumerate(pd.read_csv('customers.csv', chunksize=chunk_size)):
>             _# Clean chunk_
>             chunk = clean_customer_data(chunk)
>     
>             _# Transform_
>             chunk = apply_business_rules(chunk)
>     
>             _# Write to Azure in batches_
>             write_to_azure(chunk)
>     
>             total_records += len(chunk)
>             print(f"Processed {total_records:,} records...")
>     
>         print(f"Migration complete: {total_records:,} records")
> 
> This approach prevented memory issues and provided real-time visibility into migration progress."

* * *

## Q15: How do you connect Python to databases? What libraries have you used?

**Model Answer:**

> "Based on my projects, I've used several approaches:
> 
> **1\. SQLAlchemy (ORM and core):**
> 
> python
>     
>     
>     from sqlalchemy import create_engine, text
>     import pandas as pd
>     
>     _# Connection string_
>     engine = create_engine('postgresql://user:pass@localhost:5432/dbname')
>     
>     _# Read query directly to DataFrame_
>     df = pd.read_sql("SELECT * FROM customers WHERE signup_date > '2024-01-01'", engine)
>     
>     _# Write DataFrame to database_
>     df.to_sql('processed_customers', engine, if_exists='append', index=False)
>     
>     _# Execute raw SQL with parameters_
>     with engine.connect() as conn:
>         result = conn.execute(
>             text("SELECT * FROM claims WHERE amount > :min_amount"),
>             {"min_amount": 10000}
>         )
> 
> **2\. Database-specific connectors:**
> 
> **SQL Server (pyodbc):**  Used at Refinitiv and Affinion
> 
> python
>     
>     
>     import pyodbc
>     import pandas as pd
>     
>     conn_str = (
>         "DRIVER={ODBC Driver 17 for SQL Server};"
>         "SERVER=server_name;"
>         "DATABASE=database;"
>         "UID=username;PWD=password"
>     )
>     conn = pyodbc.connect(conn_str)
>     
>     _# Query with parameters_
>     query = "SELECT * FROM trades WHERE trade_date >= ?"
>     df = pd.read_sql(query, conn, params=['2024-01-01'])
> 
> **Oracle (cx_Oracle):**  Used at JNJ
> 
> python
>     
>     
>     import cx_Oracle
>     import pandas as pd
>     
>     dsn = cx_Oracle.makedsn('host', 1521, service_name='service')
>     conn = cx_Oracle.connect(user='username', password='password', dsn=dsn)
>     
>     _# Execute query_
>     cursor = conn.cursor()
>     cursor.execute("SELECT * FROM claims WHERE rownum <= 100")
> 
> **Snowflake connector:**  Used at Travelers
> 
> python
>     
>     
>     import snowflake.connector
>     import pandas as pd
>     
>     conn = snowflake.connector.connect(
>         user='username',
>         password='password',
>         account='account_name',
>         warehouse='COMPUTE_WH',
>         database='CLAIMS_DB',
>         schema='GOLD'
>     )
>     
>     _# Execute query_
>     cursor = conn.cursor()
>     cursor.execute("SELECT * FROM fact_claims WHERE claim_date >= '2024-01-01'")
>     
>     _# Direct to pandas_
>     df = cursor.fetch_pandas_all()
> 
> **Key considerations:**
> 
>   * Use connection pooling for production
>   * Always use parameterized queries to prevent SQL injection
>   * Set appropriate fetch size for large result sets
>   * Close connections properly (use context managers)"
> 


* * *

## Q16: How do you handle errors and exceptions in ETL pipelines?

**Model Answer:**

> "Robust error handling is critical in production ETL pipelines. Here's my approach:
> 
> **1\. Structured try-except blocks with logging:**
> 
> python
>     
>     
>     import logging
>     import traceback
>     
>     logging.basicConfig(level=logging.INFO)
>     
>     def etl_pipeline():
>         try:
>             logging.info("Starting ETL pipeline")
>     
>             _# Extract_
>             df = extract_data()
>     
>             _# Transform_
>             df_transformed = transform_data(df)
>     
>             _# Load_
>             load_data(df_transformed)
>     
>             logging.info("ETL pipeline completed successfully")
>     
>         except Exception as e:
>             logging.error(f"Pipeline failed: {str(e)}")
>             logging.error(traceback.format_exc())
>             send_alert(f"ETL pipeline failed: {str(e)}")
>             raise  _# Re-raise if needed_
> 
> **2\. Granular error handling:**
> 
> python
>     
>     
>     def process_with_error_handling():
>         errors = []
>         success_count = 0
>     
>         for i, record in enumerate(data_source):
>             try:
>                 process_record(record)
>                 success_count += 1
>             except DataQualityError as e:
>                 _# Handle expected data quality issues_
>                 errors.append({"record_id": record.id, "error": str(e), "type": "data_quality"})
>                 quarantine_record(record)
>             except ConnectionError as e:
>                 _# Handle transient connection issues_
>                 logging.warning(f"Connection error, retrying: {e}")
>                 time.sleep(5)
>                 retry_record(record)
>             except Exception as e:
>                 _# Unexpected errors_
>                 errors.append({"record_id": record.id, "error": str(e), "type": "unexpected"})
>                 logging.error(f"Unexpected error: {traceback.format_exc()}")
>     
>         _# Report_
>         if errors:
>             generate_error_report(errors)
>     
>         return {"success": success_count, "failed": len(errors)}
> 
> **3\. Retry logic with exponential backoff:**
> 
> python
>     
>     
>     import time
>     from functools import wraps
>     
>     def retry_with_backoff(max_retries=3, base_delay=1):
>         def decorator(func):
>             @wraps(func)
>             def wrapper(*args, **kwargs):
>                 for attempt in range(max_retries):
>                     try:
>                         return func(*args, **kwargs)
>                     except (ConnectionError, TimeoutError) as e:
>                         if attempt == max_retries - 1:
>                             raise
>                         delay = base_delay * (2 ** attempt)  _# Exponential backoff_
>                         logging.warning(f"Attempt {attempt+1} failed, retrying in {delay}s: {e}")
>                         time.sleep(delay)
>             return wrapper
>         return decorator
>     
>     @retry_with_backoff(max_retries=3)
>     def call_external_api():
>         _# API call that might fail_
>         pass
> 
> **4\. Dead letter queues/quarantine:**
> 
> python
>     
>     
>     def quarantine_record(record, reason):
>         """Move failed records to quarantine for later review"""
>         quarantine_df = pd.DataFrame([{
>             'record_id': record.id,
>             'data': json.dumps(record.__dict__),
>             'error_reason': reason,
>             'timestamp': datetime.now()
>         }])
>     
>         _# Append to quarantine table_
>         quarantine_df.to_sql('quarantine_records', engine, if_exists='append', index=False)
> 
> **Real example from Refinitiv:**
> 
> We built a comprehensive error handling framework for the 50+ automated checks:
> 
> python
>     
>     
>     class DataValidationFramework:
>         def __init__(self):
>             self.errors = []
>             self.warnings = []
>     
>         def validate(self, data, rules):
>             for rule in rules:
>                 try:
>                     result = rule.validate(data)
>                     if not result.passed:
>                         if result.severity == 'error':
>                             self.errors.append(result)
>                         else:
>                             self.warnings.append(result)
>                 except Exception as e:
>                     self.errors.append({
>                         'rule': rule.name,
>                         'error': f"Validation rule failed: {str(e)}"
>                     })
>     
>             _# Take action based on error counts_
>             if len(self.errors) > 10:
>                 raise PipelineAbortException("Too many validation errors")
>             elif self.errors:
>                 quarantine_problematic_records(self.errors)
>     
>             return len(self.errors) == 0
> 
> This framework helped achieve the 85% reduction in reconciliation issues."

* * *

## Q17: How do you work with APIs in Python for data extraction?

**Model Answer:**

> "API integration is common in data pipelines. Here's my approach:
> 
> **1\. Using requests library:**
> 
> python
>     
>     
>     import requests
>     import pandas as pd
>     from requests.adapters import HTTPAdapter
>     from urllib3.util.retry import Retry
>     
>     _# Configure session with retries_
>     session = requests.Session()
>     retry_strategy = Retry(
>         total=3,
>         backoff_factor=1,
>         status_forcelist=[429, 500, 502, 503, 504],
>     )
>     adapter = HTTPAdapter(max_retries=retry_strategy)
>     session.mount("http://", adapter)
>     session.mount("https://", adapter)
>     
>     _# Set headers_
>     session.headers.update({
>         'Authorization': f'Bearer {API_KEY}',
>         'Content-Type': 'application/json'
>     })
>     
>     _# Make request with error handling_
>     try:
>         response = session.get(
>             'https://api.example.com/data',
>             params={'start_date': '2024-01-01', 'limit': 1000},
>             timeout=30
>         )
>         response.raise_for_status()  _# Raise exception for 4xx/5xx_
>     
>         data = response.json()
>         df = pd.DataFrame(data['results'])
>     
>     except requests.exceptions.Timeout:
>         logging.error("API request timed out")
>     except requests.exceptions.ConnectionError:
>         logging.error("Connection error")
>     except requests.exceptions.HTTPError as e:
>         logging.error(f"HTTP error: {e}")
> 
> **2\. Pagination handling:**
> 
> python
>     
>     
>     def fetch_all_pages(base_url, params):
>         all_data = []
>         page = 1
>     
>         while True:
>             params['page'] = page
>             response = session.get(base_url, params=params)
>             data = response.json()
>     
>             if not data['results']:
>                 break
>     
>             all_data.extend(data['results'])
>             page += 1
>     
>             _# Rate limiting_
>             time.sleep(0.5)  _# Respect API limits_
>     
>         return pd.DataFrame(all_data)
> 
> **3\. Real example from Google Street View project:**
> 
> We used internal APIs for defect tracking:
> 
> python
>     
>     
>     class BugTrackingAPIClient:
>         def __init__(self, api_key, base_url):
>             self.session = requests.Session()
>             self.session.headers.update({
>                 'X-API-Key': api_key,
>                 'Content-Type': 'application/json'
>             })
>             self.base_url = base_url
>     
>         def get_defects(self, date_from, date_to, project_id):
>             """Fetch defects with pagination"""
>             all_defects = []
>             page = 1
>             page_size = 100
>     
>             while True:
>                 params = {
>                     'created_from': date_from,
>                     'created_to': date_to,
>                     'project_id': project_id,
>                     'page': page,
>                     'page_size': page_size
>                 }
>     
>                 response = self.session.get(
>                     f"{self.base_url}/defects",
>                     params=params
>                 )
>     
>                 if response.status_code == 200:
>                     data = response.json()
>                     defects = data.get('defects', [])
>     
>                     if not defects:
>                         break
>     
>                     all_defects.extend(defects)
>                     page += 1
>     
>                     _# Check if last page_
>                     if len(defects) < page_size:
>                         break
>                 else:
>                     logging.error(f"API error: {response.status_code}")
>                     break
>     
>             return pd.DataFrame(all_defects)
>     
>         def update_defect(self, defect_id, status, comment):
>             """Update defect status"""
>             payload = {
>                 'status': status,
>                 'comment': comment
>             }
>     
>             response = self.session.put(
>                 f"{self.base_url}/defects/{defect_id}",
>                 json=payload
>             )
>     
>             return response.status_code == 200
> 
> This client helped streamline defect reporting and cut resolution time by 18 hours."

* * *

# 5\. ADVANCED PYTHON CONCEPTS

## Q18: What are context managers and how have you used them?

**Model Answer:**

> "Context managers manage resources by ensuring proper setup and cleanup, even if errors occur. They're used with the `with` statement.
> 
> **Built-in examples:**
> 
> python
>     
>     
>     _# File handling_
>     with open('file.txt', 'r') as f:
>         content = f.read()
>     _# File automatically closed, even if exception occurs_
>     
>     _# Database connections_
>     with engine.connect() as conn:
>         result = conn.execute(query)
>     _# Connection automatically returned to pool_
> 
> **Creating custom context managers:**
> 
> **Method 1: Class-based (using  `__enter__` and `__exit__`):**
> 
> python
>     
>     
>     class DatabaseConnection:
>         def __init__(self, connection_string):
>             self.connection_string = connection_string
>             self.conn = None
>     
>         def __enter__(self):
>             print("Opening database connection")
>             self.conn = create_connection(self.connection_string)
>             return self.conn
>     
>         def __exit__(self, exc_type, exc_val, exc_tb):
>             print("Closing database connection")
>             if self.conn:
>                 self.conn.close()
>             _# Return False to propagate exceptions, True to suppress_
>             return False
>     
>     _# Usage_
>     with DatabaseConnection(conn_string) as conn:
>         conn.execute(query)
> 
> **Method 2: Using  `contextlib` (simpler):**
> 
> python
>     
>     
>     from contextlib import contextmanager
>     
>     @contextmanager
>     def database_connection(connection_string):
>         print("Opening connection")
>         conn = create_connection(connection_string)
>         try:
>             yield conn
>         finally:
>             print("Closing connection")
>             conn.close()
>     
>     _# Usage_
>     with database_connection(conn_string) as conn:
>         conn.execute(query)
> 
> **Real example from Travelers:**
> 
> We used context managers for Snowflake connections and transactions:
> 
> python
>     
>     
>     from contextlib import contextmanager
>     
>     @contextmanager
>     def snowflake_transaction(connection_params):
>         """Context manager for Snowflake transactions with automatic rollback on error"""
>         conn = snowflake.connector.connect(**connection_params)
>         try:
>             yield conn
>             conn.commit()  _# Commit if no exception_
>         except Exception as e:
>             conn.rollback()  _# Rollback on error_
>             logging.error(f"Transaction failed: {e}")
>             raise
>         finally:
>             conn.close()
>     
>     _# Usage in ETL pipeline_
>     def load_to_snowflake(df, table_name):
>         with snowflake_transaction(conn_params) as conn:
>             cursor = conn.cursor()
>     
>             _# Stage data_
>             cursor.execute("CREATE OR REPLACE TEMPORARY STAGE ...")
>     
>             _# Copy data_
>             cursor.execute(f"COPY INTO {table_name} FROM @%...")
>     
>             _# If we get here, commit happens automatically_
> 
> This ensured data consistency—if any step failed, the transaction rolled back, preventing partial loads."

* * *

## Q19: Explain the difference between shallow copy and deep copy.

**Model Answer:**

> "Copying in Python can be shallow or deep, which matters when working with nested data structures.
> 
> **Shallow copy:**  Creates a new object but inserts references to the original child objects
> 
> python
>     
>     
>     import copy
>     
>     original = [[1, 2, 3], [4, 5, 6]]
>     shallow = copy.copy(original)
>     
>     _# Modifying a nested list affects both!_
>     shallow[0][0] = 99
>     print(original[0][0])  _# 99 — original changed!_
>     
>     _# But top-level modifications are independent_
>     shallow.append([7, 8, 9])
>     print(len(original))  _# 2 — unchanged_
> 
> **Deep copy:**  Creates a new object and recursively copies all nested objects
> 
> python
>     
>     
>     original = [[1, 2, 3], [4, 5, 6]]
>     deep = copy.deepcopy(original)
>     
>     _# Modifying nested list only affects the copy_
>     deep[0][0] = 99
>     print(original[0][0])  _# 1 — original unchanged!_
> 
> **Real example from Affinion:**
> 
> When processing customer records with nested structures, shallow copies caused issues:
> 
> python
>     
>     
>     _# Problem: Shallow copy caused unintended modifications_
>     def process_customer_batch(customers):
>         batch_copy = customers.copy()  _# Shallow copy!_
>     
>         for customer in batch_copy:
>             _# This modifies original if customer has nested lists/dicts_
>             customer['address']['city'] = customer['address']['city'].upper()
>     
>     _# Solution: Use deep copy_
>     import copy
>     
>     def process_customer_batch_safe(customers):
>         batch_copy = copy.deepcopy(customers)
>     
>         for customer in batch_copy:
>             customer['address']['city'] = customer['address']['city'].upper()
>     
>         return batch_copy  _# Original remains unchanged_
> 
> **When to use each:**
> 
>   * **Shallow copy:**  Simple flat structures, performance-critical, intentional sharing
>   * **Deep copy:**  Complex nested data, when you need true independence, for data transformation that shouldn't affect source"
> 


* * *

## Q20: What are lambda functions and how have you used them in data pipelines?

**Model Answer:**

> "Lambda functions are small, anonymous functions defined with the `lambda` keyword. They can have any number of arguments but only one expression.
> 
> **Syntax:**
> 
> python
>     
>     
>     lambda arguments: expression
>     
>     _# Example_
>     square = lambda x: x**2
>     print(square(5))  _# 25_
> 
> **Common use cases in data engineering:**
> 
> **1\. With Pandas  `apply()`:**
> 
> python
>     
>     
>     _# Simple transformation_
>     df['amount_usd'] = df['amount_eur'].apply(lambda x: x * 1.18)
>     
>     _# Conditional logic_
>     df['risk_level'] = df['claim_amount'].apply(
>         lambda x: 'High' if x > 10000 else 'Medium' if x > 5000 else 'Low'
>     )
> 
> **2\. With  `map()` for Series:**
> 
> python
>     
>     
>     _# Clean text data_
>     df['clean_name'] = df['raw_name'].map(lambda x: x.strip().lower())
> 
> **3\. With  `filter()`:**
> 
> python
>     
>     
>     _# Filter records_
>     high_value_claims = list(filter(lambda claim: claim['amount'] > 10000, claims))
> 
> **4\. With  `sorted()` custom sorting:**
> 
> python
>     
>     
>     _# Sort by multiple criteria_
>     sorted_customers = sorted(customers, key=lambda x: (x['region'], -x['revenue']))
> 
> **Real example from Google Street View:**
> 
> We used lambda functions for dynamic column creation:
> 
> python
>     
>     
>     _# Classify latency issues_
>     df['priority'] = df.apply(lambda row:
>         'Critical' if row['latency_days'] > 30 else
>         'High' if row['latency_days'] > 15 else
>         'Medium' if row['latency_days'] > 7 else
>         'Low',
>         axis=1
>     )
>     
>     _# Create composite keys for deduplication_
>     df['unique_key'] = df.apply(
>         lambda row: f"{row['region']}_{row['capture_date']}_{row['image_id']}",
>         axis=1
>     )
> 
> **When NOT to use lambdas:**
> 
>   * Complex logic (use regular functions for readability)
>   * Reusable logic (define named functions)
>   * Debugging (lambdas are harder to debug)
> 

> 
> **Performance note:**  For simple operations, lambdas are fine. For performance-critical code, vectorized operations are better."

* * *

# 6\. PYTHON FOR CLOUD & BIG DATA

## Q21: How do you use Python with Azure services? (Based on your Affinion project)

**Model Answer:**

> "In my Affinion project, we used Python extensively with Azure services:
> 
> **1\. Azure Blob Storage:**
> 
> python
>     
>     
>     from azure.storage.blob import BlobServiceClient
>     import pandas as pd
>     from io import StringIO
>     
>     _# Connect to Blob Storage_
>     conn_str = "DefaultEndpointsProtocol=https;AccountName=...;AccountKey=...;EndpointSuffix=core.windows.net"
>     blob_service_client = BlobServiceClient.from_connection_string(conn_str)
>     container_client = blob_service_client.get_container_client("data-lake")
>     
>     _# Upload file_
>     blob_client = container_client.get_blob_client("raw/customers.csv")
>     blob_client.upload_blob(df.to_csv(index=False), overwrite=True)
>     
>     _# Download file_
>     blob_client = container_client.get_blob_client("processed/customers.parquet")
>     download_stream = blob_client.download_blob()
>     df = pd.read_parquet(BytesIO(download_stream.readall()))
> 
> **2\. Azure Data Lake Storage Gen2:**
> 
> python
>     
>     
>     from azure.storage.filedatalake import DataLakeServiceClient
>     
>     _# ADLS Gen2 client_
>     service_client = DataLakeServiceClient.from_connection_string(conn_str)
>     file_system_client = service_client.get_file_system_client("bronze")
>     
>     _# Create directory structure_
>     directory_client = file_system_client.get_directory_client("year=2024/month=01")
>     directory_client.create_directory()
>     
>     _# Upload file_
>     file_client = directory_client.get_file_client("customers.parquet")
>     file_client.upload_data(df.to_parquet(), overwrite=True)
> 
> **3\. Azure SQL Database:**
> 
> python
>     
>     
>     import pyodbc
>     import pandas as pd
>     
>     _# Connection string_
>     conn_str = (
>         "Driver={ODBC Driver 17 for SQL Server};"
>         "Server=tcp:server.database.windows.net,1433;"
>         "Database=mydb;"
>         "Uid=user;Pwd=password;"
>         "Encrypt=yes;TrustServerCertificate=no;"
>     )
>     
>     _# Write DataFrame to SQL_
>     def write_to_azure_sql(df, table_name):
>         conn = pyodbc.connect(conn_str)
>         cursor = conn.cursor()
>     
>         _# Create table if not exists_
>         _# Use bulk insert for performance_
>         for index, row in df.iterrows():
>             cursor.execute(f"""
>                 INSERT INTO {table_name} (col1, col2, col3)
>                 VALUES (?, ?, ?)
>             """, row['col1'], row['col2'], row['col3'])
>     
>         conn.commit()
>         conn.close()
> 
> **4\. Azure Functions (serverless):**
> 
> python
>     
>     
>     import azure.functions as func
>     import logging
>     import pandas as pd
>     
>     def main(req: func.HttpRequest) -> func.HttpResponse:
>         logging.info('Python HTTP trigger function processed a request.')
>     
>         try:
>             _# Get parameters_
>             date = req.params.get('date')
>     
>             _# Process data_
>             result = process_daily_data(date)
>     
>             return func.HttpResponse(
>                 f"Successfully processed data for {date}",
>                 status_code=200
>             )
>         except Exception as e:
>             return func.HttpResponse(
>                 f"Error: {str(e)}",
>                 status_code=500
>             )
> 
> **Key lessons from Affinion:**
> 
>   * Use connection pooling for production
>   * Implement retry logic for transient failures
>   * Monitor costs—Python loops can be expensive at scale"
> 


* * *

## Q22: How do you use Python with Snowflake? (Based on your Travelers project)

**Model Answer:**

> "At Travelers, we used Python extensively with Snowflake:
> 
> **1\. Snowflake Connector for Python:**
> 
> python
>     
>     
>     import snowflake.connector
>     import pandas as pd
>     
>     _# Connection_
>     conn = snowflake.connector.connect(
>         user='username',
>         password='password',
>         account='account_name',
>         warehouse='COMPUTE_WH',
>         database='CLAIMS_DB',
>         schema='BRONZE'
>     )
>     
>     _# Execute query_
>     cursor = conn.cursor()
>     cursor.execute("SELECT * FROM raw_claims WHERE claim_date >= '2024-01-01' LIMIT 1000")
>     
>     _# Fetch as pandas DataFrame_
>     df = cursor.fetch_pandas_all()
> 
> **2\. Snowpark Python (Snowflake's native Python API):**
> 
> python
>     
>     
>     from snowflake.snowpark import Session
>     from snowflake.snowpark.functions import col, avg
>     
>     _# Create session_
>     connection_params = {
>         "account": "account_name",
>         "user": "username",
>         "password": "password",
>         "role": "DATA_ENGINEER",
>         "warehouse": "COMPUTE_WH",
>         "database": "CLAIMS_DB",
>         "schema": "GOLD"
>     }
>     
>     session = Session.builder.configs(connection_params).create()
>     
>     _# Load table_
>     claims_df = session.table("fact_claims")
>     
>     _# Transform (executed in Snowflake, not locally)_
>     result = (claims_df
>         .filter(col("claim_date") >= "2024-01-01")
>         .group_by("region")
>         .agg(avg("claim_amount").alias("avg_claim"))
>         .order_by("avg_claim", ascending=False)
>     )
>     
>     _# Collect results (triggers execution)_
>     result_df = result.to_pandas()
> 
> **3\. Bulk data loading:**
> 
> python
>     
>     
>     def load_to_snowflake(df, table_name):
>         """Load pandas DataFrame to Snowflake efficiently"""
>     
>         _# Convert to CSV in memory_
>         from io import StringIO
>         csv_buffer = StringIO()
>         df.to_csv(csv_buffer, index=False, header=False)
>         csv_buffer.seek(0)
>     
>         _# Use COPY command for bulk load_
>         cursor = conn.cursor()
>         cursor.execute("""
>             CREATE OR REPLACE TEMPORARY STAGE my_stage
>         """)
>     
>         _# Put file to stage_
>         cursor.execute("PUT file://... @my_stage")  _# Or use internal stage_
>     
>         _# Copy into table_
>         cursor.execute(f"""
>             COPY INTO {table_name}
>             FROM @my_stage
>             FILE_FORMAT = (TYPE = CSV FIELD_OPTIONALLY_ENCLOSED_BY = '"')
>             ON_ERROR = 'CONTINUE'
>         """)
> 
> **4\. ETL pipeline example:**
> 
> python
>     
>     
>     def bronze_to_silver_pipeline():
>         """Transform raw claims to silver layer"""
>     
>         _# Connect to Snowflake_
>         conn = snowflake.connector.connect(**conn_params)
>     
>         _# Read from bronze_
>         bronze_df = pd.read_sql("""
>             SELECT *
>             FROM bronze.raw_claims
>             WHERE ingestion_date = CURRENT_DATE()
>         """, conn)
>     
>         _# Transform with Python_
>         silver_df = (bronze_df
>             .pipe(clean_claim_data)
>             .pipe(validate_claims)
>             .pipe(enrich_with_reference_data)
>         )
>     
>         _# Load to silver_
>         load_to_snowflake(silver_df, "silver.claims_cleaned")
>     
>         conn.close()
> 
> This approach helped us achieve the Bronze-Silver-Gold architecture with efficient data processing."

* * *

# 7\. PYTHON TESTING & DEBUGGING

## Q23: How do you test Python code in data pipelines?

**Model Answer:**

> "Testing is critical for reliable data pipelines. Here's my approach:
> 
> **1\. Unit tests with pytest:**
> 
> python
>     
>     
>     _# test_transformations.py_
>     import pytest
>     import pandas as pd
>     import numpy as np
>     
>     def test_clean_claim_data():
>         _# Arrange_
>         input_df = pd.DataFrame({
>             'claim_id': [1, 2, 3],
>             'claim_amount': [1000, -500, None],
>             'claim_date': ['2024-01-01', 'invalid', '2024-01-03']
>         })
>     
>         _# Act_
>         result = clean_claim_data(input_df)
>     
>         _# Assert_
>         assert len(result) == 1  _# Only valid record_
>         assert result.iloc[0]['claim_id'] == 1
>         assert result.iloc[0]['claim_amount'] == 1000
> 
> **2\. Data quality tests with Great Expectations:**
> 
> python
>     
>     
>     import great_expectations as ge
>     
>     def test_data_quality(df):
>         _# Convert to Great Expectations DataFrame_
>         ge_df = ge.from_pandas(df)
>     
>         _# Define expectations_
>         ge_df.expect_column_values_to_not_be_null('claim_id')
>         ge_df.expect_column_values_to_be_between('claim_amount', 0, 1000000)
>         ge_df.expect_column_values_to_match_regex('claim_date', r'\d{4}-\d{2}-\d{2}')
>     
>         _# Validate_
>         results = ge_df.validate()
>     
>         if not results['success']:
>             failed_expectations = [
>                 exp for exp in results['results'] if not exp['success']
>             ]
>             raise DataQualityError(f"Data quality failed: {failed_expectations}")
> 
> **3\. Integration tests:**
> 
> python
>     
>     
>     def test_end_to_end_pipeline():
>         """Test full ETL pipeline with small dataset"""
>     
>         _# Setup test data_
>         test_data = generate_test_data(100)
>     
>         _# Run pipeline_
>         result = run_etl_pipeline(test_data)
>     
>         _# Verify_
>         assert len(result) == expected_count
>         assert result['amount'].sum() == expected_sum
>     
>         _# Cleanup_
>         cleanup_test_data()
> 
> **4\. Mocking external dependencies:**
> 
> python
>     
>     
>     from unittest.mock import Mock, patch
>     
>     @patch('database_connector.get_connection')
>     def test_extract_data(mock_get_connection):
>         _# Mock database connection_
>         mock_conn = Mock()
>         mock_cursor = Mock()
>         mock_cursor.fetchall.return_value = [(1, 'test', 100)]
>         mock_conn.cursor.return_value = mock_cursor
>         mock_get_connection.return_value = mock_conn
>     
>         _# Test function_
>         result = extract_data()
>     
>         _# Assert_
>         assert len(result) == 1
>         mock_get_connection.assert_called_once()
> 
> **Real example from JNJ:**
> 
> We used X-Ray for test automation in CI/CD pipeline:
> 
> python
>     
>     
>     _# Jenkins pipeline stage_
>     stage('Run Python Tests') {
>         steps {
>             sh '''
>                 pytest tests/ --junitxml=reports/test-results.xml
>                 coverage run -m pytest
>                 coverage report --fail-under=80
>             '''
>         }
>     }
> 
> This approach caught issues early and maintained code quality across the 24-person team."

* * *

## Q24: How do you debug Python code effectively?

**Model Answer:**

> "Effective debugging is essential for complex data pipelines. Here's my toolkit:
> 
> **1\. Logging (better than print):**
> 
> python
>     
>     
>     import logging
>     import sys
>     
>     _# Configure logging_
>     logging.basicConfig(
>         level=logging.INFO,
>         format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
>         handlers=[
>             logging.FileHandler('pipeline.log'),
>             logging.StreamHandler(sys.stdout)
>         ]
>     )
>     
>     logger = logging.getLogger(__name__)
>     
>     def process_data(df):
>         logger.info(f"Starting processing with {len(df)} records")
>     
>         logger.debug(f"Data types: {df.dtypes}")
>         logger.debug(f"Null counts: {df.isnull().sum()}")
>     
>         try:
>             result = complex_transformation(df)
>             logger.info(f"Processing complete: {len(result)} records")
>             return result
>         except Exception as e:
>             logger.error(f"Processing failed: {str(e)}", exc_info=True)
>             raise
> 
> **2\. Using pdb (Python debugger):**
> 
> python
>     
>     
>     import pdb
>     
>     def complex_function(df):
>         _# Set breakpoint_
>         pdb.set_trace()
>     
>         _# Now you can inspect variables:_
>         _# p df.head()_
>         _# p df.dtypes_
>         _# n (next line)_
>         _# c (continue)_
>         _# q (quit)_
>     
>         result = df.groupby('category').agg(...)
>         return result
>     
>     _# Post-mortem debugging_
>     try:
>         result = risky_function()
>     except Exception:
>         import pdb
>         pdb.post_mortem()  _# Start debugger at exception point_
> 
> **3\. Interactive debugging with Jupyter:**
> 
> python
>     
>     
>     _# In Jupyter notebook_
>     %debug  _# After an exception, start debugger_
>     
>     _# Or use ipdb_
>     import ipdb
>     ipdb.set_trace()
> 
> **4\. Profiling to find bottlenecks:**
> 
> python
>     
>     
>     import cProfile
>     import pstats
>     from io import StringIO
>     
>     def profile_function():
>         profiler = cProfile.Profile()
>         profiler.enable()
>     
>         _# Code to profile_
>         result = expensive_operation()
>     
>         profiler.disable()
>     
>         _# Print results_
>         stream = StringIO()
>         stats = pstats.Stats(profiler, stream=stream)
>         stats.sort_stats('cumulative')
>         stats.print_stats(20)  _# Top 20_
>     
>         print(stream.getvalue())
> 
> **5\. Pandas-specific debugging:**
> 
> python
>     
>     
>     _# Check intermediate results_
>     def debug_pipeline(df):
>         print(f"Shape: {df.shape}")
>         print(f"Columns: {df.columns.tolist()}")
>         print(f"Data types:\n{df.dtypes}")
>         print(f"Missing values:\n{df.isnull().sum()}")
>         print(f"Sample:\n{df.head(3)}")
>     
>         _# Check for unexpected values_
>         print(f"Unique values in category: {df['category'].unique()}")
>         print(f"Value ranges:\n{df.describe()}")
>     
>         return df
>     
>     _# Insert into pipeline_
>     df = (df
>         .pipe(step1)
>         .pipe(debug_pipeline)  _# Debug intermediate_
>         .pipe(step2)
>     )
> 
> **Real example from Refinitiv:**
> 
> When debugging the 50+ validation checks, we used conditional breakpoints:
> 
> python
>     
>     
>     def validate_trade(trade):
>         _# Only break for specific failing case_
>         if trade['id'] == 12345:  _# Known problematic record_
>             pdb.set_trace()
>     
>         _# Validation logic..._
>         return is_valid
> 
> This targeted approach saved hours of debugging time."

* * *

# 8\. PYTHON DESIGN PATTERNS & BEST PRACTICES

## Q25: How do you structure Python code for maintainable data pipelines?

**Model Answer:**

> "Based on my experience across multiple projects, here's how I structure Python code for maintainability:
> 
> **1\. Project structure:**
> 
> text
>     
>     
>     project_name/
>     ├── src/
>     │   ├── __init__.py
>     │   ├── extract/
>     │   │   ├── __init__.py
>     │   │   ├── base_extractor.py
>     │   │   ├── sql_extractor.py
>     │   │   └── api_extractor.py
>     │   ├── transform/
>     │   │   ├── __init__.py
>     │   │   ├── base_transformer.py
>     │   │   ├── cleaning.py
>     │   │   └── aggregations.py
>     │   ├── load/
>     │   │   ├── __init__.py
>     │   │   ├── base_loader.py
>     │   │   ├── snowflake_loader.py
>     │   │   └── azure_loader.py
>     │   ├── utils/
>     │   │   ├── __init__.py
>     │   │   ├── logging_config.py
>     │   │   ├── error_handling.py
>     │   │   └── validation.py
>     │   └── config/
>     │       ├── __init__.py
>     │       ├── settings.py
>     │       └── logging.conf
>     ├── tests/
>     │   ├── __init__.py
>     │   ├── test_extractors.py
>     │   ├── test_transformers.py
>     │   └── test_loaders.py
>     ├── notebooks/
>     │   └── exploration.ipynb
>     ├── scripts/
>     │   └── run_pipeline.py
>     ├── requirements.txt
>     ├── setup.py
>     └── README.md
> 
> **2\. Configuration management:**
> 
> python
>     
>     
>     _# config/settings.py_
>     import os
>     from dataclasses import dataclass
>     from typing import Optional
>     
>     @dataclass
>     class DatabaseConfig:
>         host: str
>         port: int
>         database: str
>         username: str
>         password: str
>     
>         @classmethod
>         def from_env(cls, prefix: str = "DB_"):
>             return cls(
>                 host=os.getenv(f"{prefix}HOST", "localhost"),
>                 port=int(os.getenv(f"{prefix}PORT", "5432")),
>                 database=os.getenv(f"{prefix}NAME"),
>                 username=os.getenv(f"{prefix}USER"),
>                 password=os.getenv(f"{prefix}PASSWORD")
>             )
>     
>     @dataclass
>     class PipelineConfig:
>         batch_size: int = 10000
>         retry_attempts: int = 3
>         retry_delay: int = 5
>         log_level: str = "INFO"
>     
>     _# Load configuration_
>     db_config = DatabaseConfig.from_env()
>     pipeline_config = PipelineConfig()
> 
> **3\. Abstract base classes for extensibility:**
> 
> python
>     
>     
>     _# extract/base_extractor.py_
>     from abc import ABC, abstractmethod
>     import pandas as pd
>     from typing import Optional
>     
>     class BaseExtractor(ABC):
>         """Abstract base class for all data extractors"""
>     
>         def __init__(self, config):
>             self.config = config
>             self.logger = logging.getLogger(self.__class__.__name__)
>     
>         @abstractmethod
>         def extract(self, **kwargs) -> pd.DataFrame:
>             """Extract data from source"""
>             pass
>     
>         def validate_extract(self, df: pd.DataFrame) -> bool:
>             """Common validation logic"""
>             if df.empty:
>                 self.logger.warning("Extracted empty DataFrame")
>                 return False
>             return True
>     
>     _# extract/sql_extractor.py_
>     class SQLExtractor(BaseExtractor):
>         def __init__(self, config, connection_string):
>             super().__init__(config)
>             self.connection_string = connection_string
>     
>         def extract(self, query: str, **kwargs) -> pd.DataFrame:
>             self.logger.info(f"Executing query: {query[:100]}...")
>     
>             try:
>                 df = pd.read_sql(query, self.connection_string)
>     
>                 if self.validate_extract(df):
>                     self.logger.info(f"Extracted {len(df)} records")
>                     return df
>                 else:
>                     return pd.DataFrame()
>     
>             except Exception as e:
>                 self.logger.error(f"Extraction failed: {e}")
>                 raise
> 
> **4\. Factory pattern for pipeline components:**
> 
> python
>     
>     
>     class ExtractorFactory:
>         """Factory for creating appropriate extractor based on source type"""
>     
>         @staticmethod
>         def create_extractor(source_type: str, config):
>             if source_type == "sql":
>                 return SQLExtractor(config, config.sql_connection)
>             elif source_type == "api":
>                 return APIExtractor(config, config.api_key)
>             elif source_type == "file":
>                 return FileExtractor(config)
>             else:
>                 raise ValueError(f"Unknown source type: {source_type}")
>     
>     _# Usage_
>     extractor = ExtractorFactory.create_extractor(source_type, config)
>     df = extractor.extract(**params)
> 
> **5\. Pipeline orchestration:**
> 
> python
>     
>     
>     _# scripts/run_pipeline.py_
>     import argparse
>     import logging
>     from src.extract import ExtractorFactory
>     from src.transform import Transformer
>     from src.load import LoaderFactory
>     from src.utils.error_handling import PipelineErrorHandler
>     
>     def main():
>         parser = argparse.ArgumentParser()
>         parser.add_argument("--date", required=True)
>         parser.add_argument("--source", default="sql")
>         args = parser.parse_args()
>     
>         _# Load config_
>         config = PipelineConfig.from_env()
>     
>         _# Initialize error handler_
>         error_handler = PipelineErrorHandler(config)
>     
>         try:
>             _# Extract_
>             extractor = ExtractorFactory.create_extractor(args.source, config)
>             df = extractor.extract(date=args.date)
>     
>             _# Transform_
>             transformer = Transformer(config)
>             df_transformed = transformer.transform(df)
>     
>             _# Load_
>             loader = LoaderFactory.create_loader("snowflake", config)
>             loader.load(df_transformed, f"table_{args.date}")
>     
>             logging.info(f"Pipeline completed successfully for {args.date}")
>     
>         except Exception as e:
>             error_handler.handle(e)
>             raise
>     
>     if __name__ == "__main__":
>         main()
> 
> **6\. Dependency management:**
> 
> bash
>     
>     
>     _# requirements.txt_
>     pandas==2.0.3
>     numpy==1.24.3
>     snowflake-connector-python==3.0.4
>     azure-storage-blob==12.16.0
>     pyodbc==4.0.39
>     sqlalchemy==2.0.19
>     pytest==7.4.0
>     black==23.7.0  _# Code formatting_
>     flake8==6.1.0  _# Linting_
>     mypy==1.4.1    _# Type checking_
> 
> This structure kept our 24-person JNJ team productive and made onboarding new members easier."

* * *

# 📋 QUICK REFERENCE: Python Topics by Project

Project| Python Topics to Reference  
---|---  
**Google Street View**|  API integration, automation, Pandas for geospatial, decorators for retry logic, logging  
**JNJ**|  Data cleaning, missing value handling, SQLAlchemy, testing with pytest, error handling  
**Refinitiv**|  Performance optimization, NumPy vectorization, real-time processing, multiprocessing  
**Affinion**|  Azure SDK, chunking large datasets, ETL patterns, context managers for connections  
**Travelers**|  Snowpark, data validation frameworks, memory management, batch processing  
  
* * *

# 🎯 Final Tips for Python Interview

Tip| Why  
---|---  
**Connect to your projects**|  Every answer should reference your real experience  
**Show code examples**|  Be ready to write code on whiteboard/shared editor  
**Discuss trade-offs**|  Show you understand pros/cons of different approaches  
**Mention testing**|  Production code needs tests—mention pytest, Great Expectations  
**Talk about performance**|  Show you care about efficiency at scale  
**Be honest about gaps**|  Acknowledge what you don't know, show how you'd learn
