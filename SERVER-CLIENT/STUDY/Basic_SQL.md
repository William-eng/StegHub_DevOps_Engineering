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

      SELECT * FROM users WHERE username = 'john_doe'; # Select with a condition
      SELECT * FROM users LIMIT 5;  # Select with a limit

- INSERT: Add new records into a table.

        INSERT INTO users (username, email) VALUES 
         ('alice', 'alice@example.com'),
        ('bob', 'bob@example.com');
  

