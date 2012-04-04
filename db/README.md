# Relational Databases and SQL

# Data Definition Language (DDL)
The data definition language (DDL) for SQL allows for the manipulation
of database *tables*. In particular, it provides language support for
*creating*, *altering*, and *deleting* tables in a relational
database. To create a table we use the `CREATE` statement:

    CREATE TABLE <table name> (<attribute list>);

The `CREATE` statement constructs a new table named `<table name>`
with the attributes (and their types) specified in the `<attribute
list>`. An example usage would be the following:

    CREATE TABLE sailors (
        int sid,
        sname varchar(50),
        rating int,
        age real
    );

Here we create a new table named `sailors` with the attributes `sid`,
`sname`, `rating`, and `age`. We use the SQL data types `int`,
`varchar(50)`, `int`, and `real` respectively. There are many data
types available in SQL. Although SQL is meant to be a universal
language that applies to all relational databases, this is
unfortunately not the case. For example, here is a list of
[general data types][general-dt] and here is a list of
[data types that are specific to the PostgreSQL][postgres-dt] database
platform. There is significant overlap, however, many database
implementations provide special data types for their own platform.

If we have the need to make a change to an existing table we can use
the `ALTER` statement:

    ALTER TABLE <table name> ADD COLUMN <attribute>;

So, if we would like to add a new attribute to the `sailors` table for
recording the home town of a sailor we can do this:

    ALTER TABLE sailors ADD COLUMN town int;

Lastly, if we want to delete a table we can use the `DROP`
statement:

    DROP TABLE <table name>;

This has the effect of deleting the table named `<table name>`
including all the records stored in that table.

An important aspect of relational databases is to ensure the
*integrity* of the data stored in its tables. In particular, it is
possible to apply *constraints* to a table to ensure that any
update to the data contained within the table satisfies the imposed
constraints. Three of the most often used constraints are ensuring
that a particular attribute can't be *NULL*, identifying a *primary
key*, and indicating that a set of attributes must be *unique*. 

`NULL` has special significance in a relation database it represents
an *unknown value*. There are times when we do want to use `NULL` to
represent missing information. For example, we might not know the
*rating* of a sailor when the sailor is added to the
database. However, it is probably important that the unique identifier
for a sailor can never be `NULL`. To impose this constraint on the
sailors table we would do the following:

    CREATE TABLE sailors (
        int sid not null,
        sname varchar(50),
        rating int,
        age real
    );

This would ensure that it is impossible for us to add data to this
table that had a missing `sid` value. In fact, the database would
report an error if an attempt was made. A second constraint is to
indicate the *primary key* of a table. The primary key represents the
smallest set of attributes that uniquely identify a row in a
table. Here is an example of specifying that the `sid` attribute of
the sailors table should be a primary key:

    CREATE TABLE sailors (
        int sid not null,
        sname varchar(50),
        rating int,
        age real,
        primary key (sid)
    );

Specifying the primary key may or may not cause the database system to
enforce a uniqueness constraint on that attribute. In most cases,
however, it is a *hint* to the database system to construct a *index*
on that attribute to increase performance. An index is simply a
mechanism to improve performance by building a data structure on top
of the table that improves lookup performance on the primary key. To
enforce a *uniqueness* constraint on table attributes we can do this:

    CREATE TABLE sailors (
        int sid not null,
        sname varchar(50),
        rating int,
        age real,
        primary key (sid),
        unique (sname)
    );

In this example we use `unique (sname)` to indicate that we do not
allow duplicate `sname` values in the `sailors` table.

It is often the case where we want to have the database automatically
provide a unique identifier for a table. For example, in the sailors
table we want to identify a sailor using its `sid`. It is not
important what the `sid` is, rather it is important that it be
different for each record in the table. Unfortunately, different
database vendors have their own approach to providing this
functionality. For PostgreSQL we create a [sequence][sequence] and use
it to generate a unique value each time we insert a record into the
database. Here is an example:

    CREATE SEQUENCE sailors_sid_seq;
    CREATE TABLE sailors (
        int sid not null default nextval('sailors_sid_seq'),
        sname varchar(50),
        rating int,
        age real,
        primary key (sid),
        unique (sname)
    );

When we add a new record to the `sailors` table it will automatically
choose a unique next value as the default value for the `sid` field.

In addition to a *primary key* we may also want to impose a *foreign
key* constraint. A *foreign key* is an attribute that is used to
*reference* the primary key of another table. This is the fundamental
technique in relational databases for linking information between
multiple tables. To add a foreign key constraint we would do the
following:

    CREATE TABLE reserves (
      sid int,
      bid int,
      day date,
      foreign key (sid, bid) references sailors(sid), boats(bid);
    );

# Data Manipulation Language (DML)
## Record Updates
After we have created our tables we are ready to add records. To do
this we use the *data manipulation language* (DML) part of SQL. There
are three modifications that are interesting: *insert*, *delete*, and
*update*. To insert a new record into a table we use the `INSERT`
statement:

    INSERT INTO <table name> VALUES (<value list>);

An example of inserting a record into the `sailors` table is:

    INSERT INTO sailors VALUES (default, 'caleb', 12, 6);
    
This will insert a new record into the `sailors` table with the given
value list. To delete a record we would do the following:

    DELETE FROM sailor WHERE sname='horatio';
    
This would delete the record in sailor for the sailor named
horatio. To update a record we use the `UPDATE` statement:

    UPDATE sailor SET rating=20 WHERE sname='caleb';

## Querying the Database
### Select-From-Where
The basic form used for querying data in a relational database is
referred to as the *select-from-where* clause. Its general form is:

    SELECT [DISTINCT] <attribute-list>
    FROM   table-list
    WHERE  conditions;

An example usage of this is:

    SELECT sid, sname, rating
    FROM   sailors
    WHERE  age > 21;

Because a table may include duplicate table entries we may want to
exclude duplicates using the `DISTINCT` qualifier:

    SELECT DISTINCT sid, sname, rating
    FROM   sailors
    WHERE  age > 21;

Because we enforced a uniqueness constraint in the definition of the
sailors table we ensure that the table will only contain unique
entries. So, in this particular case the use of `DISTINCT` is not
necessary.

### Aggregate Functions
SQL provides a number of useful *aggregate functions* for computing a
value based on the records contained in a table. The typical aggregate
functions include:

- `COUNT(*)`
- `COUNT ([DISTINCT] <attribute>)`
- `SUM ([DISTINCT] <attribute>)`
- `AVG ([DISTINCT] <attribute>)`
- `MAX ([DISTINCT] <attribute>)`
- `MIN ([DISTINCT] <attribute>)`

An example of counting the number of rows in a table is:

    SELECT COUNT(*)
    FROM   sailors
    WHERE  rating=4;

To get the average age we might do the following:

    SELECT AVG(age)
    FROM   sailors;

### Group By / Having
It is often the case that you want to query a table by grouping on an
attribute. For example, perhaps we want the average age of sailors
grouped by rating where the sum of their ages exceeds 10:

    SELECT rating, AVG(age)
    FROM sailors 
    GROUP BY rating 
    HAVING SUM(age) > 10;

This might return the following if executed using the `psql` tool in
PostgreSQL:

     rating  |       avg        
     --------+------------------
          10 |             25.5
           8 |             40.5
           7 |               40
           3 | 38.1666666666667
           2 |               53
           1 |               33
    (6 rows)


[general-dt]: http://www.w3schools.com/sql/sql_datatypes.asp
[postgres-dt]: http://www.postgresql.org/docs/7.4/static/datatype.html
[sequence]: http://www.postgresql.org/docs/8.1/static/sql-createsequence.html
