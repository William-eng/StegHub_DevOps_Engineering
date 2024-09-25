- SHOW: Display information about databases, tables, and columns.

      
      SHOW COLUMNS FROM table_name;
      SHOW DATABASES;


- CREATE: Create new databases or tables.


      CREATE TABLE users (
      id INT AUTO_INCREMENT PRIMARY KEY,
      username VARCHAR(50) NOT NULL,
      email VARCHAR(100),
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );

- DROP: Delete databases or tables (destructive).

      ALTER TABLE users DROP COLUMN email;

- SELECT: Query data from a table.

      SELECT * FROM users WHERE username = 'john_doe';
      SELECT * FROM users LIMIT 5;

- INSERT: Add new records into a table.
