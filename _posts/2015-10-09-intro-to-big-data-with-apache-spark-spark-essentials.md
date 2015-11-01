---
layout: post
title: "Big Data with Apache Spark: Spark Essentials"
comments: True
---

#### PySpark: Python Programming Interface to Spark

A Spark program first creates a SparkContext object

- Tells Spark how and where to access a cluster
- pySpark shell and Databricks Cloud automatically create the **sc** variable
- iPython and programs must use a constructor to create a new SparkContext

Use **SparkContext** to create RDDs

The master parameter for **sc** determines which type and size of cluster to use

- local clusters. can use 1 or K (#cores) worker threads
- remote clusters. can connect to a Spark cluster or an Apache Mesos cluster
- In the lab, the master param is set

Resilient Distributed Datasets

- The primary abstraction in Spark
- **Immutable** once constructed
- Track lineage info to efficiently recompute lost data
- Enable operations on collection of elements in parallel

You construct RDDs

- by **parallelizing** existing Python collections (lists)
- by **transforming** an existing RDD
- from **files** in HDFS or any other storage system

<img src="/assets/images/RDD1.png" alt="Drawing" style="width: 400px;"/>
<img src="/assets/images/RDD2.png" alt="Drawing" style="width: 400px;"/>
<img src="/assets/images/RDD3.png" alt="Drawing" style="width: 400px;"/>
<img src="/assets/images/RDD4.png" alt="Drawing" style="width: 400px;"/>
<img src="/assets/images/RDD5.png" alt="Drawing" style="width: 600px;"/>

Transformations are not calculated right away, they are like a recipe

Review: Python lambda functions
- small anonymous functions (not bound to a name)
- `lambda a, b: a + b`
- Can use lambda functions when function objects are required
- Restricted to a single expression

Transformations: (`map`, `filter` run in drivers, `lambda`s run in workers)

```
>>> rdd = sc.parallelize([1, 2, 3, 4])
>>> rdd.map(lambda x: x*2)
RDD: [1, 2, 3, 4] -> [2, 4, 6, 8]

>>> rdd.filter(lambda x: x%2 == 0)
RDD: [1, 2, 3, 4] -> [2, 4]

>>> rdd2 = sc.parallelize([1, 4, 2, 2, 3])
>>> rdd2.distinct()
RDD: [1, 4, 2, 2, 3] -> [1, 4, 2, 3]

>>> rdd = sc.parallelize([1, 2, 3])
>>> rdd.Map(lambda x: [x, x+5])
RDD: [1, 2, 3] -> [[1, 6], [2, 7], [3, 8]]

// flatMap: flattens the list of lists -> list

>>> rdd.flatMap(lambda x: [x, x+5])
RDD: [1, 2, 3] -> [1, 6, 2, 7, 3, 8]

// Function literals are closures automatically passed to workers
```

Transformation is not executed right away: **lazy evaluation**. 
Spark remembers the recipe, and perform it when there’s an action.

<br>
#### Spark Actions

<img src="/assets/images/RDD6.png" alt="Drawing" style="width: 600px;"/>

```
>>> rdd = sc.parallelize([1, 2, 3])
>>> rdd.reduce(lambda a, b: a * b)
Value: 6

>>> rdd.take(2)
Value: [1, 2] # as list

>>> rdd.collect()
Value: [1, 2, 3] # as list. Careful, make sure the driver’s memory is big enough!

# getting data out of rdd
>>> rdd = sc.parallelize([5, 3, 2, 1])
>>> rdd.takeOrdered(3, lambda s: -1 * s)
Value: [5, 3, 2] # as list at the driver. lambda s: -1 * s makes the list in decreasing value
```

<br>
#### Spark Programming Model

```
lines = sc.textFile(“…”, 4)     # 4 partitions
print lines.count() # count: action
# count(): read data; sum # lines within partitions; combine sum in driver

comments = lines.filter(isComment)
print lines.count(), comments.count()   # comments.count() will cause recomputation, re-reading the data
```

To Avoid Reloading the Data

```
lines = sc.textFile(“…”, 4)     # 4 partitions
lines.cache()   # save, don’t recompute! Will run much much faster!
comments = lines.filter(isComment)
print lines.count(), comments.count()
```

Spark life cycle

- Create RDDs from external data or **parallelize** a collection in your driver program
- Lazily transform them into new RDDs
- `cache()` some RDDs for reuse
- Perform **actions** to execute parallel computation and produce results

Spark Key-Value RDDs

- Similar to MapReduce, Spark supports Key-Value pairs
- Each element of a Pair RDD is a pair tuple

```
>>> rdd = sc.parallelize([(1, 2), (3, 4)])
RDD: [(1, 2), (3, 4)]
```

- Key-Value Transformation: `reduceByKey(func)`, `sortByKey(func)`, `groupByKey(func)`

```
>>> rdd = sc.parallelize([(1, 2), (3, 4), (3, 6)])
>>> rdd.reduceByKey(lambda a, b: a + b)
RDD: [(1, 2), (3, 4), (3, 6)] -> [(1, 2), (3, 10)]

>>> rdd2 = sc.parallelize([(1, ‘a’), (2, ‘c’), (1, ‘b’)])
>>> rdd2.sortByKey()
RDD: [(1, ‘a’), (2, ‘c’), (1, ‘b’)] -> [(1, ‘a’), (1, ‘b’), (2, ‘c’)]

>>> rdd2 = sc.parallelize([(1, ‘a’), (2, ‘c’), (1, ‘b’)])
>>> rdd2.groupByKey()
RDD: [(1, ‘a’), (2, ‘c’), (1, ‘b’)] -> [(1, [‘a’, ‘b’]), (2, [‘c’])]
# be careful, groupByKey() can cause a lot of data movement across the network and create large Iterables at workers
```

<br>
#### PySpark Closures

PySpark shared variables

- Broadcast Variables
  - efficiently send large, read-only value to all workers
  - saved at workers for use in one or more Spark operations
  - like sending a large, read-only lookup table to all the nodes

- Accumulators
  - aggregate values from workers back to driver
  - only driver can access value of accumulator
  - for tasks, accumulators are write-only.
  - use to count errors seen in RDD across workers

**Broadcast variables**: send large dataset to workers, cache at workers

```python
# at the driver
>>> broadcastVar = sc.broadcast([1, 2, 3])
# at the worker (code passed via a closure)
>>> broadcastVar.value
[1, 2, 3]
```

Example,

```python
# Lookup the locations of the call signs on the RDD contactCounts.
# We load a list of call sign prefixes to country code to support this lookup.
# signPrefixes = loadCallSignTable(), load repeatedly for each worker, NOT GOOD
signPrefixes = sc.broadcast(loadCallSignTable())

def processSignCount(sign_count, signPrefixes):
    country = lookupCountry(sign_count[0], signPrefixes.value)
    count = sign_count[1]
    return (country, count)

countryContactCounts = 
(contactCounts.map(processSignCount).reduceByKey(lambda x, y: x+y)))
```

**Accumulators**

- variables that can only be “added” to by associative operations
- used to efficiently implement parallel counters and sum
- only driver can read an accumulator’s value, not tasks

```python
>>> accum = sc.accumulator(0) # create the accumulator, set value to 0
>>> rdd = sc.parallelize([1, 2, 3, 4])
>>> def f(x):
>>>     global accum
>>>     accum += x

>>> rdd.foreach(f)
>>> accum.value
Value: 10

////

# another example, count empty lines
# create the rdd: file
file = sc.textFile(inputFile)
# create accumulator[int] init to 0
blanklines = sc.accumulator(0)

def extractCallSigns(line):
    global blankLines  # make the global variable accessible
    if (line == “”):
        blankLines += 1
    return line.split(“ “)

# new rdd: callSigns
callSigns = file.flatMap(extractCallSigns)
print “Blank lines: %d” % blankLines.value
```

Tasks at workers cannot access accumulator’s values
Tasks see accumulators as write-only vars
Accumulators can be used in actions and transformations
Actions: each task’s update to accumulator is applied only once
Transformations: no guarantees (use only for debugging)
Types: integers, double, long, float. See lab for example of custom type.
Summary:
master param: specify the # workers
when create an RDD, specify # partitions for the RDD, and Spark will automatically create that RDD spread across the workers.
when perform transforms or actions, Spark automatically push a closure containing that function to the workers so that it can run at the workers.

<br>
#### Spark tutorial

- transforms: `map()`; `filter()`
- actions: `count()`; `collect()`; `first()` is equivalent to `take(1)`; `take()`; `top()` (descending order); 
`takeOrdered()` (ascending order); `reduce()`
- advanced actions: `takeSample(withReplacement=T/F, num, seed)`, `countByValue()`: a histogram dictionary
- additional transforms: `flatMap()`; `groupByKey()`: only use when `reduceByKey()` is not helpful; `reduceByKey()`
- advanced transforms: `mapPartitions()`, `mapPartitionWithIndex()`
- Caching RDDs: `.cache()`, `.is_cached`
- `.unpersist()`, `.getStorageLevel()`, `.toDebugString()` 

When learning Spark, for easy debugging, this form is recommended:

```python
RDD.transformation1()
RDD.action1()
RDD.transformation2()
RDD.action2()
```

When more experienced with Spark, write like this,

```python
RDD.transformation1().transformation2().action()
```

Readability and code style: to make the expert coding style more readable, enclose the statement in 
parentheses and put each method, transformation, or action on a separate line.

```python
# Final version
(sc
 .parallelize(data)
 .map(lambda y: y - 1)
 .filter(lambda x: x < 10)
 .collect())
```



