## What is relational Database?
A relational database is a collection of information that
organizes data in predefined relationships where data is stored
in on or more tables (or "relations") of columns and rows, making it
easy to see and understand how different data structure relate to 
each other.

Relationships are a logical connection between different tables, established
on the basis of interaction among these tables.

## What is SQL?
SQL (Structured  Query Language) is the primary programming language
used to manage and interact with *relational databases*. SQL can perform
various operations such as creating, updating and deleting within a database.

```sql

# selects all fields from the users.
SELECT * FROM users;

```

## SQL `SELECT` statement
Let's write our own SQL statement from scratch! A `SELECT` statement
is the most common operation is SQL called "query". `SELECT` retrieves
data from one or more tables. Standard `SELECT` statements do not alter
the state of the database.

```sql

SELECT id FROM users;

```

- Selecting a single field.
`SELECT id from users;`

- Selecting multiple fields.
`SELECT id, name FROM users;`

- Selecting all Fields
If you want to select every field in a record you can use the 
shorthand `*` syntax.
`SELECT * FROM users;`

## Which databases Use SQL?
SQL is just a query language. You typically use it to interact with
a specific database technology.
- SQLite
- PostgreSQL
- MySQL
- Oracle
- BigQuery

Many different databases use the SQL language, most of them will have 
their own dialect. Different DBs use SQL that does not mean they are equal. 

## NoSQL vs SQL.
NoSQL is a database that does not use SQL.
Each NoSQL typically has its own way of writing and executing
queries.
For example MongoDB uses MQL and EasticSearch simply has JSON API.

Relational Databases are fairly similar. NoSQL databases tend to be
fairly unique and are used for more niche purpose.
- NoSQL databases are usually non-relational, SQL databases are usually
relational
- SQL DBs usually have a defined schema, NoSQL databases usually
have dynamic schema.
- SQL DBs are table-based, NoSQL databases have a variety of different
storage methods, such as document, key-value, graph, wide-column and more.

## Types of NoSQL Databases
- Document Databases
- Key-Value store
- Wide-Column
- Graph

A few most popular NoSQL databases are:
- MongoDB
- Cassandra
- Redis (caching)
- CouchDB
- DynamoDB
- ElasticSearch (searching)
- Firebase (Realtime/mobile apps)

A few most popular SQL databases are:
- Cockroach DB - build for scale

## SQLite vs PostgreSQL
SQLite is a serverless DBMS  that has the ability to run within
applications

PostgreSQL uses a client-server model and requires a server to be
installed and listening on a network, similar to an HTTP server.

## Creating a Table
To create tables use `CREATE TABLE`

```sql

CREATE TABLE employees (
	id INTEGER,
	name TEXT,
	age INTEGER,
	is_manager BOOLEAN,
	salary INTEGER
);

```

NOTE: INTEGER and INT are slightly different data types.

## Altering a table
To make changes to your database

- Renaming a Table or Column 
```sql

ALTER TABLE employees
RENAME TO contractors;

ALTER TABLE employees
RENAME COLUMN salary TO invoice;

```

- Add or Drop a column

```sql

ALTER TABLE contractors
ADD COLUMN job_title TEXT;

ALTER TABLE contractors
DROP COLUMN job_title TEXT;

ALTER TABLE contractors
DROP COLUMN is_manager;

```

Unlike some SQL databases, SQLite does not support adding multiple 
columns in a single ALTER TABLE statement. Each column must be added
in a separate ALTER TABLE command.

## Intro to migration
A database migration is a set of changes to a relational database.
`ALTER TABLE` statements are an example of database migration.

## Types of migration
Let's say we want to add a new column to our database this is considered
a database migration.

- Add -> add columns to the database then updating code accordingly
safe to do.

- Delete -> update code then update the database, 
not so save to do.

- Update ->
some approaches:
Downtime.
Few bugs for a seconds.
Copy database. Their can be some new data in old table. 
Aliasing (some DBs support aliasing feature some don't)
very dangerous/hardest to do.

## Up migration and down migration
When writing reversible migrations, we use the terms "up" and "down" migrations.
An "up" migration is simply the set of changes you want to make,
like altering/removing/adding/editing a table in some way. A "down"
migration includes the changes that would revert any of the "up" migration's changes.


- up migration

```sql

ALTER TABLE transactions
ADD COLUMN was_successful BOOLEAN;

ALTER TABLE transactions
ADD COLUMN transaction_type TEXT;

```

-- down migration

```sql

ALTER TABLE transactions
DROP COLUMN was_successful;

ALTER TABLE transactions
DROP COLUMN transaction_type;

```


## SQL Data Types
Data that your RDBMS supports will vary depending upon your specific 
database you are using.

NULL - Null value
INTEGER - signed integers
REAL - Floating point value stored as a 64-bit IEEE floating point numbers
TEXT - text string
BLOB - short for binary large object and typically used for images, audio
and other multimedia.
BOOLEAN - boolean values are written in SQLite queries as true or false
but are recorded as 1 or 0.

NOTE: sqlite does not have BOOLEAN storage class
it will let you write boolean but will convert it into integers underneath. 

- 0 = false
- 1 = true

## NULL Values
In SQL, a cell with a `NULL` value indicates that value is missing.
A `NULL` value is very different from a zero value.

## Constraints
When creating a table we an define whether or not a field can or 
cannot be NULL and that's a kind of `constraint`

A constraint is a rule we create on a database that enforces some
specific behaviour. For example setting a NOT NULL constraint on a 
column ensures that the column will not accept NULL values.

```sql

CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL,
  age INTEGER NOT NULL,
  country_code TEXT NOT NULL,
  username TEXT UNIQUE NOT NULL,
  password TEXT NOT NULL,
  is_admin BOOLEAN
);

```

## Primary Keys
The key defines and protects relationships between tables. A primary key
is a special column that uniquely indentifies records within a table. Each
table can have one and only one primary key.

NOTE: Primary Key will almost always be the "id" column
It's very common to have a column named id on each table in DB and that
`id` is the primary key for that table. No two rows in that table 
can share an `id`.

## Foreign Keys
Foreign keys are what make relational database relational. Foreign Keys
define the relationship between tables. Simply put a `Foreign Key` is 
field in one table that references another table's `Primary Key`.


## Creating a Foreign Key in Sqlite
Creating a foreign key in SQLite happens at table creation After we
define table fields and constraints we add named `CONSTRAINT` where 
we define the `FOREGIN KEY` column and its `REFERENCES`.

```sql

CREATE TABLE departements (

  id INTEGER PRIMARY KEY,
  department_name TEXT NOT NULL

);

CREATE TABLE employee (

  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL,
  department_id INTEGER,
  CONSTRAINT fk_departments
  FOREIGN KEY (department_id)
  REFERENCES department(id)

);


```
In this example, an `employee` has a `department_id`. The `department_id`
must be the same as the `id` field of record from the `departments` table.
`fk_departments` is the specified name of the constraint
1 - `CONSTRAINT fk_departments`: create a constraint called `fk_departments`
2 - `FOREIGN KEY (department_id)`: make this constraint a foreign key 
assigned to this `department_id`
3 - REFERENCES department(id): link the foreign field `id` from the 
`departments` table

## Schema
A database's schema describes how data is organized within it.

Data type, table names, field names, constraints and the relationships
between all of those entities are part a database *schema*.

## There is not perfect way to architect a database schema.
When designing a database schema there typically isn't a "correct" solution.
We do our best to choose a sane set of tables, fields, constraints, etc
that will accomplish our project's goals. Like many things in programming,
different schema designs come with different tradeoffs.

## Relational Databases
A *relational database* is a type of database that stores data so 
that it can easily related to other data. For example, a `user` can have
many `tweets`. There a relationship between a `user` and their `tweet`.

In a relational database:
1 - Data is typically represented in "tables"
2 - Each table has "columns" or "fields" that hold
attribute related to the record.
3 - Each row or entry in the table is called a *record*.
4 - Typically, each record has a unique `Id` called the `primary key`.

## Relational vs Non-Relational Databases
The big difference relational and non-relational databases is that
non-relational databases *nest* their data. Instead of keeping a record
of separate tables, they store records *within* other records.

To over simplify, you can think of non-relational databases as a giant
JSON blobs. If user can have multiple courses, you might just add all
the courses to the user record.

```json
{
  "users": [
    {
      "id": 0,
      "name": "Elon",
      "courses": [
        {
          "name": "Biology",
          "id": 0
        },
        {
          "name": "Biology",
          "id": 0
        }
      ]
    }
  ]
}
```
There does result in a *duplicate data* within the databases. That's 
obviously less then ideal. But it does have some benefits 

## AUTO INCREAMENT
SQLite does auto increment automatically for if you have 
`INTEGER PRIMARY KEY`

## Count
`SELECT COUNT(*) FROM employees;`
give the count

## WHERE
It allows us to be very specific with our instructions.

## Finding NULL values
You can use WHERE clause to filter values by wheather or not they're
NULL

IS NULL
`SELECT name FROM users WHERE first_name IS NULL;`

IS NOT NULL
`SELECT name FROM users WHERE first_name IS NOT NULL;`

## DELETE
A `DELETE` statement removes all records from a table that march
the WHERE clause.
```sql
DELETE FROM emplyees
WHERE id = 251;

```
## The danger of deleting database
Take snapshots of your data  daily or hourly (for more small companies this is fine)

## Update Query in SQL

```sql

UPDATE employees
SET job_title = 'Backend Engineer', salary = 20000
where id = 251;

```

Update Statement
The `UPDATE` statement in SQL allows us to update the fields of a record.
We can even update many records depending on how we write the statement

## Object Relational Mapping
An object-relational-mapping or an *ORM* for short, is a tool that
allows you to perform CRUD operations on a database using a traditional
programming language. These typically come in the form of a library 
or framework ta you would use in your backend server


```go

type User struct {
  ID int
  Name string
  isAdmin bool
}

user := User {
  ID: 10,
  Name: "Lane",
  isAdmin: false,

}

// generate a SQL statement and run it,
// creating a new record in the users table
db.Create(user)
db.Exec("INSERT INTO users (Id, name, is_admin) VALUES (?, ?, ?);", user.ID, user.Name, user.isAdmin);


```
ORM trades simplicity over control

## As Clause in SQL
'AS' clause allows us to "alias" a peice of data in our query.
It only exists for the duration of the query.

```sql

SELECT employee_id AS id, employee_name AS name
FROM employees;

SELECT employee_id, employee_name
FROM employees;

```

## SQL Functions
SQL is a programming language and like nearly all programming languages, it supports functions. We can use functions and aliases to calculate new columns in a query. This is similar to how you might use formulas in Excel.

A calculated column is a new column that doesn't exist in the original
table but is created on the fly when you run a query.

## IFF Function
`IFF` function works like a ternary.

```sql

SELECT quantity,
    IIF(quantity < 10, 'Order more', 'In Stock') AS directive
    FROM products;

```

```sql

IFF(carA > carB, 'Car a is bigge', 'Car b in bigger')

```

## BETWEEN
You can also us WHERE with BETWEEN to pick records in a particular
range.

```sql

SELECT employee_name, salary
FROM employees
WHERE salary BETWEEN 30000 AND 60000;

```

## DISTINCT
Sometimes you need to to return table without duplicates.

```sql

SELECT DISTINCT previous_company FROM employees;

```

## AND
We can use AND  operator to bundle conditions

```sql

SELECT product_name, quantity, shipment_status
    FROM products
    WHERE shipment_status = 'pending'
    AND quantity BETWEEN 0 and 10;

```

## OR
```sql

SELECT product_name, quantity, shipment_status
    FROM products
    WHERE shipment_status = 'out of stock'
    OR quantity BETWEEN 10 and 100;

```

## IN
Another variation to the WHERE clause we can utilize is the `IN` operator.
IN returns true or false if the first operand matches any of the values in the second operand. The IN operator is a shorthand for multiple OR conditions.

```sql

SELECT product_name, shipment_status
    FROM products
    WHERE shipment_status IN ('shipped', 'preparing', 'out of stock');

```

```sql

SELECT product_name, shipment_status
    FROM products
    WHERE shipment_status = 'shipped'
        OR shipment_status = 'preparing'
        OR shipment_status = 'out of stock';

```

## LIKE
LIKE keywords allows us to use '%' and '_' wildcard operator

```sql

## starts with banana
SELECT * FORM products
WHERE product_name LIKE 'banana%';

## ends with banana
SELECT * FORM products
WHERE product_name LIKE '%banana';

# contains banana
SELECT * FORM products
WHERE product_name LIKE '%banana%';

```

## underscore
'_' wildcard operator only matches a *single* character.

## LIMIT
The LIMIT keyword can be used at the end of a select statement  to reduce
the number of records returned

```sql

SELECT * FROM products
  WHERE product_name LIKE '%berry%'
  LIMIT 58;

```

## ORDER BY
SQL also offers us the ability to sort the results of a query
using ORDER BY. The default for this is ASC, but it also supports
DESC.


The below query will returns records in ascending order
and will order the records by price.
```sql

SELECT name, price, quantity FROM products
  ORDER BY price;

```

Here we are still ordering by price but in descending order.
```sql

SELECT name, price, quantity FROM products
  ORDER BY quanity DESC;

```

NOTE: ORDER BY and LIMIT 
When using both ORDER BY must come first

## What Are Aggregations?
An "aggregation" is a single value that's derived by combining
several other values. We perform an aggregation earlier when we used
the COUNT statement to count the number of records in table.

## Why aggregations?
Data stored in database should generally be stored raw. When we need 
to calculate some additional data from the raw data, we can use an
aggregation.


It is simpler to store the products in a single place and run an
aggregation when we need to derive additional information from the 
raw data.


## SUM
The SUM aggregation function returns the sum of a set of values.

So this query below will return sum of all the salary field in
employee table.

```sql

SELECT SUM(salary)
 FROM employees;

```

## MAX 
As you may expect, the MAX function retrieves the *largest* value from
set of values

```sql

SELECT MAX(price) FROM products;

````

## MIN
works same way as MAX but opposite.

```sql

SELECT MIN(price) FROM products;

```

## GROUP BY
There are times we need to group data based on specific values.
SQL offers GROUP BY clause which can group rows that have similar
values in "summary" rows. It returns one row for each group. The interesting
part is that each group can have an aggregate function applied to it
that operates only on the grouped data.

```sql

SELECT album_id, COUNT(song_id)
  FROM songs
  GROUP BY album_id;

```

## AVERAGE
Just like we may want to find the min or max values within a dataset
sometimes we need to know the average

SQL offers us the AVG() function, Similar to MAX() and MIN()

```sql

SELECT AVG(song_length)
  from album;

```

## HAVING
When we need to filter the results of a GROUP BY query even further,
we can use HAVING clause. The HAVING clause specifies a search condition
for a group.

The HAVING clause is similar to the WHERE clause, but it operates on
groups after they've been grouped, rather than rows before they've been
grouped

```sql

SELECT album_id, COUNT(id) as COUNT
  FROM songs
  GROUP BY album_id
  HAVING count > 5;

```
the query returns the album_id and count of its songs, but only
for albums with more than 5 songs.

## HAVING vs WHERE is SQL
The difference in fairly simple.
- A `WHERE` condition is applied to all the data in a query before
it's grouped by a `GROUP BY` clause.
- A `HAVING` condition is only applied to the grouped rows that are
returned after a `GROUP BY` is applied.

This simply means if you want to filter based on the result of an
aggregation, you need to use `HAVING` if you want to filter on a value
that's present in the raw data, you should use `WHERE` clause.

## ROUND
Sometimes we need to round some numbers, particularly when working
with the results of an aggregation. We can use the `ROUND()` function
get the job done.

The SQL `ROUND()` function allows you to specify both the value you 
wish to round and the precision to which  you wish to round it.

```sql

ROUND(value, precision)

```

If no precision is given, SQL will round the value to the nearest
whole value.

```sql

SELECT song_name, ROUND(AVG(song_length), 1)
  FROM songs

```

## Subqueries
Sometime a single query is not enough to retrieve the specific 
records we need.

It is possible to run a query on the *result* set of anther query
a query within a query.

Subqueries can be very useful in a number situations  when trying
to retrieve specific data that wouldn't be accessible by simply
querying a single table.

### Retrieving Data from Multiple Tables
```sql

SELECT id, song_name, artist_id
FROM songs
WHERE artist_id IN (
  SELECT id
  FROM songs
  WHERE artist_name LIKE 'RICK%'
);

```

## Subquery Syntax
The only syntax unique to a subquery is the parentheses surrounding
the nested query. The `IN` operator could be different, for example, we
could use the `=` operator if we expect a single value to be returned.

## No Tables
SQL is a full programming language

```sql

SELECT 5 + 4 as sum;

# 9
```

## Normalization

###  Table relationships
Relational database are powerful because of the relationships between
the tables. These relationships help us to keep to our databases clean 
and efficient.

A relationship between tables assumes that one of these tables has
a FOREIGN KEY that reference PRIMARY KEY of another table.

## Types of Relationships
1 - one to one
2 - one to many
3 - many to many

## One to One
A `One to one` relationships most often manifests as a field  or set
of fields on a row in a table. For example, a `user` will have exactly
one `password`

Settings field might be another example of a one to one relationships
A user will have exactly one `email_preference` and exactly one `birthday`.

## One to Many
When talking about the relationships between *tables*, a one to many
relationship is probably the most commonly used relationship.

A one-to-many relationships occurs when a single record in one table
is related to potentially  many records in another table.

```
The one->many relation only goes one way; a record in the second table
can not be related to multiple records in the first table!

```
Example:
A `customer` table and a `orders` table. Each customer has
`0`, `1` or many order that they've placed.

A `users` table and a `transactions' table. Each `user` has 0, 1 or
many transactions that they've taken part in.

```sql
CREATE TABLE customers (
  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL
);

CREATE TABLE orders (
  id INTEGER PRIMARY KEY,
  amount INTEGER NOT NULL,
  customer_id INTEGER,
  CONSTRAINT fk_customers
  FOREGIN KEY (customer_id)
  REFERENCES customer(id)
);

```

## Many to Many
A many to many relationship occurs when multiple records in one table
can be related to multiple records in another table.

- A products table and suppliers table -
Products may have 0 to many suppliers, and suppliers can supply 0 to
many products.

- A classes table and a students table - Students can take personally
many classes and classes can have multiple students enrolled.

## Joining Table
Joining table help define many to many relationships between data in
a database.
As an example when defining the relationship above between products
and suppliers, we would define a joining table called `product_suppliers`
that contains the primary keys from the tables to be joined.

Then, when we want to see if a supplier supplies a specific product,
we can look in the joining table to see if the ids share a row.

## Unique Constraint Across 2 fields
When enforcing specific schema constraints we may need to enforce the 
`Unique` constraint across two different fields.


```sql

CREATE TABLE product_suppliers(
  product_id INTEGER,
  supplier_id INTEGER,
  UNIQUE(product_id, supplier_id) # this prevents one two fields being the same
);

```
This lets multiple rows share the same product_id or supplier_id, but it prevents any two rows from having both the same product_id and supplier_id.


## Database Normalization
Database normalization is a method for structuring your database schema
in a way that helps:

- improve data integrity
- reduce data redundancy

Data integrity:
"Data integrity" refers to the accuracy and consistency of data. 
For example, if user's age is stored is a database, rather than their
birthday, that data becomes incorrect automatically with the passage of 
time.

It would be better to store birthday and calculate the age when required.



Data redundancy:
Data redundancy occurs when the same piece of data is stored in multiple
places. For example, saving the same file multiple times to different
hard drives.

Data redundancy can be problematic, especially when data in changed 
a place that makes it no longer consistent across all the copies of that 
data.


## Normal Forms
The creator of database normalization, Edgar f. Codd described 
different "normal forms" a database can adhere to.
- First Normal form (1NF)
- Second Normal form (2NF)
- Third Normal form (3NF)
- Boyce-Codd Normal form (1NF)

short version 1NF form is the least "normalized" form, and Boyce-Codd
is the most "normalized" form

The more normalized the better data integrity and less duplication.

## Primary key means something else during database normalization
In the context of database normalization, we're going to use the term "primary key" slightly differently. When we're talking about SQLite, a "primary key" is a single column that uniquely identifies a row.

When we're talking more generally about data normalization, the term "primary key" means the collection of columns that uniquely identify a row. That can be a single column, but it can actually be any number of columns that form a composite key. A primary key is the minimum number of columns needed to uniquely identify a row in a table.

If you think back to the many-to-many joining table product_suppliers, that table's "primary key" was actually a combination of the 2 ids, product_id and supplier_id:

```sql

CREATE TABLE product_suppliers (
  product_id INTEGER,
  supplier_id INEGER,
  UNIQUE(product_id, supplier_id)
);

```

## 1NF
To be compliant with first normal form, a database table simply need 
to follow to rules:
- it must have a unique primary key.
- a cell can't have a nested table as its value (depending on the database
you're using this may not even be possible)

## 2NF
A table in second normal form follows all the rules of 1st normal form
and one additional rule which only applies to composite primary keys.

- All the columns that are not part of the primary keys are dependent
on the entire primary key and not just one of the columns in the primary
keys

What this means, when you have COMPOSITE PRIMARY KEY in a table one column
should not partially be dependent on only of the keys

| first_name | last_name | first_initial |
| --------------- | --------------- | --------------- |
|  Lane | Wagner | l |
| Allen | Small | a |

The primary key in the above table is combination of first_name and 
last_name.

This table does not follow 2NF as the first_initial is dependent on
first_name column

One way to convert the table above to 2NF is to add a new table that
maps a first_name directly to its first_initial.

Let's think it through: In the original table (before 2NF), for every row with the same first_name but different last_name, the first_initial is repeated. For example:

first_name	last_name	first_initial
Lane	Wagner	l
Lane	Small	l
Here, "Lane" maps to "l" twice—imagine thousands of rows!

In 2NF, by making a separate table:

The mapping table (first_name → first_initial) contains only one row per unique first name.
The main table doesn't store first_initial at all, so if "Lane" appears a hundred times as a first name, we only map it to "l" once in the new table.


## 3NF

A table in 3NF follows all the rules of 2NF and one additional rule:
- all the column that aren't part of primary key are dependent solely
on the primary key.

Notice that this is only *slightly* different from second normal form. 
In second normal form we can't have column completely dependent of part
of the primary key and in the 3NF we can't have a column that is entirely
dependent on anything that isn't primary key.

| id | name | first_initial | email
| --------------- | --------------- | --------------- |
| 1 | Lane | l | lane.work@gmail.com |
| 2 | Breana| b | breana.work@gmail.com |
| 3 | Allen | a | allen.work@gmail.com |

This table is in 2NF form because first_initial is not dependent on part
of the primary key, however because it is dependent on the name column
it doesn't adhere to 3NF.

Converting to 3NF is to add new table that maps name directory
to its first_initial

| id | name | email |
| --------------- | --------------- | --------------- |
| 1 | Lane | lane@gmail.com |
| 2 | Breana | breana@gmail.com |



| name | first_initial |
| -------------- | --------------- |
| Lane | l |
| Breana | b |

## Boyce-Codd Normal Form (BCNF)

A table is BCNF follows all the rules of 3NF plus one additional rule:
- A column that's part of primary key can not be entirely dependent on
a column that's not part of that primary key.


This is only comes into play when there are multiple possible primary
key combination that overlap. Another name for this is "overlapping 
candidate keys."


release_year	release_date	sales	name
2001	2001-01-02	100	Kiss me tender
2001	2001-02-04	200	Bloody Mary
2002	2002-04-14	100	I wanna be them
2002	2002-06-24	200	He got me

This table does not follow boyce-codd law as release_year depennds on
release_date


What you can do is make release_year and release_day_and_month

release_year	release_day_and_month	sales	name
2001	01-02	100	Kiss me tender
2001	02-04	200	Bloody Mary
2002	04-14	100	I wanna be them
2002	06-24	200	He got me

## Joins

Joins are the most important features SQL offers. Joins allow us to make
use of the relationships we have set between our tables.
Joins allow us to query multiple tables at the same time.

### Inner Join
The simplest and most common type of join in SQL is the INNER JOIN. By
default JOIN command is an INNER JOIN. An INNER JOIN returns all of 
the records in table_a that have matching records in table_b.



## On
To perform a table join, we need to tell the DB how to "match up" the
rows from each table. The ON clause specifies the column from each
table that should be compared.

When the same column name exists in both tables, we have to specify
which table each column comes from using the table name (or an alias)
followed by a dot '.' before the column name.

```sql

SELECT * 
FROM employees
INNER JOIN departments
ON emplyess.department_id = departments.id;

```

In this query:
- employees.department_id refers to the department_id column from the
employee table.
- departments.id refers to id in the departments table.

ON clause ensures that row are matched based on these columns, creating
a relationship between two tables.

## INNER JOIN
A LEFT JOIN well return every record from table_a regardless of whether
or not  any of those records have a match table_b. A left join will
also return any matching records from table_b.

There is also simple trick for alias

```sql

SELECT e.name, d.name 
FROM employee e
INNER JOIN department d
ON e.department_id = d.id;

```
Notice e and d for employees and department 

## Right Join

A RIGHT JOIN is as you may expect opposite of LEFT JOIN. It returns
from table_b regardless matches, and all matching records between
the two tables.

A RIGHT JOIN is just like LEFT JOIN with the order of the tables
switched so in most cases LEFT JOIN is preferred for readability.


## FULL JOIN
A FULL JOIN combines the result set of LEFT JOIN and RIGHT JOIN commands
It returns all records from both table_a and table_b regardless of whether
or not they have matches.


## SQL Indexes
An index is an in-memory structure that ensure that queries were run on
a database are performant, that is to say they run quickly. If you can 
remember back to the data structure course most indexes are just binary
trees or b-trees. The binary tree can be stored in ram as well as on disk
and it makes it easy to look up the location of an entire row.

PRIMARY KEY columns are indexed by default, ensuring you can look up
a row by its id very quickly. However, if you have other columns that
you want to be able to do quick lookups on, you'll need to index them.

```sql

CREATE INDEX index_name ON table_name (column_name);

```

It is fairly common to name an index after the column it's created on
with suffix of '_idx'.


## Why can't you index every database?
While indexes make specific kinds of lookups much faster, they also add performance overhead - they can slow down a database in other ways. Think about it, if you index every column, you could have hundreds of b-trees in memory! That needlessly bloats the memory usage of your database. It also means that each time you insert a record, that record needs to be added to many trees - slowing down your insert speed.


## Multi column indexes
Multi-Column indexes are useful for the exact reason you might think
they speed up lookups that depend on multiple columns.

```sql

CREATE INDEX first_name_last_name_age_idx
ON users (first_name, last_name, age);

```

A multi-column index is sorted by the first column first, the second column next, and so forth. A lookup on only the first column in a multi-column index gets almost all of the performance improvements that it would get from its own single-column index. However, lookups on only the second or third column will have very degraded performance.

## Rule of thumb
Unless you have specific reasons to do something special, only add
multi-column indexes if you're doing frequent lookups on a specific
combination of columns.

## Denormalizing for Speed
Denormalizing can make your database lookups performant, but you will
have duplicate data that can endup being buggy.


