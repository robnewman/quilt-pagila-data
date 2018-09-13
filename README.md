# quilt-pagila-data

## Goal

Demonstrate how large datasets, which are commonly used in ML and Data Science projects, can be rapidly prepared for analysis using [Quilt](https://www.quiltdata.com) data packages, and how data packages are significantly faster than using a standard Relational Database Management System (RDMS) such as PostgreSQL or MySQL.

## Data Source

We are going to be using data provided by the [Pagila](https://github.com/devrimgunduz/pagila) database which is a PostgreSQL port of the [Sakila example database available for MySQL](https://dev.mysql.com/doc/sakila/en/). The data represents a DVD rental store (remember those!?). It is intended to provide a standard schema that can be used for examples in books, tutorials, articles, samples, etc and is a good standard for benchmarking performance.

## 1. Clone this repository

Use `git clone https://www.github.com/robnewman/quilt-kiva-lending-data` to get this repository on your localhost.

## 2. Clone the Pagila database repository

Use `git clone https://github.com/devrimgunduz/pagila.git` to get the data repository on your localhost.

## 3. Create a relational database and add the data

Now we have the data, we are going to add it to a PostgreSQL database.

### 3.1. Installing PostgreSQL

If don't already have Postgres installed on your localhost, download a native Mac OSX version from https://postgresapp.com/ or from https://www.postgresql.org/ (the official distribution for all platforms). Follow the install instructions.

### 3.2. Add the Pagila data to PostgreSQL

You first need to build the database schema. Run the following from the command line:

`"/Applications/Postgres.app/Contents/Versions/10/bin/psql" -p5432 -d "postgres" < /path/to/pagila/pagila-schema.sql`

and:

`"/Applications/Postgres.app/Contents/Versions/10/bin/psql" -p5432 -d "postgres" < /path/to/pagila/pagila-insert-data.sql`

where `/path/to/pagila` is your localhost directory path to the Pagila database that you just downloaded.

We now have a complete database that contains the following tables:

![alt text][pagila_erd]

### 3.3. Run some simple queries

## 4. Create a Quilt package

## References

[pagila_erd]: https://github.com/robnewman/quilt-pagila-data/raw/master/src/assets/images/dvd-rental-sample-database-diagram.png "Pagila Entity Relationship Diagram"
