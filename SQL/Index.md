## Slow Indexes, Part I

* Rebuilding an index does not improve performance on the long run.

### Ingredients for a slow index lookup

* Leaf node chain
    * In a case that an index lookup not only needs to perform the tree traversal, but also needs to follow the leaf node chain.
    * e.g. not-unique 
* Accessing the table
    * Even a single leaf node might contain many hits—often hundreds. 
    * The corresponding table data is usually scattered across many table blocks. That means that there is an additional table access for each hit.
    * TBD...

### An index lookup requires three steps

* (1) the tree traversal
* (2) following the leaf node chain
* (3) fetching the table data. 
    * The tree traversal is the only step that has an upper bound for the number of accessed blocks—the index depth. 
    * The other two steps might need to access many blocks—they cause a slow index lookup.

### Three distinct operations that describe a basic index lookup of the Oracle database

* INDEX UNIQUE SCAN
    * It performs the tree traversal only. 
    * The Oracle database uses this operation if a unique constraint ensures that the search criteria will match no more than one entry.
    
* INDEX RANGE SCAN
    * It performs the tree traversal and follows the leaf node chain to find all matching entries. 
    * This is the fall­back operation if multiple entries could possibly match the search criteria.
    * The important point is that an INDEX RANGE SCAN can potentially read a large part of an index. If there is one more table access for each row, the query can become slow even when using an index.

* TABLE ACCESS BY INDEX ROWID
    * This operation retrieves the row from the table. 
    * This operation is (often) performed for every matched record from a preceding index scan operation.

# The Where Clause

* This chapter explains how different operators affect index usage and how to make sure that an index is usable for as many queries as possible. 
* The last section shows common anti-patterns and presents alternatives that deliver better performance.

## The Equality Operator
* This section shows how to verify index usage and explains how concatenated indexes can optimize combined conditions. 

### Primary Keys
* The database automatically creates an index for the primary key.
*  Select from pk:

| Id  | Operation                    | Name           | Rows | Cost |
| --- | ---------------------------- | -------------- | ---- | ---- |
|  0  | SELECT STATEMENT             |                |   1  |   2  |
|  1  |  TABLE ACCESS BY INDEX ROWID | EMPLOYEES      |   1  |   2  |
|  2  |   INDEX UNIQUE SCAN          | EMPLOYEES_PK   |   1  |   1  |

* INDEX UNIQUE SCAN
	* The operation that only traverses the index tree. 
	* It fully utilizes the logarithmic scalability of the index to find the entry very quickly—almost independent of the table size.

* TABLE ACCESS BY INDEX ROWID operation. This operation can become a performance bottleneck
	* but there is no such risk in connection with an INDEX UNIQUE SCAN. This operation cannot deliver more than one entry 
	* so it cannot trigger more than one table access. 
	* That means that the ingredients of a slow query are not present with an INDEX UNIQUE SCAN.

### Concatenated Indexes
* A concatenated index is one index across multiple columns.
* Database creates an index on all primary key columns—a so-called concatenated index. 
* Note that the column order of a concatenated index has great impact on its usability so it must be chosen carefully.
* Whenever a query uses the complete primary key, the database can use an INDEX UNIQUE SCAN—no matter how many columns the index has.
* When using only one of the indexed columns, the index will not be used. Instead it performs a TABLE ACCESS FULL.
    * As a result the database reads the entire table and evaluates every row against the where clause.
* But first index column is always usable for searching.
* The most important consideration when defining a concatenated index is how to choose the column order so it can be used as often as possible.
* In general, a database can use a concatenated index when searching with the leading (leftmost) columns. 
    * An index with three columns can be used when 
        * searching for the first column
        * searching with the first two columns together
        * searching using all columns.
* Even though the two-index solution delivers very good select performance as well, the single-index solution is preferable. 
    * It not only saves storage space, but also the maintenance overhead for the second index. 
* The fewer indexes a table has, the better the insert, delete and update performance.
* To define an optimal index you must understand more than just how indexes work—you must also know how the application queries the data. 
    * This means you have to know the column combinations that appear in the where clause.      
 
### Slow Indexes, Part II
*  This section explains the way databases pick an index and demonstrates the possible side effects when changing existing indexes.

#### The Query Optimizer
* The query optimizer, or query planner, is the database component that transforms an SQL statement into an execution plan. 
* This process is also called compiling or parsing. There are two distinct optimizer types.

* Cost-based optimizers (CBO)
    * Generate many execution plan variations and calculate a cost value for each plan. 
    * The cost calculation is based on the operations in use and the estimated row numbers. 
    * In the end the cost value serves as the benchmark for picking the “best” execution plan.

* Rule-based optimizers (RBO)
    * Generate the execution plan using a hard-coded rule set. 
    * Rule based optimizers are less flexible and are seldom used today.
    
## Execution Plans
* https://use-the-index-luke.com/sql/explain-plan


### References 
* [1][Slow Indexes, Part I](https://use-the-index-luke.com/sql/anatomy/slow-indexes)
* [2][The Equality Operator](https://use-the-index-luke.com/sql/where-clause/the-equals-operator)
    

