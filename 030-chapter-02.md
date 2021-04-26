### Chapter 2 - Relational Databases

## Introduction

In this book, we'll discuss a few relational database platforms that are popular in the Ruby on Rails community. We'll discuss the terminology used in each platform, which can differ. We'll also cover the common data types offered by each platform.

## Terminology

Some of the terminology differs across database platforms, depending on which documention you read or who you ask. Most of the time, though they're using different terminology to describe the same things. Here are a few terms that we'll use throughout the book.

* Blocking - A situation where one process has locked a resource such as a row or table to prevent access to the same resource from another process
* Columns - Also known as fields
* Concurrency - Simultaneous access to view or manipulate data
* Commit - Successfully closing out a transaction
* Connection pool - A cache of connections that are reusable
* Create, Read, Update, Delete (CRUD) - Commonly used to describe the basic interactions between an application and a database platform
* Data Definition Language (DDL) - Statements that can create, modify or drop database structures
* Data Manipulation Language (DML) - Statements that can create, modify or remove rows from a table
* Database - A collection of objects, such as tables, where records are stored
* Deadlock - A situation where two or more database connections are waiting for a lock on a resource (row, table, etc) to be released
* Fields - Distinct attributes that describe an individual record
* Function - Logic that can run against one or more columns inside of a transaction
* Index - A data structure that provides quick access to data
* Join - Used to combine the results of one or more relations
* Keys - Uniquely identifies rows in a table
* Lock - Concurrent database systems provide locking mechanisms which allow rows or tables to be locked for exclusive access
* Materialized View - A PostgreSQL only feature where the results of a query can be permanently stored. Materialized views cannot be updated, only dropped and recreated. Commonly used to denormalize data for fast retrieval.
* Procedure - Logic that can include conditional statements and variables
* Query - A method of retrieving data from a database relation, such as a table or view
* Record - An individual entity or row that represents a collection of fields
* Relation - An object in a relational database management system which presents data, such as a table or view
* Relational Database Management System (RDBMS) - A collection of relational objects (views, tables, etc) which store data and can be queried and manipulated
* Replication - Read-only copies of the database. Commonly used to provide another near-time copy of the data to reduce query loads on the primary database.
* Rollback - When a failure occurs during a transaction, the current and prior statements are switched back to their state before the transaction started
* Structured Query Language (SQL) - A computer language used to retrieve and manipulate data in a database
* Table - A collection of rows and columns
* Table Scan - When the entire table is read to find the results of a query
* Transaction - An independent unit of work that may involved multiple operations, such as inserting into one table and deleting from another table
* Tuple - Also known as a record
* View - A pre-defined query of data from one or more tables that can itself be queried as if it were a table

## ACID

ACID is a set of properties that help guarantee the validity of data committed to a database. We'll define each of the components of ACID below. PostgreSQL and MySQL (when using the InnoDB and NDB Cluster Storage engines) are both ACID compliant.

* Atomicity - The transaction and all of its operations will fully succeed or fully roll back
* Consistency - Data must be validated against existing rules before it is committed
* Isolation - When running multiple transactions at the same time, each transaction executes against the database only when other transactions against the same database objects have been committed
* Durability - The transaction is only reported as being successfully committed after it has been persisted

## Resources

* https://en.wikipedia.org/wiki/Relational_database

[Next >>](040-chapter-03.md)
