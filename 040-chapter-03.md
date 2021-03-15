### Chapter 3 - Keys

## Introduction

Your application will most likely be designed on top of one or more database tables, whether they are stored in a relational database or a NoSQL database. As the application grows, you'll define relationships between entities that will need to be defined both in the application and in the database.

A key is a unique value for a row in a table. A key can be a single field. It can also be a combination of fields. It needs to be something that uniquely represents the data in that row.

### Natural vs Surrogate

Keys can be natural or surrogate. A natural key is composed of the data in one or more fields that represent the unique data for that row or entity. For example, a physical street address, consisting of street address, city, state and zipcode is unique, because in most cases street addresses are unique for a city and state. This key would consist of all of the fields, because 101 West Maple Street could exist in multiple cities among multiple states.

A surrogate key is a key that is automatically generated for each individual row in a table. This key could be an incrementing integer or an unique identifier, like a UUID.

## Primary Keys

Primary keys are attributes used to identify the uniqueness of a row in a relation (also known as a table in database terminology).

## Foreign Keys

Foreign keys are used to associate child or dependent records to another relation. The foreign key data type should match the primary key from the related table.

## Data Types

For surrogate keys, the data type matters. Fields can and should be indexed. Integer types are more suitable for storing integer values, while universally unique identifiers (UUIDs) should be stored in UUID type fields (if the database supports it). Storing the values in their native form can provide storage savings and better performance.

### Integer Types

If you decide to use integers as the primary key, you'll need to try to imagine, in a far out future when your application is widely used, how many tables may exist each of its tables. If you need a domain table to store enum values, you could get away with a PostgreSQL `smallint` or a MySQL `tinyint` datatype. If you're storing blog comments where you expect massive growth over time, you should pick a larger integer datatype which will not hit its upper limit for a long period of time.

### UUID

Can be generated either by the Rails application or the database. The advantage of generating the UUIDs in the application is that you can create the recordset in memory with all of the primary and foreign key relationships generated and set in place before the records are committed to the database.

## Resources

* https://dev.mysql.com/doc/refman/8.0/en/integer-types.html
* https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_uuid
* https://www.postgresql.org/docs/9.5/datatype-uuid.html
* https://ruby-doc.org/stdlib-2.5.1/libdoc/securerandom/rdoc/SecureRandom.html

[Next >>](050-chapter-04.md)
