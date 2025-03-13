<h1 align="center"> DuckDB </h1>

# Content

1. [Chapter 1: DuckDB Overview](#chapter1)
    - [Chapter 1 - Part 1: What is DuckDB](#chapter1part1)
2. [Appendix A: Useful DuckDB Code Snippet](#appendixa)
    - [Appendix A - Part 1: Remove characters from VARCHARS using REGEXP_REPLACE](#appendixapart1)
    - [Appendix A - Part 2: Check if a column have different values in other column](#appendixapart2)
    - [Appendix A - Part 3: Check for duplicate lines](#appendixapart3)
    - [Appendix A - Part 4: Find a character in a VARCHAR field](#appendixapart4)
    - [Appendix A - Part 5: Split a field and create new columns](#appendixapart5)
    
## <a name="chapter1"></a>Chapter 1: DuckDB Overview

#### <a name="chapter1part1"></a>Chapter 1 - Part 1: What is DuckDB

## <a name="appendixa"></a>Appendix A: Useful DuckDB Code Snippet

#### <a name="appendixapart1"></a>Appendix A - Part 1: Remove characters from VARCHARS using REGEXP_REPLACE

```
CREATE TABLE test AS SELECT CONCAT(chr(13),chr(10),'Hello World') AS example;

SELECT * FROM test;

┌─────────────────┐
│     example     │
│     varchar     │
├─────────────────┤
│ \r\nHello World │
└─────────────────┘

SELECT REGEXP_REPLACE(example, '[\n\r]+'::text, ' '::text, 'g'::text) AS transformed_example FROM test;

┌─────────────────────┐
│ transformed_example │
│       varchar       │
├─────────────────────┤
│  Hello World        │
└─────────────────────┘
```

#### <a name="appendixapart2"></a>Appendix A - Part 2: Check if a column have different values in other column

```
CREATE TABLE products (sku VARCHAR(10),price DECIMAL(10,2));

INSERT INTO products (sku, price) VALUES
('AA', 20),
('AA', 30),
('BB', 40),
('CC', 50),
('CC', 50),
('DD', 60),
('DD', 70);

SELECT * FROM products;
┌─────────┬───────────────┐
│   sku   │     price     │
│ varchar │ decimal(10,2) │
├─────────┼───────────────┤
│ AA      │         20.00 │
│ AA      │         30.00 │
│ BB      │         40.00 │
│ CC      │         50.00 │
│ CC      │         50.00 │
│ DD      │         60.00 │
│ DD      │         70.00 │
└─────────┴───────────────┘

SELECT sku FROM products GROUP BY sku HAVING COUNT(DISTINCT price) > 1;

┌─────────┐
│   sku   │
│ varchar │
├─────────┤
│ AA      │
│ DD      │
└─────────┘
```

#### <a name="appendixapart3"></a>Appendix A - Part 3: Check for duplicate lines

```
CREATE TABLE products (sku VARCHAR(10),price DECIMAL(10,2));

INSERT INTO products (sku, price) VALUES
('AA', 20),
('AA', 30),
('BB', 40),
('CC', 50),
('CC', 50),
('DD', 60),
('DD', 70);

SELECT * FROM products;
┌─────────┬───────────────┐
│   sku   │     price     │
│ varchar │ decimal(10,2) │
├─────────┼───────────────┤
│ AA      │         20.00 │
│ AA      │         30.00 │
│ BB      │         40.00 │
│ CC      │         50.00 │
│ CC      │         50.00 │
│ DD      │         60.00 │
│ DD      │         70.00 │
└─────────┴───────────────┘

SELECT sku, price, COUNT(*) FROM products GROUP BY sku, price HAVING COUNT(*) > 1 ORDER BY sku ASC;

┌─────────┬───────────────┬──────────────┐
│   sku   │     price     │ count_star() │
│ varchar │ decimal(10,2) │    int64     │
├─────────┼───────────────┼──────────────┤
│ CC      │         50.00 │            2 │
└─────────┴───────────────┴──────────────┘
```

#### <a name="appendixapart4"></a>Appendix A - Part 4: Find a character in a VARCHAR field

```
CREATE TABLE products (sku VARCHAR(10),price DECIMAL(10,2));

INSERT INTO products (sku, price) VALUES
('AA', 20),
('AA', 30),
('B|B', 40),
('CC', 50),
('CC', 50),
('DD', 60),
('DD', 70);

SELECT * FROM products;

┌─────────┬───────────────┐
│   sku   │     price     │
│ varchar │ decimal(10,2) │
├─────────┼───────────────┤
│ AA      │         20.00 │
│ AA      │         30.00 │
│ B|B     │         40.00 │
│ CC      │         50.00 │
│ CC      │         50.00 │
│ DD      │         60.00 │
│ DD      │         70.00 │
└─────────┴───────────────┘

SELECT * FROM products WHERE sku LIKE '%|%';

┌─────────┬───────────────┐
│   sku   │     price     │
│ varchar │ decimal(10,2) │
├─────────┼───────────────┤
│ B|B     │         40.00 │
└─────────┴───────────────┘
```

#### <a name="appendixapart5"></a>Appendix A - Part 5: Split a field and create new columns

```
CREATE TABLE products (line_number INT, sku VARCHAR(10), price DECIMAL(10,2), metafields VARCHAR);

INSERT INTO products (line_number, sku, price, metafields) VALUES
  (1, 'AA', 20.0, 'name|maria#lastname|joao#gender|femea'),
  (2, 'BB', 30.0, 'name|carlos#gender|male'),
  (3, 'CC', 40.0, 'name|joana');
  
SELECT * FROM products;

┌─────────────┬─────────┬───────────────┬───────────────────────────────────────┐
│ line_number │   sku   │     price     │              metafields               │
│    int32    │ varchar │ decimal(10,2) │                varchar                │
├─────────────┼─────────┼───────────────┼───────────────────────────────────────┤
│           1 │ AA      │         20.00 │ name|maria#lastname|joao#gender|femea │
│           2 │ BB      │         30.00 │ name|carlos#gender|male               │
│           3 │ CC      │         40.00 │ name|joana                            │
└─────────────┴─────────┴───────────────┴───────────────────────────────────────┘

WITH pairs AS (
  SELECT
 line_number AS LineNumber,
 sku AS SKU,
 price AS PRICE,
 regexp_split_to_table(metafields, '#') AS metafields
 FROM products
 ),
 key_value AS (
 SELECT
  LineNumber AS LineNumber,
  SKU AS SKU,
  PRICE AS PRICE,
  split_part(metafields, '|', 1) AS key,
 split_part(metafields, '|', 2) AS pair
 FROM pairs
 ),
 firstNameQuery AS (
 SELECT
   LineNumber AS LineNumber,
   SKU AS SKU,
   PRICE AS PRICE,
   CASE 
      WHEN key = 'name' 
      THEN pair 
      ELSE NULL 
  END AS FirstName
 FROM key_value
 )
 
 SELECT
   LineNumber AS LineNumber,
   SKU AS SKU,
   PRICE AS PRICE,
   FirstName AS FIRSTNAME
 FROM firstNameQuery
 WHERE(CASE WHEN FirstName IS NOT NULL AND FirstName != '' THEN 1 ELSE 0 END) >= 1;
 
┌────────────┬─────────┬───────────────┬───────────┐
│ LineNumber │   SKU   │     PRICE     │ FIRSTNAME │
│   int32    │ varchar │ decimal(10,2) │  varchar  │
├────────────┼─────────┼───────────────┼───────────┤
│          1 │ AA      │         20.00 │ maria     │
│          2 │ BB      │         30.00 │ carlos    │
│          3 │ CC      │         40.00 │ joana     │
└────────────┴─────────┴───────────────┴───────────┘
```
