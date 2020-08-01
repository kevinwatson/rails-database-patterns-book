## Outline

#### Title: Rails Database Patterns Book

1. Cover
1. Preface
1. Who this book is for
1. What's in this book
1. What you need
1. Online resources
1. Acknowledgements
1. Chapter 1 - Ruby on Rails
1. Chapter 2 - Relational Database Terminology
   * MySQL vs PostgreSQL vs SQLite table
     * Databases
     * Tables
     * Rows and Tuples
     * Indexes
     * Functions
     * Procedures
1. Chapter 3 - Keys
   * Primary
     * Composite
   * Foreign
   * Int4 vs Int16 vs Int32 vs Int64
   * UUID
1. Chapter 4 - Active Record
   * Caching
   * Scopes
1. Chapter 5 - Query Interface
   * Dynamic queries
1. Chapter 6 - Migrations
1. Chapter 7 - PostgreSQL Query Plans
1. Chapter 8 - MySQL Query Plans
1. Chapter 9 - SQLite Query Plans
1. Chapter -- - Arel
1. Chapter -- - Raw SQL
1. Chapter -- - Batch Processing
   * find_in_batches
   * in_batches
1. Chapter -- - Aggregate Functions
1. Chapter -- - Indexes
   * (are a storage tradeoff - indexes take up disk space to produce faster queries)
   * Unique
   * Not Unique
   * Composite
   * Primary Keys
   * Foreign Keys
   * Partial Indexes
     * Index the rows that have data, ignoring NULL rows
   * Deciding when they're needed
   * Deciding when they're no longer needed
1. Chapter -- - SQL Functions
1. Chapter -- - SQL Stored Procedures
1. Chapter -- - SQL Regex
1. Chapter -- - SQL Profiling
1. Chapter -- - Master and Slave Databases
1. Chapter -- - Database Client
1. Chapter -- - Profiling
1. Chapter -- - Locking
1. Chapter -- - Sharding
1. Chapter -- - Seeing is Believing
   * Set up a test database
   * Seeding
1. Chapter -- - Data Transfer Costs
1. Chapter -- - When You Don't Need the Entire Record
   * Pluck
1. Chapter -- - Batch Operations
   * update_all
1. Chapter -- - Document Databases
   * update_all
1. Chapter -- - Optimized queries
   * Inner joins vs left joins
   * Common Table Expressions (CTE)
1. Bulk inserts
1. Antipatterns

