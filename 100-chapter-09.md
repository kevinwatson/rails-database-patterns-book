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
  account_id INTEGER NOT NULL REFERENCES accounts (id),
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
* key_len: `4` tells us that the 4 bytes were used by the index because the indexed column `account_id` is an `int` type
* ref: `const`
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

```console

```

```sql
EXPLAIN
SELECT *
FROM posts
INNER JOIN accounts ON accounts.id = posts.account_id;
```

`EXPLAIN` output

```console

```

```sql
EXPLAIN
SELECT *
FROM posts
INNER JOIN accounts ON accounts.id = posts.account_id
ORDER BY accounts.id;
```

`EXPLAIN` output

```console

```

```sql
EXPLAIN
SELECT *
FROM posts
INNER JOIN accounts ON accounts.id = posts.account_id
ORDER BY accounts.id, posts.id;
```

`EXPLAIN` output

```console

```

## Resources

* https://dev.mysql.com/doc/refman/8.0/en/explain-extended.html
* https://dev.mysql.com/doc/refman/8.0/en/explain-output.html
* https://dev.mysql.com/doc/refman/8.0/en/storage-requirements.html
* https://www.sitepoint.com/using-explain-to-write-better-mysql-queries
* https://stackoverflow.com/q/25098747
* https://dba.stackexchange.com/a/164360
