# What is SQL Syntax?
---
SQL syntax is a unique set of rules and guidelines to be followed while writing SQL statements. 
All the SQL statements start with any of the keywords like SELECT, INSERT, UPDATE, DELETE, ALTER, DROP, CREATE and all the statements end with a semicolon (;).
## Case Sensitivity
---
The most important point to be noted here is that SQL is case insensitive, which means SELECT and Select have same meaning in SQL statements. Whereas, MySQL makes difference in table names.
So, if you are working with MySQL, then you need to give table names as they exist in the database.

## SQL Table
---
Let us consider a table with the name CUSTOMERS shown below, and use it as a reference to demonstrate all the SQL Statements on the same.
|ID|	NAME|	AGE|	ADDRESS|	SALARY|
|--|------|----|---------|--------|
1|	Ramesh	|32	|Ahmedabad	|2000.00
2	|Khilan	|25	|Delhi	|1500.00
3	|kaushik|	23	|Kota	|2000.00
4	|Chaitali	|25	|Mumbai	|6500.00

## SQL Statements
---
All the SQL statements require a semicolon (;) at the end of each statement. 
Semicolon is the standard way to separate different SQL statements which allows to include multiple SQL statements in a single line.

## SQL CREATE DATABASE Statement
---
To store data within a database, you first need to create it. This is necessary to individualize the data belonging to an organization.

You can create a database using the following syntax −

                CREATE DATABASE database_name;
## SQL USE Statement
---
Once the database is created, it needs to be used in order to start storing the data accordingly. 
Following is the syntax to change the current location to required database −       


                            USE database_name;
## SQL DROP DATABASE Statement
---
If a database is no longer necessary, you can also delete it. To delete/drop a database, use the following syntax −     

                    DROP DATABASE database_name;

## SQL CREATE TABLE Statement
---
In an SQL driven database, the data is stored in a structured manner, i.e. in the form of tables. To create a table, following syntax is used −

                  
                  CREATE TABLE table_name(
                     column1 datatype,
                     column2 datatype,
                     column3 datatype,
                     .....
                     columnN datatype,
                     PRIMARY KEY( one or more columns )
                  );
  ## SQL SELECT Statement
  ---
In order to retrieve the result-sets of the stored data from a database table, we use the SELECT statement. Following is the syntax −


                SELECT column1, column2....columnN FROM table_name;
To retrieve the data from CUSTOMERS table, we use the SELECT statement as shown below.

                SELECT * FROM CUSTOMERS;


## SQL UPDATE Statement
---
When the stored data in a database table is outdated and needs to be updated without having to delete the table, we use the UPDATE statement.
Following is the syntax −

          UPDATE table_name
          SET column1 = value1, column2 = value2....columnN=valueN

## SQL ALTER TABLE Statement (Rename)
The ALTER TABLE statement is also used to change the name of a table as well. Use the syntax below −


          ALTER TABLE table_name RENAME TO new_table_name;
          

## SQL DELETE Statement
Without deleting the entire table from the database, you can also delete a certain part of the data by applying conditions.
This is done using the DELETE FROM statement. Following is the syntax −

## DELETE FROM table_name WHERE  {CONDITION};
The following code has a query, which will DELETE a customer, whose ID is 6.

 
          DELETE FROM CUSTOMERS WHERE ID = 6;

## SQL DROP TABLE Statement
To delete a table entirely from a database when it is no longer needed, following syntax is used −

          DROP TABLE table_name;
This query will drop the CUSTOMERS table from the database.                                    
