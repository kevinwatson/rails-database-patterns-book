### Chapter 10 - Arel

## Introduction

ActiveRecord implements a library named Arel to define relationship behaviors between models and build queries to interact with the database. Arel provides a domain specific language (DSL) to help developers write database agnostic code.

This API allows Rails developers to write Ruby code to query and modify the database without knowing the specific implementation details of each SQL database (SQLite, MySQL, PostgreSQL, etc). Because of Arel, we can write small code fragments that can be reused in our code base. As an example, `User.arel_table[:created_at].between(Date.today..Date.tomorrow)` could be wrapped in a method in the `User` model and reused in a scope or wherever it is needed. This helps us DRY up our code when we can combine small fragments into larger, more complicated statements which are still easy for developers to read.

Most of the time, Rails developers will build their queries and CRUD statements using the ActiveRecord::Relation API. This API provides the standard `where`, `joins`, `or` and other related methods for a model. Under the hood, ActiveRecord is using the Arel API to track and build the associations, filters and related data to eventually build the SQL statements that are needed to interact with the database.

The ActiveRecord API is written with a general use case in mind, and does not provide all of the functionality that may be needed at times. For example, a developer may need to find records that were created after a specific date. The ActiveRecord API doesn't provide a 'greater than' clause. The developer is left to either use a combination of the ActiveRecord API and a string, or by using a combination of the ActiveRecord and Arel APIs. An implementation using ActiveRecord and a string could be `User.where("created_at >= ?", Date.today.midnight)` while an ActiveRecord and Arel implementation might look like `User.where(User.arel_table[:created_at].gt(Date.today.midnight))`. While the ActiveRecord/Arel combination might be more verbose, it is database agnostic and the Arel code could be extracted out into it's own reusable method.

As a rule of thumb, developers should use the methods provided by the ActiveRecord::Relation API to generate their queries, and when a developer needs to generate a specific database query that the ActiveRecord API doesn't support, the developer should then methods provided by the Arel API.

## The Arel API

Let's imagine we have a User model with a corresponding users table in the database. We'll compose simple Arel queries that can be used to construct larger queries to retrieve data from this table.

The tables we'll use in our example will have of the following structure:

```sql
CREATE TABLE users (id integer, email varchar, created_at datetime, login_count integer)
CREATE TABLE posts (id integer, user_id integer, comment varchar)
```

To gain access to the instance of the predefined Arel table, we can get a reference via the model's `arel_table` method. We'll use the following `users` and `posts` variables to generate some of our queries.

```ruby
users = User.arel_table
posts = Post.arel_table
```

The (nearly) full list of public Arel methods with examples are included in the table below.

| Description | Arel method | Arel example | SQL | Links|
|---|---|---|---|---|
| Not equal | not_eq | users[:email].not_eq('t@g.com') | users.email != 't@g.com' |
| Not equal to any | not_eq_any | users[:email].not_eq_any(['t@g.com','a@b.com']) | (users.email != 't@g.com' OR users.email != 'a@b.com') |
| Not equal to all | not_eq_all | users[:email].not_eq_all(['t@g.com','a@b.com']) | (users.email != 't@g.com' AND users.email != 'a@b.com') |
| Equals | eq | users[:email].eq('t@g.com') | users.email = 't@g.com' |
| Is not Distinct From | is_not_distinct_from | users[:email].is_not_distinct_from('t@g.com') | users.email <=> 't@g.com' | [PostgreSQL](https://wiki.postgresql.org/wiki/Is_distinct_from) |
| Is Distinct From | is_distinct_from | users[:email].is_distinct_from('t@g.com') | NOT users.email <=> 't@g.com' | [PostgreSQL](https://wiki.postgresql.org/wiki/Is_distinct_from) |
| Equals Any | eq_any | users[:email].eq_any(['a@b.com', 'b@b.com']) | (users.email = 'a@b.com' OR users.email = 'b@b.com') |
| Equals All | eq_all | users[:email].eq_all(['a@b.com', 'b@b.com']) | (users.email = 'a@b.com' AND users.email = 'b@b.com') |
| Between | between | users[:created_at].between(Date.today..Date.tomorrow) | users.created_at BETWEEN '2020-11-04' AND '2020-11-05' |
| In | in | users[:email].in(['a@b.com', 'b@b.com']) | users.email IN ('a@b.com', 'b@b.com') |
| In Any | in_any | users[:email].in_any(['a@b.com', 'b@b.com']) | (users.email IN ('a@b.com') OR users.email IN ('b@b.com')) |
| In All | in_all | users[:email].in_all(['a@b.com', 'b@b.com']) | (users.email IN ('a@b.com') AND users.email IN ('b@b.com')) |
| Not Between | not_between | users[:created_at].not_between(Date.today..Date.tomorrow) | (users.created_at < '2020-11-04' OR users.created_at > '2020-11-05') |
| Not In | not_in | users[:created_at].not_in(Date.today..Date.tomorrow) | (users.created_at < '2020-11-04' OR users.created_at > '2020-11-05') |
| Not In Any | not_in_any | users[:created_at].not_in_any(Date.today..Date.tomorrow) | (users.created_at NOT IN ('2020-11-04') OR users.created_at NOT IN ('2020-11-05')) |
| Not In All | not_in_all | users[:created_at].not_in_all(Date.today..Date.tomorrow) | (users.created_at NOT IN ('2020-11-04') AND users.created_at NOT IN ('2020-11-05')) |
| Matches Wildcard | matches | users[:email].matches('%a.com') | users.email LIKE '%a.com' |
| Matches Regexp | matches_regexp | users[:email].matches_regexp(/a.com$/) | |
| Matches Any Wildcard | matches_any | users[:email].matches_any(['%a.com','%b.com']) | (users.email LIKE '%a.com' OR users.email LIKE '%b.com') |
| Matches All | matches_all | users[:email].matches_all(['%a.com','%b.com']) | (users.email LIKE '%a.com' AND users.email LIKE '%b.com') |
| Does Not Match Wildcard (Not Like) | does_not_match | users[:email].does_not_match('%a.com') | users.email NOT LIKE '%a.com' |
| Does Not Match Regexp | does_not_match_regexp | users[:email].does_not_match_regexp(/a.com$/) | |
| Does Not Match Any | does_not_match_any | users[:email].does_not_match_any(['%a.com','%b.com']) | (users.email NOT LIKE '%a.com' OR users.email NOT LIKE '%b.com') |
| Does Not Match All | does_not_match_all | users[:email].does_not_match_all(['%a.com','%b.com']) | (users.email NOT LIKE '%a.com' AND users.email NOT LIKE '%b.com') |
| Greater Than or Equal To | gteq | users[:created_at].gteq(Date.yesterday) | users.created_at >= '2020-11-03' |
| Greater Than or Equal To Any | gteq_any | users[:created_at].gteq_any(Date.yesterday..Date.today) | (users.created_at >= '2020-11-03' OR users.created_at >= '2020-11-04') |
| Greater Than or Equal To All | gteq_all | users[:created_at].gteq_all(Date.yesterday..Date.today) | (users.created_at >= '2020-11-03' AND users.created_at >= '2020-11-04') |
| Greater Than | gt | users[:created_at].gt(Date.yesterday) | users.created_at > '2020-11-03' |
| Greater Than Any | gt_any | users[:created_at].gt_any(Date.yesterday..Date.today) | (users.created_at > '2020-11-03' OR users.created_at > '2020-11-04') |
| Greater than All | gt_all | users[:created_at].gt_all(Date.yesterday..Date.today) | (users.created_at > '2020-11-03' AND users.created_at > '2020-11-04') |
| Less Than | lt | users[:created_at].lt(Date.yesterday) | users.created_at < '2020-11-04' |
| Less Than Any | lt_any | users[:created_at].lt_any(Date.yesterday..Date.today) | (users.created_at < '2020-11-04' OR users.created_at < '2020-11-05') |
| Less Than All | lt_all | users[:created_at].lt_all(Date.yesterday..Date.today) | (users.created_at < '2020-11-04' AND users.created_at < '2020-11-05') |
| Less Than or Equal To | lteq | users[:created_at].lteq(Date.today) | users.created_at <= '2020-11-05' |
| Less Than or Equal To Any | lteq_any | users[:created_at].lteq_any(Date.yesterday..Date.today) | (users.created_at <= '2020-11-04' OR users.created_at <= '2020-11-05') |
| Less Than or Equal To All | lteq_all | users[:created_at].lteq_all(Date.yesterday..Date.today) | (users.created_at <= '2020-11-04' AND users.created_at <= '2020-11-05') |
| Case When Then | when | Arel::Nodes::Case.new(users[:email])<br>.when('a@b.com')<br>.then('a@b.org')<br>.when('b@b.com')<br>.then('b@b.org') | CASE users.email WHEN 'a@b.com' THEN 'a@b.org' WHEN 'b@b.com' THEN 'b@b.org' END |
| Concatenation | concat | Internal use only |
| | contains | Internal use only |
| | overlaps | Internal use only |
| | quoted_array
| | count | User.count | SELECT COUNT(*) FROM users |
| | sum | User.sum(:login_count) | SELECT SUM(users.login_count) FROM users |
| | maximum | User.maximum(:created_at) | SELECT MAX(users.created_at) FROM users |
| | minimum | User.minimum(:created_at) | SELECT MIN(users.created_at) FROM users |
| | average | User.average(:login_count) | SELECT AVG(users.login_count) FROM users |
| | extract(field)
| Multiplication | * | Arel::Nodes::Multiplication.new(users[:login_count], Arel::Nodes::SqlLiteral.new('0.9')) | users.login_count * 0.9 |
| Addition | + | Arel::Nodes::Addition.new(users[:login_count], 100) | users.login_count + 100 |
| Subtraction | - | Arel::Nodes::Subtraction.new(users[:login_count], 100) | users.login_count - 100 |
| Division | / | Arel::Nodes::Division.new(users[:login_count], 10) | users.login_count / 10 |
| Bitwise AND | & | Arel::Nodes::BitwiseAnd.new(56, 53) | 56 & 53 |
| Bitwise OR | \| | Arel::Nodes::BitwiseOr.new(56, 53) | 56 \| 53 |
| Bitwise XOR | ^ | Arel::Nodes::BitwiseXor.new(56, 53) | 56 ^ 53 |
| Bitwise Shift Left | << | Arel::Nodes::BitwiseShiftLeft.new(56, 2) | 56 << 2 |
| Bitwise Shift Right | >> | Arel::Nodes::BitwiseShiftRight.new(56, 2) | 56 >> 2 |
| Bitwise NOT | ~@ | Arel::Nodes::BitwiseNot.new(56) | ~ 56 |
| Alias | as | Arel::Nodes::Addition.new(users[:login_count], 10).as('inflated_login_count') | users.login_count + 10 AS inflated_login_count |
| | lower
| | coalesce
| Order Ascending | asc | users.order(users[:email].asc).project(Arel.star) | SELECT * FROM users ORDER BY users.email ASC |
| Order Descending | desc | users.order(users[:email].desc).project(Arel.star) | SELECT users.* FROM users ORDER BY users.email DESC |
| Limit | limit
| Taken | taken (alias for limit)
| | constraints
| | offset
| | skip
| | exists
| | lock
| | on
| | group
| | from
| | froms
| Inner Join | join | users.join(posts).project(Arel.star) | SELECT * FROM users INNER JOIN posts |
| | outer_join | users.outer_join(posts).on(users[:id].eq(posts[:user_id])).project(Arel.star) | SELECT * FROM users LEFT OUTER JOIN posts ON users.id = posts.user_id |
| | having
| | window
| Project (the select clause) | project | users.project(:email) | SELECT email FROM users |
| | optimizer_hints
| | distinct
| | distinct_on
| | order | users.order(users[:email].asc).project(Arel.star) |SELECT * FROM users ORDER BY users.email ASC |
| | orders
| | where_sql
| | where | User.where(:email => 'a@a.com') | SELECT users.* FROM users WHERE users.email = 'a@a.com' |
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
| Convert a SQL string to a `Arel::Nodes::SqlLiteral` object | sql | users.order(Arel.sql('length(email)')).project(:email) | SELECT email FROM users ORDER BY length(email) |
| Select all fields | star | users.project(Arel.star.count) | SELECT COUNT(*) FROM users |
| Descending order | reverse_order | User.all.reverse_order | SELECT users.* FROM users ORDER BY users.id |

## References

* https://devhints.io/arel
* https://ernie.io/2010/05/11/activerecord-relation-vs-arel
* https://github.com/DavyJonesLocker/postgres_ext/blob/master/docs/querying.md
* https://github.com/rails/rails/tree/master/activerecord/lib/arel
* http://www.scuttle.io
* https://tanzu.vmware.com/content/blog/using-arel-to-build-complex-sql-expressions
* https://thoughtbot.com/blog/using-arel-to-compose-sql-queries
* https://www.rubydoc.info/gems/arel/7.1.1/Arel/Nodes/Concat
