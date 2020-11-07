### Chapter 10 - Arel

## Introduction

Arel is a relational algebra Ruby on Rails API which is used by ActiveRecord to compose queries. Because it provides an API, developers can use it to compose complex queries in Ruby code.



## The Arel API

The full list of public Arel methods with examples. Let's imagine we have two models, `Vehicle` and `Wheel`. They

```ruby
class Vehicle < ApplicationRecord
  has_many :wheels
end

class Wheel < ApplicationRecord
  belongs_to :vehicle
end
```

With these relationships between vehicles and wheels, we can use these two models as examples when defining the following methods exposed in the Arel API:

usr = User.arel_table

| Description | Arel method | Arel example | SQL |
|---|---|---|---|
| Not equal | not_eq | usr[:email].not_eq('t@g.com') | "`users`.`email` != 't@g.com'" |
| Not equal to any | not_eq_any | usr[:email].not_eq_any(['t@g.com','a@b.com']) | "(`users`.`email` != 't@g.com' OR `users`.`email` != 'a@b.com')" |
| Not equal to all | not_eq_all | usr[:email].not_eq_all(['t@g.com','a@b.com']) | "(`users`.`email` != 't@g.com' AND `users`.`email` != 'a@b.com')" |
| Equal | eq | usr[:email].eq('t@g.com') | "`users`.`email` = 't@g.com'" |
| | is_not_distinct_from | usr[:email].is_not_distinct_from('t@g.com') | "`users`.`email` <=> 't@g.com'" |
| | is_distinct_from | usr[:email].is_distinct_from('t@g.com') | "NOT `users`.`email` <=> 't@g.com'" |
| | eq_any | usr[:email].eq_any(['a@b.com', 'b@b.com']) | "(`users`.`email` = 'a@b.com' OR `users`.`email` = 'b@b.com')" |
| | eq_all | usr[:email].eq_all(['a@b.com', 'b@b.com']) | "(`users`.`email` = 'a@b.com' AND `users`.`email` = 'b@b.com')" |
| | between | usr[:created_at].between(Date.today..Date.tomorrow) | "`users`.`created_at` BETWEEN '2020-11-04' AND '2020-11-05'" |
| | in | usr[:email].in(['a@b.com', 'b@b.com']) | "`users`.`email` IN ('a@b.com', 'b@b.com')" |
| | in_any | usr[:email].in_any(['a@b.com', 'b@b.com']) | "(`users`.`email` IN ('a@b.com') OR `users`.`email` IN ('b@b.com'))" |
| | in_all | usr[:email].in_all(['a@b.com', 'b@b.com']) | "(`users`.`email` IN ('a@b.com') AND `users`.`email` IN ('b@b.com'))" |
| | not_between | usr[:created_at].not_between(Date.today..Date.tomorrow) | "(`users`.`created_at` < '2020-11-04' OR `users`.`created_at` > '2020-11-05')" |
| | not_in | usr[:created_at].not_in(Date.today..Date.tomorrow) | "(`users`.`created_at` < '2020-11-04' OR `users`.`created_at` > '2020-11-05')" |
| | not_in_any | usr[:created_at].not_in_any(Date.today..Date.tomorrow) | "(`users`.`created_at` NOT IN ('2020-11-04') OR `users`.`created_at` NOT IN ('2020-11-05'))" |
| | not_in_all | usr[:created_at].not_in_all(Date.today..Date.tomorrow) | "(`users`.`created_at` NOT IN ('2020-11-04') AND `users`.`created_at` NOT IN ('2020-11-05'))" |
| | matches | usr[:email].matches('%a.com') | "`users`.`email` LIKE '%a.com'" |
| | matches_regexp | usr[:email].matches_regexp(/a.com$/) | TODO: NotImplementedError (~ not implemented for this db) |
| | matches_any | usr[:email].matches_any(['%a.com','%b.com']) | "(`users`.`email` LIKE '%a.com' OR `users`.`email` LIKE '%b.com')" |
| | matches_all | usr[:email].matches_all(['%a.com','%b.com']) | "(`users`.`email` LIKE '%a.com' AND `users`.`email` LIKE '%b.com')" |
| | does_not_match | usr[:email].does_not_match('%a.com') | "`users`.`email` NOT LIKE '%a.com'" |
| | does_not_match_regexp | usr[:email].does_not_match_regexp(/a.com$/) | TODO: NotImplementedError (!~ not implemented for this db) |
| | does_not_match_any | usr[:email].does_not_match_any(['%a.com','%b.com']) | "(`users`.`email` NOT LIKE '%a.com' OR `users`.`email` NOT LIKE '%b.com')" |
| | does_not_match_all | usr[:email].does_not_match_all(['%a.com','%b.com']) | "(`users`.`email` NOT LIKE '%a.com' AND `users`.`email` NOT LIKE '%b.com')" |
| | gteq | usr[:created_at].gteq(Date.yesterday) | "`users`.`created_at` >= '2020-11-03'" |
| | gteq_any | usr[:created_at].gteq_any(Date.yesterday..Date.today) | "(`users`.`created_at` >= '2020-11-03' OR `users`.`created_at` >= '2020-11-04')" |
| | gteq_all | usr[:created_at].gteq_all(Date.yesterday..Date.today) | "(`users`.`created_at` >= '2020-11-03' AND `users`.`created_at` >= '2020-11-04')" |
| | gt | usr[:created_at].gt(Date.yesterday) | "`users`.`created_at` > '2020-11-03'" |
| | gt_any | usr[:created_at].gt_any(Date.yesterday..Date.today) | "(`users`.`created_at` > '2020-11-03' OR `users`.`created_at` > '2020-11-04')" |
| | gt_all | usr[:created_at].gt_all(Date.yesterday..Date.today) | "(`users`.`created_at` > '2020-11-03' AND `users`.`created_at` > '2020-11-04')" |
| | lt | usr[:created_at].lt(Date.yesterday) | "`users`.`created_at` < '2020-11-04'" |
| | lt_any | usr[:created_at].lt_any(Date.yesterday..Date.today) | "(`users`.`created_at` < '2020-11-04' OR `users`.`created_at` < '2020-11-05')" |
| | lt_all | usr[:created_at].lt_all(Date.yesterday..Date.today) | "(`users`.`created_at` < '2020-11-04' AND `users`.`created_at` < '2020-11-05')" |
| | lteq | usr[:created_at].lteq(Date.today) | "`users`.`created_at` <= '2020-11-05'" |
| | lteq_any | usr[:created_at].lteq_any(Date.yesterday..Date.today) | "(`users`.`created_at` <= '2020-11-04' OR `users`.`created_at` <= '2020-11-05')" |
| | lteq_all | usr[:created_at].lteq_all(Date.yesterday..Date.today) | "(`users`.`created_at` <= '2020-11-04' AND `users`.`created_at` <= '2020-11-05')" |
| | when | Arel::Nodes::Case.new(usr[:email]).when('a@b.com').then('a@b.org').when('b@b.com').then('b@b.org') | "CASE `users`.`email` WHEN 'a@b.com' THEN 'a@b.org' WHEN 'b@b.com' THEN 'b@b.org' END" |
| | concat | 
| | contains
| | overlaps
| | quoted_array

## References

* https://devhints.io/arel
* https://github.com/DavyJonesLocker/postgres_ext/blob/master/docs/querying.md
* https://github.com/rails/rails/tree/master/activerecord/lib/arel
* https://github.com/rails/rails/blob/master/activerecord/lib/arel/predications.rb
* https://thoughtbot.com/blog/using-arel-to-compose-sql-queries
* https://www.rubydoc.info/gems/arel/7.1.1/Arel/Nodes/Concat

