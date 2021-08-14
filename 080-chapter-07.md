### Chapter 7 - Migrations

## Introduction

Ruby on Rails provides a method for maintaining a database schema called migrations. A migration is a specific unit that can manipulate the database schema by creating or dropping tables, adding columns, etc. Migrations are written in Ruby, and are database agnostic - we can write the same migration for a SQLite database that we would use for a PostgreSQL database.

## Creating a Migration

Migrations are generated when you use Rails scaffolding to generate models, controllers and views. Migrations can also be generated separately with the `rails generate migration` command. The generator command creates the migration files, which are created with timestamped filenames to make them unique. The files can be committed to source control and provide a history of changes to the database.

## Running a Migration

Once a migration has been created, it needs to be run against the database. To do so, we need to run another command to apply the migration. Running this command will produce some output, either displaying a successful result or an error.

```
rails db:migrate
```

Each time the `migrate` task is run, it will compare the database schema to the `db/schema.rb` or `db/structure.sql` files. If they are different, the file in the `db` directory will be modified to match the database schema.

## Patterns

1. Fields in new tables can be created with default values. When adding one or more fields with default values, the database may lock the table, which could cause errors in a live application. As an alternative to the default value in the database, the model class's constructor can be used to set a default value when an object is instantiated. The record will then be inserted into the table with the defaults set.
1. Keep migrations small. For example, when adding fields to separate tables, separate each table's changes into a single migration. This makes troubleshooting and rollbacks easier.
1. Don't seed values in a migration. Create a rake task instead, or for new projects, add data to the `db/seeds.rb` file. This keeps the schema separate from the data and allows developers and deployers to run each separately.
1. For distributed applications: when dropping a column, first roll out a change that adds the field to the model's `ignored_columns` array. Second, follow that change with a migration that drops the column. This prevents older versions of the application from trying to access the field and causing errors.

## Examples

Generate a new resource via scaffolding

```bash
rails generate scaffold User email:string password:string
```

Add a field to the `users` table. The first parameter passed to the scaffold command is `AddLoginCountToUser`. This first parameter contains information that will be used by the generator. Let's break this down into its individual parts. First, the name is Pascal cased, the same way that classes and modules are cased in Ruby. The migration class will use this full name. Second, the generator looks for key words, like `To`. It ignores the words to the left and tries to parse the words to the left. In this case, it found the word `User` and will automatically add a `change_table :users` block to the migration class. The optional parameters after the class name can specify additional options that should automatically be added to the migration class. In the example below, `login_count:integer` specifies the new column name and it's type. This will add a `add_column :users, :login_count, :integer` line inside the `change_table :users` block.

This rails command

```bash
rails generate migration AddLoginCountToUser login_count:integer
```

Will create a file in the `db/migrations` directory that looks like this

```ruby
class AddLoginCountToUser < ActiveRecord::Migration
  def change
    add_column :users, :login_count, :integer
  end
end
```

## Resources

* https://guides.rubyonrails.org/active_record_migrations.html
* https://guides.rubyonrails.org/command_line.html#rails-generate

## Wrap-up

[Next >>](090-chapter-08.md)
