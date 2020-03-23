### Chapter 7 - PostgreSQL Query Plans

## Introduction

Query plans are a way to find out what's happening under the hood. They explain how the rows are being retrieved and their relative cost. They can help us determine how to increase performance when retrieving data.

We'll review the query plans from both MySQL and PostgreSQL. We'll dig into the nuances of each platform and add queries to improve the performance of our queries.

## Definitions

* B-tree Index:
* Bitmap: A temporary data structure that is populated in memory while running a query
* Bitmap Heap Scan:
* Bitmap Index: An index that is encoded using bitmaps (e.g. 1s and 0s) for fast data comparison
* Bitmap Index Scan:
* BitmapAnd: 
* Block: Also known as a page, a block is an 8KB chunk of disk space where the permanent data is stored. Before the data can be read or modified it is loaded into a bitmap.
* Buffers: 
* Exact: The opposite of lossy, when the bitmap is within the constraints of the `work_mem` setting
* Explain: A breakdown of how the query will be executed, based on metrics and statistics that may not be up to date
* Explain Analyze: Runs the query and displays a breakdown of how the query was executed
* GiST Index:
* GIN Index:
* Hash Index:
* Heap: 
* Heap Blocks: 
* Index: A small copy of the data in the table, usually containing only one or two fields that can be used to find the data more quickly than scanning the original table
* Index Only Scan: The results are returned from the index, the table is not scanned
* Index Scan: The results are returned from both the index and the table
* High-cardinality data: Data that is mostly unique
* Lossy: When a bitmap becomes too large (larger than the available `work_mem`), the processor? is no longer tracking the individual tuples, but starts storing the bitmap that will be used later to rescan for the specific tuples.
* Parallel Seq Scan (parallel sequential scan): Two workers work on the filter condition at the same time
* Page: Also known as a block, a page is an 8KB chunk of disk space where the permanent data is stored. Before the data can be read or modified it is loaded into a bitmap.
* Recheck Cond (recheck condition): When the bitmap is larger than the `work_mem` setting, it is converted to a 'lossy' style. When this happens, only the pages are tracked instead of the individual tuples. The table-visiting phase has to examine each tuple on the page and recheck the scan condition to see which tuples to return
* Seq Scan (sequential scan): Each row in a table is inspected in the order the records are stored in the table
* SP-GiST Index:

## Optimization

In general, when running a query that requests a majority of the rows in a table, the query processor may choose to run a sequential scan instead of an index scan. This is because it is cheaper to read the table into memory than it is to find the position of the requested rows in an index and then go to the table to get the actual rows. When the table contains more rows than can be loaded into memory, the following diagram can be used as a reference to determine how efficient the query will run. For complex queries, the items on the left are more efficient than the items on the right.

```console
Index Scan > Bitmap Index Scan > Bitmap Heap Scan > Sequential Scan
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

This time the output is drastically different.

Line 1 Shows that a `Bitma Heap scan` was performed on the table. Bitmap heap scans . This line also shows us that we have a cost of `1355.69`. This value is much smaller than the original `4503.25` we got when we ran the query without the index on the `account_id` field. We also learn from this line that 506 rows were found that match our filter.

Line 2 Shows that a recheck condition occurred - the bitmap that was built during processing grew larger than the current settings allow, so pages were returned instead of individual tuples. The processor will need to `recheck` the final pages to find the individual tuples from those pages. Also included on this line is the filter that was used for the `recheck` - `account_id = 1`

Line 3 Shows that the processor? visited 465 blocks. Exact means that it was not lossy (the individual tuples were tracked).

Line 4 Shows 

Line 5 Shows 

Line 6 Shows 

Line 7 Shows 

## Resources

* https://www.postgresql.org/docs/9.5/using-explain.html
