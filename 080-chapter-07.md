### Chapter 7 - PostgreSQL Query Plans

## Introduction

Query plans are a way to find out what's happening under the hood. They explain how the rows are being retrieved and their relative cost. They can help us determine how to increase performance when retrieving data.

We'll review the query plans from both MySQL and PostgreSQL. We'll dig into the nuances of each platform and add queries to improve the performance of our queries.

## Definitions

* B-tree Index: A self-balancing tree data structure. The default and most common type of index in PostgreSQL.
* Bitmap: A temporary data structure that is populated in memory while running a query.
* Bitmap Heap Scan: A 'bitmap index scan' was used to find all of the matching records, the index results are sorted using an in-memory 'bitmap' data structure, and then the records are retrieved from the table. More efficient than an 'Index Scan' for retrieving larger sets of data.
* Bitmap Index: An index that is encoded using bitmaps (e.g. 1s and 0s) for fast data comparison.
* Bitmap Index Scan: An index was used and the results are parsed and store in an in-memory bitmap structure.
* BitmapAnd: The intersection of the results from two bitmap index scans are returned.
* BitmapOr: Similar to BitmapAnd, the union of the results from two bitmap index scans are returned.
* Block: Also known as a page, a block is an 8KB chunk of disk space where the permanent data is stored. Before the data can be read or modified it is loaded into a bitmap.
* Buffers: Identify which parts of the query are the most input/output intensive.
* Exact: The opposite of lossy, when the bitmap is within the constraints of the `work_mem` setting.
* Explain: A breakdown of how the query will be executed, based on metrics and statistics that may not be up to date.
* Explain Analyze: Runs the query and displays a breakdown of how the query was executed.
* GiST Index (Generalized Search Tree): A class of index types that support B-trees, R-trees and other indexing schemes. Supports custom data types.
* GIN Index (Generalized Inverted Index): A type of index that supports composite values, such as documents. Implemented as a set of key-value pairs.
* Hash Index: A type of index that is designed for simple equality comparisons. Creating Hash indexes is generally discouraged because of several limitations.
* Hash Join: The first table is scanned and loaded into a hash table. The second table is scanned and matching attributes are loaded into a hash table. 
* Heap: The structure used to store the table data on disk.
* Heap Blocks: Eight kilobyte (8KB) blocks of disk space where the table data is stored.
* High-cardinality data: Data that is mostly unique.
* Index: A small copy of the data in the table, usually containing only one or two fields that can be used to find the data more quickly than scanning the original table.
* Index Only Scan: The results are returned from the index only. The table is not scanned. More efficient than an index scan.
* Index Scan: The index is scanned, and as each item in the index is found, the matching table record is added to the output (if data is needed from the table). Most efficient for retrieving small sets of data.
* Lossy: When a bitmap becomes too large (larger than the available `work_mem`), the processor? is no longer tracking the individual tuples, but starts storing the bitmap that will be used later to rescan for the specific tuples.
* Merge Join: Each table is sorted on the joined attributes, the two tables are scanned and matching rows are combined. Faster than a Nested Loop.
* Nested Loop: The default join algorithm. Slower than an index join.
* Parallel Seq Scan (parallel sequential scan): Two workers work on the filter condition at the same time.
* Page: Also known as a block, a page is an 8KB chunk of disk space where the permanent data is stored. Before the data can be read or modified it is loaded into a bitmap.
* Recheck Cond (recheck condition): When the bitmap is larger than the `work_mem` setting, it is converted to a 'lossy' style. When this happens, only the pages are tracked instead of the individual tuples. The table-visiting phase has to examine each tuple on the page and recheck the scan condition to see which tuples to return.
* Seq Scan (sequential scan): Each row in a table is inspected in the order the records are stored in the table. The default method for retrieving rows from a table, it's also the most efficient way to retrieve records for a query that has no filters or a query that has a filter that will return most of the records in a table.
* SP-GiST Index (Space-Partitioned GiST Index): Supports partitioned search trees which map in-memory nodes to disk pages so that a search is optimized and only needs access to a few disk pages.

## Optimization

In general, when running a query that requests a majority of the rows in a table, the query processor may choose to run a sequential scan instead of an index scan. This is because it is cheaper to read the table into memory than it is to find the position of the requested rows in an index and then go to the table to get the actual rows. When the table contains more rows than can be loaded into memory, the following can be used as a reference to determine query efficiency. For complex queries, the items on the left are more efficient than the items on the right.

```console
Index Only Scan > Index Scan > Bitmap Index Scan > Bitmap Heap Scan > Sequential Scan
```

When the query will return most of the records in a table, a sequential scan is the most efficient.

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
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  comment TEXT NOT NULL
);

CREATE TABLE words (word TEXT);
```

Now let's populate the tables with some random text. We'll use a file called `words` installed on most Unix-based systems to populate the tables.

```sql
\copy words (word) FROM '/usr/share/dict/words';

INSERT INTO accounts (name)
SELECT
  ( SELECT string_agg(word, ' ') FROM (TABLE words ORDER BY random() * n LIMIT 2) AS words (word) )
FROM generate_series(1, 100) AS s(n);

INSERT INTO posts (account_id, created_at, comment)
SELECT
  RANDOM() * 99 + 1,
  NOW() - ('1 days'::INTERVAL * random() * 1000),
  ( SELECT string_agg(word, ' ') FROM (TABLE words ORDER BY random() * n LIMIT 20) AS words (word) )
FROM generate_series(1, 100000) AS s(n);
```

## Execution

The first thing we'll want to do is run a command to optimize the database and provide the most efficient execution plans for our queries.

```console
VACUUM ANALYZE;
```

If we were to skip this command, our planning time and execution time estimates would be inflated.

### Without an Index

Let's run a series of queries and inspect the query plans of each. At this point, PostgreSQL has automatically added unique indexes to our table on the primary key. The next query we run will filter on the `account_id` field, which is a foreign key and does not have an index.

```sql
EXPLAIN ANALYZE
SELECT *
FROM posts
WHERE account_id = 1;
```

`EXPLAIN ANALYZE` output

```console
1 Seq Scan on posts  (cost=0.00..4503.25 rows=504 width=230) (actual time=0.950..16.560 rows=506 loops=1)
2   Filter: (account_id = 1)
3   Rows Removed by Filter: 99494
4 Planning Time: 1.663 ms
5 Execution Time: 20.904 ms
```

Let's inspect the output.

Line 1 Shows that a `sequential scan` was performed on the table. Sequential scans read the table from the beginning to the end, like picking up and thumbing through a book from the beginning to get to the page you're interested in. This line also shows us that we have a cost of `4503.25`. This value is relative and doesn't mean much in the context of a single query. We'll use it later when we compare it to other queries. We also learn from this line that 506 rows were found that match our filter.

Line 2 Shows our filter - `account_id = 1`

Line 3 Shows how many rows were read and discarded because they didn't match our filter - `99494`

Line 4 Shows how much time was spent by the query planner deciding what would be the quickest path to executing the query - `1.663 milliseconds`

Line 5 Shows how much time was spent running the query and generating the output - `20.904 milliseconds`

### With an Index

Sequential scans are the slowest way to filter the rows on a table. Let's add an index and run the query again.

```sql
CREATE INDEX ON posts (account_id);
```

Now let's run the same query with `EXPLAIN ANALYZE` to get the new query plan.

```sql
EXPLAIN ANALYZE
SELECT *
FROM posts
WHERE account_id = 1;
```

`EXPLAIN ANALYZE` output

```console
1 Bitmap Heap Scan on posts  (cost=12.32..1355.69 rows=503 width=230) (actual time=0.868..1.598 rows=506 loops=1)
2   Recheck Cond: (account_id = 1)
3   Heap Blocks: exact=465
4   ->  Bitmap Index Scan on posts_account_id_idx  (cost=0.00..12.19 rows=503 width=0) (actual time=0.814..0.814 rows=506 loops=1)
5         Index Cond: (account_id = 1)
6 Planning Time: 7.757 ms
7 Execution Time: 1.861 ms
```

This time the output is much different.

Line 1 Shows that a `Bitma Heap scan` was performed on the table. This line also shows us that we have a cost of `1355.69`. This value is much smaller than the original `4503.25` we got when we ran the query without the index on the `account_id` field. We also learn from this line that 506 rows were found that match our filter.

Line 2 Shows that a recheck condition occurred - the bitmap that was built during processing grew larger than the current settings allow, so pages were returned instead of individual tuples. The processor will need to `recheck` the final pages to find the individual tuples from those pages. Also included on this line is the filter that was used for the `recheck` - `account_id = 1`

Line 3 Shows that the planner visited 465 blocks. Exact means that it was not lossy (the individual tuples were tracked).

Line 4 Shows that a bitmap index scan used index `posts_account_id_idx` and returned 506 rows.

Line 5 Shows the filter condition that was used in the index: `account_id = 1`

Line 6 Shows how much time was spent by the query planner deciding what would be the quickest path to executing the query - `7.757 milliseconds`

Line 7 Shows how much time was spent running the query and generating the output - `1.861 milliseconds`. In this case the planner took more time that it did to actually run the query.

### Inner Joins

The next query we'll run will join two tables together. This join will be an inner join, which only includes the intersection of the records from both tables in the output.

```sql
EXPLAIN ANALYZE
SELECT *
FROM posts
INNER JOIN accounts ON accounts.id = posts.account_id
WHERE posts.account_id = 1;
```

`EXPLAIN ANALYZE` output

```console
1  Nested Loop  (cost=12.32..1362.97 rows=503 width=254) (actual time=1.085..1.674 rows=506 loops=1)
2    ->  Seq Scan on accounts  (cost=0.00..2.25 rows=1 width=24) (actual time=0.466..0.473 rows=1 loops=1)
3          Filter: (id = 1)
4          Rows Removed by Filter: 99
5    ->  Bitmap Heap Scan on posts  (cost=12.32..1355.69 rows=503 width=230) (actual time=0.126..0.600 rows=506 loops=1)
6          Recheck Cond: (account_id = 1)
7          Heap Blocks: exact=465
8          ->  Bitmap Index Scan on posts_account_id_idx  (cost=0.00..12.19 rows=503 width=0) (actual time=0.069..0.069 rows=506 loops=1)
9                Index Cond: (account_id = 1)
10 Planning Time: 54.826 ms
11 Execution Time: 2.259 ms
```

The output from this query differs from the previous queries in that line 1 shows that the planner used a Nested Loop to filter out the records that do not match the `accounts.id = posts.account_id` condition. Nested Loops are not usually the most efficient join algorithm, but because the planner is running a Seq Scan on the accounts table (most likely because its a small table), the Nested Loop was determined to be the most efficient method to find this data. let's run a query that will use the Hash Join algorithm.

```sql
EXPLAIN ANALYZE
SELECT *
FROM posts
INNER JOIN accounts ON accounts.id = posts.account_id;
```

`EXPLAIN ANALYZE` output

```console
1 Hash Join  (cost=3.25..4528.88 rows=100000 width=254) (actual time=1.098..46.099 rows=100000 loops=1)
2   Hash Cond: (posts.account_id = accounts.id)
3   ->  Seq Scan on posts  (cost=0.00..4252.00 rows=100000 width=230) (actual time=0.582..14.067 rows=100000 loops=1)
4   ->  Hash  (cost=2.00..2.00 rows=100 width=24) (actual time=0.507..0.507 rows=100 loops=1)
5         Buckets: 1024  Batches: 1  Memory Usage: 14kB
6         ->  Seq Scan on accounts  (cost=0.00..2.00 rows=100 width=24) (actual time=0.484..0.491 rows=100 loops=1)
7 Planning Time: 2.558 ms
8 Execution Time: 50.571 ms
```

In this particular case, lines 1 and 2 show that we are using hashes to build the query. Removing the extra filter in the where clause `WHERE posts.account_id = 1` performs the query more optimially, but may return more records than we want.

## Resources

* https://www.postgresql.org/docs/9.5/planner-optimizer.html
* https://www.postgresql.org/docs/9.5/using-explain.html
* https://www.postgresql.org/message-id/12553.1135634231@sss.pgh.pa.us
* https://www.postgresql.org/docs/9.2/spgist-intro.html
