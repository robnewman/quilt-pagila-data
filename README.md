# quilt-kiva-lending-data

## Goal

Demonstrate how large datasets, which are commonly used in ML and Data Science projects, can be rapidly prepared for analysis using [Quilt](https://www.quiltdata.com) data packages, and how data packages are significantly faster than using a standard Relational Database Management System (RDMS) such as PostgreSQL or MySQL.

## Data Source

We are going to be using data from [Kiva](https://www.kiva.org): a San Francisco-based non-profit that lends money in more than 80 countries on 5 continents. In their own words:

> Kiva is an international nonprofit, founded in 2005 and based in San Francisco, with a mission to connect people through lending to alleviate poverty. We celebrate and support people looking to create a better future for themselves, their families and their communities.

Kiva maintains an open data model which can be queried using their RESTful [API](https://build.kiva.org). This allows anyone to download their loan data. Conveniently you can easily download large datasets in the form of [data snapshots](http://build.kiva.org/docs/data/snapshots) which are ZIP archives in either JSON or CSV formats.

## 1. Clone this repository

Use `git clone https://www.github.com/robnewman/quilt-kiva-lending-data` to get this repository on your localhost.

## 2. Download the latest data snapshot from Kiva

Change directories into the `data` directory and run the following command, which will retrieve the most recent snapshot of loan data from an AWS S3 bucket:

`curl http://s3.kiva.org/snapshots/kiva_ds_csv.zip -o kiva_ds_csv.zip`

Unzip the archive. As of writing, this is a 902MB zip file containing three files:

    kiva_ds_json /
      lenders.json
      loans.json
      loans_lenders.json

## 3. Create a relational database and add the data

Now we have the data, we are going to add it to a PostgreSQL database.

### 3.1. Installing PostgreSQL

If don't already have Postgres installed on your localhost, download a native Mac OSX version from https://postgresapp.com/ or from https://www.postgresql.org/ (the official distribution for all platforms). Follow the install instructions.

## 4. Create a Quilt package
