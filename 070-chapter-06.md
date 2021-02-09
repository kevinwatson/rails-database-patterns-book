### Chapter 6 - Active Record Query Interface

## Introduction

The Ruby on Rails guide [Active Record Query Interface](https://guides.rubyonrails.org/active_record_querying.html) is an excellent resource of the available methods for writing Ruby code to retrieve and modify data in a database.

I won't try to reproduce what the Query Interface guide authors have already written, but in an effort to provide a comprehensive guide here, I will include a few examples and possible optimizations in this chapter.

In the previous chapter we discussed the Arel library. The Active Record Query Interface methods use Arel methods under the hood to generate the necessary SQL to access the data in the database. The query interface makes it easy to access the database without the need to use lower-level Arel methods. In my experience, as a Rails developer, you will most often use the public methods on the query interface's ActiveRecord::Relation class and rarely need to drop down to the Arel methods, but it is a good idea to become familiar with the Arel library, so that you can use it when needed.

## Examples

Let's say we have two models, a `Chair` model and a `Leg` model. We'll define the relationship between the models as a chair can have many legs.

Defining our models

```ruby
class Chair
  has_many :legs
end

class Leg
  belongs_to :chair
end
```

Let's assume the following schema in the database.

```sql
CREATE TABLE chairs (id bigserial, kind varchar, created_at timestamp, updated_at timestamp);
CREATE TABLE legs (id bigserial, chair_id, created_at timestamp, updated_at timestamp);
```

And the Rails scaffolding equivalent

```ruby
rails g scaffold chair kind:string
rails g scaffold leg chair:belongs_to
```

For the purposes of this example, we won't generate the related tables, but based on the classes above, we can assume that the `chairs` table has an `id` field, and the `legs` table has both an `id` and a foreign key `chair_id` field.

Let's start with simple examples and work our way up to more complex examples. The first thing we'll do is use the Query Interface to retrieve all Chair records as Ruby objects.

```ruby
chairs = Chair.all
```

The chair variable is a `ActiveRecord::Relation` object, which has various methods such as `where` that can be used to add additional filters to the final SQL query. The data is lazy loaded, which allows us to chain `where` methods together before the SQL query is finally generated and sent to the database.

Next, let's retrieve a limited set of chair objects.

```ruby
chairs = Chair.where(kind: 'dining')
```

This will retrieve all chair records where the `kind` is set to `dining`. Again, because of performance efficiencies in the `ActiveRecord::Relation` library, the query is not sent to the database until we start to perform other operations on the `chairs` variable, such as iterate over the items in the object. For example, the following code will generate the query and make a call to the database when the `each` method is called.

```ruby
chairs.each do |chair|
  puts chair.id
end
```

Next, let's modify a database record. We'll use a query to retrieve the chair records where the `kind` field is `NULL` and set it to a default value. We'll provide examples of two ways to do this, the first will iterate over each record and the second will be more efficient.

First, let's retrieve the records and update those records one by one.

```ruby
chairs = Chair.where(kind: nil)

chairs.each do |chair|
  chair.kind = 'unknown'
  chair.save!
end
```

In the example above, we used the `save!` bang method. The regular `save` method is also available. The `save` method will return `true` or `false`, depending on whether model validations passed and the record was saved. The bang method, `save!`, will raise an error if either validations failed or there was a problem saving the record. In this example, I used to the bang method to error out and exit the code on the first record that failed. Depending on the application's requirements, this may or may not be the preferred method for handling errors.

Let's dig deeper and find out what's going on behind the scenes.

When the first line is run in Rails console, it generates the SQL query and sends the query to the database. If this line was in an app or a script, it would delay the request until the data is needed.

```ruby
chairs = Chair.where(kind: nil)
```

In the output, we see it build and run a `SELECT` statement, converting our Ruby `nil` value to a SQL `NULL` value. The object that is returned is a `ActiveRecord::Relation` object which contains a list of Chair objects.

```ruby
irb(main):012:0> chairs = Chair.where(kind: nil)
  Chair Load (14.0ms)  SELECT  "chairs".* FROM "chairs" WHERE "chairs"."kind" IS NULL LIMIT ?  [["LIMIT", 11]]
=> #<ActiveRecord::Relation [#<Chair id: 2, kind: nil, created_at: "2021-01-19 14:44:34", updated_at: "2021-01-19 14:44:34">]>
```

Now, let's iterate over the collection and update the records one by one.

```ruby
chairs.each do |chair|
  chair.kind = 'unknown'
  chair.save!
end
```

```ruby
irb(main):013:0> chairs.each do |chair|
irb(main):014:1*   chair.kind = 'unknown'
irb(main):015:1>   chair.save!
irb(main):016:1> end
  Chair Load (6.2ms)  SELECT "chairs".* FROM "chairs" WHERE "chairs"."kind" IS NULL
   (0.1ms)  begin transaction
  SQL (17.4ms)  UPDATE "chairs" SET "kind" = ?, "updated_at" = ? WHERE "chairs"."id" = ?  [["kind", "unknown"], ["updated_at", "2021-01-20 14:39:24.823544"], ["id", 3]]
   (11.3ms)  commit transaction
   (0.2ms)  begin transaction
  SQL (16.2ms)  UPDATE "chairs" SET "kind" = ?, "updated_at" = ? WHERE "chairs"."id" = ?  [["kind", "unknown"], ["updated_at", "2021-01-20 14:39:24.859795"], ["id", 4]]
   (10.1ms)  commit transaction
   (0.1ms)  begin transaction
  SQL (15.9ms)  UPDATE "chairs" SET "kind" = ?, "updated_at" = ? WHERE "chairs"."id" = ?  [["kind", "unknown"], ["updated_at", "2021-01-20 14:39:24.893203"], ["id", 5]]
   (10.7ms)  commit transaction
=> [#<Chair id: 3, kind: "unknown", created_at: "2021-01-20 14:38:56", updated_at: "2021-01-20 14:39:24">, #<Chair id: 4, kind: "unknown", created_at: "2021-01-20 14:38:57", updated_at: "2021-01-20 14:39:24">, #<Chair id: 5, kind: "unknown", created_at: "2021-01-20 14:39:03", updated_at: "2021-01-20 14:39:24">]
```

In the output above, we see the `SELECT` statement that was lazy loaded. We also see that because we're going to manipulate data, Active Record wraps a database transaction around each `UPDATE` statement.

The query interface provides methods to perform actions on database records without loading each record into memory. Notice that Active Record does not wrap the `UPDATE` statement in a database transaction.

```ruby
Chair.where(kind: nil).update_all(kind: 'unknown')

  SQL (33.1ms)  UPDATE "chairs" SET "kind" = 'unknown' WHERE "chairs"."kind" IS NULL
=> 3
```

If we wanted to run multiple SQL statements wrapped in a transaction, we can use a block.

```ruby
Chair.transaction do
  Chair.where(kind: nil).update_all(kind: 'unknown')
  Chair.where(kind: 'plush').update_all(kind: 'super plush')
end

(7.6ms)  begin transaction
  SQL (20.8ms)  UPDATE "chairs" SET "kind" = 'unknown' WHERE "chairs"."kind" IS NULL
  SQL (24.4ms)  UPDATE "chairs" SET "kind" = 'super plush' WHERE "chairs"."kind" = ?  [["kind", "plush"]]
(15.9ms)  commit transaction
```

Both `UPDATE` statements in this `transaction` block are wrapped in a single database transaction. If either of the `UPDATE` statements caused an error, the database would `ROLLBACK` the transaction and Active Record would raise an `ActiveRecord::Rollback` error. When a rollback occurs, the database changes are switched back to their previous state, as if our `UPDATE` statement never occurred.

### Scopes

Scopes are pre-defined, reusable Active Record filters that you can define in your models. Like any `ActiveRecord::Relation` object, scopes are chainable. They can be used to hide the implementation details for complex queries.

Let's look at some examples. The first example will retrieve all chair records which do not have any assigned leg records. This allows us to call `Chair.without_legs` from another class which makes this specific database access code reusable. If we didn't use a scope but wanted the same database query, we would write the scope below as `Chair.left_joins(:legs).where(Leg.arel_table[:id].eq(nil))`, which is a little harder to read than `Chair.without_legs`. It also allows us to abstract the logic behind the method `without_legs` away from any other classes that need this functionality.

```ruby
class Chair < ApplicationRecord
  scope :without_legs, -> {
    left_joins(:legs).
    where(Leg.arel_table[:id].eq(nil))
  }
end
```

Scopes can be simple one-liners like `where(kind: 'plush')`, or chained `ActiveRecord::Relation` objects as the example above shows.

## Resources

* https://guides.rubyonrails.org/active_record_querying.html

## Wrap-up
