# Data Warehouse Project Starter Code

The objective of this project is to apply what we've learned on data warehouses and AWS to build an ETL pipeline for a database hosted on Redshift.

<!--more-->

[//]: # (Image References)

[image1]: ./images/redshift_start.jpg "Start of the redshift cluster"
[image2]: ./images/create_tables.jpg "Creation of the tables"
[image3]: ./images/etl.jpg "Transformation of data"
[image4]: ./images/redshift_stop.jpg "Stop of the redshift cluster"

---


#### How to run the program with our own code

## Project Requirements

The requirements for the project are a valid aws account, with accompanying security credentials, as well as a python environment, which satisfies the module requirements given.

You will need to add aws access key and secret information to the dwf.cfg file, under [AWS_ACCESS]. This is not to be comitted to git.

We also have to create our security group which has to be assigned to the default VPC. Because the creation of our cluster has to belong to a VPC.

**NOTE:** _To follow IAC (Infrastructure as Code) practices, and to allow us to easily start and stop the redshift cluster to save costs, we can use the following scripts;_

    redshift_start.py
    redshift_stop.py

The scripts will create/remove the neccessary resources for redshift to run.

For the execution of our own code, we go to the project workspace or to our own terminal in Visual Code.

In the project workspace we can open a terminal and run the following files:

To start the redshift cluster.
```bash
  python redshift_start.py
```
![alt text][image1]

To create the tables.
```bash
  python create_tables.py
```
![alt text][image2]

For data transformations.
```bash
  python etl.py
```
![alt text][image3]

To stop the redshift cluster.
```bash
  python redshift_stop.py
```
![alt text][image4]

---

The summary of the files and folders within repo is provided in the table below:

| File/Folder              | Definition                                                                                                   |
| :----------------------- | :----------------------------------------------------------------------------------------------------------- |
| images/*                 | Folder containing the images of the project.                                                                 |
|                          |                                                                                                              |
| auxiliary.py             | Auxiliary functions for creating the connection to the RedShift cluster.                                     |
| create_tables.py         | Functions to create the fact and dimension tables for the star schema in Redshift.                           |
| dwh.cfg                  | Contains the parameterization offdhcvv z Redshift, IAM role, AWS access and S3.                                      |
| etl.py                   | Contains the queries for loading the S3 data into staging tables on Redshift and then process that data into our analytics tables on Redshift. |
| redshift_start.py        | Contains the necessary processes to create and start up our cluster.                                         |
| redshift_stop.py         | Contains the necessary processes to stop and delete our cluster.                                             |
| sql_queries.py           | Contains the SQL statements that will be implemented for the creation of the tables and for the etl process. |
|                          |                                                                                                              |
| README.md                | Contains the project documentation.                                                                          |
| README.pdf               | Contains the project documentation in PDF format.                                                            |


---

**Steps to complete the project:**

#### Create Table Schemas

1. Design schemas for your fact and dimension tables
2. Write a SQL CREATE statement for each of these tables in sql_queries.py
3. Complete the logic in create_tables.py to connect to the database and create these tables
4. Write SQL DROP statements to drop tables in the beginning of create_tables.py if the tables already exist. This way, you can run create_tables.py whenever you want to reset your database and test your ETL pipeline.
5. Launch a redshift cluster and create an IAM role that has read access to S3.
6. Add redshift database and IAM role info to dwh.cfg.
7. Test by running create_tables.py and checking the table schemas in your redshift database. You can use Query Editor in the AWS Redshift console for this.


#### Build ETL Pipeline

1. Implement the logic in etl.py to load data from S3 to staging tables on Redshift.
2. Implement the logic in etl.py to load data from staging tables to analytics tables on Redshift.
3. Test by running etl.py after running create_tables.py and running the analytic queries on your Redshift database to compare your results with the expected results.
4. Delete your redshift cluster when finished.



## [Rubric Points](https://review.udacity.com/#!/rubrics/2501/view)
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
## Scenario

A music streaming startup, Sparkify, has grown their user base and song database and want to move their processes and data onto the cloud. Their data resides in S3, in a directory of JSON logs on user activity on the app, as well as a directory with JSON metadata on the songs in their app.

As their data engineer, you are tasked with building an ETL pipeline that extracts their data from S3, stages them in Redshift, and transforms data into a set of dimensional tables for their analytics team to continue finding insights in what songs their users are listening to. You'll be able to test your database and ETL pipeline by running queries given to you by the analytics team from Sparkify and compare your results with their expected results.


## Denormalizing the Database.

The goal of denormalization in this context is to reduce the amount of time needed to read data. Unlike relational databases, non-relational databases have been optimized for fast reads and writes; therefore, denormalization is a must!
Always thinking about the necessary queries first and designing the denormalization scheme accordingly. One table per query is a good strategy.

Denormalization changes the application: First, it means data redundancy, which translates to significantly increased storage costs. Second, fixing data inconsistency is now the main job of the application.

Again, Data Modeling in Apache Cassandra is query focused – and that focus needs to be put on the WHERE clause. This clause allows to do fast reads. Note that the partition key always needs to be included in the query! The clustering columns can be used to put the results in order.

The tables that will contain the data of our consultations are:

![alt text][image1]

## Apache Cassandra.

Aside from being a backbone for Facebook, Uber, and Netflix, Cassandra is a very scalable and resilient database that is easy to master and simple to configure. Apache Cassandra uses its own query language – CQL – which is similar to SQL. Note that JOINS, GROUP BY, or subqueries are not supported by CQL.

Some terms used in Cassandra differ from those we already know:

A keyspace, for example, is analogous to the term database in a relational database.

Another example is a partition, which is a collection of rows. Cassandra organizes data into partitions; there, each partition consists of multiple columns.

Partitions are stored on a node. Nodes (or servers) are generally part of a cluster where each node is responsible for a fraction of the partitions.

The Primary Key defines how each row can be uniquely identified and how the data is distributed across the nodes in our system. A partition key is responsible for identifying the partition or node in the cluster that stores a row – whereas the purpose of a clustering key (or clustering column) is to store row data within a partition in a sorted order.

When we have only one partition key and no clustering column, it is called a Single Primary Key. Should we use one (or more) partition key(s) and one (or more) clustering column(s) instead, we call it a Compound Primary Key or Composite Primary Key.


## Data.

The data used in this project, it's better to understand what they represents.

#### Song Dataset.

We'll be working with dataset: event_data. The directory of CSV files partitioned by date. For example, here are the file paths for this dataset.

```bash
  event_data/2018-11-01-events.csv
  event_data/2018-11-02-events.csv
  event_data/2018-11-03-events.csv
  .
  .
  event_data/2018-11-30-events.csv
```

These files are in CSV format and contains several records with the song data separated by a comma, below is an example of what a single song file, 2018-11-01-events.csv, looks like.

```bash
  Black Eyed Peas,Logged In,Sylvie,F,0,Cruz,214.93506,free,"Washington-Arlington-Alexandria, DC-VA-MD-WV",PUT,NextSong,1.54027E+12,9,Pump It,200,1.54111E+12,10
```

The code to pre-process the CSV files was provided already. So no need to go in-depth for it.


## ETL Pipeline.

Extract, transform, load (ETL) is the general procedure of copying data from one or more sources into a destination system which represents the data differently from, or in a different context than, the sources.

#### ETL Pipeline for Creating and Querying NoSQL Database.

We need to create a streamlined CSV file from all these. The final file will be used to extract and insert data into Apache Cassandra tables.

The event_datafile_new.csv has 6821 rows and contains the following columns:

* artist
* firstName of user
* gender of user
* item number in session
* last name of user
* length of the song
* level (paid or free song)
* location of the user
* sessionId
* song title
* userId

The image below is a screenshot of what the denormalized data should appear like in the event_datafile_new.csv after the code above is run:

![alt text][image2]

## Apache Cassandra Coding Portion.

We will model our data based on the queries provided to us by the analytics team at Sparkify. But first, let's setup Apache Cassandra for this. This is a three step process:

#### Create a Cluster.

We create a cluster and connect it to our local host. This makes a connection to a Cassandra instance on our local machine.

```bash
# This should make a connection to a Cassandra instance your local machine (127.0.0.1).
from cassandra.cluster import Cluster

try:
    # Connect to local Apache Cassandra instance.
    cluster = Cluster(['127.0.0.1'])
    # To establish connection and begin executing queries, need a session.
    session = cluster.connect()

except Exception as e:
    print(e)
```

#### Create a Keyspace.

```bash
# Create a keyspace called sparkify.
try:
    session.execute("""
        CREATE KEYSPACE IF NOT EXISTS sparkify
        WITH REPLICATION = {'class': 'SimpleStrategy', 'replication_factor': 1}"""
    )

except Exception as e:
    print(e)
```

#### Set Keyspace.

```bash
# Set KEYSPACE to the keyspace specified above.
try:
    session.set_keyspace("sparkify")

except Exception as e:
    print(e)
```


## Data Modeling.

In Apache Cassandra, we model our data based on the queries we will perform. Aggregation like GROUP BY, JOIN are highly discouraged in Cassandra. This is because we shouldn't scan the entire data because it is distributed on multiple nodes. It will slow down our system because sending all of that data from multiple nodes to a single machine will crash it.

Now we will create the tables to run the following queries:

1. Give me the artist, song title and song's length in the music app history that was heard during sessionId = 338, and itemInSession = 4.

2. Give me only the following: name of artist, song (sorted by itemInSession) and user (first and last name) for userid = 10, sessionid = 182.

3. Give me every user name (first and last) in my music app history who listened to the song "All Hands Against His Own".

To gain more technical detail about the coding portion please view the ETL notebook.


## Conclusion.

This project provides Sparkify startup customers with tools to analyze their data and help answer their key business questions, such as "Which artist and song was heard in a specified session," "Which artist, song and user was heard in a specified session," or "Which users heard a certain song".