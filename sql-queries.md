A nice overview of several SQL queries and operations can be found here: [SQL Queries](https://learnsql.com/blog/sql-basics-cheat-sheet/sql-basics-cheat-sheet-a3.pdf)

# Structuring SQL Queries

## `WITH` Define temporary data tables in advance

```sql
WITH temp_table AS (
    SELECT
        column1,
        column2
    FROM
        table
    WHERE
        column3 = 'value'
)[, xxx as (SELECT ...)]
SELECT ...
```

## Create Views to structure Data

```sql
CREATE VIEW view_name AS
    SELECT column1, column2
    FROM table_name
    WHERE condition;
```
Views are accessible like tables and can be used in queries.

# Operations

## Select latest record for e.g. account

Sometimes state changes are recorded in a table and you want to select the latest record for each account.

```sql
SELECT DISTINCT ON (account_id) *
FROM history_table
order by account_id, history_table.id desc
```
`desc` sorting provides the latest record for each account_id.
`asc` sorting provides the oldest record for each account_id.


## `VALUES` Provide Data in Query

```sql
SELECT *
   FROM ( VALUES ('xxx'), ('yyy'), ('zzz') ) [AS table_name(column_name)]

```

## Execute Queries in Transactions

If you manipulate data with sql, there is normally no `Crtl-Z` to undo your changes. 
To avoid data corruption, you can execute your queries in a transaction.
If you are not happy with the result, you can rollback the transaction.

```sql
BEGIN;
    -- your queries
ROLLBACK; -- or COMMIT
```

## `UNION` Combine Results

```sql
SELECT column1, column2
FROM table1
UNION
SELECT column1, column2
FROM table2
```
Duplicates are removed by default. Use `UNION ALL` to keep duplicates.

# Analyse Queries

## `EXPLAIN` Theoretical Analyse of Query

```sql
EXPLAIN 
delete from table
where id='x';

```

## `EXPLAIN ANALYZE` Practical Analyse of Query
    
```sql
BEGIN;
EXPLAIN ANALYSE 
delete from table
where id='x';
ROLLBACK;

```
Query is executed and would alter data. Use `ROLLBACK` to undo changes.