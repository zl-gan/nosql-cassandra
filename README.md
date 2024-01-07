# NoSQL - Cassandra

## Introduction 
NoSQL databases are non-relational databases that provide a mechanism for data storing and retrieving models that are different than relational tables and not fixed schema. \
NoSQL originally refers to “Not-SQL” or “Not Only SQL” and this concept was introduced in 1998 by Carl Strozzi. \
NoSQL databases are increasingly used in big data and real-time web applications due to the flexibility of all kinds of databases formats, such as Twitter, Facebook, and Google. 

The objective of NoSQL is to make data storage and recovery easy, regardless of its structure and content. In contrast, SQL databases are relational databases that store data in structured tables with fixed schemas. SQL databases use the Structured Query Language (SQL) to manage and query data. SQL databases are widely used in traditional applications that require complex transactions and data relationships. SQL databases are known for their ability to ensure data consistency and integrity through the use of transactions and constraints.

There are four main types of NoSQL databases: key-value stores, document-based stores, column-based stores, and graph-based stores. 
Each type has its own characteristics and is designed to have features such as flexible schemas, horizontal scaling, fast queries, and ease of use which promote simplicity. \ 
In this guide, Cassandra will be used to demonstrate the capabilities of NoSQL. 

## Cassandra
Cassandra is an open-source NoSQL distributed database trusted by thousands of companies for its scalability and high availability without comprising performance. Its features of linear scalability and proven fault-tolerance on commodity hardware or cloud infrastructure make it the perfect platform for mission-critical data. 

### Installation Steps
The following are the steps to install Cassandra on you local Windows PC. 
1)	Download the installation package for [Java SE Development Kit 8u311](https://www.oracle.com/java/technologies/downloads/#java8-windows) from Oracle. 
2)	Install the Java SE Development Kit. 
3)	Configure the environment variable for Java. 
Define JAVA_HOME as the system variable with the directory of Java as its variable value. \
![Picture1](https://github.com/zl-gan/nosql-cassandra/assets/69247135/52929a71-d345-44cc-9693-3d32e99e4276)\
*Fig. 1	Configuring the environment variable for Java*
4)	Download Apache Cassandra version 3.11.11 from this [link](https://www.apache.org/dyn/closer.lua/cassandra/4.0.1/apache-cassandra-4.0.1-bin.tar.gz). 
5)	Unzip the compressed file to local directory. (C:\Casssandra)
6)	Configure the environment variable for Cassandra. \
![Picture2](https://github.com/zl-gan/nosql-cassandra/assets/69247135/6f6a4c36-f456-4618-9356-8cd6eb91e9a3)\
*Fig. 2	Configuring environmental variable for Cassandra*
7)	Download Python (version 2.7.18) from this [link](https://www.python.org/downloads/release/python-2718/). 
8)	Install Python (version 2.7.18) on local machine. 
9)	Add environment variable for Python.
10)	Rename sigar-amd64-winnt.dll from C:\Cassandra\apache-cassandra-3.11.11\lib\sigar-bin to sigar-amd64-winntt.dll to prevent execution failure. 
11)	Modify the cqlsh.py file from Cassandra bin folder (C:\Cassandra\apache-cassandra-3.11.11\bin\). Change the parameter DEFAULT_REQUEST_TIMEOUT_SECONDS from 10 to 1000s as shown in Fig. 20. This is to prevent any timeout from occurring when querying the data. \
![Picture3](https://github.com/zl-gan/nosql-cassandra/assets/69247135/eddfbc9c-2e5c-4d70-8caf-4713bf823b70)\
*Fig. 3	Modifying DEFAULT_REQUEST_TIMEOUT_SECONDS from 10 to 1000*
12)	Modify the cassandra.yaml file from Cassandra conf folder (C:\Cassandra\apache-cassandra-3.11.11\conf). Change the parameter tombstone_warn_threshold and tombstone_failure_threshold from 1000 and 100000 to 100000 and 10000000 respectively to prevent ReadTimeout error when querying using cassandra-driver on Python. 

### Construct Database
The following steps were taken to construct our database. 
1)	In command prompt, change the path to where bin folder for Cassandra is located. (C:\Cassandra\apache-cassandra-3.11.11\bin\). 
2)	Run the command “cassandra.bat -f” in the command prompt to start Cassandra server. A message stating “Startup complete” will appear when the Cassandra server is ready. 
3)	With the previous still running, open a separate command prompt to execute cqlsh. Navigate to the bin folder of the Cassandra distribution in the command prompt. (C:\Cassandra\apache-cassandra-3.11.11\bin\)
4)	Start cqlsh by running the command “cqlsh”. The shell will be ready when “cqlsh>” is shown in the command prompt. 
5)	Execute the following CQL query to create a new keyspace.
> CREATE KEYSPACE cds502 WITH replication = {'class':'SimpleStrategy', 'replication_factor':1} AND DURABLE_WRITES = true;
6)	Execute the following CQL query to verify that the keyspace was created.
> SELECT * FROM system_schema.keyspaces;
7)	Use the keyspace by executing the following code.
> USE cds502;

### Construct a Data Table and Populate It
The following steps were executed to construct a data table and populate it with the Chicago crimes dataset, which can be downloaded from [Kaggle](https://www.kaggle.com/chicago/chicago-crime). This dataset is around 1.48GB, with 22 columns and 6.44 million rows. 

1)	The following command is executed in CQLSH to create a new table. 
> CREATE TABLE crime("ID" int PRIMARY KEY, "Case Number" varchar, "Date" timestamp, "Block" varchar, "IUCR" varchar, "Primary Type" varchar, "Description" varchar, "Location Description" varchar, "Arrest" boolean, "Domestic" boolean, "Beat" int, "District" int, "Ward" int, "Community Area" int, "FBI Code" varchar, "X Coordinate" decimal, "Y Coordinate" decimal, "Year" int, "Updated On" timestamp,"Latitude" decimal, "Longitude" decimal, "Location" varchar);

2)	Execute the following command to verify the creation of the table. A blank table with only the column name will be printed as shown in Fig. 4. 
> SELECT * FROM crime;

![Picture4](https://github.com/zl-gan/nosql-cassandra/assets/69247135/01254193-b295-4d9c-ad5a-ae176fe9a019)\
*Fig. 4	Blank data table output from newly created table.*

3)	Populate the data table by copying the csv file to the data table using the following code. The processing time will be shown at the end of the process as shown in Fig. 5. 
> COPY crimes("ID", "Case Number", "Date", "Block", "IUCR", "Primary Type", "Description", "Location Description", "Arrest", "Domestic", "Beat", "District", "Ward", "Community Area", "FBI Code", "X Coordinate", "Y Coordinate", "Year", "Updated On", "Latitude", "Longitude", "Location") FROM 'C:\Cassandra\crimes.csv' WITH NULL='null' and DELIMITER=',' AND HEADER=TRUE AND DATETIMEFORMAT='%m/%d/%Y %H:%M:%S %p';

![Picture5](https://github.com/zl-gan/nosql-cassandra/assets/69247135/c74d0964-2598-4548-aaac-9ea20ec47411)\
*Fig. 5	Output message from CQLSH after successfully ingesting the CSV file to data table.*

4)	Two materialized view are created in order to assist with querying process, particularly for those queries which involves aggregation operation GROUP BY. The codes used were as follow. 
> CREATE MATERIALIZED VIEW arrests AS SELECT "ID", "Year", "Case Number", "Arrest" from crime WHERE "Year" IS NOT NULL AND "Case Number" IS NOT NULL AND "Arrest" IS NOT NULL PRIMARY KEY ("ID", "Year"); 
> CREATE MATERIALIZED VIEW crime_by_location AS SELECT "Location Description", "Primary Type", "Year", "Case Number" from crime WHERE "Location Description" IS NOT NULL AND "ID" IS NOT NULL PRIMARY KEY ("Location Description", "ID");

### Queries
Data query can be performed in python using the cassandra library. Please visit the attached [Jupyter notebook](https://github.com/zl-gan/nosql-cassandra/blob/main/Cassandra.ipynb) for the code. 

Five sample queries were made to obtain relevant insights from the data table. The queries made and executed in order were as follow. 
1)	Number of arrested crimes by year
>`SELECT "Year", COUNT("Case Number") FROM arrests WHERE "Arrest"=True GROUP BY "Year" ALLOW FILTERING;`
2)	Number of crimes by location
>`SELECT "Location Description", COUNT("Case Number") FROM crime_by_location GROUP BY "Location Description";`
3)	Number of robbery cases in District 1
>`SELECT count(*) FROM crime WHERE "District"=1 AND "Primary Type"='ROBBERY' ALLOW FILTERING;`
4)	Number of theft crime by location in year 2015
>`SELECT "Location Description", COUNT("Case Number") FROM crime_by_location WHERE "Year"=2015 AND "Primary Type"='THEFT' GROUP BY "Location Description" ALLOW FILTERING;`
5)	Number of thefts occurring in residences with arrests being made in year 2010.
>`SELECT count(*) FROM crime WHERE "Primary Type"='THEFT' and "Location Description"='RESIDENCE' and "Arrest"= True and "Year"=2010 ALLOW FILTERING;`

**Sample Video**

https://github.com/zl-gan/nosql-cassandra/assets/69247135/9d281bd7-e18d-4e83-a8d0-aaa42cbaedb2

## Lessons Learned
It is imperative to model the data in NoSQL database based on the operations that will be performed on the data table. For instance, in Cassandra, the GROUP BY aggregate function can only be done over primary keys. As such, the user would have to define the primary key beforehand during the creation of a new data table in Cassandra. Another workaround here is to create a materialized view of the data table with new primary keys defined for the materialized view.  

In Cassandra, the filtering function using WHERE clause is optimized for primary keys or columns with secondary index defined. When the filter is done on non-primary key columns or columns without indexes, we need to explicitly allow Cassandra to perform full table scan using ALLOW FILTERING option, which will be a penalty for query time. Thus, it is imperative to have the primary key or index configured based on the queries that will be performed on the data table in order to speed up the queries in Cassandra. 
