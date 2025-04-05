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
    - [Appendix A - Part 6: Split a field and aggregate values](#appendixapart6)
    - [Appendix A - Part 7: Select just even numbers](#appendixapart7)
    - [Appendix A - Part 8: Find the difference of Duplicates](#appendixapart8)
    - [Appendix A - Part 9: Find the Min and Max Length of a String and ordered alphabetically](#appendixapart9)
    
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

 #### <a name="appendixapart6"></a>Appendix A - Part 6: Split a field and aggregate values

 ```
CREATE TABLE products (sku VARCHAR(10),price DECIMAL(10,2), size VARCHAR(10));

INSERT INTO products (sku, price, size) VALUES
  ('AA', 20, 'S'),
  ('AA', 20, 'M'),
  ('AA', 20, 'L'),
  ('BB', 30, '38'),
  ('BB', 30, '39');

SELECT * FROM products;

┌─────────┬───────────────┬─────────┐
│   sku   │     price     │  size   │
│ varchar │ decimal(10,2) │ varchar │
├─────────┼───────────────┼─────────┤
│ AA      │         20.00 │ S       │
│ AA      │         20.00 │ M       │
│ AA      │         20.00 │ L       │
│ BB      │         30.00 │ 38      │
│ BB      │         30.00 │ 39      │
└─────────┴───────────────┴─────────┘

SELECT
    price AS PRICE,
    CONCAT(sku,'|',string_agg(TRIM(size), '_' ORDER BY size)) AS ConcatSize
FROM
    products
GROUP BY
    sku, price;

┌───────────────┬────────────┐
│     PRICE     │ ConcatSize │
│ decimal(10,2) │  varchar   │
├───────────────┼────────────┤
│         30.00 │ BB|38_39   │
│         20.00 │ AA|L_M_S   │
└───────────────┴────────────┘
```

 #### <a name="appendixapart7"></a>Appendix A - Part 7: Split a field and aggregate values

```
CREATE TABLE EXAMPLE (ID INTEGER,NAME VARCHAR(17),COUNTRYCODE VARCHAR(3), DISTRICT VARCHAR(20), POPULATION INTEGER);

INSERT INTO EXAMPLE (ID, NAME, COUNTRYCODE, DISTRICT, POPULATION) VALUES
  (1, 'AA', 'IT', 'ABC', 100000),
  (2, 'BB', 'DE', 'DEF', 99999),
  (3, 'CC', 'FR', 'GHI', 100001);

SELECT * FROM EXAMPLE;

┌───────┬─────────┬─────────────┬──────────┬────────────┐
│  ID   │  NAME   │ COUNTRYCODE │ DISTRICT │ POPULATION │
│ int32 │ varchar │   varchar   │ varchar  │   int32    │
├───────┼─────────┼─────────────┼──────────┼────────────┤
│     1 │ AA      │ IT          │ ABC      │     100000 │
│     2 │ BB      │ DE          │ DEF      │      99999 │
│     3 │ CC      │ FR          │ GHI      │     100001 │
└───────┴─────────┴─────────────┴──────────┴────────────┘

SELECT DISTINCT
      *
      FROM EXAMPLE
      WHERE
      CASE
          WHEN MOD(ID,2) = 0
          THEN 1
          ELSE 0
      END;

┌───────┬─────────┬─────────────┬──────────┬────────────┐
│  ID   │  NAME   │ COUNTRYCODE │ DISTRICT │ POPULATION │
│ int32 │ varchar │   varchar   │ varchar  │   int32    │
├───────┼─────────┼─────────────┼──────────┼────────────┤
│   2   │ BB      │ DE          │ DEF      │   99999    │
└───────┴─────────┴─────────────┴──────────┴────────────┘
```

 #### <a name="appendixapart8"></a>Appendix A - Part 8: Find the difference of Duplicate Values

 Find the difference between the total number of COUNTRYCODE entries in the table and the number of distinct COUNTRYCODE entries in the table

 ```
CREATE TABLE EXAMPLE (ID INTEGER,NAME VARCHAR(17),COUNTRYCODE VARCHAR(3), DISTRICT VARCHAR(20), POPULATION INTEGER);

INSERT INTO EXAMPLE (ID, NAME, COUNTRYCODE, DISTRICT, POPULATION) VALUES
(1, 'AA', 'IT', 'ABC', 100000),
(2, 'BB', 'IT', 'DEF', 99999),
(3, 'CC', 'FR', 'GHI', 100001);

SELECT COUNT(COUNTRYCODE) - COUNT(DISTINCT COUNTRYCODE) AS Difference FROM EXAMPLE;

┌────────────┐
│ Difference │
│   int64    │
├────────────┤
│     1      │
└────────────┘

```

 #### <a name="appendixapart9"></a>Appendix A - Part 9: Find the Min and Max Length of a String and ordered alphabetically

```
CREATE TABLE EXAMPLE (ID INTEGER,NAME VARCHAR(17),COUNTRYCODE VARCHAR(3), DISTRICT VARCHAR(20), POPULATION INTEGER);

INSERT INTO EXAMPLE (ID, NAME, COUNTRYCODE, DISTRICT, POPULATION) VALUES
(1, 'Abruzzo', 'IT', 'ABC', 100000),
(2, 'Roma', 'IT', 'DEF', 99999),
(3, 'Paris', 'FR', 'GHI', 100001),
(4, 'Lima', 'PE', 'JKL', 101001);

SELECT * FROM EXAMPLE;

┌───────┬─────────┬─────────────┬──────────┬────────────┐
│  ID   │  NAME   │ COUNTRYCODE │ DISTRICT │ POPULATION │
│ int32 │ varchar │   varchar   │ varchar  │   int32    │
├───────┼─────────┼─────────────┼──────────┼────────────┤
│     1 │ Abruzzo │ IT          │ ABC      │     100000 │
│     2 │ Roma    │ IT          │ DEF      │      99999 │
│     3 │ Paris   │ FR          │ GHI      │     100001 │
│     4 │ Lima    │ PE          │ JKL      │     101001 │
└───────┴─────────┴─────────────┴──────────┴────────────┘

WITH name_lengths AS (
SELECT NAME, LENGTH(NAME) nameLength FROM EXAMPLE ORDER BY NAME
),
min_length AS (
SELECT NAME, nameLength FROM name_lengths WHERE nameLength = (SELECT MIN(nameLength) FROM name_lengths) LIMIT 1
),
max_length AS (
SELECT NAME, nameLength FROM name_lengths WHERE nameLength = (SELECT MAX(nameLength) FROM name_lengths) LIMIT 1
),
final_query AS (
SELECT * FROM min_length
UNION
SELECT * FROM max_length
)

SELECT * FROM final_query ORDER BY NAME;

┌─────────┬────────────┐
│  NAME   │ nameLength │
│ varchar │   int64    │
├─────────┼────────────┤
│ Abruzzo │          7 │
│ Lima    │          4 │
└─────────┴────────────┘
```

With inner Joins

```
CREATE TABLE EXAMPLE (ID INTEGER,NAME VARCHAR(17),COUNTRYCODE VARCHAR(3), DISTRICT VARCHAR(20), POPULATION INTEGER);

INSERT INTO EXAMPLE (ID, NAME, COUNTRYCODE, DISTRICT, POPULATION) VALUES
(1, 'Abruzzo', 'IT', 'ABC', 100000),
(2, 'Roma', 'IT', 'DEF', 99999),
(3, 'Paris', 'FR', 'GHI', 100001),
(4, 'Lima', 'PE', 'JKL', 101001);

SELECT * FROM EXAMPLE;

┌───────┬─────────┬─────────────┬──────────┬────────────┐
│  ID   │  NAME   │ COUNTRYCODE │ DISTRICT │ POPULATION │
│ int32 │ varchar │   varchar   │ varchar  │   int32    │
├───────┼─────────┼─────────────┼──────────┼────────────┤
│     1 │ Abruzzo │ IT          │ ABC      │     100000 │
│     2 │ Roma    │ IT          │ DEF      │      99999 │
│     3 │ Paris   │ FR          │ GHI      │     100001 │
│     4 │ Lima    │ PE          │ JKL      │     101001 │
└───────┴─────────┴─────────────┴──────────┴────────────┘

WITH name_lengths AS (
SELECT NAME, LENGTH(NAME) nameLength FROM EXAMPLE ORDER BY NAME
),
min_length AS (
SELECT NAME, nameLength FROM name_lengths WHERE nameLength = (SELECT MIN(nameLength) FROM name_lengths) LIMIT 1
),
max_length AS (
SELECT NAME, nameLength FROM name_lengths WHERE nameLength = (SELECT MAX(nameLength) FROM name_lengths) LIMIT 1
)

SELECT nl.NAME, nl.nameLength
FROM name_lengths nl
INNER JOIN min_length minl
ON minl.NAME = nl.NAME
UNION
SELECT nl.NAME, nl.nameLength
FROM name_lengths nl
INNER JOIN max_length maxl
ON maxl.NAME = nl.NAME;
