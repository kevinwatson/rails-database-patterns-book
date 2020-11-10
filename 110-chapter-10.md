### Chapter 10 - Arel

## Introduction

ActiveRecord uses a library named Arel to define relationship behaviors between models and build queries to interact with the database. Arel is a relational algebra library which provides a domain specific language (DSL) to help developers write database agnostic code.

## The Arel API

Let's imagine we have a User model with a corresponding users table in the database. We'll compose simple Arel queries that can be used to construct larger queries to retrieve data from this table.

The table we'll use in our example will consist of the following structure:

```sql
CREATE TABLE users (email varchar(1000), created_at datetime, login_count int)
```

To gain access to the instance of the predefined Arel table, we can get a reference via the model's `arel_table` method. We'll use the following `usr` variable to generate some of our queries.

```ruby
usr = User.arel_table
```

The (nearly) full list of public Arel methods with examples are included in the table below.

| Description | Arel method | Arel example | SQL | Links|
|---|---|---|---|---|
| Not equal | not_eq | usr[:email].not_eq('t@g.com') | users.email != 't@g.com' |
| Not equal to any | not_eq_any | usr[:email].not_eq_any(['t@g.com','a@b.com']) | (users.email != 't@g.com' OR users.email != 'a@b.com') |
| Not equal to all | not_eq_all | usr[:email].not_eq_all(['t@g.com','a@b.com']) | (users.email != 't@g.com' AND users.email != 'a@b.com') |
| Equal | eq | usr[:email].eq('t@g.com') | users.email = 't@g.com' |
| Is not Distinct From | is_not_distinct_from | usr[:email].is_not_distinct_from('t@g.com') | users.email <=> 't@g.com' | [PostgreSQL](https://wiki.postgresql.org/wiki/Is_distinct_from) |
| Is Distinct From | is_distinct_from | usr[:email].is_distinct_from('t@g.com') | NOT users.email <=> 't@g.com' | [PostgreSQL](https://wiki.postgresql.org/wiki/Is_distinct_from) |
| Equals Any | eq_any | usr[:email].eq_any(['a@b.com', 'b@b.com']) | (users.email = 'a@b.com' OR users.email = 'b@b.com') |
| Equals All | eq_all | usr[:email].eq_all(['a@b.com', 'b@b.com']) | (users.email = 'a@b.com' AND users.email = 'b@b.com') |
| Between | between | usr[:created_at].between(Date.today..Date.tomorrow) | users.created_at BETWEEN '2020-11-04' AND '2020-11-05' |
| In | in | usr[:email].in(['a@b.com', 'b@b.com']) | users.email IN ('a@b.com', 'b@b.com') |
| In Any | in_any | usr[:email].in_any(['a@b.com', 'b@b.com']) | (users.email IN ('a@b.com') OR users.email IN ('b@b.com')) |
| In All | in_all | usr[:email].in_all(['a@b.com', 'b@b.com']) | (users.email IN ('a@b.com') AND users.email IN ('b@b.com')) |
| Not Between | not_between | usr[:created_at].not_between(Date.today..Date.tomorrow) | (users.created_at < '2020-11-04' OR users.created_at > '2020-11-05') |
| Not In | not_in | usr[:created_at].not_in(Date.today..Date.tomorrow) | (users.created_at < '2020-11-04' OR users.created_at > '2020-11-05') |
| Not In Any | not_in_any | usr[:created_at].not_in_any(Date.today..Date.tomorrow) | (users.created_at NOT IN ('2020-11-04') OR users.created_at NOT IN ('2020-11-05')) |
| Not In All | not_in_all | usr[:created_at].not_in_all(Date.today..Date.tomorrow) | (users.created_at NOT IN ('2020-11-04') AND users.created_at NOT IN ('2020-11-05')) |
| Match Wildcard | matches | usr[:email].matches('%a.com') | users.email LIKE '%a.com' |
| Match Regexp | matches_regexp | usr[:email].matches_regexp(/a.com$/) | TODO: NotImplementedError (~ not implemented for this db) |
| Match Any Wildcard | matches_any | usr[:email].matches_any(['%a.com','%b.com']) | (users.email LIKE '%a.com' OR users.email LIKE '%b.com') |
| Match All | matches_all | usr[:email].matches_all(['%a.com','%b.com']) | (users.email LIKE '%a.com' AND users.email LIKE '%b.com') |
| Does Not Match Wildcard (Not Like) | does_not_match | usr[:email].does_not_match('%a.com') | users.email NOT LIKE '%a.com' |
| Does Not Match Regexp | does_not_match_regexp | usr[:email].does_not_match_regexp(/a.com$/) | TODO: NotImplementedError (!~ not implemented for this db) |
| Does Not Match Any | does_not_match_any | usr[:email].does_not_match_any(['%a.com','%b.com']) | (users.email NOT LIKE '%a.com' OR users.email NOT LIKE '%b.com') |
| Does Not Match All | does_not_match_all | usr[:email].does_not_match_all(['%a.com','%b.com']) | (users.email NOT LIKE '%a.com' AND users.email NOT LIKE '%b.com') |
| Greater Than or Equal To | gteq | usr[:created_at].gteq(Date.yesterday) | users.created_at >= '2020-11-03' |
| Greater Than or Equal To Any | gteq_any | usr[:created_at].gteq_any(Date.yesterday..Date.today) | (users.created_at >= '2020-11-03' OR users.created_at >= '2020-11-04') |
| Greater Than or Equal To All | gteq_all | usr[:created_at].gteq_all(Date.yesterday..Date.today) | (users.created_at >= '2020-11-03' AND users.created_at >= '2020-11-04') |
| Greater Than | gt | usr[:created_at].gt(Date.yesterday) | users.created_at > '2020-11-03' |
| Greater Than Any | gt_any | usr[:created_at].gt_any(Date.yesterday..Date.today) | (users.created_at > '2020-11-03' OR users.created_at > '2020-11-04') |
| Greater than All | gt_all | usr[:created_at].gt_all(Date.yesterday..Date.today) | (users.created_at > '2020-11-03' AND users.created_at > '2020-11-04') |
| Less Than | lt | usr[:created_at].lt(Date.yesterday) | users.created_at < '2020-11-04' |
| Less Than Any | lt_any | usr[:created_at].lt_any(Date.yesterday..Date.today) | (users.created_at < '2020-11-04' OR users.created_at < '2020-11-05') |
| Less Than All | lt_all | usr[:created_at].lt_all(Date.yesterday..Date.today) | (users.created_at < '2020-11-04' AND users.created_at < '2020-11-05') |
| Less Than or Equal To | lteq | usr[:created_at].lteq(Date.today) | users.created_at <= '2020-11-05' |
| Less Than or Equal To Any | lteq_any | usr[:created_at].lteq_any(Date.yesterday..Date.today) | (users.created_at <= '2020-11-04' OR users.created_at <= '2020-11-05') |
| Less Than or Equal To All | lteq_all | usr[:created_at].lteq_all(Date.yesterday..Date.today) | (users.created_at <= '2020-11-04' AND users.created_at <= '2020-11-05') |
| Case When Then | when | Arel::Nodes::Case.new(usr[:email])<br>.when('a@b.com')<br>.then('a@b.org')<br>.when('b@b.com')<br>.then('b@b.org') | CASE users.email WHEN 'a@b.com' THEN 'a@b.org' WHEN 'b@b.com' THEN 'b@b.org' END |
| Concatenation | concat | 
| | contains
| | overlaps
| | quoted_array
| | count
| | sum
| | maximum
| | minimum
| | average
| | extract(field)
| Math Multiplication | *
| Math Addition | +
| Math Subtraction | -
| Math Division | /
| Bitwise AND | &
| Bitwise OR | \|
| Bitwise XOR | ^
| Bitwise Shift Left | <<
| Bitwise Shift Right | >>
| Bitwise NOT | ~@
| Alias | as
| | lower
| | coalesce
| Order Ascending | asc
| Order Descending | desc
| Limit | limit
| Taken | taken (alias for as limit)
| | constraints
| | offset
| | skip
| | exists
| | lock
| | on
| | group
| | from
| | froms
| | join (notice that this is different than joins)
| | outer_join
| | having
| | window
| | project
| | projections
| | optimizer_hints
| | distinct
| | distinct_on
| | order
| | orders
| | where_sql
| | where
| | union
| | intersect
| | except
| | lateral
| | with
| | take
| | join_sources
| | source
| | comment
| | over

## References

* https://devhints.io/arel
* https://github.com/DavyJonesLocker/postgres_ext/blob/master/docs/querying.md
* https://github.com/rails/rails/tree/master/activerecord/lib/arel
* https://thoughtbot.com/blog/using-arel-to-compose-sql-queries
* https://www.rubydoc.info/gems/arel/7.1.1/Arel/Nodes/Concat

