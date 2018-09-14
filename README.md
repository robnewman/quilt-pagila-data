# quilt-pagila-data

## Goal

Demonstrate how large datasets, which are commonly used in ML and Data Science projects, can be rapidly prepared for analysis using [Quilt](https://www.quiltdata.com) data packages, and how data packages are significantly faster than using a standard Relational Database Management System (RDMS) such as PostgreSQL or MySQL.

## Data Source

We are going to be using data provided by the [Pagila](https://github.com/devrimgunduz/pagila) database which is a PostgreSQL port of the [Sakila example database available for MySQL](https://dev.mysql.com/doc/sakila/en/). The data represents a DVD rental store (remember those!?). It is intended to provide a standard schema that can be used for examples in books, tutorials, articles, samples, etc and is a good standard for benchmarking performance.

## 1. Clone this repository

Use `git clone https://www.github.com/robnewman/quilt-kiva-lending-data` to get this repository on your localhost.

### 1.1 Build and activate a new virtual environment

To build the virtual environment (called `quilt-py3`) with all the packages used in this post, you need to run the following commands (assuming you have installed Python3 using the [Homebrew](http://brew.sh/) package manager):

First, build a new virtual environment called `quilt-py3`:

`$ python3 -m venv ~/path/to/virtual/environments/quilt-py3`

Next, activate the new environment:

`$ source /path/to/virtual/environments/quilt-py3/bin/activate`

(Depending on how your shell is configured, your command line prompt may automatically update with the virtual environment name as a prefix)

### 1.2. Check that you have a Python3 interpreter installed

```
[quilt-py3] $ python3 --version
Python 3.6.5
```

### 1.3. Install Python packages

Finally, install all the Python packages from the `requirements.txt` file in this repository:

`pip install -r requirements.txt`

Now you have all you need to work through the rest of this post, _except_ for the data!

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

![Pagila ERD][pagila_erd]

### 3.3. Run some simple queries

```
SELECT
 customer.customer_id,
 customer.first_name customer_first_name,
 customer.last_name customer_last_name,
 customer.email,
 staff.first_name staff_first_name,
 staff.last_name staff_last_name,
 amount,
 payment_date
FROM
 customer
INNER JOIN payment ON payment.customer_id = customer.customer_id
INNER JOIN staff ON payment.staff_id = staff.staff_id;
```
```
customer_id | customer_first_name | customer_last_name |                  email                   | staff_first_name | staff_last_name | amount |         payment_date          
-------------+---------------------+--------------------+------------------------------------------+------------------+-----------------+--------+-------------------------------
        269 | CASSANDRA           | WALTERS            | CASSANDRA.WALTERS@sakilacustomer.org     | Jon              | Stephens        |   1.99 | 2017-01-24 21:40:19.996577-08
        269 | CASSANDRA           | WALTERS            | CASSANDRA.WALTERS@sakilacustomer.org     | Mike             | Hillyer         |   0.99 | 2017-01-25 15:16:50.996577-08
        269 | CASSANDRA           | WALTERS            | CASSANDRA.WALTERS@sakilacustomer.org     | Jon              | Stephens        |   6.99 | 2017-01-28 21:44:14.996577-08
        269 | CASSANDRA           | WALTERS            | CASSANDRA.WALTERS@sakilacustomer.org     | Jon              | Stephens        |   0.99 | 2017-01-29 00:58:02.996577-08
        269 | CASSANDRA           | WALTERS            | CASSANDRA.WALTERS@sakilacustomer.org     | Mike             | Hillyer         |   4.99 | 2017-01-29 08:10:06.996577-08
        269 | CASSANDRA           | WALTERS            | CASSANDRA.WALTERS@sakilacustomer.org     | Jon              | Stephens        |   2.99 | 2017-01-31 12:23:14.996577-08
        270 | LEAH                | CURTIS             | LEAH.CURTIS@sakilacustomer.org           | Mike             | Hillyer         |   1.99 | 2017-01-26 05:10:14.996577-08
```

### 3.4. Add some benchmarking tools to the SQL prompt

You can do this in a couple of ways:

*Add `EXPLAIN ANALYZE` as the first line to your SQL statement:*

```
EXPLAIN ANALYZE
SELECT
 customer.customer_id,
 customer.first_name customer_first_name,
 customer.last_name customer_last_name,
 customer.email,
 staff.first_name staff_first_name,
 staff.last_name staff_last_name,
 amount,
 payment_date
FROM
 customer
INNER JOIN payment ON payment.customer_id = customer.customer_id
INNER JOIN staff ON payment.staff_id = staff.staff_id;
```

The output now starts:

```
QUERY PLAN                                                              
-------------------------------------------------------------------------------------------------------------------------------------
Hash Join  (cost=40.13..427.17 rows=16995 width=127) (actual time=3.760..29.513 rows=15635 loops=1)
Hash Cond: (payment_p2017_01.staff_id = staff.staff_id)
->  Hash Join  (cost=22.48..364.35 rows=16995 width=65) (actual time=3.582..21.913 rows=15635 loops=1)
Hash Cond: (payment_p2017_01.customer_id = customer.customer_id)
->  Append  (cost=0.00..296.95 rows=16995 width=18) (actual time=0.054..10.099 rows=15635 loops=1)
->  Seq Scan on payment_p2017_01  (cost=0.00..20.26 rows=1126 width=18) (actual time=0.053..0.682 rows=1126 loops=1)
->  Seq Scan on payment_p2017_02  (cost=0.00..40.12 rows=2312 width=18) (actual time=0.053..1.441 rows=2312 loops=1)
->  Seq Scan on payment_p2017_03  (cost=0.00..98.44 rows=5644 width=18) (actual time=0.074..2.948 rows=5644 loops=1)
->  Seq Scan on payment_p2017_04  (cost=0.00..110.71 rows=6371 width=18) (actual time=0.033..2.861 rows=6371 loops=1)
->  Seq Scan on payment_p2017_05  (cost=0.00..3.82 rows=182 width=17) (actual time=0.046..0.108 rows=182 loops=1)
->  Seq Scan on payment_p2017_06  (cost=0.00..23.60 rows=1360 width=24) (actual time=0.013..0.013 rows=0 loops=1)
->  Hash  (cost=14.99..14.99 rows=599 width=49) (actual time=3.492..3.492 rows=599 loops=1)
Buckets: 1024  Batches: 1  Memory Usage: 57kB
->  Seq Scan on customer  (cost=0.00..14.99 rows=599 width=49) (actual time=0.055..0.510 rows=599 loops=1)
->  Hash  (cost=13.40..13.40 rows=340 width=68) (actual time=0.099..0.099 rows=2 loops=1)
Buckets: 1024  Batches: 1  Memory Usage: 9kB
->  Seq Scan on staff  (cost=0.00..13.40 rows=340 width=68) (actual time=0.072..0.074 rows=2 loops=1)
Planning time: 6.699 ms
Execution time: 32.699 ms
(19 rows)
```

*Add the `timing` functionality to the PostgreSQL prompt:*

```
postgres=# \timing
Timing is on.
```

```
customer_id | customer_first_name | customer_last_name |                  email                   | staff_first_name | staff_last_name | amount |         payment_date          
-------------+---------------------+--------------------+------------------------------------------+------------------+-----------------+--------+-------------------------------
        269 | CASSANDRA           | WALTERS            | CASSANDRA.WALTERS@sakilacustomer.org     | Jon              | Stephens        |   1.99 | 2017-01-24 21:40:19.996577-08
        ....        
        284 | SONIA               | GREGORY            | SONIA.GREGORY@sakilacustomer.org         | Jon              | Stephens        |   0.99 | 2017-01-29 14:59:08.996577-08
        286 | VELMA               | LUCAS              | VELMA.LUCAS@sakilacustomer.org           | Jon              | Stephen
Time: 83.096 ms
```

## 4. Query the database using a notebook

Everything prior to now has been correctly setting up our environment and getting the data loaded locally. Now we are going to actually _work like a data engineer_ and pull in some real data from the PostgreSQL Pagila database and work with it in a Jupyter notebook.

### 4.1. Make sure your notebook is using your virtual environment kernel

To ensure that all the Python packages that you are using from your virtual environment (my current one is called `quilt-py3`, see above) are available in your notebook, you need to add it to the notebook kernel.

`$ ipython kernel install --user --name=quilt-py3`

At this point, you can start Jupyter, create a new notebook and select the new kernel for your environment using the Kernel menu  > Change kernel (red box) > quilt-py3 (blue box):

![Notebook Kernel Selection][notebook_kernel]:

You will see the kernel update in the top-right of the notebook (green box).

## 5. Create a Quilt package

## References

[pagila_erd]: https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/dvd-rental-sample-database-diagram.png "Pagila Entity Relationship Diagram"

[notebook_kernel]: https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/notebook_kernel.png "Selecting the notebook kernel"
