### Chapter 2 - Relational Databases

## Introduction

In this book, we'll discuss a few relational database platforms that are popular in the Ruby on Rails community. We'll discuss the terminology used in each platform, which can differ. We'll also cover the common data types offered by each platform.

## Terminology

Some of the terminology differs across database platforms, depending on which documention you read or who you ask. Most of the time, though they're using different terminology to describe the same things. Here are a few terms that we'll use throughout the book.

* Columns - Also known as fields
* Database - A collection of objects, such as tables, where records are stored
* Fields - Distinct attributes that describe an individual record
* Function - Logic that can run against one or more columns inside of a transaction
* Procedure - Logic that can include 
* Records - An individual entity that represents a collection of fields
* Table - A collection of individual records
* Transaction - An independent unit of work that may involved multiple operations, such as inserting into one table and deleting from another table
* Tuple - Also known as records
* Commit - Successfully closing out a transaction
* Rollback - When a failure occurs during a transaction, the current and prior statements are switched back to their state before the transaction started
* Concurrency - Simultaneous access to view or manipulate data
* Data Definition Language (DDL) - Statements that can create, modify or drop database structures
* Data Manipulation Language (DML) - Statements that can create, modify or remove rows from a table
* Create, Read, Update, Delete (CRUD) - 
* Keys - 
* Lock - 
* Block - 
* Query - 
* Replication - 
* Join - 
* Index - 
* Scan - 

## ACID

ACID is a set of properties that help guarantee the validity of data committed to a database. We'll define each of the components of ACID below. PostgreSQL and MySQL (when using the InnoDB and NDB Cluster Storage engines) are both ACID compliant.

* Atomicity - The transaction and all of its operations will fully succeed or fully roll back
* Consistency - Data must be validated against existing rules before it is committed
* Isolation - When running multiple transactions at the same time, each transaction executes against the database only when other transactions against the same database objects have been committed
* Durability - The transaction is only reported as being successfully committed after it has been persisted

## Resources

* 
