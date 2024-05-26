## Basic SQL Commands with Examples

Structured Query Language (SQL) is the standard language for managing and manipulating relational databases. SQL commands are used to perform various operations on the data stored in a database. This article covers the basic SQL commands with examples to help you get started.

### 1. Creating a Database and Table

#### CREATE DATABASE
The `CREATE DATABASE` command is used to create a new database.
```sql
CREATE DATABASE my_database;
```

#### CREATE TABLE
The `CREATE TABLE` command is used to create a new table in a database.
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    position VARCHAR(50),
    salary DECIMAL(10, 2),
    hire_date DATE
);
```

### 2. Inserting Data

#### INSERT INTO
The `INSERT INTO` command is used to insert new rows into a table.
```sql
INSERT INTO employees (id, name, position, salary, hire_date)
VALUES (1, 'John Doe', 'Software Engineer', 70000.00, '2021-01-15');
```
You can also insert multiple rows at once:
```sql
INSERT INTO employees (id, name, position, salary, hire_date)
VALUES 
(2, 'Jane Smith', 'Project Manager', 90000.00, '2020-03-22'),
(3, 'Emily Johnson', 'Data Analyst', 65000.00, '2019-07-30');
```

### 3. Querying Data

#### SELECT
The `SELECT` command is used to retrieve data from a database.
```sql
SELECT * FROM employees;
```
To select specific columns:
```sql
SELECT name, position FROM employees;
```
With conditions:
```sql
SELECT * FROM employees WHERE salary > 65000;
```
To order the results:
```sql
SELECT * FROM employees ORDER BY hire_date DESC;
```

### 4. Updating Data

#### UPDATE
The `UPDATE` command is used to modify existing data in a table.
```sql
UPDATE employees
SET salary = 75000.00
WHERE id = 1;
```

### 5. Deleting Data

#### DELETE
The `DELETE` command is used to remove rows from a table.
```sql
DELETE FROM employees
WHERE id = 3;
```

### 6. Filtering Data

#### WHERE
The `WHERE` clause is used to filter records.
```sql
SELECT * FROM employees
WHERE position = 'Data Analyst';
```

#### LIKE
The `LIKE` operator is used in a `WHERE` clause to search for a specified pattern.
```sql
SELECT * FROM employees
WHERE name LIKE 'J%';
```

#### BETWEEN
The `BETWEEN` operator selects values within a given range.
```sql
SELECT * FROM employees
WHERE hire_date BETWEEN '2020-01-01' AND '2021-12-31';
```

### 7. Aggregating Data

#### COUNT, SUM, AVG, MIN, MAX
Aggregate functions are used to perform calculations on multiple values.
```sql
SELECT COUNT(*) FROM employees;

SELECT AVG(salary) FROM employees;

SELECT SUM(salary) FROM employees;

SELECT MIN(salary) FROM employees;

SELECT MAX(salary) FROM employees;
```

### 8. Grouping Data

#### GROUP BY
The `GROUP BY` statement groups rows that have the same values into summary rows.
```sql
SELECT position, AVG(salary) as avg_salary
FROM employees
GROUP BY position;
```

### 9. Joining Tables

#### INNER JOIN
The `INNER JOIN` clause is used to combine rows from two or more tables based on a related column.
```sql
SELECT employees.name, departments.department_name
FROM employees
INNER JOIN departments ON employees.department_id = departments.id;
```

### 10. Altering Tables

#### ALTER TABLE
The `ALTER TABLE` command is used to add, delete, or modify columns in an existing table.
```sql
ALTER TABLE employees
ADD email VARCHAR(100);

ALTER TABLE employees
DROP COLUMN email;

ALTER TABLE employees
MODIFY COLUMN salary DECIMAL(15, 2);
```

### 11. Dropping Tables and Databases

#### DROP TABLE
The `DROP TABLE` command is used to delete a table.
```sql
DROP TABLE employees;
```

#### DROP DATABASE
The `DROP DATABASE` command is used to delete a database.
```sql
DROP DATABASE my_database;
```

### Conclusion

These basic SQL commands are fundamental to managing and manipulating relational databases. By mastering these commands, you can perform essential operations such as creating and modifying databases and tables, inserting and querying data, and managing database structures. Whether you are a beginner or looking to refresh your knowledge, understanding these SQL basics is crucial for effective database management.