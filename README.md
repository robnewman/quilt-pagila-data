# quilt-pagila-data

## Goal

1. Clearly demonstrate how small- to large-datasets, which are commonly used in ML and Data Science projects, can be rapidly and easily prepared for analysis using [Quilt](https://www.quiltdata.com) data packages.
2. How Quilt data packages are **significantly** faster to load data into Panda's dataframes than using a direct query to a standard Relational Database Management System (RDMS) such as PostgreSQL or MySQL.

## Data Source

We are going to be using data provided by the [Pagila](https://github.com/devrimgunduz/pagila) database which is a PostgreSQL port of the [Sakila example database available for MySQL](https://dev.mysql.com/doc/sakila/en/). The data represents a DVD rental store (remember those!?). It is intended to provide a standard schema that can be used for examples in books, tutorials, articles, samples, etc and is a good standard for benchmarking performance.

## 1. Clone this repository

Use `git clone https://www.github.com/robnewman/quilt-pagila-data` to get this repository on your localhost.

## 1.1 Project directory structure

Data engineering projects need a sensible (and repeatable) directory structure. This repository follows the recommendations from [Cookie Cutter Data Science](http://drivendata.github.io/cookiecutter-data-science/#directory-structure):

```
├── LICENSE
├── README.md
├── assets
├── data
│   ├── external
│   ├── interim
│   ├── processed
│   └── raw
├── notebooks
├── packages
├── requirements.txt
└── src
    └── data
```

There are a couple of minor differences to the recommendation:

1. `assets` is where all the images and other assets for this repository are stored.
2. `packages` is where we will be storing the Quilt data packages that we generate and work with.

The `data/raw` directory is where we are storing all the data files for building the database we will use during this post.

The `data/interim` directory is where we are dumping the CSV files created by our SQL queries within Postgres.

### 1.1. Build and activate a new virtual environment

To build the virtual environment (called `quilt-py3`) with all the packages used in this post, you need to run the following commands (assuming you have installed Python3 using the [Homebrew](http://brew.sh/) package manager):

First, build a new virtual environment called `quilt-py3`:

```bash
$ python3 -m venv ~/path/to/virtual/environments/quilt-py3
```

Next, activate the new environment:

```bash
$ source /path/to/virtual/environments/quilt-py3/bin/activate
```

(Depending on how your shell is configured, your command line prompt may automatically update with the virtual environment name as a prefix)

### 1.2. Check that you have a Python3 interpreter installed

```bash
[quilt-py3] $ python3 --version
Python 3.6.5
```

### 1.3. Install Python packages

Finally, install all the Python packages from the `requirements.txt` file in this repository:

```bash
[quilt-py3] pip install -r requirements.txt
```

Now you have all you need to work through the rest of this post, _except_ for the data!

## 2. Clone the Pagila database repository

Use `git clone https://github.com/devrimgunduz/pagila.git` to get the data repository on your localhost. Move the files to the `data > raw` directory, so that the contents of the directory look like:

```
data/raw
└── pagila
    ├── pagila-0.10.1
    │   ├── README
    │   ├── pagila-data.sql
    │   ├── pagila-insert-data.sql
    │   └── pagila-schema.sql
    └── pagila-0.10.1.zip
```

## 3. Create a relational database and add the data

Now we have the raw data, we are going to use it to create a tables in the default local PostgreSQL database called `postgres`.

### 3.1. Installing PostgreSQL

If don't already have Postgres installed on your localhost, download a native Mac OSX version from https://postgresapp.com/ or from https://www.postgresql.org/ (the official distribution for all platforms). Follow the install instructions.

### 3.2. Add the Pagila data to PostgreSQL

You first need to build the database schema. Run the following from the command line:

```bash
[quilt-py3] "/Applications/Postgres.app/Contents/Versions/10/bin/psql" -p5432 -d "postgres" < /path/to/data/raw/pagila/pagila-schema.sql
```

and:

```bash
[quilt-py3] "/Applications/Postgres.app/Contents/Versions/10/bin/psql" -p5432 -d "postgres" < /path/to/data/raw/pagila/pagila-insert-data.sql
```

where `/path/to/data/raw/pagila` is your localhost directory path to the Pagila database that you just downloaded.

We now have a complete database that contains the following tables and relationships:

![Pagila ERD][pagila_erd]

### 3.3. Run some simple queries

Let's test that we have the data correctly stored in Postgres  by running some simple queries. This assumes that you have a postgres command line prompt open.

```sql
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

The results:

```sql
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

Great! We have all the data loaded and accessible.

### 3.4. Add some benchmarking tools to the SQL prompt

You can do this in a couple of ways:

#### 3.4.1. Add `EXPLAIN ANALYZE` as the first line to your SQL statement

```sql
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

```sql
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

#### 3.4.2. Add the `timing` functionality to the PostgreSQL prompt

You can toggle this functionality on and off using the following command:

```sql
postgres=# \timing
Timing is on.
```

Lets take a look at the previous SQL command, with the `timing` flag on:

```sql
customer_id | customer_first_name | customer_last_name |                  email                   | staff_first_name | staff_last_name | amount |         payment_date          
-------------+---------------------+--------------------+------------------------------------------+------------------+-----------------+--------+-------------------------------
        269 | CASSANDRA           | WALTERS            | CASSANDRA.WALTERS@sakilacustomer.org     | Jon              | Stephens        |   1.99 | 2017-01-24 21:40:19.996577-08
        ....        
        284 | SONIA               | GREGORY            | SONIA.GREGORY@sakilacustomer.org         | Jon              | Stephens        |   0.99 | 2017-01-29 14:59:08.996577-08
        286 | VELMA               | LUCAS              | VELMA.LUCAS@sakilacustomer.org           | Jon              | Stephen
Time: 83.096 ms
```

Note the ending **Time: 83.096 ms**. That's the nugget of helpful information.

## 4. Query the database using a notebook

Everything prior to now has been correctly setting up our environment, downloading data, and getting the data loaded locally into our relational database management system. Now we are going to actually _work like a data engineer_ and pull in some real data from the Pagila database tables and work with it in a Jupyter notebook.

### 4.1. Ensure your notebooks are using your virtual environment packages by making them available as a kernel

To ensure that all the Python packages that you are using from your virtual environment (my current one is called `quilt-py3`, see above) are available in your notebook, you need to make it available as a notebook kernel.

```bash
[quilt-py3] $ ipython kernel install --user --name=quilt-py3
```

### 4.2. Ensure your notebook has the ability to benchmark queries

The whole point of this blog post is to benchmark the performance of SQL vs. CSV files vs. Quilt, so we need to install a [Juypter notebook extension](https://github.com/ipython-contrib/jupyter_contrib_nbextensions) that allows benchmarking. A good one is the  [ExecuteTime](https://github.com/ipython-contrib/jupyter_contrib_nbextensions/tree/master/src/jupyter_contrib_nbextensions/nbextensions/execute_time) extension. This displays when the last execution of a code cell occurred, and how long it took. Install the suite of user-contributed Jupyter extensions with the following command:

```bash
[quilt-py3] $ jupyter contrib nbextension install --user
```

To be able to configure your Jupyter notebooks to use this extension, you need to enable the *extension configurator*:

```bash
[quilt-py3] $ jupyter nbextensions_configurator enable --user
```

Now when you start up Jupyter you can select which extensions to enable by checking boxes in the `Nbextensions` tab (<span style="color:red">red box</span>):

![Selecting and enabling specific Nbextensions][notebook_nbextensions]

Ensure you check the `ExecuteTime` (<span style="color:blue">blue box</span>) extension to turn it on.

### 4.3. Fire up your notebook and select your kernel

At this point, you can start Jupyter, create a new notebook and select the new kernel for your environment using the Kernel menu  > Change kernel (<span style="color:red">red box</span>) > quilt-py3 (<span style="color:blue">blue box</span>):

![Notebook Kernel Selection][notebook_kernel]

You will see the kernel update in the top-right of the notebook (<span style="color:green">green box</span>).

## 5. Benchmarking PostgreSQL

Now we are able to benchmark our PostgreSQL database tables directly within Jupyter. Provided you have cloned this repository, you should be able to do exactly this by opening the `0.1-robnewman-pagila.ipynb` notebook in the `notebooks` directory.

## 5.1. A note on flushing the cache

In order to correctly benchmark, you need to ensure that all caches are flushed. To do this for Postgres, do the following:

1. Stop the Postgres service: How you do this depends on how you have it setup. Because I am using the Postgres app, I just turn the service off in the GUI:

![Turn off PostgreSQL server][postgres-app-stop]

2. Run `sync`: Running the `sync` command forces completion of any pending disk writes, essentially **flushing the cache**.

3. Optionally, on OSX run `purge`. This **forces** the disk cache to be purged (flushed and emptied). Typically this is what happens during a system reboot. You need administrator privileges to run this command (via `sudo`).

## 5.2. `pd.read_sql():` Direct SQL performance

A simple SQL query to select all records from the `payment` table and insert into a Panda's dataframe (**15635 rows × 6 columns**) takes **119ms**:

![Select all payments][sql-select-all-payments]

## 6. Create a Quilt package

Now we are going to do the same operations in Quilt. Following the [docs](https://docs.quiltdata.com/get-started/step-by-step#build-a-package), we need to first export the unstructured data from Postgres into a CSV file.

### 5.1. Export the data

Let's export payment data (the highest volume of records) from the Pagila database into a CSV:

```sql
postgres=# \COPY (SELECT * FROM payment) TO '/path/to/data/interim/payments.csv' DELIMITER ',' CSV HEADER;
COPY 15635
Time: 279.119 ms
```

Copy `/path/to/data/interim/payments.csv` to the `packages` directory.

### 5.2. Authenticate to the Quilt registry & generate the data package

In order to generate a data package, we first need to login to the Quilt data registry:

```bash
[quilt-py3] $ quilt login
Launching a web browser...
If that didn't work, please visit the following URL: https://pkg.quiltdata.com/login

Enter the code from the webpage:
```

And generate the `build.yml` file:

```bash
[quilt-py3] $ quilt generate packages
Generated build-file packages/build.yml.
```

We can take a peak inside `build.yml`:

```yaml
contents:
  payments:
    file: payments.csv
```

### 5.3. Build the Quilt package

```bash
[quilt-py3] rnewman@donut2 ~/sandbox/quilt-pagila-data (master) $ quilt build robnewman/payments packages/build.yml
Inferring 'transform: csv' for payments.csv
Built robnewman/payments successfully.
```

We can easily check if the Quilt data package was successfully built with the command `quilt ls` which lists all user packages in the Quilt data registry:

```bash
$ quilt ls
/Users/rnewman/Library/Application Support/QuiltCli/quilt_packages
robnewman/payments             latest               fc05fd571f3b0bc5769cf83b10195ebed9cbc264c0b41cdfbcaa18b58d462dbe
```

Now we're ready to use the data package in our Jupyter notebook. We `import` it just like any other standard Python library and load the data into a Pandas data frame:

!![Select all payments using Quilt data package][quilt-select-all-payments]

The equivalent command to insert the Quilt package data into a Panda's dataframe (**15635 rows × 6 columns**) takes **45ms**

You immediately see **x2.5** performance difference between the Pandas `pd.read_sql()` query and the Quilt conversion into a data frame.

## References

[pagila_erd]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/dvd-rental-sample-database-diagram.png "Pagila Entity Relationship Diagram"

[notebook_nbextensions]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/notebook-nbextensions.png "Choosing which Nbextensions to enable"

[notebook_kernel]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/notebook-kernel.png "Selecting the notebook kernel"

[postgres-app-stop]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/postgres-app-stop.png "Stop the PostgreSQL app server"

[sql-select-all-payments]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/sql-select-all-payments.png "Select all payments via direct SQL query"

[quilt-select-all-payments]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/quilt-select-all-payments.png "Select all payments using Quilt data package"
