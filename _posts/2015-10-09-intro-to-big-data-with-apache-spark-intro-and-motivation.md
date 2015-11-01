---
layout: post
title: "Big Data with Apache Spark: Intro and Motivation"
---

#### Data Science vs Databases:

In contrast to databases which have valuable data such as bank or medical records, data science has 
cheap data like web logs, GPS data, sensor data. So data science deals with Big Data.

**CAP theorem** for data science: we can only have 2 of the 3 following properties, we can’t have all.

- Consistency
- Availability
- Partition tolerance

Databases use SQL, in data science we use NoSQL data stores. Much of the data in data science are 
non-structured or semi-structured, cannot be stored in relational databases. NoSQL has much more rich 
queries than SQL.

#### Data Science vs Scientific Computing

**SC**: physics-based models; **DS**: general ML classifier

**SC**: mostly deterministic, precise; **DS**: statistical models, handle unmodeled complexity

**SC**: run on supercomputers; **DS**: run on cheap clusters like EC2

#### Data Science vs Machine Learning

**ML**: developing new models; **DS**: explore models, build and tune hybrids

**ML**: Prove mathematical properties of models; **DS**: Understand empirical properties

**ML**: Improve/validate on a few small clean datasets; **DS**: Develop/use tools that can handle massive datasets

**ML**: Publish a paper; **DS**: take action!

#### Doing data science

1. Identify the problem
2. Collect data
3. Clean the data
4. Build models
5. Evaluate models
6. Communicate results

Cloud Computing enables Data Science: cheap clusters you can rent.

#### What’s hard about data science

- Overcoming assumptions
- Making ad-hoc explanations of data patterns
- Not checking enough (validate models, data pipeline integrity, etc.)
- Overgeneralizing
- Communication
- Using statistical tests correctly
- Prototype —> Production
- Data pipeline complexity

#### **ETL**: Extract, Transform, Load

1. **Extract** data from the sources: file, database, event log, website, Hadoop Distributed File System (HDFS)
2. **Load** data into the sink: Python, R, SQLite, NoSQL store, files, HDFS, Relational Database Management System (RDBMS)
3. **Transform** data at the source or sink

Plus:

- Data transfer, serialization/deserialization
- Transformation pipeline or workflow
- Recording the execution of a workflow is known as capturing lineage or provenance. 
Spark’s Resilient Distributed Datasets do this automatically.

#### Data Science Roles:

**2 kinds of individual: business person, programmer.**

- businessperson: excel functions/charts, VB
- programmer: 
  - Data sources: web scraping, web services API; excel sheets exported as Comma Separated Values (csv); 
  database queries
  - ETL: wget, curl, Beautiful Soup, lxml
  - Data Warehouse: flat files
  - Business Intelligence and Analytics: Numpy, Matplotlib, R, Matlab, Octave

**2 kinds of organizations: an enterprise and a web company.**

Other archetypes of individual:

- Hackers: efficient programmers who uses R/Matlab, Python, SQL/Pig
- Scripters: R/Matlab
- Application Users: SAS, SPSS

#### Big Data and Hardware Trends

Traditional data analysis tools: Unix shell cmd, Pandas, R. They all run on a single machine.
Now, 1TB of disk is only $35, but it takes up to 3 hours to read 1TB of data.
Solution: distribute data over large clusters.

For example, a task such as counting words in really big documents can be done better with a cluster.
In the cluster, some machines only process/store a specific part of the result.

Why is cluster computing difficult?

- Moving data is hard
- Node failure
- Node runs slow

Solution 1: MapReduce

- MapReduce deals with failures and slow tasks by re-launching the tasks on other machines.
- MapReduce handles the execution of Map and Reduce functions on many machines including the **shuffling 
of data between Map and Reduce functions**. It also automatically **recovers from both machine failures 
and slow machines.**

Solution 2: Spark

- Spark motivation: Disk I/O is very slow. Running MapReduce iteratively —> speed of disk
- Memory is cheap now: process in-memory, do not write to disk
- Spark abstraction: Resilient Distributed Datasets (RDDs)

RDDs enable us to write programs in terms of operations on distributed datasets. They are partitioned
collections of objects spread across a cluster, stored in memory or on disk. RDDs are built and manipulated
through a diverse set of parallel **transformations**: `map`, `filter`, `join`; and **actions**: `count`, 
`collect`, `save`. Also, RDDs are automatically rebuilt on machine failure.

On top of Apache Spark, there are **Spark SQL**, **Spark Streaming**, **MLlib** (machine learning), 
**GraphX** (graph).

*Comments:*

Using memory instead of disks offers two huge benefits. The first benefit is that memory is much faster 
than disks. The time to read or write a value to memory is only a few nanoseconds, while the time to 
read or write is several milliseconds - that means **memory is a million times faster than disks**. The 
second benefit is that keeping intermediate results in memory means that **they do not have to be converted 
into a format that can be stored on disks**. The process of converting a memory object to a disk object is 
called serialization and the process of converting a disk object to a memory object is called 
deserialization. Serializing and deserializing objects is a very expensive and time consuming process. 
Keeping intermediate results in memory avoids this significant overhead. Taken together, the faster 
access times and avoidance of serialization/deserialization overhead make Spark much faster than Map Reduce 
- up to 100 times faster!

<br>

#### Difference between Spark and Hadoop MapReduce

<img src="/assets/images/sparkVsHadoop.png" alt="Drawing" style="width: 800px;"/>


