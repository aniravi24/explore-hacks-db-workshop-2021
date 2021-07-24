---
title: 'Introduction to Database Design'
author: 'Ani'
date: 2021-07-24
patat:
  eval:
    sql_eval:
      command: PGPASSWORD="password" psql -U postgres -h localhost -p 5432 -d explorehacks2021
      fragment: true
      replace: true
  slideLevel: 2
  wrap: true
  margins:
        left: 10
        right: 10
  images:
    backend: auto
...

# Why the Database is All You Are

## The goals for this workshop

. . .

To explain why a database is important.

. . .

To showcase some basic database operations.

. . .

Show some more cool advanced stuff that is possible, but I will hand-wave a bit as there is too much to cover. I will leave it to you all to take a look at the resources in my appendix.

. . .

Give you a starting point, not make you an expert.

## What is a Database

. . .

A place to store data.

. . .

We need some mechanism to store data in a persistent manner.

. . .

Let's look at what forms a database could be in and what you would put in it.


## Types of "databases"

Your file system:

 - stores files
 - could be a text file, document, game, or some other program

. . .

A library:

 - database of books
 - organized in a particular way so you can find the books later

. . .


## What kinds of things would you put in a database?

Medical Records

. . .

Money Movements

. . .

Messages from loved ones

. . .

McDonalds locations where the ice cream machine works

(h/t <https://mcbroken.com/>)

## Why is it important to get data storage right?

May have extremely sensitive or vulnerable data

If you get it wrong, you may lose important data or have incorrect data

Data is a set of facts. It makes it harder for you to make decisions or automate things if your data is wrong

## Don't ignore the database

A trend in the web development ecosystem is to push aside database concerns

. . .

In reality, it's the most important part of your app to get right

. . .

Your business, project, and your users depend on this being accurate, possibly for their livelihoods

. . .

Databases are not "dummy data stores". They are full-fledged applications

# Intro to PostgreSQL

## What is PostgreSQL (often called Postgres)?

>  PostgreSQL, also known as Postgres, is a free and open-source relational database management system emphasizing extensibility and SQL compliance.

. . .

_Thanks Wikipedia_

. . .

### What does this mean?

All you need to know for now is:

 - Postgres is free to use and is open-source
 - Postgres is an application that can hold multiple databases
 - Postgres is a "relational" database management system and supports something called SQL

## What is SQL

SQL is a language to talk to relational databases.

A relational database, roughly, is a database that stores data with pre-defined relationships.

Following the relational model proposed by Edgar Codd in 1970.

## How is data stored in SQL?

Data is stored in tables with rows and columns.

Ever seen a spreadsheet? Just like that! A bit more complicated though..

## What do I need to know for this workshop?

You need to know 5 operations.

CREATE, INSERT, SELECT, UPDATE, DELETE

Last 4 are often called (CRUD - Create, Read, Update, Delete)

The official categories/names for these operations can be found in the Terminology Appendix.

# Code time!

## Creating a database and the first table

~~~~~{.sql}
CREATE DATABASE explorehacks2021;
~~~~~

. . .

~~~~~{.sql}
CREATE TABLE IF NOT EXISTS users (
    id serial PRIMARY KEY,
    first_name text,
    last_name text NOT NULL,
    email text NOT NULL UNIQUE,
    password text NOT NULL
);
~~~~~

SQL is NOT cAsE-SeNsItIvE. I uppercase keywords to make them easier to see.

Syntax for the column definitions are _<column_name>_ _<data_type>_ _<other_options>_. It can get more sophisticated than this but we will start here.

## Let's run it!

~~~~~{.sql .sql_eval}
CREATE TABLE IF NOT EXISTS users (
    id serial PRIMARY KEY,
    first_name text,
    last_name text NOT NULL,
    email text NOT NULL UNIQUE,
    password text NOT NULL
);
~~~~~

Congrats! We just created our first table.

## Now let's insert a row into that table.

The syntax is _INSERT INTO_, followed by parenthesized list of columns, _VALUES_, followed by data in the same order as the columns.

Notice whitespace doesn't matter. It might be easier to read this way.

~~~~~{.sql .sql_eval}
INSERT INTO users (first_name, last_name, email, password)
VALUES ('ani', 'ravi', 'ani@todo.com', 'NEVER DO THIS')
RETURNING *;

INSERT INTO users (first_name, last_name, email, password)
VALUES ('Jane', 'Doe', 'jane@doe.com', 'NEVER DO THIS')
RETURNING *;
~~~~~

You do not need the _RETURNING_ _\*_ in the query. That is just so we can view the data after it's inserted.

By default, postgres will only show you the number of rows inserted, not what was inserted.

## Important PSA on passwords

Never, ever, store passwords in plaintext like we did on the last slide.

Always hash your passwords in the application code or using database functions (application code is preferred).

Some popular hashing algorithms are _BCrypt_ and _Argon2_. Most languages have libraries that can do this for you given a string.

## Now let's select the rows we just inserted.

The syntax is SELECT <identifiers> FROM <table_name>.

~~~~~{.sql .sql_eval}
SELECT * from users;
~~~~~

## Let's update all the users!

The syntax is UPDATE <table_name> SET <column> = <value>.

~~~~~{.sql .sql_eval}
UPDATE users SET first_name = 'Dhanish' RETURNING *;
~~~~~

Oh no! We just updated every user! How can we prevent this?

## RIP

SHOW GIF PLZ

## The WHERE clause

The WHERE keyword is one of the most important keywords you will use in your operations.

. . .

It helps SQL figure out what conditions a row has to meet before it will show it to you.


## Let's select only the user with id 1.

~~~~~{.sql .sql_eval}
SELECT * from users WHERE id = 1;
~~~~~

## Let's only update the user with id 1.

~~~~~{.sql .sql_eval}
UPDATE users SET first_name = 'Ani' WHERE id = 1;
~~~~~

## Delete me!

~~~~~{.sql .sql_eval}
DELETE FROM users WHERE id = 1;
~~~~~

Never delete without a where clause. It will delete every row in that table.

If you ACTUALLY want to do that, use _TRUNCATE <table_name>_ instead.

If you want to get rid of the table altogether, you can use _DROP TABLE <table_name>_.

## Let's create one more table that references the users table

~~~~~{.sql .sql_eval}
CREATE EXTENSION IF NOT EXISTS pgcrypto;
~~~~~

Notice the REFERENCES keyword. This creates a _foreign key_.

~~~~~{.sql .sql_eval}
CREATE TABLE books (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id int NOT NULL REFERENCES users,
  title text NOT NULL,
  subtitle text
);
~~~~~

## Foreign keys

Similar to a spreadsheet, instead of having to copy information from different rows, you can directly reference the column.

. . .

It is considered _good database design practice_ to use foreign keys to refer to data, rather than repeating data.

. . .

This is called _normalization_ (splitting out common data pieces into their own tables, and not duplicating it across tables).

See appendix for more on normalization rules and the different _normal forms_.

## Let's insert a row into the books table.

Notice we are not inserting the id ourselves. The database is doing it for us because we defined the default.

~~~~~{.sql .sql_eval}
INSERT INTO books (user_id) VALUES (1);
~~~~~

> ERROR:  null value in column "title" violates not-null constraint
> DETAIL:  Failing row contains (920214b1-c385-47a8-a586-f0d2b7320aa9, 1, null, null).

The DB caught us! Wow!

## Importance of correct constraints

If you get your constraints correct, then the database can prevent us from making mistakes.

It also makes it easier for anyone looking to know how your data is structured.

. . .

I cannot describe how valuable this is in production when users will break your application in every which way.

## Let's insert a row into the books table (correctly this time).

~~~~~{.sql .sql_eval}
INSERT INTO books (user_id, title)
VALUES (1, 'Explore Hacks 2021 Epic')
RETURNING *;
~~~~~

I know it looks weird. UUIDs are big.

## Reflection

Let's think about what our database looks like now.

. . .

We have a users table. We have a books table that references the users table.

Whenever a book is inserted, we know that each book is tied to a user.

. . .

This isn't modeled well though - why would a book only be tied to one user?

## Many-to-Many

There's several types of relationships that tables can have with each other.

- One-to-One
- One-to-Many
- Many-to-One
- Many-to-Many

. . .

If we think about the relationship between a user and a book, a user can have many books, and a book could have many users..

## Many-to-Many

This is accomplished with what's called a _junction table_ (goes by several other names as well).

Notice how we only used foreign keys to model this relationship.

And there's a primary key that's not only on one column - it's two columns together! (Known as a _composite key_).

~~~~~{.sql .sql_eval}
CREATE TABLE IF NOT EXISTS users_books (
 user_id int REFERENCES users,
 book_id uuid REFERENCES books,
 PRIMARY KEY (user_id, book_id)
);
~~~~~

## Inserting the relationship between users and books

See that subquery? Pretty cool right?

~~~~~{.sql .sql_eval}
INSERT INTO users_books (user_id, book_id)
VALUES (1, (SELECT id FROM books LIMIT 1))
RETURNING *;
~~~~~

This gives us a way to know which users have which books (and vice-versa).

The way we'd query this out would be through what's called a _JOIN_, one of the most powerful tools in SQL to make use of these relationships.

## Unfortunately covering JOINs will take a while.

There are many different types of joins, to join tables in different ways. I'd

Here's a small taste of what you can do.

~~~~~{.sql .sql_eval}
SELECT u.id, u.first_name AS "First Name", b.title AS "Book Title"
FROM users u
JOIN users_books ub ON u.id=ub.user_id
JOIN books b ON b.id=ub.book_id;
~~~~~

## Cleaning up the book table

Let's clean up that extra column we created on the book table.

~~~~~{.sql .sql_eval}
ALTER TABLE books DROP COLUMN user_id;
~~~~~

## Postgres can do A TON

A not even close to exhaustive list on what's possible...

- Can handle JSON data natively (can query it, update it, index on it, and much more)

. . .

- Supports dozens of datatypes, everything from date ranges to IP addresses to geospatial data (with PostGIS)

. . .

- Can extend functionality via extensions (think of them like plugins)

. . .

- With these extensions, you can do everything from write code in other programming languages (e.g. Python, JavaScript) to connecting to foreign data in other databases (e.g. MongoDB, Redis, MySQL), or even send an email... maybe 😉

. . .

- Can handle geospatial data, being able to take a latitude/longitude and do queries on it, such as "find me the nearest coffee shop from my location" (directly in Postgres!)

. . .

- ACID compliant, has transactions (incredibly valuable, we use this everywhere), and has MVCC (Multi-Version Concurrency Control)

. . .

- Handle permissions directly in the database, with multiple users, and have multiple schemas or databases to separate tables

. . .

- Related tools that give you cool functionality on top of Postgres (e.g. PostGraphile or Hasura, which let you generate a GraphQL API on top of your database)

See Appendix for more of these examples (like how someone basically built something like Google Maps routing in Postgres).

## There's a lot of basics we didn't cover on these slides

We didn't discuss indexes, or get deep into joins.

We didn't cover how to modify the schema or data once it's in the database.

We didn't cover the many functions available in Postgres to do everything from trimming strings to turning rows into json.

We didn't cover how companies manage changes to their schema, usually with something called a migration.

See the examples section in the appendix to see some real migrations that were run in production. We basically wrote a part of a migration today!


## Final demos

You've stared at a screen long enough..

. . .

Let's see what's possible visually with some cool tools and data that's available!

We'll use Metabase to visualize some data and we can look at some geospatial data on a map, to get a sense of how cool this can be.

For example..

lat: 33.4457415,
lng: -112.073389

Want to figure out where this is? What does this query do?

~~~~~{.sql}
SELECT name, a.address_line_1
FROM property p
JOIN address a ON p.address_id=a.id
ORDER BY a.geom <-> ST_SetSRID(ST_MakePoint(-112.073389,33.4457415),4326)
LIMIT 10;
~~~~~

## The final step

All good things must come to an end...

~~~~~{.sql}
DROP DATABASE explorehacks2021;
~~~~~

# Thank you for participating! Any Questions?

# Appendix

## Installation

Installing Postgres:
<https://www.postgresqltutorial.com/install-postgresql/>

Installing and Running Metabase:

<https://www.metabase.com/docs/latest/operations-guide/running-the-metabase-jar-file.html>

## Tutorials

<https://www.postgresqltutorial.com/> Hands down one of my favorite resources, you can go step by step and learn a ton. It's a good reference and will get you far with the basics.

<https://hasura.io/learn/database/postgresql/introduction/> Good introduction into Postgres, covers more of the basics than we could in this workshop!

## Terminology

<https://www.youtube.com/watch?v=y1tcbhWLiUM> Basics of DB design

<http://putham.com/post/2/What+is+DDL,+DML,+DCL+and+TCL+in+sql+,+oracle+,+postgresql?>

<https://www.sisense.com/blog/sql-query-order-of-operations/> Explains which order the keywords are evaluated in.

<https://www.guru99.com/database-normalization.html> Normalization

<https://retool.com/blog/whats-an-acid-compliant-database/> ACID

<https://devcenter.heroku.com/articles/postgresql-concurrency> MVCC

## Examples

<https://github.com/buzzwordlabs/foodfeed-public/tree/main/server/migrations/committed> (Notice all the migrations have camelcased columns. This is a tradeoff we made working with TypeScript because everything in JavaScript is camelcased. More tedious to write SQL queries, easier to use in your backend code. All columns and usages of those columns in queries need to be double quoted so that SQL doesn't automatically lowercase it.)

## Indexes

<https://www.citusdata.com/blog/2017/10/17/tour-of-postgres-index-types/>

<https://www.citusdata.com/blog/2017/10/11/index-all-the-things-in-postgres/>

<https://www.youtube.com/watch?v=HAn1xu6_SW0> (a talk about indexes)

<https://gist.github.com/popravich/d6816ef1653329fb1745> (index naming convention)

## Performance

<https://www.citusdata.com/blog/2017/09/29/what-performance-can-you-expect-from-postgres>

<https://youtu.be/5M2FFbVeLSs>

<https://www.craigkerstiens.com/2013/01/10/more-on-postgres-performance/>

<https://medium.com/braintree-product-technology/postgresql-at-scale-database-schema-changes-without-downtime-20d3749ed680>

## PostGIS

<https://gis.stackexchange.com/questions/131363/choosing-srid-and-what-is-its-meaning>

<http://duspviz.mit.edu/tutorials/intro-postgis/>

<https://www.percona.com/blog/2020/04/15/working-with-postgresql-and-postgis-how-to-become-a-gis-expert/>

<https://www.bostongis.com/PrinterFriendly.aspx?content_name=postgis_tut01>

<https://blog.daftcode.pl/find-your-way-with-the-power-of-postgis-pgrouting-66d620ef201b>


## Tools/Software

<https://www.pgadmin.org/> DB management tool, comes with Postgres installation typically and the defacto tool

<https://dbeaver.io/> DB management tool, one I've enjoyed using and lots of functionality available

<https://tableplus.com/> Simpler DB management tool with better UI/UX and easier to use for most developers, but less features available overall and limited functionality without paying for it.

<https://github.com/graphile/migrate> I have tried several migration tools. This is the best one I've found for getting started, even for growing companies!

<https://github.com/dhamaniasad/awesome-postgres>

<https://www.graphile.org/postgraphile/> Generate GraphQL API on top of PostgreSQL database (I learned a lot about SQL by reading the postgraphile docs!)

<https://hasura.io/> Generate GraphQL API on top of PostgreSQL database

## Misc

<https://medium.com/hackernoon/how-to-query-jsonb-beginner-sheet-cheat-4da3aa5082a3> (beginner jsonb queries)

<https://stackoverflow.com/questions/11352056/postgresql-composite-primary-key>

<https://leopard.in.ua/2016/09/20/safe-and-unsafe-operations-postgresql>

<https://gist.github.com/checco/40960cd7e993e568e4bd4ede27a25197> (postgres RO, read-only, read only user)

<https://gist.github.com/esfand/9444313> (lookup table)

<https://dba.stackexchange.com/questions/97963/how-to-build-a-table-for-a-private-messaging-system-that-supports-replies> (messaging system for replies)

Column ordering matters
<https://www.2ndquadrant.com/en/blog/on-rocks-and-sand/>

<https://dba.stackexchange.com/questions/183119/modelling-a-database-structure-for-multiple-user-types-and-their-contact-informa> (user types and contact info)

<https://aniravi.com/posts/move-off-orms-in-typescript.html> My blog post on ORMs in JS/TS

<https://aniravi.com/posts/your-database-is-all-your-are.html> My blog post on databases

## Credits

Jasper for the cool Haskell program that makes this presentation <https://github.com/jaspervdj/patat>

Cool Retro Term for being the terminal that I ran this presentation in to make it look retro! <https://github.com/Swordfish90/cool-retro-term>