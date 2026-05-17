---
title: "SQL Server"
date: 2013-08-13 12:02:44 
author: shajeebhameed
layout: page
slug: sql-server
status: publish
---

1\. Which TCP/IP port does SQL Server run on? How can it be changed? 

SQL Server runs on port 1433. It can be changed from the Network Utility TCP/IP properties. 

2\. What are the difference between clustered and a non-clustered index? 

  1. **A clustered index** is a special type of index that reorders the way records in the table are physically stored. Therefore table can have only one clustered index. The leaf nodes of a clustered index contain the data pages.
  2. **A non clustered index** is a special type of index in which the logical order of the index does not match the physical stored order of the rows on disk. The leaf node of a non clustered index does not consist of the data pages. Instead, the leaf nodes contain index rows.

With a clustered index the rows are stored physically on the disk in the same order as the index. There can therefore be only one clustered index. With a non clustered index there is a second list that has pointers to the physical rows. You can have many non clustered indexes, although each new index will increase the time it takes to write new records. The primary key created for the `StudId `column will create a clustered index for the `Studid `column. A table can have only one clustered index on it. When creating the clustered index, SQL server 2005 reads the `Studid `column and forms a Binary tree on it. This binary tree information is then stored separately in the disc. Expand the table `Student `and then expand the `Index`es. A non-clustered index is useful for columns that have some repeated values. Say for example, `AccountType `column of a bank database may have 10 million rows. But, the distinct values of account type may be 10-15. A clustered index is automatically created when we create the primary key for the table. We need to take care of the creation of the non-clustered index. 

3\. What are the different index configurations a table can have? 

A table can have one of the following index configurations: 

  1. No indexes
  2. A clustered index
  3. A clustered index and many nonclustered indexes
  4. A nonclustered index
  5. Many nonclustered indexes



4\. What are different types of Collation Sensitivity? 

  1. **Case sensitivity** \- A and a, B and b, etc.
  2. **Accent sensitivity**
  3. **Kana Sensitivity** \- When Japanese kana characters Hiragana and Katakana are treated differently, it is called Kana sensitive.
  4. **Width sensitivity** \- A single-byte character (half-width) and the same character represented as a double-byte character (full-width) are treated differently than it is width sensitive.



5\. What is OLTP (Online Transaction Processing)? 

In OLTP - online transaction processing systems relational database design use the discipline of data modeling and generally follow the Codd rules of data normalization in order to ensure absolute data integrity. Using these rules complex information is broken down into its most simple structures (a table) where all of the individual atomic level elements relate to each other and satisfy the normalization rules. 

6\. What's the difference between a primary key and a unique key? 

Both primary key and unique key enforces uniqueness of the column on which they are defined. But by default primary key creates a clustered index on the column, where are unique creates a nonclustered index by default. Another major difference is that, primary key doesn't allow NULLs, but unique key allows one NULL only. 

7\. What is difference between DELETE and TRUNCATE commands? 

Delete command removes the rows from a table based on the condition that we provide with a WHERE clause. Truncate will actually remove all the rows from a table and there will be no data in the table after we run the truncate command. 

  1. **TRUNCATE** : 
     1. TRUNCATE is faster and uses fewer system and transaction log resources than DELETE.
     2. TRUNCATE removes the data by deallocating the data pages used to store the table's data, and only the page deallocations are recorded in the transaction log.
     3. TRUNCATE removes all rows from a table, but the table structure, its columns, constraints, indexes and so on, remains. The counter used by an identity for new rows is reset to the seed for the column.
     4. You cannot use TRUNCATE TABLE on a table referenced by a FOREIGN KEY constraint. Because TRUNCATE TABLE is not logged, it cannot activate a trigger.
     5. TRUNCATE cannot be rolled back.
     6. TRUNCATE is DDL Command.
     7. TRUNCATE Resets identity of the table
  2. **DELETE** : 
     1. DELETE removes rows one at a time and records an entry in the transaction log for each deleted row.
     2. If you want to retain the identity counter, use DELETE instead. If you want to remove table definition and its data, use the DROP TABLE statement.
     3. DELETE Can be used with or without a WHERE clause
     4. DELETE Activates Triggers.
     5. DELETE can be rolled back.
     6. DELETE is DML Command.
     7. DELETE does not reset identity of the table.

Note: DELETE and TRUNCATE both can be rolled back when surrounded by TRANSACTION if the current session is not closed. If TRUNCATE is written in Query Editor surrounded by TRANSACTION and if session is closed, it can not be rolled back but DELETE can be rolled back. 

8\. When is the use of UPDATE_STATISTICS command? 

This command is basically used when a large processing of data has occurred. If a large amount of deletions any modification or Bulk Copy into the tables has occurred, it has to update the indexes to take these changes into account. UPDATE_STATISTICS updates the indexes on these tables accordingly. 

9\. What is the difference between a HAVING CLAUSE and a WHERE CLAUSE? 

They specify a search condition for a group or an aggregate. But the difference is that HAVING can be used only with the SELECT statement. HAVING is typically used in a GROUP BY clause. When GROUP BY is not used, HAVING behaves like a WHERE clause. Having Clause is basically used only with the GROUP BY function in a query whereas WHERE Clause is applied to each row before they are part of the GROUP BY function in a query. 

10\. What are the properties and different Types of Sub-Queries? 

  1. **Properties of Sub-Query**
     1. A sub-query must be enclosed in the parenthesis.
     2. A sub-query must be put in the right hand of the comparison operator, and
     3. A sub-query cannot contain an ORDER-BY clause.
     4. A query can contain more than one sub-query.
  2. **Types of Sub-Query**
     1. Single-row sub-query, where the sub-query returns only one row.
     2. Multiple-row sub-query, where the sub-query returns multiple rows,. and
     3. Multiple column sub-query, where the sub-query returns multiple columns



11\. What is SQL Profiler? 

SQL Profiler is a graphical tool that allows system administrators to monitor events in an instance of Microsoft SQL Server. You can capture and save data about each event to a file or SQL Server table to analyze later. For example, you can monitor a production environment to see which stored procedures are hampering performances by executing too slowly. Use SQL Profiler to monitor only the events in which you are interested. If traces are becoming too large, you can filter them based on the information you want, so that only a subset of the event data is collected. Monitoring too many events adds overhead to the server and the monitoring process and can cause the trace file or trace table to grow very large, especially when the monitoring process takes place over a long period of time. 

12\. What are the authentication modes in SQL Server? How can it be changed? 

Windows mode and Mixed Mode - SQL and Windows. To change authentication mode in SQL Server click Start, Programs, Microsoft SQL Server and click SQL Enterprise Manager to run SQL Enterprise Manager from the Microsoft SQL Server program group. Select the server then from the Tools menu select SQL Server Configuration Properties, and choose the Security page. 

13\. Which command using Query Analyzer will give you the version of SQL server and operating system? 

`SELECT SERVERPROPERTY ('productversion'), SERVERPROPERTY ('productlevel'), SERVERPROPERTY ('edition').`

14\. What is SQL Server Agent? 

SQL Server agent plays an important role in the day-to-day tasks of a database administrator (DBA). It is often overlooked as one of the main tools for SQL Server management. Its purpose is to ease the implementation of tasks for the DBA, with its full- function scheduling engine, which allows you to schedule your own jobs and scripts. 

15\. Can a stored procedure call itself or recursive stored procedure? How much level SP nesting is possible? 

Yes. Because Transact-SQL supports recursion, you can write stored procedures that call themselves. Recursion can be defined as a method of problem solving wherein the solution is arrived at by repetitively applying it to subsets of the problem. A common application of recursive logic is to perform numeric computations that lend themselves to repetitive evaluation by the same processing steps. Stored procedures are nested when one stored procedure calls another or executes managed code by referencing a CLR routine, type, or aggregate. You can nest stored procedures and managed code references up to 32 levels. 

16\. What is Log Shipping? 

Log shipping is the process of automating the backup of database and transaction log files on a production SQL server, and then restoring them onto a standby server. Enterprise Editions only supports log shipping. In log shipping the transactional log file from one server is automatically updated into the backup database on the other server. If one server fails, the other server will have the same db and can be used this as the Disaster Recovery plan. The key feature of log shipping is that it will automatically backup transaction logs throughout the day and automatically restore them on the standby server at defined interval. 

17\. Name 3 ways to get an accurate count of the number of records in a table? 

`SELECT * FROM table1 SELECT COUNT(*) FROM table1 SELECT rows FROM sysindexes WHERE id = OBJECT_ID(table1) AND indid < 2 `

18\. What does it mean to have QUOTED_IDENTIFIER ON? What are the implications of having it OFF? 

When SET QUOTED_IDENTIFIER is ON, identifiers can be delimited by double quotation marks, and literals must be delimited by single quotation marks. When SET QUOTED_IDENTIFIER is OFF, identifiers cannot be quoted and must follow all Transact-SQL rules for identifiers. 

19\. What is the difference between a Local and a Global temporary table? 

  1. **A local temporary** table exists only for the duration of a connection or, if defined inside a compound statement, for the duration of the compound statement.
  2. **A global temporary** table remains in the database permanently, but the rows exist only within a given connection. When connection is closed, the data in the global temporary table disappears. However, the table definition remains with the database for access when database is opened next time.



20\. What is the STUFF function and how does it differ from the REPLACE function? 

STUFF function is used to overwrite existing characters. Using this syntax, STUFF (string_expression, start, length, replacement_characters), string_expression is the string that will have characters substituted, start is the starting position, length is the number of characters in the string that are substituted, and replacement_characters are the new characters interjected into the string. REPLACE function to replace existing characters of all occurrences. Using the syntax REPLACE (string_expression, search_string, replacement_string), where every incidence of search_string found in the string_expression will be replaced with replacement_string. 

21\. What is PRIMARY KEY? 

A PRIMARY KEY constraint is a unique identifier for a row within a database table. Every table should have a primary key constraint to uniquely identify each row and only one primary key constraint can be created for each table. The primary key constraints are used to enforce entity integrity. 

22\. What is UNIQUE KEY constraint? 

A UNIQUE constraint enforces the uniqueness of the values in a set of columns, so no duplicate values are entered. The unique key constraints are used to enforce entity integrity as the primary key constraints. 

23\. What is FOREIGN KEY? 

A FOREIGN KEY constraint prevents any actions that would destroy links between tables with the corresponding data values. A foreign key in one table points to a primary key in another table. Foreign keys prevent actions that would leave rows with foreign key values when there are no primary keys with that value. The foreign key constraints are used to enforce referential integrity. 

24\. What is CHECK Constraint? 

A CHECK constraint is used to limit the values that can be placed in a column. The check constraints are used to enforce domain integrity. 

25\. What is NOT NULL Constraint? 

A NOT NULL constraint enforces that the column will not accept null values. The not null constraints are used to enforce domain integrity, as the check constraints. 

26\. How to get @@ERROR and @@ROWCOUNT at the same time? 

If @@Rowcount is checked after Error checking statement then it will have 0 as the value of @@Recordcount as it would have been reset. And if @@Recordcount is checked before the error-checking statement then @@Error would get reset. To get @@error and @@rowcount at the same time do both in same statement and store them in local variable. `SELECT @RC = @@ROWCOUNT, @ER = @@ERROR `

27\. What is a Scheduled Jobs or What is a Scheduled Tasks? 

Scheduled tasks let user automate processes that run on regular or predictable cycles. User can schedule administrative tasks, such as cube processing, to run during times of slow business activity. User can also determine the order in which tasks run by creating job steps within a SQL Server Agent job. E.g. back up database, Update Stats of Tables. Job steps give user control over flow of execution. If one job fails, user can configure SQL Server Agent to continue to run the remaining tasks or to stop execution. 

28\. What are the advantages of using Stored Procedures? 

  1. Stored procedure can reduced network traffic and latency, boosting application performance.
  2. Stored procedure execution plans can be reused, staying cached in SQL Server's memory, reducing server overhead.
  3. Stored procedures help promote code reuse.
  4. Stored procedures can encapsulate logic. You can change stored procedure code without affecting clients.
  5. Stored procedures provide better security to your data.



29\. What is a table called, if it has neither Cluster nor Non-cluster Index? What is it used for? 

Unindexed table or Heap. Microsoft Press Books and Book on Line (BOL) refers it as Heap. A heap is a table that does not have a clustered index and, therefore, the pages are not linked by pointers. The IAM pages are the only structures that link the pages in a table together. Unindexed tables are good for fast storing of data. Many times it is better to drop all indexes from table and then do bulk of inserts and to restore those indexes after that. 

30\. Can SQL Servers linked to other servers like Oracle? 

SQL Server can be linked to any server provided it has OLE-DB provider from Microsoft to allow a link. E.g. Oracle has an OLE-DB provider for oracle that Microsoft provides to add it as linked server to SQL Server group. 

31\. What is BCP? When does it used? 

BulkCopy is a tool used to copy huge amount of data from tables and views. BCP does not copy the structures same as source to destination. BULK INSERT command helps to import a data file into a database table or view in a user-specified format. 

32\. How to implement one-to-one, one-to-many and many-to-many relationships while designing tables? 

One-to-One relationship can be implemented as a single table and rarely as two tables with primary and foreign key relationships. One-to-Many relationships are implemented by splitting the data into two tables with primary key and foreign key relationships. Many-to-Many relationships are implemented using a junction table with the keys from both the tables forming the composite primary key of the junction table. 

33\. What is an execution plan? When would you use it? How would you view the execution plan? 

An execution plan is basically a road map that graphically or textually shows the data retrieval methods chosen by the SQL Server query optimizer for a stored procedure or ad- hoc query and is a very useful tool for a developer to understand the performance characteristics of a query or stored procedure since the plan is the one that SQL Server will place in its cache and use to execute the stored procedure or query. From within Query Analyzer is an option called "Show Execution Plan" (located on the Query drop-down menu). If this option is turned on it will display query execution plan in separate window when query is ran again. 34\. function and stored proc difference. 

> **Store Procedure Vs. Function in sql server**
> 
> **•** Procedure can return zero or n values whereas function can return one value which is mandatory. **•** Procedures can have input/output parameters for it whereas functions can have only input parameters. **•** Procedure allows select as well as DML statement in it whereas function allows only select statement in it. **•** Functions can be called from procedure whereas procedures cannot be called from function. **•** Exception can be handled by try-catch block in a procedure whereas try-catch block cannot be used in a function. **•** We can go for transaction management in procedure whereas we can't go in function. **•** Procedures can not be utilized in a select statement whereas function can be embedded in a select statement. **•** UDF can be used in the SQL statements anywhere in the WHERE/HAVING/SELECT section where as Stored procedures cannot be. **•** UDFs that return tables can be treated as another rowset. This can be used in JOINs with other tables. **•** Inline UDF's can be though of as views that take parameters and can be used in JOINs and other Rowsetoperations.

>
