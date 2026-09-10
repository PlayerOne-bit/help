# SQL Tutorial

## SQL Data Types:
### 1. Numeric Types

INT / INTEGER: Whole numbers.
SMALLINT / BIGINT: Smaller or larger integer ranges.
DECIMAL(p, s) / NUMERIC(p, s): Exact precision numbers; p is total digits, s is digits after decimal.
FLOAT / REAL / DOUBLE: Approximate decimal numbers for floating points.

### 2. Character/String Types

CHAR(n): Fixed-length string, padded with spaces if shorter.
VARCHAR(n): Variable-length string with a maximum of n characters.
TEXT / CLOB: Large text fields without specific length limits (varies by DB).

### 3. Date and Time Types

DATE: Stores date in YYYY-MM-DD format.
TIME: Stores time in HH:MM:SS format.
DATETIME / TIMESTAMP: Stores date and time together.

### 4. Boolean Types

BOOLEAN / BIT: Stores TRUE or FALSE values. Implementation may vary; some use 0 for false, 1 for true.

### 5. Other Common Types

BLOB / BYTEA: Binary large objects like images or files.
JSON / JSONB: Stores JSON-formatted data, supported in modern databases like PostgreSQL.

## SQL Syntax:

### 1. DATABASE:
- Creating a database;
```sql
CREATE DATABASE database_name;
USE database_name;
```
- Dropping a database
```sql
DROP DATABASE database_name
```
### 2. TABLES
- Creating a table
```sql
CREATE TABLE table_name (
    column1 datatype constraint,
    column2 datatype constraint,
    column3 datatype constraint,
    ...
);
```
- Dropping a table
```sql
DROP DATABASE table_name;
```
Always add this line before below
```sql
ALTER TABLE table_name
```
- Add a column
```sql
ADD column_name data_type [constraint];
```
- Modify an existing column's name
```sql
ALTER COLUMN column_name data_type;
```
- Modify a data type with constraint
```sql
MODIFY COLUMN column_name data_type;
```
- Drop a column
```sql
DROP COLUMN column_name;
```
- Add a constraint
```sql
ADD CONSTRAINT constraint_name constraint_type (column_name);
```
- Drop a constraint
```sql
DROP CONSTRAINT constraint_name;
```

### 3. CRUD SYNTAX:
- Creating a data
```sql
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);
```
- Reading a data
```sql
SELECT column1, column2 FROM table_name WHERE condition;
```
- Updating a data
```sql
UPDATE table_name
SET column1 = value1, column2 = value2
WHERE condition;
```
- Deleting a data
```sql
DELETE FROM table_name WHERE condition;
```