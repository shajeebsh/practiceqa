---
title: "DB Qstns NES"
date: 2014-04-16 05:40:22 
author: shajeebhameed
layout: page
slug: db-qstns-nes
status: publish
---

Q 1. | Which of the following statements is TRUE for a primary key?  
---|---  
A primary key can only consist of a single column.  
A primary key is the same as a unique key.  
A primary key can only be an integer.  
None of the above.  
Q 2. | If I want to restrict the range of values in a table, which constraint must I use?  
CHECK constraint.  
PRIMARY KEY constraint.  
FOREIGN KEY constraint.  
DEFAULT constraint.  
Q 3. | I need to create a many-to-many mapping between two tables A and B. How do I do so?  
By creating a foreign key on table A that references a primary key on table B.  
By creating a foreign key on table B that references a primary key on table A.  
By creating a new table called C that has foreign keys to both tables A and B.  
Using a and b.  
Q 4. | Which of the statements is NOT TRUE about an index?  
An index increases the storage requirements of a database.  
Once created, an index cannot be dropped from a table.  
An index can speed up query performance.  
An index can be applied to more than one column in a table.  
Q 5. | What does the following SQL statement do? select * from Person where 1=1;  
Select all records from Person where the primary key is 1.  
Select all records from Person.  
The query has an error and no records are returned.  
Selects a single record randomly from amongst all the records in the table.  
Q 6. | A query "select distinct name from Person" produces 50 records. A different query "select distinct personId, name from Person" produces 55 records. Which of the following statements is definitely TRUE?  
The queries are not working correctly and need to be debugged.  
There are multiple people in the Person table with the same same personId.  
There are multiple people in the Person table with the same name but different person Id.  
There are 5 people in the Person table with the same name but different person Id.  
Q 7. | Which of the following statements is TRUE about the following SQL query? select name, age, dob from person order by age, name desc;  
We cannot predict the order in which the records are returned.  
The records will be returned in descending order of name.  
The records will be returned in ascending order of age. People with same age will be ordered in descending order of name.  
The records will be returned in descending order of age and name.  
Q 8. | Which of the following statements is TRUE for a column XYZ with a NOT NULL constraint?  
I can never insert a record into the table unless I specify a value for the XYZ column during insert.  
I can insert a record into the table without specifying a value for XYZ column during insert, provided that column XYZ also has a DEFAULT constraint.  
I can never create an index on the column XYZ.  
Both a and c are true.  
Q 9. | How can I create a 1-to-many relationship between table A and B, such that many records in B reference a single record in A?  
By creating a foreign key on table A that references a primary key on table B.  
By creating a foreign key on table B that references a primary key on table A.  
By creating a foreign key on table B that references a unique key on table A.  
Either b or c.  
Q 10. | Which of the following statements is FALSE?  
The Primary Key in a table cannot be null.  
The Primary Key in a table must always be numeric.  
There cannot be more than one Primary Key in a table.  
A table does not have to have a Primary Key.  
Q 11. | If a query involving a table join is not performing well, which option will NOT improve performance?  
Adding an index on a column that is not involved in the join and is not part of the WHERE clause.  
Making sure that the query uses indexes and the index statistics are updated.  
Denormalizing the tables.  
Restructuring the query to avoid table scans.  
Q 12. | Which statement about triggers is TRUE?  
You can create a trigger on a table to fire on insert/update/delete operations on a table.  
You can create an INSTEAD OF trigger on a view to fire on an insert/update/delete operation on a view.  
You can create a trigger on a table to just fire on an insert, but not for updates or deletes.  
All of the above is true.  
Q 13. | Here is a CUSTOMERS table with the following data in the table.  | ID | NAME | AGE | CITY | PURCHASE  
---|---|---|---|---  
1 | Jeremy | 32 | Boston | 2000.00  
2 | Ethan | 25 | Paris | 1500.00  
3 | Barbara | 23 | Charlotte | 2000.00  
4 | Brian | 25 | London | 6500.00  
5 | Elliott | 27 | Boston | 8500.00  
6 | Brian | 22 | Moscow | 4500.00  
7 | Helen | 24 | Istanbul | 10000.00  
I want to order the records such that the CITY is in ascending order but the AGE is in descending order. Which SQL statement will do this?  
select * from CUSTOMERS order by CITY, AGE;  
select * from CUSTOMERS order by CITY desc, AGE desc;  
select * from CUSTOMERS order by CITY desc, AGE;  
select * from CUSTOMERS order by CITY, AGE desc;  
Q 14. | The following query will do what? SELECT ID, NAME, AGE, AMOUNT FROM CUSTOMERS INNER JOIN ORDERS ON CUSTOMERS.ID = ORDERS.CUSTOMER_ID;  
Return all records from CUSTOMERS, even for records where CUSTOMERS.ID does not have a corresponding ORDERS.CUSTOMER_ID  
Return all records from ORDERS, even for records where CUSTOMERS.ID does not have a corresponding ORDERS.CUSTOMER_ID  
Return only those records where CUSTOMER.ID has a corresponding ORDERS.CUSTOMER_ID  
Return only those records from CUSTOMERS whose CUSTOMERS.ID is not null.  
Q 15. | A database schema has fact tables (with lots of redundant data), and it has other tables called dimension tables. Which of the following statements is TRUE?  
It is probably being used by an Online Transaction Processing (OLTP) system.  
It is probably being used by a Data Warehouse to do historical analysis of sales for a company.  
It is probably being used by a Data Warehouse to efficiently insert ATM transactions for bank customers.  
None of the above.
