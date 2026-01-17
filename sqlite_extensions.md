# SQLite SQL Language Extensions Reference

This document describes SQL language features that are specific to SQLite and are not part of the standard SQL specification.

## Table of Contents

1. [Data Definition Language Extensions](#data-definition-language-extensions)
2. [Data Manipulation Language Extensions](#data-manipulation-language-extensions)
3. [Query Language Extensions](#query-language-extensions)
4. [Constraint and Conflict Resolution](#constraint-and-conflict-resolution)
5. [Table Modifiers](#table-modifiers)
6. [PRAGMA Statements](#pragma-statements)
7. [Additional Features](#additional-features)

---

## Data Definition Language Extensions

### IF NOT EXISTS Clause

SQLite adds the IF NOT EXISTS clause to CREATE TABLE, CREATE INDEX, CREATE VIEW, and CREATE TRIGGER statements.

**Syntax:**
```sql
CREATE TABLE IF NOT EXISTS table_name (
    column_definitions
);
```

**Purpose:** Prevents errors when attempting to create database objects that already exist. If the object exists, the statement is silently ignored.

**Example:**
```sql
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL
);
```

### ALTER TABLE Limitations and Extensions

SQLite supports a limited subset of ALTER TABLE operations compared to standard SQL.

**Supported Operations:**

1. **RENAME TABLE** (standard SQL)
   ```sql
   ALTER TABLE old_name RENAME TO new_name;
   ```

2. **RENAME COLUMN** (added in SQLite 3.25.0)
   ```sql
   ALTER TABLE table_name RENAME COLUMN old_column TO new_column;
   ```

3. **ADD COLUMN**
   ```sql
   ALTER TABLE table_name ADD COLUMN column_name column_definition;
   ```
   
   Limitations when adding columns:
   - Cannot have UNIQUE or PRIMARY KEY constraints
   - If NOT NULL is specified, must provide a default value
   - Cannot use CURRENT_TIMESTAMP, CURRENT_DATE, or CURRENT_TIME as defaults
   - Foreign key columns must accept NULL as default

4. **DROP COLUMN** (added in SQLite 3.35.0)
   ```sql
   ALTER TABLE table_name DROP COLUMN column_name;
   ```

**Not Supported:**
- ALTER COLUMN (to modify column type or constraints)
- Most other ALTER TABLE operations found in standard SQL

**Workaround:** For unsupported operations, use the table recreation method:
```sql
PRAGMA foreign_keys=off;
BEGIN TRANSACTION;
CREATE TABLE new_table (new_schema);
INSERT INTO new_table SELECT ... FROM old_table;
DROP TABLE old_table;
ALTER TABLE new_table RENAME TO old_table;
COMMIT;
PRAGMA foreign_keys=on;
```

### AUTOINCREMENT Keyword

SQLite provides AUTOINCREMENT for INTEGER PRIMARY KEY columns.

**Syntax:**
```sql
CREATE TABLE table_name (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    other_columns
);
```

**Behavior:**
- Ensures that automatically generated row IDs are always increasing
- Prevents reuse of row IDs from deleted rows
- Creates and maintains a special sqlite_sequence table
- Slightly slower than regular INTEGER PRIMARY KEY
- Cannot be used with WITHOUT ROWID tables

**Example:**
```sql
CREATE TABLE orders (
    order_id INTEGER PRIMARY KEY AUTOINCREMENT,
    customer_name TEXT,
    order_date TEXT
);
```

---

## Data Manipulation Language Extensions

### REPLACE Statement

SQLite provides REPLACE as a shorthand for INSERT OR REPLACE.

**Syntax:**
```sql
REPLACE INTO table_name (columns) VALUES (values);
```

**Behavior:**
- If a UNIQUE or PRIMARY KEY constraint would be violated, deletes the existing row(s) before inserting
- Equivalent to `INSERT OR REPLACE INTO`
- Can trigger DELETE triggers if they exist

**Example:**
```sql
CREATE TABLE products (
    product_id INTEGER PRIMARY KEY,
    name TEXT,
    price REAL
);

REPLACE INTO products VALUES (1, 'Widget', 19.99);
-- If product_id 1 exists, it will be deleted first
```

### UPSERT (INSERT ... ON CONFLICT)

SQLite added UPSERT syntax in version 3.24.0 (2018-06-04), using PostgreSQL-compatible syntax.

**Syntax:**
```sql
INSERT INTO table_name (columns)
VALUES (values)
ON CONFLICT (conflict_target) DO UPDATE SET column = value
[WHERE condition];

-- Or do nothing on conflict
INSERT INTO table_name (columns)
VALUES (values)
ON CONFLICT (conflict_target) DO NOTHING;
```

**conflict_target** can be:
- Column name(s): `ON CONFLICT (column_name)`
- WHERE clause for partial indexes: `ON CONFLICT WHERE condition`

**Special keyword `excluded`:** References the row that would have been inserted.

**Examples:**
```sql
-- Update price if product already exists
INSERT INTO products (product_id, name, price)
VALUES (1, 'Widget', 19.99)
ON CONFLICT (product_id) DO UPDATE SET
    price = excluded.price;

-- Update only if new price is higher
INSERT INTO products (product_id, name, price)
VALUES (1, 'Widget', 25.00)
ON CONFLICT (product_id) DO UPDATE SET
    price = excluded.price
WHERE excluded.price > products.price;

-- Ignore duplicates silently
INSERT INTO products (product_id, name, price)
VALUES (1, 'Widget', 19.99)
ON CONFLICT (product_id) DO NOTHING;
```

**Limitation:** Cannot use SELECT in the INSERT statement before ON CONFLICT without adding another clause to disambiguate the parser.

### RETURNING Clause

SQLite added the RETURNING clause in version 3.35.0 (2021-03-12).

**Syntax:**
```sql
INSERT INTO table_name (columns) VALUES (values) RETURNING *;
UPDATE table_name SET column = value RETURNING column1, column2;
DELETE FROM table_name WHERE condition RETURNING *;
```

**Purpose:** Returns the rows affected by INSERT, UPDATE, or DELETE operations without requiring a separate SELECT.

**Examples:**
```sql
-- Get the auto-generated ID after insert
INSERT INTO users (name, email)
VALUES ('Alice', 'alice@example.com')
RETURNING id;

-- Get updated values
UPDATE products
SET price = price * 1.1
WHERE category = 'electronics'
RETURNING product_id, name, price;

-- See what was deleted
DELETE FROM old_records
WHERE created_date < '2020-01-01'
RETURNING *;
```

---

## Query Language Extensions

### LIMIT and OFFSET Clauses

SQLite supports LIMIT and OFFSET for pagination, though this is now common in many databases.

**Syntax:**
```sql
SELECT columns FROM table
LIMIT row_count;

SELECT columns FROM table
LIMIT row_count OFFSET skip_count;

-- Alternative syntax (offset first, then limit)
SELECT columns FROM table
LIMIT skip_count, row_count;
```

**Examples:**
```sql
-- First 10 rows
SELECT * FROM products LIMIT 10;

-- Skip first 20, get next 10 (page 3 of results)
SELECT * FROM products LIMIT 10 OFFSET 20;

-- Same using alternative syntax
SELECT * FROM products LIMIT 20, 10;

-- Get the 3rd highest price
SELECT price FROM products
ORDER BY price DESC
LIMIT 1 OFFSET 2;
```

**Best Practice:** Always use LIMIT with ORDER BY to ensure consistent, predictable results.

### ATTACH DATABASE

SQLite allows attaching multiple database files to a single connection.

**Syntax:**
```sql
ATTACH DATABASE 'file_path' AS schema_name;
DETACH DATABASE schema_name;
```

**Limits:**
- Maximum of 10 attached databases by default (configurable up to 125)
- Each attached database gets its own schema name

**Examples:**
```sql
-- Attach another database
ATTACH DATABASE '/path/to/other.db' AS secondary;

-- Query across databases
SELECT * FROM main.users
JOIN secondary.orders ON users.id = orders.user_id;

-- Detach when done
DETACH DATABASE secondary;
```

### Bare Columns in Aggregate Queries

SQLite allows bare columns (non-aggregated columns not in GROUP BY) in aggregate queries. This is an SQLite-specific extension that other databases typically disallow.

**Example:**
```sql
-- This works in SQLite but would fail in PostgreSQL/MySQL strict mode
SELECT department, employee_name, MAX(salary)
FROM employees
GROUP BY department;
-- Returns one row per department with an arbitrary employee_name
```

**Warning:** Results are unpredictable for bare columns. Use this feature carefully or avoid it for portability.

---

## Constraint and Conflict Resolution

### ON CONFLICT Clause

SQLite provides ON CONFLICT clause for handling constraint violations in DDL and DML.

**Resolution Algorithms:**
1. **ROLLBACK** - Abort SQL statement and roll back transaction
2. **ABORT** - Abort statement but preserve transaction (default)
3. **FAIL** - Abort statement but preserve completed changes
4. **IGNORE** - Skip the violating row and continue
5. **REPLACE** - Delete existing row(s) and insert new row

**DDL Usage (in CREATE TABLE):**
```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    email TEXT UNIQUE ON CONFLICT REPLACE,
    username TEXT NOT NULL ON CONFLICT ABORT
);
```

**DML Usage (in INSERT/UPDATE):**
```sql
-- Using OR keyword instead of ON CONFLICT
INSERT OR REPLACE INTO users (id, email) VALUES (1, 'new@example.com');
INSERT OR IGNORE INTO users (id, email) VALUES (1, 'duplicate@example.com');
UPDATE OR FAIL users SET email = 'taken@example.com' WHERE id = 5;
```

**Examples:**
```sql
-- Constraint-level conflict resolution
CREATE TABLE products (
    product_code TEXT PRIMARY KEY ON CONFLICT REPLACE,
    name TEXT NOT NULL ON CONFLICT FAIL,
    quantity INTEGER CHECK(quantity >= 0) ON CONFLICT ABORT
);

-- Statement-level conflict resolution
INSERT OR IGNORE INTO products VALUES ('ABC', 'Widget', 10);
```

---

## Table Modifiers

### WITHOUT ROWID

SQLite added WITHOUT ROWID tables in version 3.8.2 (2013-12-06).

**Syntax:**
```sql
CREATE TABLE table_name (
    primary_key_columns,
    other_columns,
    PRIMARY KEY (primary_key_columns)
) WITHOUT ROWID;
```

**Characteristics:**
- Uses a clustered index (primary key is the storage key)
- No implicit rowid/oid/_rowid_ column
- Must have an explicit PRIMARY KEY
- Cannot use AUTOINCREMENT
- All PRIMARY KEY columns are implicitly NOT NULL
- Can improve performance for tables with composite primary keys
- Can reduce storage space in some cases

**When to Use:**
- Tables with natural composite primary keys
- Tables where primary key lookups are the dominant access pattern
- Tables where reducing storage overhead matters

**Example:**
```sql
CREATE TABLE word_index (
    word TEXT,
    document_id INTEGER,
    position INTEGER,
    PRIMARY KEY (word, document_id, position)
) WITHOUT ROWID;
```

### STRICT Tables

SQLite added STRICT tables in version 3.37.0 (2021-11-27).

**Syntax:**
```sql
CREATE TABLE table_name (
    column1 datatype,
    column2 datatype
) STRICT;
```

**Allowed Datatypes in STRICT Tables:**
- INT or INTEGER
- REAL
- TEXT
- BLOB
- ANY (allows any type in that column)

**Behavior:**
- Enforces strict type checking (like standard SQL databases)
- Values must match the declared type or be NULL
- Type coercion follows standard affinity rules
- Rejects values that cannot be losslessly converted
- Every column must have an explicit datatype

**Examples:**
```sql
-- Strict table with type enforcement
CREATE TABLE measurements (
    id INTEGER PRIMARY KEY,
    temperature REAL,
    reading_time TEXT,
    notes TEXT
) STRICT;

-- This will fail (cannot insert text into REAL column)
INSERT INTO measurements VALUES (1, 'hot', '2024-01-01', 'test');

-- Flexible column using ANY
CREATE TABLE events (
    id INTEGER PRIMARY KEY,
    event_type TEXT,
    event_data ANY  -- Can store any type
) STRICT;
```

### GENERATED Columns

SQLite added generated (computed) columns in version 3.31.0 (2020-01-22).

**Syntax:**
```sql
CREATE TABLE table_name (
    regular_column datatype,
    computed_column datatype GENERATED ALWAYS AS (expression) [VIRTUAL | STORED]
);

-- Simplified syntax (GENERATED ALWAYS is optional)
computed_column datatype AS (expression) [VIRTUAL | STORED]
```

**Types:**
- **VIRTUAL** (default) - Computed when read, not stored
- **STORED** - Computed when written, takes storage space

**Rules:**
- Expression can only reference columns in the same row
- Cannot reference ROWID directly (but can reference INTEGER PRIMARY KEY)
- Cannot use subqueries, aggregates, or window functions
- Cannot be part of PRIMARY KEY
- Cannot have DEFAULT clause
- Cannot modify generated columns directly

**Examples:**
```sql
CREATE TABLE products (
    product_id INTEGER PRIMARY KEY,
    name TEXT,
    price REAL,
    tax_rate REAL,
    
    -- Virtual column (computed on read)
    price_with_tax REAL GENERATED ALWAYS AS (price * (1 + tax_rate)) VIRTUAL,
    
    -- Stored column (computed on write)
    price_category TEXT AS (
        CASE
            WHEN price < 10 THEN 'cheap'
            WHEN price < 100 THEN 'moderate'
            ELSE 'expensive'
        END
    ) STORED
);

-- Can query generated columns like regular columns
SELECT name, price, price_with_tax FROM products;

-- Cannot insert/update generated columns
-- This will fail:
-- UPDATE products SET price_with_tax = 25.00;
```

**ALTER TABLE Limitation:** Can only add VIRTUAL generated columns with ALTER TABLE, not STORED columns.

---

## PRAGMA Statements

PRAGMA is a SQLite-specific SQL extension for querying and modifying database settings and internal state.

**Syntax:**
```sql
PRAGMA [schema_name.]pragma_name;
PRAGMA [schema_name.]pragma_name = value;
PRAGMA [schema_name.]pragma_name(value);
```

**Common PRAGMAs:**

### Schema Information
```sql
-- List all tables
PRAGMA table_list;

-- Show table structure
PRAGMA table_info(table_name);

-- Show indexes
PRAGMA index_list(table_name);

-- Show foreign keys
PRAGMA foreign_key_list(table_name);
```

### Database Configuration
```sql
-- Enable foreign key constraints
PRAGMA foreign_keys = ON;

-- Set journal mode (WAL is recommended for concurrency)
PRAGMA journal_mode = WAL;

-- Set synchronous mode
PRAGMA synchronous = NORMAL;

-- Set page cache size (in KB, negative numbers)
PRAGMA cache_size = -64000;  -- 64MB cache
```

### Query PRAGMAs as Tables
```sql
-- PRAGMAs can be queried like tables using pragma_ prefix
SELECT * FROM pragma_table_info('users');

SELECT m.name as table_name, ii.name as column_name
FROM sqlite_schema AS m,
     pragma_index_list(m.name) AS il,
     pragma_index_info(il.name) AS ii
WHERE m.type = 'table';
```

**Important Notes:**
- PRAGMA statements are not part of standard SQL
- Some PRAGMAs take effect at compile time, not execution time
- Unknown PRAGMAs are silently ignored (no errors)
- PRAGMA values may not be backwards compatible across SQLite versions

---

## Additional Features

### Date and Time Functions

SQLite stores dates/times as TEXT, REAL, or INTEGER, not as dedicated DATE/TIME types. It provides functions for working with ISO 8601 format strings.

**Common Functions:**
```sql
SELECT date('now');                    -- Current date
SELECT time('now');                    -- Current time
SELECT datetime('now');                -- Current timestamp
SELECT strftime('%Y-%m-%d', 'now');   -- Formatted date

-- Date arithmetic
SELECT date('now', '+7 days');
SELECT datetime('now', '-1 month', 'start of month');
```

### Boolean Literals

SQLite added TRUE and FALSE keywords in version 3.23.0 (2018-04-02).

**Usage:**
```sql
-- TRUE and FALSE are synonyms for 1 and 0
CREATE TABLE settings (
    id INTEGER PRIMARY KEY,
    is_enabled INTEGER DEFAULT TRUE
);

INSERT INTO settings VALUES (1, FALSE);

-- IS TRUE / IS FALSE operators
SELECT * FROM settings WHERE is_enabled IS TRUE;
SELECT * FROM settings WHERE is_enabled IS NOT FALSE;
```

### Flexible Typing (Default Behavior)

In non-STRICT tables, SQLite uses dynamic typing where values can be of any type regardless of declared column type.

**Example:**
```sql
CREATE TABLE flexible (
    number_col INTEGER,
    text_col TEXT
);

-- This works (though not recommended)
INSERT INTO flexible VALUES ('hello', 123);
INSERT INTO flexible VALUES (456, 'world');
```

**Type Affinity Rules:** SQLite attempts to convert values based on the declared column type but will store the value as-is if conversion fails.

---
