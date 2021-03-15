### Chapter 10 - Batch Processing

## Introduction

Batch processing is defined as processing large amounts of data without user interaction. Batches can be scheduled to run when computing resources are available. An example of a batch process could be a process that generates emails for an email campaign targeted to a large number of users. Batches can also be broken down into smaller chunks to manage hardware resources in your Rails app.

While ActiveRecord makes it easy to retrieve multiple records from the database, when our process needs to find and process hundreds, thousands or millions of records, creating ActiveRecord models from all of those records can cause the app to consume large amounts of memory. Consuming large amounts of memory can cause our application to slow down, and at worst cause our application to crash. One method to work around this issue is to load a subset of records at a time, process those records, and then repeat until all of the pending records have been processed.

In the following sections we'll discuss efficient ways to process large amounts of data.

## Update All

Depending on the requirements, plain old SQL can be one of the most efficient ways that a process can update one or more records. The ActiveRecord Relation class provides an `update_all` method that allows the developer to construct a SQL `update` statement using the ActiveRecord Query Interface. The `update_all` method will generate the `update` statement and send the request to the database without requiring the application to load any of the related database records into memory.

### Example

Let's update our `users` table to set all of the `login_count` fields to `0` where the field is currently set to `NULL`.

```ruby
User.where(login_count: nil).update_all(login_count: 0)
```

## ActiveRecord::Batches

The ActiveRecord framework includes a `Batches` module that provides a few methods that allow us to process large numbers of records efficiently. We'll list each with an example. Note that each method has various options that can be used to adjust the batch size, etc.

### find_each

The `find_each` method loads a batch of records (the default is 1000 records at a time) into memory. It allows us to call methods and access the data on each ActiveRecord model object.

```ruby
User.where(login_count: 0).find_each do |user|
	user.send_reminder_email
end
```

### find_in_batches

The `find_in_batches` method yields a batch of records that can be iterated through individually.

```ruby
User.where(login_count: 0).find_in_batches do |user_batch|
	User.each do |user|
		user.send_reminder_email
	end
end
```

### in_batches

The `in_batches` method yields a Relation object that can be used to perform actions on the subset of records.

```ruby
User.where(login_count: 0).in_batches do |user_batch|
	user_batch.delete_all
end
```

## Pluck Each

Sometimes you don't need to load the entire record, but you need the primary key value of the records that need to be processed. The [Pluck Each](https://github.com/abrandoned/pluck_each) gem provides a way to retrieve only the specified fields of the records in a batch. For heavy models that have a lot of fields or data that require extra memory, loading only the required fields like the `id` or the `name` fields can be more efficient.

### pluck_each

The `pluck_each` method combines the ActiveRecord::Relation `pluck` and the ActiveRecord::Batches `find_each` method to load a subset of records that match the filter criteria but only retrieve the requested fields.

```ruby
User.where(login_count: 0).pluck_each(:email) do |email|
	UserMailer.with(email: email).welcome_email.deliver_later
end
```

### pluck_in_batches

The `pluck_in_batches` method combines the ActiveRecord::Relation `pluck` and the ActiveRecord::Batches `find_in_batches` method to load a subset of records that match the filter criteria but only retrieve the requested fields.

```ruby
User.where(login_count: 0).pluck_in_batches(:email) do |emails|
	User.send_emails(emails)
end
```

## Resources

* https://api.rubyonrails.org/classes/ActiveRecord/Batches.html
* https://api.rubyonrails.org/classes/ActiveRecord/Relation.html
* https://github.com/abrandoned/pluck_each

## Wrap-up

When large numbers of records need to be processed, Rails and associated gems provide efficient ways to process these records. Depending on whether we want to process a single record at a time, multiple records at a time or only specific fields, there are multiple efficient ways to retrieve the data.

[Next >>](120-chapter-11.md)
