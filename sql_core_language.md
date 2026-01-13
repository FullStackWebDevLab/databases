# Comprehensive SQL Language Reference

## Table of Contents

1. [Introduction to SQL](#introduction-to-sql)
2. [SQL Syntax Fundamentals](#sql-syntax-fundamentals)
3. [Data Types](#data-types)
4. [Database Operations](#database-operations)
5. [Table Operations](#table-operations)
6. [Data Manipulation](#data-manipulation)
7. [Querying Data](#querying-data)
8. [Filtering and Sorting](#filtering-and-sorting)
9. [Joins](#joins)
10. [Aggregate Functions](#aggregate-functions)
11. [Grouping and Having](#grouping-and-having)
12. [Subqueries](#subqueries)
13. [Constraints](#constraints)
14. [Indexes](#indexes)
15. [Views](#views)
16. [Transactions](#transactions)

---

## Introduction to SQL

### What is SQL?

SQL (Structured Query Language) is the standard language for managing and manipulating data in relational database management systems. SQL became a standard of the American National Standards Institute (ANSI) in 1986, and of the International Organization for Standardization (ISO) in 1987.

### Purpose

SQL enables you to perform the following operations:
- Create and modify database structures
- Insert, update, and delete data
- Retrieve and query data
- Control access to data
- Manage transactions

### Common Database Systems

SQL is supported by major database systems including:
- MySQL
- PostgreSQL
- Microsoft SQL Server
- Oracle Database
- SQLite
- IBM DB2

---

## SQL Syntax Fundamentals

### Basic Rules

1. **Case Sensitivity**: SQL keywords are not case-sensitive, but it is conventional to write them in UPPERCASE for readability.
2. **Semicolons**: SQL statements end with a semicolon (;).
3. **Whitespace**: SQL treats multiple spaces, tabs, and line breaks as a single space.
4. **Comments**: Use `--` for single-line comments or `/* */` for multi-line comments.

### Statement Structure

```sql
-- Single-line comment
SELECT column_name
FROM table_name
WHERE condition;

/* Multi-line comment
   spanning multiple lines */
```

---

## Data Types

### Numeric Types

| Type | Description | Example |
|------|-------------|---------|
| INTEGER or INT | Whole numbers | 42, -17, 0 |
| SMALLINT | Small whole numbers | 100 |
| BIGINT | Large whole numbers | 9223372036854775807 |
| DECIMAL(p,s) | Fixed-point numbers | DECIMAL(10,2) stores 12345678.90 |
| NUMERIC(p,s) | Exact numeric | NUMERIC(8,2) |
| FLOAT | Floating-point numbers | 3.14159 |
| REAL | Single precision floating-point | 2.71828 |
| DOUBLE PRECISION | Double precision floating-point | 1.414213562 |

### String Types

| Type | Description | Example |
|------|-------------|---------|
| CHAR(n) | Fixed-length string | CHAR(10) |
| VARCHAR(n) | Variable-length string | VARCHAR(255) |
| TEXT | Variable-length text | Long text content |

### Date and Time Types

| Type | Description | Format |
|------|-------------|--------|
| DATE | Date value | YYYY-MM-DD |
| TIME | Time value | HH:MM:SS |
| DATETIME | Date and time | YYYY-MM-DD HH:MM:SS |
| TIMESTAMP | Timestamp | YYYY-MM-DD HH:MM:SS |
| YEAR | Year value | YYYY |

### Other Types

| Type | Description |
|------|-------------|
| BOOLEAN | True or false values |
| BLOB | Binary large object |
| NULL | Represents missing or unknown value |

---

## Database Operations

### Create Database

Creates a new database.

**Syntax:**
```sql
CREATE DATABASE database_name;
```

**Example:**
```sql
CREATE DATABASE company;
```

### Drop Database

Deletes an existing database and all its contents.

**Syntax:**
```sql
DROP DATABASE database_name;
```

**Example:**
```sql
DROP DATABASE company;
```

**Warning:** This operation permanently deletes all data in the database.

### Select Database

Selects a database to work with (syntax varies by database system).

**MySQL Example:**
```sql
USE company;
```

---

## Table Operations

### Create Table

Creates a new table with specified columns and data types.

**Syntax:**
```sql
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    column3 datatype constraints,
    ...
);
```

**Example:**
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    hire_date DATE,
    salary DECIMAL(10, 2),
    department_id INT
);
```

### Alter Table

Modifies the structure of an existing table.

#### Add Column

**Syntax:**
```sql
ALTER TABLE table_name
ADD column_name datatype;
```

**Example:**
```sql
ALTER TABLE employees
ADD phone_number VARCHAR(15);
```

#### Drop Column

**Syntax:**
```sql
ALTER TABLE table_name
DROP COLUMN column_name;
```

**Example:**
```sql
ALTER TABLE employees
DROP COLUMN phone_number;
```

#### Modify Column

**Syntax:**
```sql
ALTER TABLE table_name
ALTER COLUMN column_name new_datatype;
```

**Example:**
```sql
ALTER TABLE employees
ALTER COLUMN salary DECIMAL(12, 2);
```

#### Rename Column

**Syntax (varies by database):**
```sql
-- MySQL/PostgreSQL
ALTER TABLE table_name
RENAME COLUMN old_name TO new_name;

-- SQL Server
EXEC sp_rename 'table_name.old_name', 'new_name', 'COLUMN';
```

### Drop Table

Deletes a table and all its data.

**Syntax:**
```sql
DROP TABLE table_name;
```

**Example:**
```sql
DROP TABLE employees;
```

### Truncate Table

Removes all rows from a table but keeps the table structure.

**Syntax:**
```sql
TRUNCATE TABLE table_name;
```

**Example:**
```sql
TRUNCATE TABLE employees;
```

---

## Data Manipulation

### INSERT Statement

Adds new rows to a table.

#### Insert Single Row

**Syntax:**
```sql
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);
```

**Example:**
```sql
INSERT INTO employees (employee_id, first_name, last_name, email, hire_date, salary)
VALUES (1, 'John', 'Doe', 'john.doe@company.com', '2024-01-15', 75000.00);
```

#### Insert Multiple Rows

**Syntax:**
```sql
INSERT INTO table_name (column1, column2, ...)
VALUES 
    (value1, value2, ...),
    (value1, value2, ...),
    (value1, value2, ...);
```

**Example:**
```sql
INSERT INTO employees (employee_id, first_name, last_name, salary)
VALUES 
    (2, 'Jane', 'Smith', 80000.00),
    (3, 'Bob', 'Johnson', 65000.00),
    (4, 'Alice', 'Williams', 90000.00);
```

#### Insert Without Column Names

If inserting values for all columns in order:

**Syntax:**
```sql
INSERT INTO table_name
VALUES (value1, value2, value3, ...);
```

### UPDATE Statement

Modifies existing data in a table.

**Syntax:**
```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

**Example:**
```sql
UPDATE employees
SET salary = 82000.00, department_id = 5
WHERE employee_id = 2;
```

**Important:** Always include a WHERE clause to avoid updating all rows.

**Update All Rows Example:**
```sql
UPDATE employees
SET salary = salary * 1.05;  -- Give everyone a 5% raise
```

### DELETE Statement

Removes rows from a table.

**Syntax:**
```sql
DELETE FROM table_name
WHERE condition;
```

**Example:**
```sql
DELETE FROM employees
WHERE employee_id = 4;
```

**Delete All Rows:**
```sql
DELETE FROM employees;
```

**Warning:** Without a WHERE clause, all rows will be deleted.

---

## Querying Data

### SELECT Statement

Retrieves data from one or more tables.

#### Select All Columns

**Syntax:**
```sql
SELECT * FROM table_name;
```

**Example:**
```sql
SELECT * FROM employees;
```

#### Select Specific Columns

**Syntax:**
```sql
SELECT column1, column2, column3
FROM table_name;
```

**Example:**
```sql
SELECT first_name, last_name, salary
FROM employees;
```

#### Select with Aliases

**Syntax:**
```sql
SELECT column_name AS alias_name
FROM table_name;
```

**Example:**
```sql
SELECT 
    first_name AS "First Name",
    last_name AS "Last Name",
    salary * 12 AS "Annual Salary"
FROM employees;
```

#### Select Distinct

Removes duplicate rows from results.

**Syntax:**
```sql
SELECT DISTINCT column1, column2
FROM table_name;
```

**Example:**
```sql
SELECT DISTINCT department_id
FROM employees;
```

---

## Filtering and Sorting

### WHERE Clause

Filters rows based on conditions.

**Syntax:**
```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

**Example:**
```sql
SELECT first_name, last_name, salary
FROM employees
WHERE salary > 70000;
```

### Comparison Operators

| Operator | Description | Example |
|----------|-------------|---------|
| = | Equal to | WHERE age = 30 |
| <> or != | Not equal to | WHERE age <> 30 |
| > | Greater than | WHERE salary > 50000 |
| < | Less than | WHERE age < 40 |
| >= | Greater than or equal | WHERE salary >= 60000 |
| <= | Less than or equal | WHERE age <= 35 |

### Logical Operators

#### AND Operator

**Syntax:**
```sql
SELECT column1, column2
FROM table_name
WHERE condition1 AND condition2;
```

**Example:**
```sql
SELECT first_name, last_name
FROM employees
WHERE salary > 60000 AND department_id = 3;
```

#### OR Operator

**Syntax:**
```sql
SELECT column1, column2
FROM table_name
WHERE condition1 OR condition2;
```

**Example:**
```sql
SELECT first_name, last_name
FROM employees
WHERE department_id = 2 OR department_id = 5;
```

#### NOT Operator

**Syntax:**
```sql
SELECT column1, column2
FROM table_name
WHERE NOT condition;
```

**Example:**
```sql
SELECT first_name, last_name
FROM employees
WHERE NOT department_id = 3;
```

### IN Operator

Tests whether a value matches any value in a list.

**Syntax:**
```sql
SELECT column1, column2
FROM table_name
WHERE column_name IN (value1, value2, value3);
```

**Example:**
```sql
SELECT first_name, last_name
FROM employees
WHERE department_id IN (1, 3, 5);
```

### BETWEEN Operator

Tests whether a value falls within a range.

**Syntax:**
```sql
SELECT column1, column2
FROM table_name
WHERE column_name BETWEEN value1 AND value2;
```

**Example:**
```sql
SELECT first_name, last_name, salary
FROM employees
WHERE salary BETWEEN 60000 AND 80000;
```

### LIKE Operator

Pattern matching in strings.

**Wildcards:**
- `%` - Matches any sequence of characters
- `_` - Matches any single character

**Syntax:**
```sql
SELECT column1, column2
FROM table_name
WHERE column_name LIKE pattern;
```

**Examples:**
```sql
-- Names starting with 'J'
SELECT first_name, last_name
FROM employees
WHERE first_name LIKE 'J%';

-- Names ending with 'son'
SELECT first_name, last_name
FROM employees
WHERE last_name LIKE '%son';

-- Names containing 'oh'
SELECT first_name, last_name
FROM employees
WHERE first_name LIKE '%oh%';

-- Three-letter names
SELECT first_name
FROM employees
WHERE first_name LIKE '___';
```

### IS NULL / IS NOT NULL

Tests for NULL values.

**Syntax:**
```sql
SELECT column1, column2
FROM table_name
WHERE column_name IS NULL;

SELECT column1, column2
FROM table_name
WHERE column_name IS NOT NULL;
```

**Example:**
```sql
SELECT first_name, last_name
FROM employees
WHERE department_id IS NULL;
```

### ORDER BY Clause

Sorts the result set.

**Syntax:**
```sql
SELECT column1, column2
FROM table_name
ORDER BY column1 [ASC|DESC], column2 [ASC|DESC];
```

**Examples:**
```sql
-- Ascending order (default)
SELECT first_name, last_name, salary
FROM employees
ORDER BY salary;

-- Descending order
SELECT first_name, last_name, salary
FROM employees
ORDER BY salary DESC;

-- Multiple columns
SELECT first_name, last_name, department_id, salary
FROM employees
ORDER BY department_id ASC, salary DESC;
```

### LIMIT Clause

Restricts the number of rows returned.

**Syntax (varies by database):**
```sql
-- MySQL, PostgreSQL, SQLite
SELECT column1, column2
FROM table_name
LIMIT number;

-- SQL Server
SELECT TOP number column1, column2
FROM table_name;
```

**Example:**
```sql
-- Get top 5 highest paid employees
SELECT first_name, last_name, salary
FROM employees
ORDER BY salary DESC
LIMIT 5;
```

---

## Joins

Joins combine rows from two or more tables based on related columns.

### INNER JOIN

Returns only rows when the join condition is satisfied in both tables.

**Syntax:**
```sql
SELECT table1.column1, table2.column2
FROM table1
INNER JOIN table2
ON table1.common_column = table2.common_column;
```

**Example:**
```sql
SELECT employees.first_name, employees.last_name, departments.department_name
FROM employees
INNER JOIN departments
ON employees.department_id = departments.department_id;
```

### LEFT JOIN (LEFT OUTER JOIN)

Returns all rows from the left table, along with matching rows from the right table. If there is no match, NULL values are returned for columns from the right table.

**Syntax:**
```sql
SELECT table1.column1, table2.column2
FROM table1
LEFT JOIN table2
ON table1.common_column = table2.common_column;
```

**Example:**
```sql
SELECT employees.first_name, employees.last_name, departments.department_name
FROM employees
LEFT JOIN departments
ON employees.department_id = departments.department_id;
```

### RIGHT JOIN (RIGHT OUTER JOIN)

Returns all rows from the right table and matching rows from the left table. If there is no match, NULL values are returned for columns from the left table.

**Syntax:**
```sql
SELECT table1.column1, table2.column2
FROM table1
RIGHT JOIN table2
ON table1.common_column = table2.common_column;
```

**Example:**
```sql
SELECT employees.first_name, employees.last_name, departments.department_name
FROM employees
RIGHT JOIN departments
ON employees.department_id = departments.department_id;
```

### FULL JOIN (FULL OUTER JOIN)

Creates the result-set by combining results of both LEFT JOIN and RIGHT JOIN. The result-set will contain all the rows from both tables. For the rows for which there is no matching, the result-set will contain NULL values.

**Syntax:**
```sql
SELECT table1.column1, table2.column2
FROM table1
FULL JOIN table2
ON table1.common_column = table2.common_column;
```

**Example:**
```sql
SELECT employees.first_name, employees.last_name, departments.department_name
FROM employees
FULL JOIN departments
ON employees.department_id = departments.department_id;
```

### CROSS JOIN

Returns the Cartesian product of both tables (all possible combinations).

**Syntax:**
```sql
SELECT table1.column1, table2.column2
FROM table1
CROSS JOIN table2;
```

**Example:**
```sql
SELECT employees.first_name, projects.project_name
FROM employees
CROSS JOIN projects;
```

### SELF JOIN

Joins a table to itself.

**Syntax:**
```sql
SELECT a.column1, b.column2
FROM table_name a
JOIN table_name b
ON a.common_column = b.common_column;
```

**Example:**
```sql
-- Find employees and their managers
SELECT e.first_name AS employee_name, m.first_name AS manager_name
FROM employees e
JOIN employees m
ON e.manager_id = m.employee_id;
```

---

## Aggregate Functions

Aggregate functions perform calculations on multiple rows and return a single value.

### COUNT()

Counts the number of rows.

**Syntax:**
```sql
SELECT COUNT(column_name)
FROM table_name
WHERE condition;
```

**Examples:**
```sql
-- Count all employees
SELECT COUNT(*) FROM employees;

-- Count employees in department 3
SELECT COUNT(*) FROM employees WHERE department_id = 3;

-- Count non-null emails
SELECT COUNT(email) FROM employees;

-- Count distinct departments
SELECT COUNT(DISTINCT department_id) FROM employees;
```

### SUM()

Calculates the sum of a numeric column.

**Syntax:**
```sql
SELECT SUM(column_name)
FROM table_name
WHERE condition;
```

**Example:**
```sql
SELECT SUM(salary) AS total_payroll
FROM employees;
```

### AVG()

Calculates the average of a numeric column.

**Syntax:**
```sql
SELECT AVG(column_name)
FROM table_name
WHERE condition;
```

**Example:**
```sql
SELECT AVG(salary) AS average_salary
FROM employees
WHERE department_id = 2;
```

### MAX()

Returns the maximum value.

**Syntax:**
```sql
SELECT MAX(column_name)
FROM table_name
WHERE condition;
```

**Example:**
```sql
SELECT MAX(salary) AS highest_salary
FROM employees;
```

### MIN()

Returns the minimum value.

**Syntax:**
```sql
SELECT MIN(column_name)
FROM table_name
WHERE condition;
```

**Example:**
```sql
SELECT MIN(hire_date) AS earliest_hire
FROM employees;
```

---

## Grouping and Having

### GROUP BY Clause

Groups rows that have the same values in specified columns.

**Syntax:**
```sql
SELECT column1, aggregate_function(column2)
FROM table_name
WHERE condition
GROUP BY column1;
```

**Example:**
```sql
SELECT department_id, COUNT(*) AS employee_count, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id;
```

### HAVING Clause

Filters groups based on aggregate conditions. Used with GROUP BY.

**Syntax:**
```sql
SELECT column1, aggregate_function(column2)
FROM table_name
WHERE condition
GROUP BY column1
HAVING aggregate_condition;
```

**Example:**
```sql
SELECT department_id, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 70000;
```

**Difference Between WHERE and HAVING:**
- WHERE filters rows before grouping
- HAVING filters groups after aggregation

**Example:**
```sql
SELECT department_id, COUNT(*) AS count, AVG(salary) AS avg_salary
FROM employees
WHERE hire_date > '2020-01-01'
GROUP BY department_id
HAVING COUNT(*) >= 5;
```

---

## Subqueries

A subquery is a query nested inside another query.

### Subquery in WHERE Clause

**Syntax:**
```sql
SELECT column1, column2
FROM table_name
WHERE column_name operator (SELECT column_name FROM table_name WHERE condition);
```

**Example:**
```sql
-- Employees earning more than average
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### Subquery with IN

**Example:**
```sql
-- Employees in departments located in New York
SELECT first_name, last_name
FROM employees
WHERE department_id IN (
    SELECT department_id 
    FROM departments 
    WHERE location = 'New York'
);
```

### Subquery in FROM Clause

**Example:**
```sql
SELECT dept_info.department_id, dept_info.avg_salary
FROM (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) AS dept_info
WHERE dept_info.avg_salary > 75000;
```

### Correlated Subquery

A subquery that references columns from the outer query.

**Example:**
```sql
SELECT e1.first_name, e1.last_name, e1.salary
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
);
```

### EXISTS Operator

Tests for the existence of rows in a subquery.

**Example:**
```sql
SELECT first_name, last_name
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.employee_id = e.employee_id
);
```

---

## Constraints

Constraints enforce rules on data in tables.

### PRIMARY KEY

Uniquely identifies each row in a table. A table can have only one primary key.

**Syntax:**
```sql
CREATE TABLE table_name (
    column1 datatype PRIMARY KEY,
    column2 datatype,
    ...
);

-- Or as table constraint
CREATE TABLE table_name (
    column1 datatype,
    column2 datatype,
    PRIMARY KEY (column1)
);
```

**Example:**
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50)
);
```

### FOREIGN KEY

Links two tables together, enforcing referential integrity.

**Syntax:**
```sql
CREATE TABLE table_name (
    column1 datatype,
    column2 datatype,
    FOREIGN KEY (column1) REFERENCES other_table(column)
);
```

**Example:**
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    order_date DATE,
    employee_id INT,
    FOREIGN KEY (employee_id) REFERENCES employees(employee_id)
);
```

### UNIQUE

Ensures all values in a column are different.

**Syntax:**
```sql
CREATE TABLE table_name (
    column1 datatype UNIQUE,
    column2 datatype,
    ...
);
```

**Example:**
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    ssn VARCHAR(11) UNIQUE
);
```

### NOT NULL

Ensures a column cannot have NULL values.

**Syntax:**
```sql
CREATE TABLE table_name (
    column1 datatype NOT NULL,
    column2 datatype NOT NULL,
    ...
);
```

**Example:**
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100)
);
```

### CHECK

Ensures values in a column satisfy a specific condition.

**Syntax:**
```sql
CREATE TABLE table_name (
    column1 datatype CHECK (condition),
    column2 datatype,
    ...
);
```

**Example:**
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    age INT CHECK (age >= 18),
    salary DECIMAL(10, 2) CHECK (salary > 0)
);
```

### DEFAULT

Sets a default value for a column when no value is specified.

**Syntax:**
```sql
CREATE TABLE table_name (
    column1 datatype DEFAULT default_value,
    column2 datatype,
    ...
);
```

**Example:**
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    hire_date DATE DEFAULT CURRENT_DATE,
    status VARCHAR(20) DEFAULT 'active'
);
```

### Adding Constraints to Existing Tables

**Syntax:**
```sql
ALTER TABLE table_name
ADD CONSTRAINT constraint_name constraint_type (column_name);
```

**Example:**
```sql
ALTER TABLE employees
ADD CONSTRAINT unique_email UNIQUE (email);

ALTER TABLE orders
ADD CONSTRAINT fk_employee 
FOREIGN KEY (employee_id) REFERENCES employees(employee_id);
```

---

## Indexes

Indexes improve query performance by creating fast lookup structures.

### Create Index

**Syntax:**
```sql
CREATE INDEX index_name
ON table_name (column1, column2, ...);
```

**Example:**
```sql
CREATE INDEX idx_last_name
ON employees (last_name);

-- Multi-column index
CREATE INDEX idx_name
ON employees (last_name, first_name);
```

### Create Unique Index

**Syntax:**
```sql
CREATE UNIQUE INDEX index_name
ON table_name (column_name);
```

**Example:**
```sql
CREATE UNIQUE INDEX idx_email
ON employees (email);
```

### Drop Index

**Syntax (varies by database):**
```sql
-- MySQL
DROP INDEX index_name ON table_name;

-- SQL Server
DROP INDEX table_name.index_name;

-- PostgreSQL
DROP INDEX index_name;
```

**Example:**
```sql
DROP INDEX idx_last_name ON employees;
```

---

## Views

A view is a virtual table based on a SQL query.

### Create View

**Syntax:**
```sql
CREATE VIEW view_name AS
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

**Example:**
```sql
CREATE VIEW high_earners AS
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE salary > 80000;
```

### Using Views

**Example:**
```sql
SELECT * FROM high_earners;

SELECT first_name, last_name
FROM high_earners
WHERE salary > 90000;
```

### Update View

**Syntax:**
```sql
CREATE OR REPLACE VIEW view_name AS
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

**Example:**
```sql
CREATE OR REPLACE VIEW high_earners AS
SELECT employee_id, first_name, last_name, salary, department_id
FROM employees
WHERE salary > 75000;
```

### Drop View

**Syntax:**
```sql
DROP VIEW view_name;
```

**Example:**
```sql
DROP VIEW high_earners;
```

---

## Transactions

Transactions ensure data integrity by grouping multiple operations into a single unit of work.

### Transaction Commands

**BEGIN TRANSACTION** - Starts a new transaction
**COMMIT** - Saves all changes made in the transaction
**ROLLBACK** - Undoes all changes made in the transaction

### Using Transactions

**Syntax:**
```sql
BEGIN TRANSACTION;

-- SQL statements
UPDATE ...
INSERT ...
DELETE ...

COMMIT;  -- or ROLLBACK;
```

**Example:**
```sql
BEGIN TRANSACTION;

UPDATE accounts
SET balance = balance - 500
WHERE account_id = 1;

UPDATE accounts
SET balance = balance + 500
WHERE account_id = 2;

COMMIT;
```

### Rollback Example

**Example:**
```sql
BEGIN TRANSACTION;

DELETE FROM employees
WHERE department_id = 5;

-- Oops, wrong department!
ROLLBACK;
```

### ACID Properties

Transactions follow ACID properties:
- **Atomicity**: All operations succeed or all fail
- **Consistency**: Database remains in a valid state
- **Isolation**: Transactions do not interfere with each other
- **Durability**: Committed changes are permanent

---

## Additional SQL Features

### UNION

Combines result sets from multiple SELECT statements.

**Syntax:**
```sql
SELECT column1, column2 FROM table1
UNION
SELECT column1, column2 FROM table2;
```

**Example:**
```sql
SELECT first_name, last_name FROM employees
UNION
SELECT first_name, last_name FROM contractors;
```

### UNION ALL

Like UNION but includes duplicate rows.

**Example:**
```sql
SELECT first_name FROM employees
UNION ALL
SELECT first_name FROM contractors;
```

### CASE Expression

Provides conditional logic in queries.

**Syntax:**
```sql
SELECT column1,
    CASE
        WHEN condition1 THEN result1
        WHEN condition2 THEN result2
        ELSE result3
    END AS alias_name
FROM table_name;
```

**Example:**
```sql
SELECT first_name, last_name, salary,
    CASE
        WHEN salary < 50000 THEN 'Low'
        WHEN salary BETWEEN 50000 AND 80000 THEN 'Medium'
        ELSE 'High'
    END AS salary_level
FROM employees;
```

### COALESCE

Returns the first non-NULL value in a list.

**Syntax:**
```sql
SELECT COALESCE(column1, column2, default_value) AS alias_name
FROM table_name;
```

**Example:**
```sql
SELECT first_name, last_name,
    COALESCE(phone_number, email, 'No contact info') AS contact
FROM employees;
```

---

## SQL Best Practices

### Query Optimization

1. **Use indexes** on columns frequently used in WHERE, JOIN, and ORDER BY clauses
2. **Select only needed columns** instead of using SELECT *
3. **Use LIMIT** to restrict result set size when testing
4. **Avoid functions on indexed columns** in WHERE clauses
5. **Use EXPLAIN** to analyze query execution plans

### Security

1. **Never concatenate user input** into SQL queries (SQL injection risk)
2. **Use parameterized queries** or prepared statements
3. **Grant minimal necessary permissions** to database users
4. **Validate and sanitize** all user input
5. **Use transactions** for operations that must succeed or fail together

### Naming Conventions

1. **Use lowercase** with underscores for table and column names
2. **Be descriptive** but concise (e.g., employee_id, not emp_id)
3. **Use singular names** for tables (employee, not employees - though both conventions exist)
4. **Prefix indexes** with idx_ (e.g., idx_last_name)
5. **Prefix foreign keys** with fk_ (e.g., fk_department_id)

### Data Integrity

1. **Always use constraints** to enforce data rules
2. **Include NOT NULL** where appropriate
3. **Use foreign keys** to maintain referential integrity
4. **Add CHECK constraints** for business rule validation
5. **Back up data regularly**

---

## Common SQL Patterns

### Pagination

**Example:**
```sql
-- Get page 3 with 10 results per page
SELECT * FROM employees
ORDER BY employee_id
LIMIT 10 OFFSET 20;
```

### Find Duplicates

**Example:**
```sql
SELECT email, COUNT(*) as count
FROM employees
GROUP BY email
HAVING COUNT(*) > 1;
```

### Remove Duplicates

**Example:**
```sql
DELETE FROM employees
WHERE employee_id NOT IN (
    SELECT MIN(employee_id)
    FROM employees
    GROUP BY email
);
```

### Ranking

**Example (using window functions):**
```sql
SELECT 
    first_name, 
    last_name, 
    salary,
    RANK() OVER (ORDER BY salary DESC) AS salary_rank
FROM employees;
```

### Top N Per Group

**Example:**
```sql
-- Top 3 highest paid employees per department
SELECT *
FROM (
    SELECT 
        first_name,
        last_name,
        department_id,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rn
    FROM employees
) ranked
WHERE rn <= 3;
```

### Running Total

**Example:**
```sql
SELECT 
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;
```

---

## Common String Functions

### CONCAT

Concatenates strings together.

**Syntax:**
```sql
SELECT CONCAT(string1, string2, ...) FROM table_name;
```

**Example:**
```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM employees;
```

### UPPER / LOWER

Converts string to uppercase or lowercase.

**Example:**
```sql
SELECT UPPER(first_name) AS upper_name, LOWER(last_name) AS lower_name
FROM employees;
```

### LENGTH

Returns the length of a string.

**Example:**
```sql
SELECT first_name, LENGTH(first_name) AS name_length
FROM employees;
```

### SUBSTRING

Extracts a portion of a string.

**Syntax:**
```sql
SUBSTRING(string, start_position, length)
```

**Example:**
```sql
SELECT SUBSTRING(email, 1, 5) AS email_prefix
FROM employees;
```

### TRIM

Removes leading and trailing spaces.

**Example:**
```sql
SELECT TRIM(first_name) AS trimmed_name
FROM employees;
```

### REPLACE

Replaces occurrences of a substring.

**Example:**
```sql
SELECT REPLACE(phone_number, '-', '') AS clean_phone
FROM employees;
```

---

## Common Date Functions

### CURRENT_DATE / CURRENT_TIME / CURRENT_TIMESTAMP

Returns current date, time, or timestamp.

**Example:**
```sql
SELECT CURRENT_DATE AS today;
SELECT CURRENT_TIME AS now;
SELECT CURRENT_TIMESTAMP AS right_now;
```

### DATE_ADD / DATE_SUB

Adds or subtracts time intervals from dates.

**Example (MySQL):**
```sql
SELECT DATE_ADD(hire_date, INTERVAL 1 YEAR) AS anniversary
FROM employees;

SELECT DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY) AS thirty_days_ago;
```

### DATEDIFF

Calculates difference between two dates.

**Example:**
```sql
SELECT DATEDIFF(CURRENT_DATE, hire_date) AS days_employed
FROM employees;
```

### EXTRACT

Extracts part of a date.

**Example:**
```sql
SELECT 
    EXTRACT(YEAR FROM hire_date) AS hire_year,
    EXTRACT(MONTH FROM hire_date) AS hire_month
FROM employees;
```

---

## Common Mathematical Functions

### ROUND

Rounds a number to specified decimal places.

**Example:**
```sql
SELECT salary, ROUND(salary, 2) AS rounded_salary
FROM employees;
```

### CEIL / FLOOR

Rounds up or down to nearest integer.

**Example:**
```sql
SELECT 
    CEIL(salary / 1000) AS rounded_up,
    FLOOR(salary / 1000) AS rounded_down
FROM employees;
```

### ABS

Returns absolute value.

**Example:**
```sql
SELECT ABS(-100) AS absolute_value;
```

### POWER

Raises a number to a power.

**Example:**
```sql
SELECT POWER(2, 10) AS result;  -- 2^10 = 1024
```

### MOD

Returns remainder of division.

**Example:**
```sql
SELECT employee_id, MOD(employee_id, 2) AS is_even
FROM employees;
```

---

## Window Functions

Window functions perform calculations across a set of rows related to the current row.

### ROW_NUMBER

Assigns a unique sequential integer to rows.

**Example:**
```sql
SELECT 
    first_name,
    last_name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num
FROM employees;
```

### RANK

Assigns a rank with gaps for ties.

**Example:**
```sql
SELECT 
    first_name,
    last_name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS salary_rank
FROM employees;
```

### DENSE_RANK

Assigns a rank without gaps for ties.

**Example:**
```sql
SELECT 
    first_name,
    last_name,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
FROM employees;
```

### PARTITION BY

Divides result set into partitions for window function calculation.

**Example:**
```sql
SELECT 
    first_name,
    last_name,
    department_id,
    salary,
    AVG(salary) OVER (PARTITION BY department_id) AS dept_avg_salary
FROM employees;
```

### LAG / LEAD

Access data from previous or next row.

**Example:**
```sql
SELECT 
    order_date,
    amount,
    LAG(amount, 1) OVER (ORDER BY order_date) AS previous_amount,
    LEAD(amount, 1) OVER (ORDER BY order_date) AS next_amount
FROM orders;
```

---

## Common Table Expressions (CTE)

CTEs provide a way to write auxiliary statements for use in a larger query.

### Basic CTE

**Syntax:**
```sql
WITH cte_name AS (
    SELECT column1, column2
    FROM table_name
    WHERE condition
)
SELECT * FROM cte_name;
```

**Example:**
```sql
WITH high_earners AS (
    SELECT employee_id, first_name, last_name, salary
    FROM employees
    WHERE salary > 80000
)
SELECT * FROM high_earners
ORDER BY salary DESC;
```

### Multiple CTEs

**Example:**
```sql
WITH 
dept_stats AS (
    SELECT 
        department_id,
        COUNT(*) AS emp_count,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
),
dept_info AS (
    SELECT department_id, department_name, location
    FROM departments
)
SELECT 
    di.department_name,
    di.location,
    ds.emp_count,
    ds.avg_salary
FROM dept_stats ds
JOIN dept_info di ON ds.department_id = di.department_id;
```

### Recursive CTE

Used for hierarchical or recursive data.

**Example:**
```sql
-- Find all employees in management chain
WITH RECURSIVE management_chain AS (
    -- Base case: start with CEO
    SELECT employee_id, first_name, last_name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: find direct reports
    SELECT e.employee_id, e.first_name, e.last_name, e.manager_id, mc.level + 1
    FROM employees e
    JOIN management_chain mc ON e.manager_id = mc.employee_id
)
SELECT * FROM management_chain
ORDER BY level, employee_id;
```

---
