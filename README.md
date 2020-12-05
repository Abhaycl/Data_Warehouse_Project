# Data Warehouse Project Starter Code

The objective of this project is to apply what we've learned on data warehouses and AWS to build an ETL pipeline for a database hosted on Redshift.

<!--more-->

[//]: # (Image References)

[image1]: ./images/redshift_start.jpg "Start of the redshift cluster"
[image2]: ./images/create_tables.jpg "Creation of the tables"
[image3]: ./images/etl.jpg "Transformation of data"
[image4]: ./images/redshift_stop.jpg "Stop of the redshift cluster"
[image5]: ./images/star_schema.jpg "Star schema"
[image6]: ./images/cluster_info.jpg "Cluster Information"
[image7]: ./images/cluster_queries.jpg "Queries"

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

Start the redshift cluster.
```bash
  python redshift_start.py
```
![alt text][image1]

Create the tables.
```bash
  python create_tables.py
```
![alt text][image2]

Data transformations.
```bash
  python etl.py
```
![alt text][image3]

Stop the redshift cluster.
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


## Schema definition

To represent this context a star schema has been used.

The songplays table is the core of this schema, is it our fact table and it contains foreign keys to four tables:

    * start_time REFERENCES time(start_time)
    * user_id REFERENCES time(start_time)
    * song_id REFERENCES songs(song_id)
    * artist_id REFERENCES artists(artist_id)

There are also two staging tables; One for event dataset and one for song dataset.
![alt text][image5]


## Preamble

In this project we are going to use two Amazon Web Services, S3 (Data storage) and Redshift (Data warehouse with columnar storage).

Data sources are provided by two public S3 buckets. One bucket contains info about songs and artists, the second has info concerning actions done by users (which song are listening, etc.. ). The objects contained in both buckets are JSON files. The song bucket has all the files under the same directory but the event ones don't, so we need a descriptor file (also a JSON) in order to extract data from the folders by path. We used a descriptor file because we don't have a common prefix on folders.

The Redshift service is where data will be ingested and transformed, in fact though COPY command we will access to the JSON files inside the buckets and copy their content on our staging tables.


## Redshift Considerations

The schema design in redshift can heavily influence the query performance associated. Some relevant areas for query performance are:

    * Defining how redshift distributes data across nodes.
    * Defining the sort keys, which can determine ordering and speed up joins.
    * Definining foreign key and primarty key constraints.


## Data Distribution

How data is distributed is orchestrated by the selected distribution style. When using a 'KEY' distribution style, we inform redshift on how the data should be distributed across nodes, as data will be distributed such that data with that particular key are allocated to the same node.

A good selection for this distribution keys is such that data is distributed evenly, such as to prevent performance hotspots, with collocating related data such that we can easily perform joins. We essentially want to perform joins on columns which are a distribution key for both the tables. Then, redshift can run joins locally instead of having to perform network I/O. We want to choose one dimension table to use as the distribution key for a fact table when using a star schema. We want to use the dimension table which is most commonly joined.

For a slowly changing dimension table, of relatively small size (<1M entries in the case of Redshift) using an 'ALL' distribution style is a good choice. This distributes the table across all nodes for each of retrieval and performance.


## Primary & Foreign Key Constraints

We can declare primary key and foreign key relationships between dimensions and fact tables for star schemas. Redshift uses this information to optimize queries, by eliminating redundant joins. We must ensure that primary key constraints are enforced, with no duplicate inserts.


## ETL process

In this project most of ETL is done with SQL (Python used just as bridge), transformation and data normalization is done by Query, check out the sql_queries python module.


## Comments.

Information on the redshift cluster.
![alt text][image6]

All the queries that are executed in the cluster.
![alt text][image7]

A possible improvement for data management would have been to use a 'sort key' determines the order with which data is stored on disk for a particular table. Query Performance is increased when the sort key is used in the where clause. Only one sort key can be specified, with multiple columns. Using a 'Compound Key', specifies precedence in columns, and sorts by the first key, then the second key. 'Interleaved Keys', treat each column with equal importance. Compound keys can improve the performance of joins, group by, and order by statements.