### Before we  Look into Database Mangement system, we must first Understand what Database is, as many a times use both Interchangeably.
## What is Database?
A database is an organized collection of structured data that is stored and accessed electronically. Databases are essential in modern applications because they help manage,
store, and retrieve data efficiently. Databases are used in various systems, from simple software applications to large enterprise systems, 
to handle tasks such as tracking transactions, storing user data, managing content, and much more.  For Example, a university database organizes the data about students, faculty, admin staff, etc. which helps in the efficient retrieval, insertion, 
and deletion of data from it.
## Now, What is a Database System ?
A Database Management System (DBMS) is  _software tool_  that enables users to interact with a database, allowing them to create, read, update, and delete (CRUD) data while ensuring the data remains consistent,
secure, and accessible. DBMS provides an environment to store and retrieve data in convenient and efficient manner.

## Key Features of DBMS
1. Data Modeling: Defines the structure and relationships of data within a database.
2. Data Storage & Retrieval: Manages data storage and offers methods to query and search the database.
3. Concurrency Control: Ensures multiple users can access data simultaneously without conflicts.
4. Data Integrity & Security: Enforces data constraints and access controls to maintain data accuracy and secure access.
Backup & Recovery: Provides tools to back up data and recover it in case of system failures.

## Types of DBMS
### Relational Database Management System (RDBMS): 
- Data is organized into tables (relations) with rows and columns, 
- Organizes data in tables (rows and columns).
- Relationships between data are maintained using primary and foreign keys.
- Suitable for structured data and complex queries (e.g., MySQL, PostgreSQL, Oracle, Microsoft SQL Server).
### Use Cases:
1. Applications requiring structured data with relationships between entities (e.g., user data, transactions).
2. Systems with strict consistency and ACID (Atomicity, Consistency, Isolation, Durability) compliance, such as financial applications.
3. Data integrity is a priority due to referential integrity constraints.
4. OLTP (Online Transaction Processing) systems like banking, e-commerce, and CRM systems.

### NoSQL DBMS:
- Designed for high-performance scenarios and large-scale data, 
- Data can be organized as key-value pairs, documents, graphs, or column-based stores.
- Designed for handling large-scale data and high-performance needs.
- More flexible, used in cases where relational models are less efficient (e.g., MongoDB, Cassandra).
### Use Cases:
1. Caching, session management, and user preference storage.
2. Systems requiring high-speed lookups with low complexity.


## Key Differences Between Relational DBMS and NoSQL DBMS
|Aspect|Relational DBMS (RDBMS) |	NoSQL DBMS| 
|------|------------------------|-----------|
|Data Structure|	Tables (rows and columns, fixed schema)	Varies| Key-Value, Document, Column-Family, Graph
|Schema|	Fixed schema, predefined relationships	|Dynamic schema, flexible structure
|Query Language	|SQL (Structured Query Language)	|Varies by type (No universal query language)
|Data Consistency	|Strong consistency, ACID compliance	|Eventual consistency (CAP theorem), flexible ACID
|Scalability	|Vertical scaling (adding more power to the existing server)	|Horizontal scaling (adding more servers)
|Use Case Suitability	|Structured, transactional data with complex relationships	|Unstructured, semi-structured, large-scale data
|Examples	|MySQL, PostgreSQL, Oracle, SQL Server|	MongoDB, Cassandra, DynamoDB, Neo4j
### Object-Oriented DBMS (OODBMS):
Stores data as objects, similar to those used in object-oriented programming, allowing for complex data representations and relationships
