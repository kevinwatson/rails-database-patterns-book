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

With the following schema in the database

```sql
CREATE TABLE chairs (id bigserial, type varchar, created_at timestamp, updated_at timestamp);
CREATE TABLE legs (id bigserial, chair_id, created_at timestamp, updated_at timestamp);
```

For the purposes of this example, we won't generate the related tables, but based on the classes above, we can assume that the `chairs` table has an `id` field, and the `legs` table has both an `id` and a foreign key `chair_id` field.

Let's start with simple examples and work our way up to more complex examples. The first thing we'll do is use the Query Interface to retrieve all Chair records as Ruby objects.

```ruby
chairs = Chair.all
```

The chair variable is a `ActiveRecord::Relation` object, which has various methods such as `where` that can be used to add additional filters to the final SQL query. The data is lazy loaded, which allows us to chain `where` methods together before the SQL query is finally generated and sent to the database.

Next, let's retrieve a limited set of chair objects.

```ruby
chairs = Chair.where(type: 'dining')
```

This will retrieve all chair records where the type is set to `dining`. Again, because of performance efficiencies in the `ActiveRecord::Relation` library, the query is not sent to the database until we start to perform other operations on the `chairs` variable, such as iterate over the items in the object. For example

```ruby
chairs.each do |chair|
  puts chair.id
end
```

## Resources

* https://guides.rubyonrails.org/active_record_querying.html

## Wrap-up
