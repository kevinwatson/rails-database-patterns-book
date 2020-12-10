### Chapter 9 - MySQL Query Plans

## Introduction

Query plans are a way to find out what's happening under the hood. They explain how the rows are being retrieved and their relative cost. They can help us determine how to increase performance when retrieving data.

We'll review the query plans for MySQL. We'll dig into the nuances of the MySQL database platform and add queries to improve the performance of our queries.

## Definitions

* Explain Output
  * Select Type: The type of selection used by the optimizer to run the query. Output could include `simple`, `primary`, `subquery`, `derived`, `dependent subquery`, `uncacheable subquery`, `union`, `dependent union` and `union result`.
  * Table: The table that this line in the output refers to
  * Partitions: The matching partitions
  * Type: The join type. This field indicates how tables were joined, and will give clues on action that can be taken on how to optimize the query. The data in this field is one of the most important pieces of information that can help us optimize our queries and apply appropriate indexes.
  * Possible Keys: The indexes that are available to choose from
  * Key: The index that was actually chosen
  * Ken Len: The length of the chosen index or key
  * Ref: The columns or constants that are used in the key column
  * Rows: Estimated number of rows that will be examined
  * Filtered: The percentage of rows that are filtered by table definition. Low values indicate that the query may not be running optimally.
  * Extra: Additional information about the query execution plan

## Optimization

In general, when running a query that requests a majority of the rows in a table, the query processor may choose to run a table scan instead of an index scan. This is because it is cheaper to read the table into memory than it is to find the position of the requested rows in an index and then go to the table to get the actual rows. When the table contains more rows than can be loaded into memory, the following can be used as a reference to determine query efficiency. For complex queries, the items on the left are more efficient than the items on the right.

```console
Index scan > Table scan
```

## Setup

In the scripts below, we'll create a test database with a couple of tables. We'll seed those tables with data. We'll then run queries against that data, inspect the query plan, add indexes, and run the queries again. We'll compare the query plans to see how our queries were affected.

First, let's create a database

```sql
CREATE DATABASE query_plans;
```

Second, let's create a few tables. The first, `accounts`, will hold our user account information. The second, `posts`, will hold our user's messages. The third table, `words`, will be used to store a list of words that we'll load into the database from a file. We'll use this `words` table to populate the `posts` table with random data.

```sql
CREATE TABLE accounts (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL
);

CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  account_id BIGINT NOT NULL REFERENCES accounts (id),
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  comment TEXT NOT NULL
);

CREATE TABLE words (word TEXT);
```

Now let's populate the tables with some random text. We'll use a file called `words` installed on most Unix-based systems to populate the tables. We'll dump the contents of this file using the `mysqlimport` command to the `words` table, which we'll then use to generate random data for the `accounts` and `posts` tables. You may need to copy the `words` file to the `/var/lib/mysql-files/` directory depending on your MySQL configuration.

```console
mysqlimport --password --user=root --columns=word query_plans /var/lib/mysql-files/words
```

Now that the data has been imported into the `words` table, we'll use this data to generate data for the other tables.

The first step is to set the `group_concat_max_len` to a large value so we can use the `group_concat` function to generate some large post data.

```sql
SET group_concat_max_len = 15000;
```

In the next couple of steps we'll use MySQL procedures to use a `WHILE` loop to insert random data into the `accounts` table.

```sql
DELIMITER $$
CREATE PROCEDURE generate_account_data()
BEGIN
  DECLARE i INT DEFAULT 0;
  WHILE i < 1000 DO
    INSERT INTO accounts (name) VALUES (
      (SELECT group_concat(word separator ' ') FROM (SELECT word FROM words ORDER BY rand() limit 2) as words)      
    );
	SET i = i + 1;
  END WHILE;
END$$
DELIMITER ;
```

Let's call the newly created procedure to generate the account data.

```sql
CALL generate_account_data();
```

Here's the procedure we'll create to generate 10000 post records that will each contain between 1 and 1000 words.

```sql
DELIMITER $$
CREATE PROCEDURE generate_post_data()
BEGIN
  DECLARE i INT DEFAULT 0;
  DECLARE post_comment TEXT DEFAULT '';
  WHILE i < 10000 DO
    SET post_comment = "";
    INSERT INTO posts (account_id, created_at, comment) VALUES (
      FLOOR(RAND()*999+1),
      FROM_UNIXTIME(UNIX_TIMESTAMP('2020-01-01 01:00:00')+FLOOR(RAND()*31536000)),
      (SELECT group_concat(word separator ' ') FROM (SELECT word FROM words ORDER BY rand() limit 6) as words)      
    );
    SET i = i + 1;
  END WHILE;
END$$
DELIMITER ;
```

Now that the `generate_post_data` procedure has been created, let's call the procedure to populate the `posts` table.

```sql
CALL generate_post_data();
```

## Execution

### Without an Index

Let's run a series of queries and inspect the query plans of each. At this point, MySQL has automatically added unique indexes to our table on the primary key fields. The next query we run will filter on the `account_id` field, which is a foreign key and does not have an index.

```sql
EXPLAIN
SELECT *
FROM posts
WHERE account_id = 1;
```

`EXPLAIN` output

| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | SIMPLE | posts | NULL | ALL | NULL | NULL | NULL | NULL | 9788 | 10.00 | Using where |
```

Let's inspect the output.

Because there is only one table in the query, we get a single line in the `EXPLAIN` output. As we make our queries more complex, we can expect one row per table in the order that they're accessed via the query optimizer. The `NULL` fields will be omitted from our analysis.

* select_type: `SIMPLE` indicates that the query accessed a table directly without any unions or subqueries
* table: The `posts` table was accessed in this query
* type: `All` indicates that a table scan was performed to find the requested records
* rows: `9788` rows were examined to produce this output
* filtered: `10.00`% of the rows were filtered out to produce the result
* Extra: `Using where` indicates that the `where` statement was used to filter the rows. When the `type` value is `ALL` and `Using where` is in this column, the query is performing optimally.

### With an Index

```sql
CREATE INDEX idx_posts_account_id ON posts (account_id);
```

Now let's run the same query with `EXPLAIN` to get the new query plan.

```sql
EXPLAIN
SELECT *
FROM posts
WHERE account_id = 1;
```

`EXPLAIN` output

| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | SIMPLE | posts | NULL | ref | idx_posts_account_id | idx_posts_account_id | 4 | const | 15 | 100.00 | NULL |

This time the output is much different.

* select_type: `SIMPLE` indicates that the query accessed a table directly without any unions or subqueries
* table: The `posts` table was accessed in this query
* type: `ref` indicates that all rows retrieved were read from the index
* possible_keys: `idx_posts_account_id` is the index that is available but may not necessarily be used by the query optimizer
* key: `idx_posts_account_id` is the actual index that was used
* key_len: `4` tells us that 4 bytes were used by the index because the indexed column `account_id` is an `int` type
* ref: `const` indicates that a constant was used. In this case the integer `1` (a constant value) was used to match on values in the index.
* rows: `15` rows were examined in the index to produce this output
* filtered: `100.00`% indicates that none of the rows needed to be filtered out after using the `idx_posts_account_id` index

### Inner Joins

The next query we'll run will join two tables together. This join will be an inner join, which only includes the intersection of the records from both tables in the output.

```sql
EXPLAIN
SELECT *
FROM posts
INNER JOIN accounts ON accounts.id = posts.account_id
WHERE posts.account_id = 1;
```

`EXPLAIN ANALYZE` output

| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | SIMPLE | accounts | NULL | const | PRIMARY,id | PRIMARY | 8 | const | 1 | 100.00 | NULL |
| 1 | SIMPLE | posts | NULL | ref | idx_posts_account_id | idx_posts_account_id | 8 | const | 15 | 100.00 | NULL |

We see that because we joined on another table, there are two rows in the output. We'll examine each row.

Row 1

* select_type: `SIMPLE` indicates that the query accessed a table directly without any unions or subqueries
* table: The `accounts` table was accessed in this query
* type: `const` indicates that rows were filtered using equality on a constant value, in this case the value of `1` (which is matched from the join on the `posts` table)
* possible_keys: `PRIMARY,id` tells us that an index named `PRIMARY` exists on the primary key field and is a possible candidate
* key: `PRIMARY` is the actual index that was used. The `PRIMARY` index is a unique index that is automatically generated on all MySQL tables.
* key_len: `8` tells us that 8 bytes were used by the index because the indexed column `account_id` is a `bigint` type
* ref: `const` indicates that a constant was used. In this case the integer `1` (a constant value) was used to match on values in the index.
* rows: `1` row was examined in the index to produce this output
* filtered: `100.00`% indicates that none of the rows needed to be filtered out after using the `PRIMARY` index

Row 2

* select_type: `SIMPLE` indicates that the query accessed a table directly without any unions or subqueries
* table: The `posts` table was accessed in this query
* type: `ref` indicates that all rows retrieved were read from the index
* possible_keys: `idx_posts_account_id` tells us that an index named `idx_posts_account_id` exists on the primary key field and is a possible candidate for the query optimizer
* key: `idx_posts_account_id` is the actual index that was used to filter records
* rows: `15` rows were examined in the index to produce this output
* filtered: `100.00`% indicates that none of the rows needed to be filtered out after using the `idx_posts_account_id` index

Next, we'll see how adding a sort to the join changes the query plan.

```sql
EXPLAIN
SELECT *
FROM posts
INNER JOIN accounts ON accounts.id = posts.account_id
ORDER BY accounts.id;
```

`EXPLAIN` output

| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | SIMPLE | posts | NULL | ALL | idx_posts_account_id | NULL | NULL | NULL | 9853 | 100.00 | Using temporary; Using filesort |
| 1 | SIMPLE | accounts | NULL | eq_ref | PRIMARY,id | PRIMARY | 8 | query_plans.posts.account_id | 1 | 100.00 | Using where |

Row 1

* select_type: `SIMPLE` indicates that the query accessed a table directly without any unions or subqueries
* table: The `posts` table was accessed in this query
* type: `All` indicates that a table scan was performed to find the requested records
* possible_keys: `idx_posts_account_id` tells us that an index named `idx_posts_account_id` exists on the primary key field and is a possible candidate
* key: `NULL` tells us that no keys were used to retrieve rows from this table because no filter was defined in a `where` clause
* rows: `9853` rows were examined in the table scan to produce this output
* filtered: `100.00`% indicates that none of the rows needed to be filtered out after using the table scan
* Extra: `Using temporary; Using filesort` tells us that because such a large number of records needed to be sorted, a temporary table was used to perform the sort

Row 2

* select_type: `SIMPLE` indicates that the query accessed a table directly without any unions or subqueries
* table: The `accounts` table was accessed in this query
* type: `eq_ref` indicates that all rows retrieved were read from the index
* possible_keys: `PRIMARY,id` tells us that an index named `idx_posts_account_id` exists on the primary key field and is a possible candidate for the query optimizer
* key: `PRIMARY` is the actual index that was used to filter records
* key_len: `8` tells us that 8 bytes were used by the index because the indexed column `account_id` is a `bigint` type
* ref: `query_plans.posts.account_id` indicates that the `posts.account_id` field was used in the join
* rows: `1` row was examined in the index to produce this output
* filtered: `100.00`% indicates that none of the rows needed to be filtered out after using the `PRIMARY` index
* Extra: `Using where` tells us that a join was used to filter rows on the `accounts` table to generate our output

## Resources

* https://dev.mysql.com/doc/refman/8.0/en/explain-extended.html
* https://dev.mysql.com/doc/refman/8.0/en/explain-output.html
* https://dev.mysql.com/doc/refman/8.0/en/storage-requirements.html
* https://www.sitepoint.com/using-explain-to-write-better-mysql-queries
* https://stackoverflow.com/q/25098747
* https://dba.stackexchange.com/a/164360

# Wrap-up

I once worked on a Rails app that was backed by a MySQL database. There was a report that was used to look back at prior months which would join several large tables and aggregate some of the data. It would take nearly 15 minutes to run and needed to be run with different inputs several times a day on the first of the month. The users asked if there was anything we could do to speed it up. I looked at the queries that the report was running, ran an `EXPLAIN` on the query and found that there were several foreign keys that did not have indexes. Once I created a migration and added an index, the query went from taking 15 minutes down to less than a second. It was so much faster that the users suspected that I modified the query and the output was no longer trustworthy. I reassured them that the query and data were the same, all that changed was that the report accessed the data via an index, instead of running a much slower table scan.

MySQL query plans are a great resources to help you optimize your queries as your tables grow. Running a query plan and inspecting the output on the `key`, `ref`, `rows`, `rows` and `Extra` fields can help you determine where to apply indexes which can make queries that take minutes down to milliseconds.
