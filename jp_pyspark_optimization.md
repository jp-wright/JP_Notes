# JP PySpark Notes (Not Coursera class)
https://data.solita.fi/pyspark-execution-logic-code-optimization/ 

# PySpark execution logic and code optimization

PySpark looks like regular python code. In reality the distributed nature of the execution requires the whole new way of thinking to optimize the PySpark code.
October 11, 2019


This article will focus on understanding PySpark execution logic and performance optimization. PySpark DataFrames are in an important role.

To try PySpark on practice, get your hands dirty with this tutorial: Spark and Python tutorial for data developers in AWS

## DataFrames in pandas as a PySpark prerequisite

PySpark needs totally different kind of engineering compared to regular Python code.

If you are going to work with PySpark DataFrames it is likely that you are familiar with the pandas Python library and its DataFrame class.

Here comes the first source of potential confusion: despite their similar names, PySpark DataFrames and pandas DataFrames behave very differently. It is also easy to confuse them in your code. You might want to use suffix like **_pDF** for pandas DataFrames and **_sDF** for Spark DataFrames.

The pandas DataFrame object stores all the data represented by the data frame within the memory space of the Python interpreter. All of the data is easily and immediately accessible. The operations on the data are executed immediately when the code is executed, line by line. It is easy to print intermediate results to debug the code.

However, these advantages are offset by the fact that you are limited by the local computer’s memory and processing power constraints – you can only handle data which fits into the local memory. But since the operations are done in memory, with a basic data processing task you do not need to wait more than a few minutes at maximum.

## PySpark DataFrames and their execution logic

The PySpark DataFrame object is an interface to Spark’s DataFrame API and a Spark DataFrame within a Spark application. The data in the DataFrame is very likely to be somewhere else than the computer running the Python interpreter – e.g. on a remote Spark cluster running in the cloud.

There are two distinct kinds of operations on Spark DataFrames: **transformations and actions**. Transformations describe operations on the data, e.g. filtering a column by value, joining two DataFrames by key columns, or sorting data. Actions are operations which take DataFrame(s) as input and output something else. Some examples from action would be showing the contents of a DataFrame or writing a DataFrame to a file system.

The key point to understand how Spark works is that **transformations are lazy**. Executing a Python command which describes a transformation of a PySpark DataFrame to another does not actually require calculations to take place. Ordering by a column and calculating aggregate values, returning another PySpark DataFrame would be such transformation. Rather, the operation is added to the graph describing what Spark should eventually do.

When an action is requested – e.g. return the contents of this Spark DataFrame as a Pandas DataFrame – Spark looks at the processing graph and then **optimizes the tasks which needs to be done**. This is the job of the [Catalyst optimizer](https://databricks.com/glossary/catalyst-optimizer), and it enables Spark to optimize the operations to very high degree.

Also, the actual computation tasks **run on the Spark cluster**, meaning that you can have huge amounts of memory and processing cores available for the actual computation, even without resorting to the top-of-the-line virtual machines offered by cloud providers.


## Consider caching to speed up PySpark

However, the highly optimized and parallelized execution comes at a cost: **it is not as easy to see what is going on at each step**. Looking at the data after some transformations means that you have to gather the data, or its subset, to a single computer. This is an action, so Spark has to determine the computation graph, optimize it, and execute it.

If your dataset is large, this may take quite some time. This is especially true **if caching is not enabled** and Spark has to start by reading the input data from a remote source – such as a database cluster or cloud object storage like S3.

You can alleviate this by *caching* the DataFrame at some suitable point. Caching causes the DataFrame partitions to be retained on the executors and not be removed from memory or disk unless there is a pressing need. In practice this means that the cached version of the DataFrame is available quickly for further calculations. However, playing around with the data is still not as easy or quick as with pandas DataFrames.

## Use small scripts and multiple environments in PySpark

As a rule of thumb, one PySpark script should perform just one well defined task. This is due to the fact that *any action triggers the transformation plan execution from the beginning*. Managing and debugging becomes a pain if the code has lots of actions.

The normal flow is to read the data, transform the data and write the data. Often the write stage is the only place where you need to execute an action. Instead of debugging in the middle of the code, you can review the output of the whole PySpark job.

With large amounts of data this approach would be slow. You would have to wait a long time to see the results after each job.

My suggestion is to create environments that have different sizes of data. In the environment **with little data you test the business logic and syntax**. The test cycle is rapid as there’s no need process gigabytes of data. Running the PySpark script with **the full dataset reveals the performance problems**.

This goes well together with the traditional dev, test, prod environment split.

## Favor DataFrame over RDD with structured data

RDD (Resilient Distributed Dataset) can be any set of items. For example, a shopping list.

["apple", "milk", "bread"]

RDD is the low-level data representation in Spark, and in earlier versions of Spark it was also the only way to access and manipulate data. However, the DataFrame API was introduced as an abstraction on top of the RDD API. As a rule of thumb, **unless you are doing something very involved** (and you really know what you are doing!), **stick with the DataFrame API**.

*DataFrame* is a tabular structure: a collection of Columns, each of which has a well defined data type. If you have a description and amount for each item in the shopping list, then a DataFrame would do better.

+-------+-----------+------+
|product|description|amount|
+-------+-----------+------+
|apple  |green      |5     |
|milk   |skimmed    |2     |
|bread  |rye        |1     |
+-------+-----------+------+

This is also a very **intuitive representation for structured data**, something that can be found from a database table. PySpark DataFrames have their own methods for data manipulation just like pandas DataFrames have.

## Avoid User Defined Functions in PySpark

As a beginner I thought PySpark DataFrames would integrate seamlessly to Python. That’s why I chose to use UDFs (User Defined Functions) to transform the data.

A UDF is simply a Python function which has been registered to Spark using PySpark’s [spark.udf.register](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.UDFRegistration) method.

With the small sample dataset it was relatively easy to get started with UDF functions. When running the PySpark script with more data, Spark *popped an OutOfMemory error*.

Investigating the issue revealed that the **code could not be optimized when using UDFs**.  To Spark’s Catalyst optimizer, the UDF is a black box. This means that Spark may have to read in all of the input data, even though the data actually used by the UDF comes from a small fragments in the input -- i.e. doing data filtering at the data read step near the data, i.e. *predicate pushdown*, cannot be used.

Additionally, there is a performance penalty: on the Spark executors, where the actual computations take place, data has to be converted (serialized) in the Spark JVM to a format Python can read, a Python interpreter spun up, the data deserialized in the Python interpreter, the UDF executed, and the result serialized and deserialized again to the Spark JVM. All of this takes significant amounts of time!

##### Double Serialization Penalty of UDFs
1. data has to be converted (serialized) in the Spark JVM to a format Python can read, 
2. a Python interpreter spun up, 
3. the data deserialized in the Python interpreter, 
4. the UDF executed, 
5. and the result serialized and deserialized again to the Spark JVM. 

The recommendation is to stay in [native PySpark dataframe functions](https://spark.apache.org/docs/2.4.0/api/python/pyspark.sql.html) whenever possible, since they are translated directly to native Scala functions running on Spark.

If you absolutely, positively need to do something with UDFs in PySpark, consider using the [Pandas vectorized UDFs introduced in Spark 2.3](https://databricks.com/blog/2017/10/30/introducing-vectorized-udfs-for-pyspark.html) – the UDFs are still a black box to the optimizer, but at least the performance penalty of moving data between JVM and Python interpreter is lot smaller.

## Number of partitions and partition size in PySpark

In order to process data in a parallel fashion on multiple compute nodes, Spark splits data into *partitions*, smaller data chunks. A DataFrame of 1,000,000 rows could be partitioned to 10 partitions having 100,000 rows each. Additionally, the computation jobs Spark runs are split into tasks, each task acting on a single data partition. Spark cluster has a driver that distributes the tasks to multiple executors. This means that the datasets can be much larger than fits into the memory of a single computer – as long as the partitions fit into the memory of the computers running the executors.

In one of the projects our team encountered an out-of-memory error that we spent a long time figuring out. Finally we found out that the problem was a result of too large partitions. The data in a partition could simply not fit to the memory of a single executor node.

Too few partitions also make the execution inefficient. Some of the executor cores idle while others are working on a full steam, if there are not as many partitions as there are available cores (or, technically, available *slots*)

However, having a large amount of small partitions is not optimal either – shuffling the data in the small partitions is inefficient. Also reading and writing to disk (not to mention a network destination) in small chunks potentially increases the total execution time.

[The Spark programming guide](https://spark.apache.org/docs/2.4.3/sql-performance-tuning.html#other-configuration-options) **recommends 128 MB partition size as the default**. For 128 GB of data this would mean 1000 partitions. Without going too deep in the details, *consider partitioning as a crucial part of the optimization toolbox*. If your partitions are too large or too small, you can use the [coalesce()](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.DataFrame.coalesce) and [repartition()](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.DataFrame.repartition) methods of DataFrame to instruct Spark to modify the partition distribution. The number of partitions in a DataFrame sDF can be checked with `sDF.rdd.getNumPartitions()`.

## Summary – PySpark basics and optimization

PySpark offers a versatile interface for using powerful Spark clusters, but it requires a completely different way of thinking and being aware of the differences of local and distributed execution models. The functionality offered by the core PySpark interface can be extended by creating User-Defined Functions (UDFs), but as a tradeoff the performance is not as good as for native PySpark functions due to lesser degree of optimization. Partitioning the data correctly and with a reasonable partition size is crucial for efficient execution – and as always, good planning is the key to success.


