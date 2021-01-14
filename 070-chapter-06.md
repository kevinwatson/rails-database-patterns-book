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

Now, let's modify the same records but with a single database call.

```ruby
Chair.where(kind: nil).update_all(kind: 'unknown')
```

### Scopes

## Resources

* https://guides.rubyonrails.org/active_record_querying.html

## Wrap-up
