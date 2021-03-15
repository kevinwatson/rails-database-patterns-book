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

## Patterns

1. Fields in new tables can be created with default values. When adding fields to new tables with thousands or more records, the database may lock the table if the migration adds a new field with a default value.
1. Keep migrations small. For example, when adding fields to separate tables, separate each table's changes into a single migration. This makes troubleshooting and rollbacks easier.
1. Don't seed values in a migration. Create a rake task instead, or for new projects, add data to the `db/seeds.rb` file.

## Examples

Generate a new resources via scaffolding

```
rails generate scaffold User email:string password:string
```

Add a field to the `users` table

```
rails generate migration AddLoginCountToUser login_count:integer
```

## Resources

* https://guides.rubyonrails.org/active_record_migrations.html
* https://guides.rubyonrails.org/command_line.html#rails-generate

## Wrap-up

[Next >>](090-chapter-08.md)
