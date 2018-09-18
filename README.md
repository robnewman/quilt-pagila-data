# quilt-pagila-data

## Goal

1. Clearly demonstrate how small- to large-datasets, which are commonly used in ML and Data Science projects, can be rapidly and easily prepared for analysis using [Quilt](https://www.quiltdata.com) data packages.
2. How Quilt data packages are **significantly** faster to load data into Panda's dataframes than using a direct query to a standard Relational Database Management System (RDMS) such as PostgreSQL or MySQL.

## Data Source

We are going to be using data provided by the [Pagila](https://github.com/devrimgunduz/pagila) database which is a PostgreSQL port of the [Sakila example database available for MySQL](https://dev.mysql.com/doc/sakila/en/). The data represents a DVD rental store (remember those!?). It is intended to provide a standard schema that can be used for examples in books, tutorials, articles, samples, etc and is a good standard for benchmarking performance.

## 1. Clone this repository

Use `git clone https://www.github.com/robnewman/quilt-pagila-data` to get this repository on your localhost.

## 1.1. Project directory structure

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
└── requirements.txt
```

There are a couple of minor differences to the recommendation:

1. `assets/images` is where the screenshots for this `README.md` are stored.
2. `packages` is where we will store the Quilt data packages that we generate and work with.

The `data/raw` directory stores the data files for building the database we will use during this article.

The `data/interim` directory is where we export CSV files created by our local Postgres SQL queries. We use these as the data sources for our Quilt package.

### 1.2. Build and activate a new virtual environment

To build the virtual environment (called `quilt-py3`) with all the packages used in this post, you need to run the following commands (assuming you have installed Python3 using the [Homebrew](http://brew.sh/) package manager):

First, build a new virtual environment called `quilt-py3`:

```bash
$ python3 -m venv /path/to/virtual/environments/quilt-py3
```

Next, activate the new environment:

```bash
$ source /path/to/virtual/environments/quilt-py3/bin/activate
```

(Depending on how your shell is configured, the command line prompt may automatically update to include the virtual environment name as a prefix once `activate` is successfully executed).

### 1.3. Check that you have a Python3 interpreter installed

```bash
[quilt-py3] $ python3 --version
Python 3.6.5
```

### 1.4. Install Python packages

Finally, install all the Python packages from the `requirements.txt` file in this repository:

```bash
[quilt-py3] pip install -r requirements.txt
```

Now you have all you need to work through the rest of this post, _except_ for the data!

## 2. Clone the Pagila database repository

Use `git clone https://github.com/devrimgunduz/pagila.git` to get the Pagila data repository on your localhost. Copy the files in the repository to the `data > raw` directory of this repository, so that the contents of the `raw` directory now look like:

```bash
raw
├── README.md
├── pagila-data.sql
├── pagila-insert-data.sql
└── pagila-schema.sql
```

## 3. Create a relational database and add the data

Now we have the raw data, we are going to use it to create a tables in the default local PostgreSQL database (`postgres`).

### 3.1. Installing PostgreSQL

If don't already have Postgres installed on your localhost, download a native Mac OSX version from https://postgresapp.com/ or from https://www.postgresql.org/ (the official distribution for all platforms). Follow the install instructions.

### 3.2. Add the Pagila tables to the PostgreSQL default database

Now we build the database schema and load the data. Run the following from the command line, which builds the tables and relations:

```bash
[quilt-py3] $ "/Applications/Postgres.app/Contents/Versions/10/bin/psql" -p5432 -d "postgres" < /path/to/data/raw/pagila-schema.sql
```

followed by:

```bash
[quilt-py3] $ "/Applications/Postgres.app/Contents/Versions/10/bin/psql" -p5432 -d "postgres" < /path/to/data/raw/pagila-insert-data.sql
```

where `/path/to/data/raw/` is the path to the Pagila SQL files on your localhost.

We now have a complete database that contains the following tables and relationships:

![Pagila ERD][pagila_erd]

### 3.3. Run some sample queries

To confirm we have data loaded into the default `postgres` database let's run some sample SQL queries. First, open Postgres:

```
[quilt-py3] $ "/Applications/Postgres.app/Contents/Versions/10/bin/psql" -p5432 -d "postgres"
psql (10.5)
Type "help" for help.

postgres=#
```

It's helpful to see how fast your queries take to complete. You can toggle this functionality using the `timing` flag:

```
postgres=# \timing
Timing is on.
```

Now run the following `SELECT` statement:

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

Screenshot of the partial results:

```sql
customer_id | customer_first_name | customer_last_name |                  email                   | staff_first_name | staff_last_name | amount |         payment_date          
-------------+---------------------+--------------------+------------------------------------------+------------------+-----------------+--------+-------------------------------
        269 | CASSANDRA           | WALTERS            | CASSANDRA.WALTERS@sakilacustomer.org     | Jon              | Stephens        |   1.99 | 2017-01-24 21:40:19.996577-08
        ....        
        284 | SONIA               | GREGORY            | SONIA.GREGORY@sakilacustomer.org         | Jon              | Stephens        |   0.99 | 2017-01-29 14:59:08.996577-08
        286 | VELMA               | LUCAS              | VELMA.LUCAS@sakilacustomer.org           | Jon              | Stephen
Time: 83.096 ms
```

Note the helpful message **Time: 83.096 ms**.

## 4. Query the database using a Jupyter notebook

Everything prior to now has been correctly setting up our environment, downloading data, and getting the data loaded locally into our relational database management system. Now we are going to actually _work like a data engineer_ and pull in some real data from the Pagila database tables and work with it in a notebook.

### 4.1. Ensure your notebooks are using your virtual environment packages by making them available as a kernel

To ensure that all the Python packages that you are using from your virtual environment (my current one is called `quilt-py3`, see above) are available in your notebook, you need to make it available as a notebook **kernel**.

```bash
[quilt-py3] $ ipython kernel install --user --name=quilt-py3
```

Now you can simply `import` Python packages into your notebook from your virtual environment.

### 4.2. Ensure your notebook has the ability to benchmark queries

The whole point of this article is to benchmark the performance of SQL vs. Quilt data packages, so we need to install a [Juypter notebook extension](https://github.com/ipython-contrib/jupyter_contrib_nbextensions) that allows benchmarking. A popular one is the  [ExecuteTime](https://github.com/ipython-contrib/jupyter_contrib_nbextensions/tree/master/src/jupyter_contrib_nbextensions/nbextensions/execute_time) extension. This displays (1) when the last execution of a code cell occurred, and (2) how long it took. Install the suite of user-contributed Jupyter extensions with the following command:

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

At this point, you can start Jupyter, create a new notebook and select the new kernel for your environment using the `Kernel` menu  > `Change kernel` (<span style="color:red">red box</span>) > `quilt-py3` (<span style="color:blue">blue box</span>):

![Notebook Kernel Selection][notebook_kernel]

You will see the kernel update in the top-right of the notebook (<span style="color:green">green box</span>).

## 5. Benchmarking PostgreSQL

Now we are able to benchmark our PostgreSQL database tables directly within Jupyter. Provided you have cloned this repository, you should be able to do exactly this by opening the `0.1-robnewman-pagila.ipynb` notebook in the `notebooks` directory.

## 5.1. A note on flushing the cache (creating "cold client, cold server" conditions)

In order to correctly benchmark, you need to ensure that all caches are flushed. To do this for Postgres, do the following:

1. **Stop the Postgres service:** How you do this depends on your PostgreSQL setup. I am using the native OSX Postgres app so I just `Stop` the service via the GUI:

![Turn off PostgreSQL server][postgres-app-stop]

2. Run `sync`: Running the `sync` command forces completion of any pending disk writes, essentially **flushing the cache**.

3. Optionally, on OSX run `purge`. This **forces** the disk cache to be purged (flushed and emptied). Typically this is what happens during a system reboot. **Warning:** You need administrator privileges to run this command (via `sudo`), so be careful. An alternative is to just _gracefully_ reboot your localhost rather than brute-force a `purge`.

## 5.2. `pd.read_sql():` Direct SQL performance

A simple SQL query to select all records from the `payment` table and insert into a Panda's dataframe (**15635 rows × 6 columns**) takes **458ms**:

![Select all payments][sql-select-all-payments]

## 5.3. What about subsequent re-runs of the same query (creating "warm client, warm server" conditions)?

The screenshot below shows three subsequent iterations of the same direct SQL query:

![Select all payments - iterations][sql-select-all-payments-iterations]

## 6. Create a Quilt package

Now we are going to do the same operations in Quilt. Following the [docs](https://docs.quiltdata.com/get-started/step-by-step#build-a-package), we need to first export the unstructured data from Postgres into a CSV file.

### 6.1. Export the data

Let's export payment data (the highest volume of records via a simple `SELECT` query with no `JOINS`) from the Pagila database into a CSV:

```sql
postgres=# \COPY (SELECT * FROM payment) TO '/path/to/data/interim/payments.csv' DELIMITER ',' CSV HEADER;
COPY 15635
Time: 279.119 ms
```

Copy `/path/to/data/interim/payments.csv` to the `/path/to/packages` directory.

### 6.2. Authenticate to the Quilt registry & generate the data package

In order to generate a data package, we first need to login to the Quilt data registry which [requires creating a free account](https://quiltdata.com/signup):

```bash
[quilt-py3] $ quilt login
Launching a web browser...
If that didn't work, please visit the following URL: https://pkg.quiltdata.com/login

Enter the code from the webpage:
```

Now generate `build.yml` using the Quilt [generate](https://docs.quiltdata.com/get-started/step-by-step#explicit-builds) command:

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

### 5.3. Build your new Quilt package

```bash
[quilt-py3] $ quilt build robnewman/performance packages/build.yml
Inferring 'transform: csv' for payments.csv
Built robnewman/performance successfully.
```

We can easily check if the Quilt data package was successfully built with the command `quilt ls` which lists all user packages in the Quilt data registry:

```bash
$ quilt ls
/Users/rnewman/Library/Application Support/QuiltCli/quilt_packages
robnewman/performance          latest               01431940787f667f46c3a0a2056ee0b16f899fc33c1c9db80ff376e337de3571
```

### 6.3. Use the data package directly from a notebook (create "cold client, cold server" conditions)

Now we're ready to use the data package in our Jupyter notebook. We `import` it just like any other standard Python library and load the data into a Pandas data frame:

![Select all payments using Quilt data package][quilt-select-all-payments]

The equivalent command to insert the Quilt package data into a Panda's dataframe (**15635 rows × 6 columns**) takes **126ms**

You immediately see **x3** performance difference between the Pandas `pd.read_sql()` query and the Quilt data package transformation into a Pandas data frame within the notebooks.

### 6.4. What about subsequent re-runs of the same data import (creating "warm client, warm server" conditions)?

The screenshot below shows three subsequent iterations of the same import of Quilt-formatted data:

![Select all payments using Quilt data package - iterations][quilt-select-all-payments-iterations]

### 6.5. Summary of direct SQL queries using Pandas vs. Quilt data package

| Query engine | Cold client,<br/>cold server (ms) | Warm client<br/>warm server [Average] (ms) |
| ------ |:------:|:------:|
| `pd.read_sql()` | 458 | 341, 207, 193 [247] |
| `import quilt.USR.PKG as pkg` | 126 | 35, 17, 18 [23] |

### 6.4. More complicated queries

That was a very simple example of a `SELECT * FROM table`. Let's create some more complex queries, including `JOINS` across tables, in our direct SQL queries and see how that compares with the same data stored in a Quilt data package.

### 6.4.1. Outer join

When joining two SQL tables, an `INNER JOIN` creates a new view with only the **matching records** from both tables, while an `OUTER JOIN` creates a view with all the records from **both tables**. An `OUTER JOIN` is therefore a way to get a much larger result set (in terms of columns).

The SQL query we are going to run joins all the DVD stores payments (_payment table_) with the details of the customers who made the payments (_customer table_) with the details of the store where the DVD was rented (_store table_) with all the specific DVD rental information (_rental table_):

```SQL
SELECT *
FROM payment p
FULL OUTER JOIN
customer c ON p.customer_id = c.customer_id
FULL OUTER JOIN
store s ON c.store_id = s.store_id
FULL OUTER JOIN
address a ON a.address_id = c.address_id
FULL OUTER JOIN
rental r ON p.rental_id = r.rental_id
```

We now have **16052** rows and **35** columns from the Pagila database (with many columns duplicated through our use of `SELECT *`, which you typically wouldn't use in a standard workflow).

### 6.4.2. Pandas read_sql() result

The result from `pandas.read_sql()` takes **1.57s** to run:

![Large outer join via direct SQL query][sql-outer-join]

### 6.4.3. Quilt data package result

Before we test the performance, we need to:

(1) Create a new CSV file from the SQL query

```SQL
postgres=# \COPY (SELECT * FROM payment p FULL OUTER JOIN \
customer c ON p.customer_id = c.customer_id FULL OUTER JOIN \
store s ON c.store_id = s.store_id FULL OUTER JOIN \
address a ON a.address_id = c.address_id FULL OUTER JOIN \
rental r ON p.rental_id = r.rental_id) TO \
'/path/to/data/interim/payments_large_outer_join.csv' DELIMITER ',' CSV HEADER;
COPY 16052
Time: 265.567 ms
```

(2) Copy this new CSV file to our `packages` directory

```bash
[quilt-py3] $ cp /path/to/data/interim/payments_large_outer_join.csv /path/to/packages/
```

(3) Regenerate our Quilt data package

```bash
[quilt-py3] $ quilt generate packages
```

The resutling new `build.yaml` file:

```yaml
contents:
  payments:
    file: payments.csv
  payments_large_outer_join:
    file: payments_large_outer_join.csv
```

(4) Add a new line to the notebook:

`performance.payments_large_outer_join()`

(4) Under the `Kernel` menu choose `Restart and Run All` in the notebook to ensure we are using the latest package.

The result from `performance.payments_large_outer_join()` takes **138ms**:

![Large outer join using Quilt data package][quilt-outer-join]

This is a **x11** speed improvement.

## Summary

We have demonstrated that Quilt data packages are significantly faster at loading structured data stored in a local Postgres database into a Pandas dataframe than Pandas own internal `read_sql()` method.

We also have a snapshot of the data that our data pipeline team can prepare in advance for the data science team to work with, version, and store separately without worrying querying against the production database. This will keep your database administrators happy.

If your data engineering pipeline could benefit from Quilt data packages, please contact me!

## References

[pagila_erd]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/dvd-rental-sample-database-diagram.png "Pagila Entity Relationship Diagram"

[notebook_nbextensions]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/notebook-nbextensions.png "Choosing which Nbextensions to enable"

[notebook_kernel]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/notebook-kernel.png "Selecting the notebook kernel"

[postgres-app-stop]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/postgres-app-stop.png "Stop the PostgreSQL app server"

[sql-select-all-payments]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/sql-select-all-payments.png "Select all payments via direct SQL query"

[sql-select-all-payments-iterations]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/assets/images/sql-select-all-payments-iterations.png "Select all payments via direct SQL query -- iterations"

[quilt-select-all-payments]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/quilt-select-all-payments.png "Select all payments using Quilt data package"

[quilt-select-all-payments-iterations]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/master/assets/images/quilt-select-all-payments-iterations.png "Select all payments using Quilt data package -- iterations"

[sql-outer-join]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/assets/images/sql-outer-join.png "Large outer join via direct SQL query"

[quilt-outer-join]:https://raw.githubusercontent.com/robnewman/quilt-pagila-data/assets/images/quilt-outer-join.png "Large outer join using Quilt data package"
