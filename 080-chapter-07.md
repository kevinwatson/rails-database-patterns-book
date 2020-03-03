### Chapter 7 - Query Plans

## Introduction

Query plans are a way to find out what's happening under the hood. They explain how the rows are being retrieved and their relative cost. They can help us determine how to increase performance when retrieving data.

We'll review the query plans from both MySQL and PostgreSQL. We'll dig into the nuances of each platform and add queries to improve the performance of our queries.

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

Let's run a series of queries and inspect the query plans of each. At this point, PostgreSQL has automatically added unique indexes to our table on the primary key. The next query we run will filter on the `account_id` field, which is a foreign key and does not yet have an index.

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

Let's inspect this output.

Line 1 Shows that a `sequential scan` was performed on the table. Sequential scans read the table from the beginning to the end, like picking up and thumbing through a book from the beginning to get to the page you're interested in. This line also shows us that we have a cost of `4503.25`. This value is relative and doesn't mean much in the context of a single query. We'll use it later to compare to other queries. We also learn from this line that 506 rows were found that match our filter.

Line 2 Shows our filter

Line 3 Shows how many rows were read and discarded because they didn't match our filter

Line 4 Shows how much time was spent by the query planner deciding what would be the quickest way to execute the query

Line 5 Shows how much time was spent running the query and generating the output

## Resources

* https://www.postgresql.org/docs/9.5/using-explain.html
