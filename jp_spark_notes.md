# JP Spark Notes

Notes about Spark generally and PySpark specifically.
Taken from the udemy.com class [spark and python for big data with pyspark](https://www.udemy.com/spark-and-python-for-big-data-with-pyspark/learn/v4).  All course materials and notes are available in the `Python-and-Spark-for-Big-Data-master` folder.  The lectures are presented in nice Jupyter Notebooks.

See the [official documentation](http://spark.apache.org/docs/2.1.0/api/python/pyspark.sql.html).

Here is a great multi-part PySpark guide from Hackers and Slackers.  It covers much of the _how_ and the _why_ of Spark.
+ [Part 1](https://hackersandslackers.com/learning-to-use-apache-spark-pyspark/)
+ [Part 2](https://hackersandslackers.com/basic-dataframe-transformations-in-pyspark/)
+ [Part 3](https://hackingandslacking.com/dataframe-transformations-in-pyspark-continued-907b1e870442) (Really great column manipulation details)

Update: they combined all parts into [one long article](https://hackersandslackers.com/transforming-pyspark-dataframes/)

## Getting Started
The Udemy class details four separate ways to install Spark.
1. Use Ubuntu and a VirtualBox (no thanks; doing this to make all systems work for vid)
2. Install locally, run on AWS EC clusters (slow, free for a year)
3. Use databricks.com for online hosting of Spark instances, free 6 GB tier (yes)
4. Install locally, run on AWS RedShift (fast and nice, not free)

The most logical way for work outside the class is to install locally and then run it on a cluster somewhere.  Running it all locally is doable for the class, but for truly "big" data that isn't an option (hence why Spark was developed in the first place, to handle sets of data that are too large to fit on a single machine).  Thus, installing locally and hosting on a cluster will be the ultimate use case at some point.  I just didn't do it for this class because of the AWS headache.  

Spark was written in Scala (and, if possible, it is best to use Scala with Spark.  I don't have any Scala experience so I'm using Python's PySpark for now), which itself comes from Java.  In order to install and run Spark locally, you will need the latest Java Virtual Machine (JVM) [development kit installed](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).  Since it comes from Oracle, there is a fair amount of surreptitious bloatware and insidious packages installed.  Google for the best ways to remove them from your machine, especially if you have an otherwise spyware-free Mac.

### Installing Spark
PySpark needs two things installed: Java and Spark.  
If you use [databricks.com](databricks.com) you don't have to install any of this since everything is handled on their servers.  I've used databricks.com largely as a testing site to accompany the online class; it appears to be fully fledged in cluster deployment if you wish to use it that way.

##### Java
Datacamp has a good overview of PySpark installation on [this page](https://www.datacamp.com/community/tutorials/apache-spark-python).  Note the [JDK](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) link in it is slightly dated (JDK SE8).  To get the most recent version of JDK, type `java` into the Terminal and choose "more info" on the ensuing dialog box.  As of March 2018, the newest version is [version 10](http://www.oracle.com/technetwork/java/javase/downloads/jdk10-downloads-4416644.html).  If you already have Java installed, this dialog box might not appear.  I'm unsure how you update Java then -- likely delete it and install a fresh new version.

Please note that Apple discontinued pre-installing Java on all their machines due to high frequency of security issues with Java.  Java will require you to frequently update it, but even in so doing you are exposing yourself to more malware risks through browser exposure.  Because of this, unless actively using it, [disabling Java is recommended](http://osxdaily.com/2012/09/08/how-to-disable-java/) and easy.  

Oracle, the makers of Java are [notorious for bundling adware](https://www.computerworld.com/article/2893607/software-integration/java-mac-ask-toolbar-oracle-itbwcw.html) into Java, causing users to have to [manually uninstall](https://www.imore.com/oracle-bundles-adware-mac-java-installer) the adware bloat.

If you wish to [uninstall Java entirely](http://osxdaily.com/2017/06/16/uninstall-java-mac/), you can.

The lasting impression to take is this:  
Unless you need to install Java on your Mac (in this case, for Spark), then don't do it.  If you do install it, complete your project in Spark, and then no longer acutely need it, you should disable it at the minimum.

##### Spark
Download the [latest version of Spark](https://spark.apache.org/downloads.html).  Unzip / tar the .zip/.tar file.  Move the resulting folder, which will be named something like `spark-2.3.0-bin-hadoop2.7`, to the following directory:  

`/usr/local/spark`

This directory may not exist yet, so create it first or use the command line from the `Downloads` directory, or wherever you untarred the Spark file (again, use whatever your Spark download version is):  

`mv spark-2.3.0-bin-hadoop2.7 /usr/local/spark`

##### PySpark
As of 2018, PySpark is now available in PyPI and Anaconda (woohoo).  Install it with:
`conda install pyspark` or `pip install pyspark`, depending upon your setup (favoring `conda` if you have Anaconda installed, of course).

You can run an interactive PySpark shell by entering the following command in the Terminal from the directory of your Spark install (e.g. `/usr/local/spark/spark-2.3.0-bin-hadoop2.7`):  

`./bin/pyspark`

If you do not have Java installed, you will be prompted to install it here.


#### Local Script / Notebook
Note I used [databricks.com](databricks.com), a site created by one of the founders of Spark, as my Spark app.  
If using a script / Notebook, would need to first create a Spark session with the following import:  

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("<AppNameHere>").getOrCreate()
df = spark.read.csv('fft_clas_stats.csv')           # many read methods available
```

This builder process can take a while depending on machine used.

#### Databricks.com
databricks.com runs instances of Spark in online Notebooks and allows a small 6 GB free cluster to be used.  We do not need to build a Spark session because every workspace in it is a session.  For PySpark, we only have to import it and define a SQL context for files we've uploaded, assuming they can be accessed via a SQL schema.

For example, I uploaded my testing csv _du jour_, `fft_class_stats.csv`, to the "data" storage in my databricks account and then used it in my first PySpark DF test.  When you upload a local data file, you can and should specify the datatype for each column. (This is another reason this FFT csv is a great tester file, because it has `str`, `int`, and `float` dtypes).

The `sqlContext` class was for Spark 1.0.  With the advent of Spark 2.0, it has been replaced by the `SparkSession` class above, though is kept alive for backward compatibility.  We cannot (to my knowledge) use the `.read.csv()` method in Databricks because it doesn't store uploaded data as a CSV (even if you upload an actual CSV) -- it is converted to a SQL DB.

Here is the old way in databricks:
```python
import pyspark
df = sqlContext.sql("SELECT * FROM fft_class_stats")
```

Here is the new way:
```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("fft").getOrCreate()
data = spark.sql("SELECT * FROM fft_class_stats")
```

Either way, we are using Spark SQL to return data from a SQL database instead of loading it directly from a .csv as above.

Now I have a Spark DF of my FFT csv ready to roll, either via a local script or databricks.com.


<BR>


## Spark Data Frames
_______

As of Spark 2.0, Spark and its various apps are transitioning to the more universal _data frame_ object, built on SQL databases, instead of the initial _RDD_.  Spark Data Frames are very similar to data frames of Pandas and R, happily.  Using PySpark offers welcomed overlap with Pandas commands and attributes.  Since Spark DFs are built on SQL databases, we also have the ability to leverage raw SQL queries to interact with Spark DFs.

There are some key differences between Spark DFs and Pandas/R DFs.  These differences stem from the distributed nature of a Spark DF vs. the local in-memory nature of Pandas and R DFs.  One significant fact to grasp is that a distributed Spark DF is _immutable_, thus we cannot assign new values or properties to an existing Spark DF and "overwrite" it.  Every operation that changes a Spark DF will result in a new Spark DF.  For example, we cannot change Column A in a Spark DF from a `string` to a `float` value via assignment, as we would in Pandas and R.

```python
## This is not allowed: 'DataFrame' object does not support item assignment
df['Column_A'] = df['Column_A'].cast('float')
```

#### From Pandas
We can convert a Pandas DF into a Spark SQL DF with the basic `createDataFrame` command by supplying an existing Pandas DF as the data:  

`df_spark = SparkSession.createDataFrame(df_pandas)` -- v2.0 or newer  
`df_spark = sqlContext.createDataFrame(df_pandas)` -- pre v2.0

To go the other way and turn a Spark DF into a Pandas DF, simply use the `df.toPandas()` command.

#### General Commands
```python
df.read.csv("<path>.csv")  
df.printSchema()
# Output:
root
 |-- Job_ID: string (nullable = true)
 |-- Job: string (nullable = true)
 |-- Identity: string (nullable = true)
 |-- Gender: string (nullable = true)
 |-- Level: integer (nullable = true)
 |-- Role: string (nullable = true)
 |-- Association: string (nullable = true)
 |-- HPm: integer (nullable = true)
 |-- MPm: integer (nullable = true)
 |-- PAm: integer (nullable = true)
 |-- MAm: integer (nullable = true)
 |-- SPm: integer (nullable = true)
 |-- Move: integer (nullable = true)
 |-- Jump: integer (nullable = true)
 |-- CEV: float (nullable = true)
 |-- HPc: integer (nullable = true)
 |-- MPc: integer (nullable = true)
 |-- PAc: integer (nullable = true)
 |-- MAc: integer (nullable = true)
 |-- SPc: integer (nullable = true)    

df.columns  
df.describe()  
df.head()  
```

#### Showing the Data
Anytime we want to see the data, we must use a `.show()` method!  
`.show(numRows, truncate)` -- both arguments optional; can use `.show(truncate=False)` to show all results.  __UPDATE__ I think this is incorrect...

`df.describe().show(100)` -- would show up to 100 rows if that many are present.   
`df.describe().show(truncate=False)` -- would show results.   
`df.columns` -- show column names; note this is an _attribute_ and not a method, a'la Pandas.


#### Collecting the Data
As expected, the `.show` method simply prints out the results of an operation.  When we want to actually return the results so we can save them in a variable or process them, we must use `.collect`.

`df.filter(df['Level'] > 20).collect()` would return (not show) a DF of characters whose level was >20.  

The return type is a `list` of `Row` objects (see more below).  This means you will have to index into the collected list to access specific rows of data!



#### Defining Schema / Datatypes
As usual, most true data is messy and Spark's internal type estimator will make mistakes or be stumped by mixed types in a column.  To manually enforce type as needed, use the following:

```python
from pyspark.sql.types import StructField, StringType, IntegerType, StructType
```

We will use these constructors to create our own Schema for the DB we are using.

##### Creating Fields and Types
Schemas can be defined by `StructType` constructors, which themselves are made up of `StructField` fields (columns).

`StructField(field_name, dtype, nullable)` is how we set a `dtype` for a specific column.  It has three arguments: __field name__ (str), __type__ (dtype), __bool nullable__ (T/F, can it be a `null` value?)

For example, let's say the FFT csv import incorrectly interpreted the column `Level` as a `string` instead of an `int`.  Note I don't know if when passing a schema to the `.read` function, such as using `final_struct` below, if we must specify a `StructField` for _every_ column in the table or not.   I can't check until I install Spark on my local.

Once a list of our desired StructFields is assembled, we pass that into a `StructType` constructor and use that as our `schema` argument upon read in.

```python
data_schema = [StructField("Level", IntegerType(), True)]]
final_struct = StructType(fields=data_schema)
df = spark.read.csv('fft_clas_stats.csv', schema=final_struct)
## Will this only modify the "Level" column leave the rest as is, or will it error b/c I left other columns undefined?
```


#### Accessing Data in the Data Frame
##### Dictionary-style Indexing
Like Pandas DFs, a Spark DF uses dictionary-style indexing where we supply the key to the DF column(s) we want.  However, there are some additional steps to grabbing data.  If we just index in a'la a Pandas DF, we return a `Column` object (called a `Series` in Pandas), and this object is not something we can currently see via `.show()`.  For many of the operations needed when working in the Spark DF we will need to provide a `Column` data type -- don't forget that this is achieved using this Pandas style dictionary index.

```python
## This returns a 'Column' object:

df['Level']
## output: Column<b'Level'>

type(df['Level'])
## output: pyspark.sql.column.Column
```

##### `.select()` Method
In order to actually access the data, we must use the `.select()` method, akin to SQL.  And to _see_ the data, we must use the `.show()` method as noted above.  The printed result is akin to Pandas in IPython.  Using the `.select()` method returns a `DataFrame` object, _not_ a Spark `Column`.  Pandas doesn't operate this way, instead returning a `Series` whenever a single row or column is sliced.  This is an important difference, as it means all DF operations are uniformly valid whether we have chosen a single row/column or multiple!  

_Small technical note_:  
We can `.select()` a single column without enclosing it in a list, such as `df.select('Level')`, and Spark should handle this.  However, there are apparently times when this might not be the case, so it is a best practice to always enclose the columns desired in a `list` inside a `.select()` statement to avoid unexpected issues.

```python
## This returns a 'DataFrame' object:

df.select(['Level'])
## output: DataFrame[Level: int]

df.select(['Level']).show()
## output:
+-----+
|Level|
+-----+
|    1|
|   10|
|   26|
|    1|
...

type(df.select('Level'))
## output: pyspark.sql.dataframe.DataFrame
```

Like Pandas, Spark DFs are `DataFrame` objects.  Unlike Pandas, Spark DFs also have separate `Column` and `Row` objects.  Slicing a row returns a `Row` object, same for a column as we saw above.  In Pandas, rows _and_ columns are actually the same type of object: a `Series`.  There are advantages to having a generalized grouping of data like a `Series`, but there are also advantages to having separate objects for both a row and a column, like Spark has.  The primary purpose for Spark's granular data types in the DF are a result of the precision needed in dealing with distributed data and computing.  Spark needs to know _exactly_ what operation is being done to which data and where that data is in the distributed system.  This makes sense.

If we had wanted to select multiple columns, we simply pass in a list of columns, `['Level', 'Role']`, as in Pandas.  _Note that since Spark DFs are distributed, we cannot (easily) access a single row, unlike in Pandas._  This is one of the larger differences between the two and one of the more difficult friction points in adapting to Spark.

#### Adding or Renaming Columns
Use the `.withColumn(name, data)` method to add new cols.  There are additions to this method for more complex operations, such as filtering and concatenation.  Note that this works when the column added (`data` in the function definition, here) is from the _same_ DF!  Appending a column from a separate DF is much messier.  

##### Adding a Column Using Data From the Same Dataframe
Arguments are __name__ (str) and __data__ (`Column` data type!).  
Use `.withColumnRenamed(old_name, new_name)` to rename columns.

These are __not__ in-place operations!

```python
## This adds a new column "Double_Level" which is just "Level" * 2
df.withColumn('Double_Level', df['Level'] * 2)

## Use the `col` function to make the data a `Column` type if needed
import pyspark.sql.functions as F
df.withColumn('Double_Level', F.col(<some_non_col_data>) * 2)
```

##### Rename a Column
```python
df.withColumnRenamed('Level', 'A_New_Level')
## Of confidence and power...
```

The other (non-`withColumn`) approaches include:
1. Using SQL by registering a Temporary Table
2. Pandas UDF
3. RDDs

They all work on a single table, meaning the data being added or manipulated must exist on the destination table already.

Read more about them in these nice articles:
1. [Towards Data Science](https://towardsdatascience.com/5-ways-to-add-a-new-column-in-a-pyspark-dataframe-4e75c2fd8c08). Very good, including how to train multiple individual models on each spark node (in Pandas UDF section)
2. [Hackers and Slackers](https://hackersandslackers.com/transforming-pyspark-dataframes/) (very good)


##### Adding a Column Using Data From a Separate Dataframe (Messy)
Okay, this gets messy.  It kind of outright sucks, to be honest.  Apart from the traditional `.join()` method, there's not a defined way to do this (which I find perplexing).  There appear to be a couple methods.

1. Join on a specific column
2. Create a unique identifier column to join on (caveat included)
3. Use `.lit()` to create a column of a single, repeated value



###### Join
Your standard SQL join that is also used in Pandas
`df1.join(df2, on=<shared column>, how={'inner', 'outer', 'left', 'right'})`

###### Create a Unique Identifier Column






Here are a couple articles giving details of these approaches.
2. [Spark By Example](https://sparkbyexamples.com/spark/spark-add-new-column-to-dataframe/)
3. [Stack Overflow](https://stackoverflow.com/questions/47028442/add-column-from-one-dataframe-to-another-dataframe-in-scala/47028782)






#### Using SQL Directly
Since this is a `spark.sql` DF, we can use SQL to access it directly.  To do so, we must first create a "temporary SQL view" and then use traditional SQL queries.  As someone who has basic knowledge of SQL, this doesn't appeal to me very much, but it is a nice feature and allows colleagues who have advanced SQL knowledge the ability to interact with Spark via SQL queries if desired.

```python
## View name as str, replaces if previously made.
df.createOrReplaceTempView("FFT_Sql")

results = spark.sql("SELECT * FROM FFT_Sql")
results.show()
```

It is also possible to use some DF methods without creating the SQL View table:

```python
## Uses the generalized df.filter() method with SQL query syntax.
df.filter("Level > 20").show()

## Can chain with .select() to return only specified columns.
df.filter("Level > 20").select(['Identity', 'Level']).show()
```


#### Filtering / Sub-selecting Data
This is how we will be operating on the data, generally.  It is very similar to Pandas masking, syntactically. The chained `.select()` method is optional, of course.  The same rules of using `&`, `|`, and `~` for the `and`, `or`, and `not` operators also apply, as does the rule of enclosing each condition of a multiple condition filter/mask in parentheses.

```python
df.filter(df['Level'] > 20).select(['Identity', 'Level']).show()

df.filter((df['Level'] > 20) & (df['Level'] < 40)).select(['Identity', 'Level']).collect()
## Note .collect() -- returns a List of Row objects; must index in for desired Rows!
```

#### `Row` and `Column` Objects
They have [many methods](http://spark.apache.org/docs/2.1.0/api/python/pyspark.sql.html#pyspark.sql.Row) available to them.  
Object | Method       | Description
-------|--------------|------------
Both   | `.asDict()`  | returns as a `dict`, can index in per normal.

The important thing to remember is the different object types returned by `.select()` vs. indexing.

Command             | Returned Object
--------------------|----------------
`.select()`         |  `DataFrame`
`df['<col_name>']`  |  `Column` / `Row`




#### Grouping / Aggregation
PySpark DFs use the `groupBy()` method.
See [documentation](http://spark.apache.org/docs/2.1.0/api/python/pyspark.sql.html#pyspark.sql.GroupedData) here.

You cannot select specific columns from a grouped object to return via the `.select` method because the cols get renamed via traditional SQL usage (e.g. "Level" becomes "avg(Level)").  Instead you must specify the col and aggregation desired in a `.agg(dict)` call.  Dict (or list) comprehension is our friend, here.  Also can traditional aggregation methods, like `.sum` or `.mean`, etc. for the entire grouped frame.  Can also use SQL star `*` to select all cols in the `.agg` call.

```python
group_cols = ['Association', 'Role']
extra_cols = ['Level', 'HPm', 'MPm', 'PAm', 'MAm']

## Show entire DF
df.groupBy(group_cols).mean().show()

## Show only desired cols - Dict Comp
df.groupBy(group_cols).agg({col: 'mean' for col in group_cols + extra_cols}).show()

## Show only desired cols - List Comp
## Note DataBricks appears to run Py3.5 so "f-strings" are not valid (fml).
df.groupBy(group_cols).mean().select(group_cols + ["avg({col})".format(col=col) for col in extra_cols]).show()
```

##### Transpose
Since Spark DFs are distributed by row, there is no way to _transpose_ them as we can in Pandas (or even Excel).  If the Spark DF is small enough to fit into local memory, we can convert it to a Pandas DF and then use Pandas' `.T` method to transpose it. Many Spark datasets are likely too large for this.

```python
df.toPandas().T
```

##### MultiIndex
Spark SQL has no "index", so things like a "multi-index" in Pandas do not apply.  In order to convert a Pandas MultiIndex to a Spark SQL DF, you have to flatten the Pandas DF first (i.e. `reset_index(drop=False, inplace=True)`)


### NaNs
PySpark DFs use the `.na` class attribute to interact with DF / row / col NaN values.  The functionality is nearly identical to Pandas.
```python
df.na.drop()                    # drops via row, any with NaN
df.na.drop(thresh=3)            # drops any row that has fewer than 3 non-null entries
df.na.drop(how='any')           # default arg.
df.na.drop(how='all')           # drops any row that has ONLY NaN vals
df.na.drop(subset=['ColA'])     # choose cols to consider NaNs in
```

Interestingly, when filling NaNs, Spark only fills by matching data types.
```python
df.na.fill(0)                   # Will only replace NaN in numeric columns
df.na.fill("0")                 # Will only replace NaN in string columns
df.na.fill(0, subset=['ColA'])  # Will only replace NaN in ColA if it is numeric...
```

To fill NaN with imputed or aggregated data, such as the mean, we must import these functions first.
```python
from pyspark.sql.functions import mean
mean_result = df.select(mean(df['Yards'])).collect()    # must .collect() to return actual data
## output: [Row(avg(Yards)=400.5)]      # note result is Row obj inside a list!
mean_val = mean_result[0][0]            # returns 400.5
df.na.fill(mean_val, subset=['Yards'])

## one line:
df.na.fill(df.select(mean(df['Yards'])).collect()[0][0], subset=['Yards'])
```

### Dates and Timestamps
Must import desired functions.
```python
from pyspark.sql.functions import hour, month, year, dayofmonth, dayofyear, weekofyear, format_number, date_format

df.select(dayofmonth(df['Date'])).show()                    ## Use functions like attributes in Pandas
new_df = df.withColumn("Year", year(df['Date']))                ## add new col 'Year'
df_yr = new_df.groupBy("Year").mean().select(['Year', 'avg(Close)'])  ## renames close -> avg(Close)
df_yr = df_yr.orderBy("Year", ascending=False)                  ## same syntax as Pandas .sort_values
df_yr = df_yr.withColumnRenamed("avg(Close)", "Avg_Close")      ## rename SQL-style col to normal
df_yr.select(["Year", format_number("Avg_Close", 2)]).show()    ## format -> round to 2 decimal places

## Output of last command:
+----+---------------------------+
|Year|format_number(Avg_Close, 2)|      ## Note horrible col name -- the is fix below
+----+---------------------------+
|2016|                     104.60|
|2015|                     120.04|
|2014|                     295.40|
|2013|                     472.63|
|2012|                     576.05|
|2011|                     364.00|
|2010|                     259.84|
+----+---------------------------+

## .alias method allows col renaming inline after a groupBy operation, like so.  
## Even after renaming via withColumnRenamed, the format_number method renamed col again
df_yr.select(["Year", format_number("Avg_Close", 2).alias("Avg_Close")])

## Output:
+----+---------+
|Year|Avg_Close|
+----+---------+
|2016|   104.60|
|2015|   120.04|
|2014|   295.40|
|2013|   472.63|
|2012|   576.05|
|2011|   364.00|
|2010|   259.84|
+----+---------+
```

#### Casting Columns to Datatypes
Apparently many operations on a Spark DF cause the datatype to be converted to a `string`.  So, a DF with correct numeric dtypes will be converted to all `string` upon using, say, `df.describe()`.  This creates problems for subsequent methods, obviously.

Use `.cast()` method to cast a column to a different data type.
```python
df['Level'].cast('float')
```

### Operating on Multiple Columns
I'm unsure of how to operate using a `.map()` or `.apply()` type method.  For now, I am using list comprehensions to act as a sort of mapping function.  So, using the `format_number` and `.cast()` methods from above, I will show an example of how to use list comps to greatly reduce the coding overhead for repeated operations. It leverages the `*` unpacking operator inside the `.select()` function.

In the FFT DF, there are a mix of numeric and `string` columns.  If I want to perform an operation on only the numeric columns, such as `format_number`, I have to first get them and then repeat the operation.  These numeric cols are assigned to the variable `numeric_cols` below.  

Pretend however that we've done some DF-level operation, such as `.describe()` that (apparently) re-types every column as a `string`, much to our chagrin.  The results of the `.describe()` operation are very long and unreadable numbers (of `string` type, mind you).  We want to fix this by using `format_number` to limit the values to two decimal places.  But since they're all `string` dtype, we can't use `format_number` on them until we first `.cast()` them into a numeric dtype, like `float`.

The base, brute force way to achieve this is to code every single desired column into the `.select()` statement, which is laborious, error-prone, and nigh unreadable.

##### Brute Force Way
```python
## After an operation that casts the numeric cols to string
df.select(format_number(dfd['Level'].cast('float'), 2).alias('Level'),
              format_number(dfd['HPm'].cast('float'), 2).alias('HPm'),
              format_number(dfd['MPm'].cast('float'), 2).alias('MPm'),
              format_number(dfd['PAm'].cast('float'), 2).alias('PAm'),
              format_number(dfd['MAm'].cast('float'), 2).alias('MAm'),
              format_number(dfd['SPm'].cast('float'), 2).alias('SPm'),
              format_number(dfd['Move'].cast('float'), 2).alias('Move'),
              format_number(dfd['Jump'].cast('float'), 2).alias('Jump'),
              format_number(dfd['CEV'].cast('float'), 2).alias('CEV'),
              format_number(dfd['HPc'].cast('float'), 2).alias('HPc'),
              format_number(dfd['MPc'].cast('float'), 2).alias('MPc'),
              format_number(dfd['PAc'].cast('float'), 2).alias('PAc'),
              format_number(dfd['MAc'].cast('float'), 2).alias('MAc'),
              format_number(dfd['SPc'].cast('float'), 2).alias('SPc'),
             ).show()
```

Ideally we could use a `.map()` method to apply the operations above repeatedly, but until I figure that out, here is the list comp approach.

##### Concise Way
```python
numeric_cols = ['Level', 'HPm', 'MPm', 'PAm', 'MAm', 'SPm', 'Move', 'Jump', 'CEV', 'HPc', 'MPc', 'PAc', 'MAc', 'SPc']
format_cols = [format_number(dfd[col].cast('float'), 2).alias(col) for col in numeric_cols]

## Note the unpacking '*' use, since .select() is a function. Otherwise we'd just pass in a list of cols and that wouldn't work.
df.select(*format_cols).show()
```

These two blocks of code achieve the same result!

Also, note the use of `.cast()` inside the `format_number` method, since we must convert to a numeric dtype before we can format it as such.  Recall the `.alias()` call at the end is simply renaming the now-changed column to its original name, else it would have a garish name like: `format_number(dfd['SPc'].cast('float'), 2)`, which is an affront to all literate humanity.


##### Best Ways to Find Max of a Column
https://intellipaat.com/community/4448/best-way-to-get-the-max-value-in-a-spark-dataframe-column


##### EDA in PySpark
Here are some commands and outputs for general EDA in PySpark:

```python
data.select(['Association']).distinct().show()
+--------------+
|   Association|
+--------------+
|        Lucavi|
|      Lion_War|
|Shrine_Knights|
|        Heroes|
|       Unknown|
|      Glabados|
|         Pawns|
|       Monster|
|       Generic|
+--------------+


data_raw.groupBy(['Association']).count().orderBy('count').show()
+--------------+-----+
|   Association|count|
+--------------+-----+
|         Pawns|    4|
|      Glabados|    5|
|        Lucavi|    8|
|      Lion_War|   11|
|       Unknown|   12|
|Shrine_Knights|   13|
|        Heroes|   19|
|       Generic|   23|
|       Monster|   52|
+--------------+-----+


## More comprehensive approach to formatting output and sorting
## GroupBy can change dtypes from numeric to string (joy...)
grp_cols = ['Association']
num_cols = [c for c in data_raw.columns if any(['m' in c.lower(), 'c' in c.lower()]) and c not in grp_cols]
dfg = data_raw.select(grp_cols + num_cols).groupBy(grp_cols).mean()

## Must format, cast to float (even if already numeric), and rename for every col you want to select with modified output
formatted_cols = [format_number(f"avg({c})", 1).cast('float').alias(f'avg_{c}') for c in num_cols]
dfg.select(grp_cols + formatted_cols).sort('avg_SPm', ascending=False).show()

+--------------+-------+-------+-------+-------+-------+--------+--------+-------+-------+-------+-------+-------+-------+
|   Association|avg_HPm|avg_MPm|avg_PAm|avg_MAm|avg_SPm|avg_Move|avg_Jump|avg_CEV|avg_HPc|avg_MPc|avg_PAc|avg_MAc|avg_SPc|
+--------------+-------+-------+-------+-------+-------+--------+--------+-------+-------+-------+-------+-------+-------+
|         Pawns|   72.5|   87.5|   75.0|  105.0|   80.0|     3.0|     2.2|    0.2|    8.2|    8.2|   37.5|   37.5|   75.0|
|       Unknown|   76.7|   87.1|   60.4|   95.8|   87.5|     3.2|     3.0|    0.1|    9.2|    9.4|   48.8|   40.2|   83.3|
|      Glabados|  112.0|  120.0|   98.0|  110.0|  100.0|     3.2|     3.0|    0.1|   10.8|   11.0|   50.0|   49.8|  104.0|
|       Generic|   89.3|   81.3|   91.6|   92.8|  101.3|     3.2|     3.2|    0.1|   11.7|   14.3|   54.0|   49.1|   97.8|
|      Lion_War|  127.7|  100.3|  106.8|   99.5|  107.3|     3.5|     3.0|    0.1|   10.5|   10.8|   49.8|   49.3|   99.5|
|        Heroes|  123.4|  102.8|  110.8|   99.0|  109.9|     3.5|     3.2|    0.1|    9.9|   16.8|   46.3|   48.7|   97.1|
|Shrine_Knights|  150.4|  109.6|  115.9|  101.9|  112.3|     3.7|     3.8|    0.2|    9.5|   10.5|   44.9|   48.1|  100.9|
|       Monster|  114.1|   80.2|  119.1|   99.9|  114.6|     3.9|     3.7|    0.1|    6.4|   28.3|   38.6|   12.0|   86.3|
|        Lucavi|   85.4|   99.8|  124.1|  127.5|  131.4|     5.1|     4.1|    0.1|   11.5|   10.4|   43.4|   46.9|   97.5|
+--------------+-------+-------+-------+-------+-------+--------+--------+-------+-------+-------+-------+-------+-------+

```

<BR>
<BR>

_______
## Machine Learning
_______

We use the new and expanding [MLlib library](http://spark.apache.org/docs/latest/ml-guide.html).  
In Spark, dataframe structure for ML is a bit different than in Pandas / R.  Primarily, Spark DFs only use one or two columns as follows:

1. Supervised - two columns: _labels_ and _features_
2. Unsupervised - one column: _labels_

This structure requires a bit more data prep on our side before using ML.  In particular, _features for supervised learning must be condensed into one column_, meaning each row is a vector of all the features that align with the given label.  For a mental image, in Excel this would mean taking an entire row (in a _tidy_ dataset) and putting it into a single vector (list) to pair with its corresponding label.  Spark / MLlib has some preprocessing methods that help achieve this, which we will cover below.

Here's an example of correctly structured data for a supervised learner:  

```python
+-------------------+--------------------+
|              label|            features|
+-------------------+--------------------+
| -9.490009878824548|(10,[0,1,2,3,4,5,.])|
| 0.2577820163584905|(10,[0,1,2,3,4,5,.])|
| -4.438869807456516|(10,[0,1,2,3,4,5,.])|
|-19.782762789614537|(10,[0,1,2,3,4,5,.])|
...
```

__Note__: the _label_ column must be numeric.  If using a classification algorithm, we must correctly encode the target classes to numbers.

<BR>


### Regression
_______

#### Generic Linear Regression Example with Already Tidy Data
Imports come from the `.ml` library and corresponding ML family, such as regression.

```python
from pyspark.ml.regression import LinearRegression
data = spark.read.format("libsvm").load("sample_linear_regression_data.txt")
lr = LinearRegression(featuresCol='features', labelCol='label', predictionCol='prediction')
mod = lr.fit(data)
```

The `libsvm` format is uncommon in Python, but Spark documentation uses them because of their formatting.  It is used to load our training data, here.  The three arguments for `LinearRegression` are the required arguments, shown here with their default names.  All the other typical regression parameters, such as `elasticNet` or `fitIntercept` would be passed here, as well. If you change the name of your data columns, you must pass in the correct name to the model constructor.

MLlib has a very nice `summary` attribute which contains all the results of the model fitting.  Using it, we can access any result or attribute of the model as such:

```python
mod_summary = mod.summary

mod_summary.coefficients
## output: [Ax1, Bx2, Cx3... Nxx]
mod_summary.intercept
## output: -2.54938
mod_summary.rootMeanSquaredError
## output: 10.17564
mod_summary.r2
## output: 0.875
## and so on...

## We can even quickly generate all the residuals for the estimate:
mod_summary.residuals.show()
+-------------------+
|          residuals|
+-------------------+
|-11.011130022096554|
| 0.9236590911176537|
|-4.5957401897776675|
|  -20.4201774575836|
|-10.339160314788181|
|-5.9552091439610555|
|-10.726906349283922|
```

Of course, for any ML work we need to have a training set and a test set.  We can use the `randomSplit([train_size, test_size])` command, where `train_size` and `test_size` are decimals between 0 and 1, respectively. (Commonly [0.7, 0.3] or [0.6, 0.4])

We can then access the respective attributes or preview the data:

```python
train_data, test_data = all_data.randomSplit([0.7, 0.3])

train_data.show()
+-------------------+--------------------+
|              label|            features|
+-------------------+--------------------+
|-26.805483428483072|(10,[0,1,2,3,4,5,.])|
|-22.837460416919342|(10,[0,1,2,3,4,5,.])|
|-21.432387764165806|(10,[0,1,2,3,4,5,.])|
| -19.66731861537172|(10,[0,1,2,3,4,5,.])|
|-17.494200356883344|(10,[0,1,2,3,4,5,.])|
```

For a quick summary, use `train_data.describe.show()`

Now that we've split our data, we want to fit our model to the training data only before we make predictions on the test data.  We can see how well our model predicted on the test data using `evaluate()`, as well as seeing all the other model and scoring attributes.  The key point to note is that `evaluate()` is used when we are predicting on _labeled_ data (i.e. known testing data).  We cannot use `evaluate()` on new, unlabeled data because we don't know the labels (answers) to compare the predictions to.

```python
correct_model = lr.fit(train_data)
test_results = correct_model.evaluate(test_data)

test_results.residuals.show()
+-------------------+
|          residuals|
+-------------------+
|-27.947989982759562|
| -21.48974651218455|
|-21.348863706083357|
| -19.11295339381348|
...

test_results.r2
## output: 0.647
```

Ultimately, we want to use this model to predict on new, unlabeled data.  For the sake of convenience, let's use some of the test data and remove the labels, pretending it is new, unlabeled data.  We can do this by grabbing the `features` column alone.

```python
unlabeled_data = test_data.select('features')
unlabeled_data.show()

+--------------------+
|            features|
+--------------------+
|(10,[0,1,2,3,4,5,.])|
|(10,[0,1,2,3,4,5,.])|
```

Once we have the "new" and unlabeled data, we can make predictions on it by using the `transform()` command.  This command is actually being used in the `evaluate()` command above.

```python
predictions = correct_model.transform(unlabeled_data)

predictions.show()
+--------------------+--------------------+
|            features|          prediction|
+--------------------+--------------------+
|(10,[0,1,2,3,4,5,.])|  1.1425065542764918|
|(10,[0,1,2,3,4,5,.])| -1.3477139047347926|
|(10,[0,1,2,3,4,5,.])|-0.08352405808245028|
|(10,[0,1,2,3,4,5,.])|  -0.554365221558238|
```

### Importing Modules and Functions
In PySpark many functions are found in the `sql.functions` module.  
By convention, if we want to import the entire functions library, we do so by calling it `F`

```python
import pyspark.sql.functions as F
```

<BR>
<BR>


### Linear Regression
_______

#### Example with FFT Data that Must Be Prepared for Spark First
##### Using `.head()` and `columns` to Preview Data
This FFT dataset, seen in examples earlier in this document, is tidy but still must be prepared for Spark to utilize in any machine learning.  Let's take a look at the data first.

`.head()` by default returns only the first row and does so in compressed _Row_ format.  

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("fft").getOrCreate()
data = spark.sql("SELECT * FROM fft_class_stats")
data.head()

## Output
Row(Job_ID='1', Job='Squire', Identity='Ramza1', Gender='Male', Level=1, Role='Story',
    Association='Heroes', HPm=125, MPm=105, PAm=111, MAm=102, SPm=107, Move=4, Jump=3,
    CEV=0.10000000149011612, HPc=11, MPc=11, PAc=50, MAc=48, SPc=95)
```

To see the data in a more readable per-line format, we just iterate through it with a for-loop.

```python
for item in data.head():
    print(item)

## Output
1
Squire
Ramza1
Male
1
Story
Heroes
125
105
111
102
107
4
3
0.10000000149011612
11
11
50
48
95
```

Adding an integer argument to the function call allows you to preview more rows. `data.head(3)` creates a 3-list with each row being an index in the list.  To access the third row, we'd index into it per normal with `data.head(3)[2]`.

Similarly, use the `.columns` attribute to see the headers.

```python
data.columns

## Output
['Job_ID',
 'Job',
 'Identity',
 'Gender',
 'Level',
 'Role',
 'Association',
 'HPm',
 'MPm',
 'PAm',
 'MAm',
 'SPm',
 'Move',
 'Jump',
 'CEV',
 'HPc',
 'MPc',
 'PAc',
 'MAc',
 'SPc']
```

#### Preparing Data for Spark ML -- Vectorizing the Features
Recall, all ML data must be in the form of two columns: ("label", "features")
To accomplish this, Spark has some convenient data preparation tools.   
According to the Udemy course I took, `Vectors` is omnipresent but not always required.  It is imported out of precaution rather than need much of the time.  `VectorAssembler` is the tool we will use most frequently to, you guessed it, assemble all features into a vector.

```python
## Import VectorAssembler and Vectors
from pyspark.ml.linalg import Vectors
from pyspark.ml.feature import VectorAssembler
```

The type of algorithm you're using will determine what type of data you are using in your feature set.  Since we are doing a standard linear regression, we are going to have a numerical target as well as numerical features.  I have a dummied version of the FFT dataset which contains all status effects for characters in dummied columns, but I'm skipping that for simplicity's sake in this instructional write up.

In order to see which columns are numeric (assuming the dataset is already cleaned up and data-typed appropriately), we can just use `printSchema()`.

```python
data.printSchema()

## Output
root
 |-- Job_ID: string (nullable = true)
 |-- Job: string (nullable = true)
 |-- Identity: string (nullable = true)
 |-- Gender: string (nullable = true)
 |-- Level: integer (nullable = true)
 |-- Role: string (nullable = true)
 |-- Association: string (nullable = true)
 |-- HPm: integer (nullable = true)
 |-- MPm: integer (nullable = true)
 |-- PAm: integer (nullable = true)
 |-- MAm: integer (nullable = true)
 |-- SPm: integer (nullable = true)
 |-- Move: integer (nullable = true)
 |-- Jump: integer (nullable = true)
 |-- CEV: float (nullable = true)
 |-- HPc: integer (nullable = true)
 |-- MPc: integer (nullable = true)
 |-- PAc: integer (nullable = true)
 |-- MAc: integer (nullable = true)
 |-- SPc: integer (nullable = true)
 ```

So, in this case every column starting with `HPm` is numeric, plus `Level`.  Those will be the columns we will want to work with for our regression.  Let's pick `Level` -- the level of the character -- as our target, the variable we are trying to predict.  For our features, we will use all the underlying core attributes of that character.  Since few will know the meaning of these stats, let me give a brief summary.


##### FFT Numeric Features

| Name | Meaning                     |
|------|-----------------------------|
|HP    | Hit Points                  |
|MP    | Magic Points                |
|PA    | Physical Attack Power       |        
|MA    | Magic Attack Power          |    
|SP    | Speed Rating                |
|Move  | Horizontal Movement Range   |            
|Jump  | Vertical Movement Range     |            
|CEV   | Class Evasion Rating        |        

The difference between the `m` and `c` suffixes is that `m` applies to in-battle ratings and `c` applies to growth (level up) ratings.  Important: a higher `m` is good while a lower `c` is good, as it means it takes less leveling up to improve in a statistic.  This is noteworthy because we would expect these values to correspond to higher level characters accordingly.


##### Preparing Features with `VectorAssembler`
Let's use all the numeric features above to create a `features` vector for our regression model.  First let's create the `assembler` object by telling it which fields we want to combine and what to call the resulting combined output column, `features` in this case.

```python
numeric_cols = [ 'HPm', 'MPm', 'PAm', 'MAm', 'SPm', 'Move', 'Jump', 'CEV', 'HPc', 'MPc', 'PAc', 'MAc', 'SPc']

assembler = VectorAssembler(
    inputCols=numeric_cols,
    outputCol="features")
```

Now that we've prepared the `VectorAssembler` object, we need to feed it the data for it to combine into a single vector.

Three notes:
1. We pass the whole dataset into the assembler.  Since we specified which columns to use in the `assembler` instantiation, it will use only those fields in the new `features` vector even if we provide it a dataset with other columns.  The _type_ of object a `VectorAssembler` returns is called a `DenseVector`.
2. The returned object is the `data` we pass in with the new `features` column appended to the end.  The returned object is _not_ simply the features vector!
2. We do this __before__ we do any train/test data split.

```python
prepped_data = assembler.transform(data)

## 'prepped_data' is now just the 'data' DF with 'features' appended to end.
prepped_data.head()

## Output
Row(Job_ID='1', Job='Squire', Identity='Ramza1', Gender='Male', Level=1, Role='Story',
    Association='Heroes', HPm=125, MPm=105, PAm=111, MAm=102, SPm=107, Move=4, Jump=3,
    CEV=0.10000000149011612, HPc=11, MPc=11, PAc=50, MAc=48, SPc=95,
    features=DenseVector([125.0, 105.0, 111.0, 102.0, 107.0, 4.0, 3.0, 0.1, 11.0, 11.0, 50.0, 48.0, 95.0]))
```

<BR>


#### Linear Regression Model
Spark documentation can be found [here.](http://spark.apache.org/docs/latest/ml-classification-regression.html#linear-regression)

Now that our data is formatted correctly for Spark to use in an algorithm, let's create our model using our prepared data and splitting it.

```python
from pyspark.ml.regression import LinearRegression

## Remember, Spark wants only two columns for its algorithms, which is why we created 'features'
model_data = prepped_data.select('Level', 'features')
model_data.describe().show()

## Output
+-------+------------------+
|summary|             Level|
+-------+------------------+
|  count|               147|
|   mean|11.006802721088436|
| stddev| 17.85078817132528|
|    min|                 0|
|    max|                75|
+-------+------------------+
```

Now let's split our data into train and test sets.

```python
train_data, test_data = model_data.randomSplit([0.7, 0.3])
```

Finally let's instantiate our model, making sure to supply the correct `labelCol` and `featuresCol`.

```python
lr = LinearRegression(featuresCol='features', labelCol='Level', predictionCol='pred')
mod = lr.fit(train_data)

test_results = mod.evaluate(test_data)
test_results.residuals.show()

## output
+-------------------+
|          residuals|
+-------------------+
| -4.426305106898241|
|-7.3172695260879514|
|  -9.69513494784088|
|-11.188580542954487|
| -9.281478815939384|
| -9.281478815939384|
...

test_results.rootMeanSquaredError
## 14.411891704276602

test_results.meanAbsoluteError
## 9.839299016890333

test_results.r2
## 0.23842515716539825
```

Well, it turns out that our data is _not_ very predictable using only the numeric features.  With a mean `Level` around 11 for the full `model_data` dataset, our two error metrics are around 10 and 14.  That's pretty bad.  Further, the standard deviation of the `model_data` is _larger_ than the mean of the data, by about 1.5x.  That tells us without even building a model that the data is likely to be highly challenging to predict.  Finally, our `r2` value helps confirm that our model is not doing a good job of explaining the variance observed in this data.

This is instructive in two ways.
1. This is 'artificial' data in the sense that it is created for a video game and the underlying features don't have to have any real relationship to the target variable.  This model goes a long way to say that, no, they don't particularly.  In the real world, there is likely to be some degree of relationship for any sensible data.

2. Using all three metrics above, plus perhaps plotting the residuals, will help understanding the results of the model.  In this case all three (or four) reviews would lead to the same conclusion: there isn't much of a relationship between the variables we included in the `features` column to the `Level` of a character.  There's _some_, but not much -- at least not in a reliable manner.


If we had new, unlabeled data (say we were given new core stats for an FFT character whose Level had not been revealed and we were asked to predict the Level), we would not be able to use `.evaluate()`, since we wouldn't have the labels (`Level`) to compare to.  Instead we'd have to use `.transform()`

```python
new_prediction = mod.transform(new_character_features)

## Since we don't have labels (answers), we can't use any metrics such as RMSE, MAE, or R2.
## We can only look at the predictions anew
new_prediction.show()
...
...
```




#### Utilizing Text Column for Numerical Modeling - One Hot Encoding
Given the poor results from using numeric-only cols to predict the Level, let's encode the `Association` col with One Hot Encoding.  This is not the same as creating `dummy` variables, which creates a column for each dummied variable containing only binary values, 0 and 1, where 1 means that the given row is true for the column and 0 means that it is not.  Dummying variables is the proper approach, but One Hot Encoding will suffice for now as it allows us to at least utilize the text / string data in our model.  This really only works well for discrete categorical columns.

The process contains two steps.
1. Creating an integer index of the categorical values in the column.
2. Encoding the categorical index into a useable column for the model.

```python
from pyspark.ml.feature import StringIndexer, OneHotEncoder
assoc_indexer = StringIndexer(inputCol='Association', outputCol='Association_int')
# data_indexed = indexer.fit(data_raw).transform(data_raw)
assoc_encoder = OneHotEncoder(inputCol='Association_int', outputCol='Association_encode')
assembler2 = VectorAssembler(
    inputCols=['Association_encode', 'HPm', 'MPm', 'PAm', 'MAm', 'SPm', 'Move', 'Jump', 'CEV', 'HPc', 'MPc', 'PAc', 'MAc', 'SPc'],
    outputCol="features")

## indexer creates an integer index for each category in the string column
## but the fit and transform appends the Indexer col to the full original dataset
data2 = assoc_indexer.fit(data).transform(data)
data2.select(['Association', 'Association_int']).show()
+-----------+---------------+
|Association|Association_int|
+-----------+---------------+
|     Heroes|            2.0|
|     Heroes|            2.0|
|   Lion_War|            5.0|
|   Lion_War|            5.0|
|      Pawns|            8.0|
|     Heroes|            2.0|
|   Glabados|            7.0|
|     Heroes|            2.0|
|   Glabados|            7.0|
|   Lion_War|            5.0|
|    Unknown|            4.0|
|      Pawns|            8.0|
...


## The encoder creates an encoded col of the features for use in the model
data2 = assoc_encoder.transform(data2)
data2.select(['Association', 'Association_int', 'Association_encode']).show()
+-----------+---------------+------------------+
|Association|Association_int|Association_encode|
+-----------+---------------+------------------+
|     Heroes|            2.0|     (8,[2],[1.0])|
|     Heroes|            2.0|     (8,[2],[1.0])|
|   Lion_War|            5.0|     (8,[5],[1.0])|
|   Lion_War|            5.0|     (8,[5],[1.0])|
|      Pawns|            8.0|         (8,[],[])|
|     Heroes|            2.0|     (8,[2],[1.0])|
|   Glabados|            7.0|     (8,[7],[1.0])|
|     Heroes|            2.0|     (8,[2],[1.0])|
|   Glabados|            7.0|     (8,[7],[1.0])|
|   Lion_War|            5.0|     (8,[5],[1.0])|
|    Unknown|            4.0|     (8,[4],[1.0])|
|      Pawns|            8.0|         (8,[],[])|
...

data2 = assembler2.transform(data2)
lr = LinearRegression(featuresCol='features', labelCol='Level', predictionCol='pred')
train_data2, test_data2 = data2.randomSplit([.7, .3])



mod2 = lr.fit(data2)
eval2 = mod2.evaluate(data2)
# eval2.predictions.select(['Identity', 'Level', 'Association', 'Association_int', 'Association_encode', 'features', 'pred']).show()
# print(eval2.residuals.show())
print('RMSE:', eval2.rootMeanSquaredError)
print('MAE:', eval2.meanAbsoluteError)
print('R2:', eval2.r2)
# for _ in data2.select('features').collect():
#   print(_)

RMSE: 8.812253598166008
MAE: 6.118139005315722
R2: 0.754628758882372
```

So judging by the result above, it appears that OneHotEncoding the Association column does help, but depends on the test data







<BR>
<BR>


### Logistic Regression (Classification)
_______

Let's look at how to do the most basic form of classification in PySpark.

First, let's make the `Association` column the classification target and all numeric columns (including `Level`) the features.

Note, in PySpark, we can directly access the evaluation metrics for classification by importing the `MulticlassClassificationEvaluator` and not the `BinaryClassificationEvaluator`, _even if we only actually have two classes!_  The four most common evaluation metrics are

+ `accuracy`
+ `weightedPrecision`
+ `weightedRecall`
+ `f1` (Harmonic mean of Precision and Recall)

```python
from pyspark.ml.evaluation import MulticlassClassificationEvaluator
acc_eval = MulticlassClassificationEvaluator(metricName='accuracy')
acc_eval.evaluate(mod_preds)
```

Here's the front matter for getting the Logistic Regressor instantiated.

```python
from pyspark.sql import SparkSession
from pyspark.ml.classification import LogisticRegression
from pyspark.ml.linalg import Vectors
from pyspark.ml.feature import VectorAssembler, StringIndexer
from pyspark.ml.evaluation import MulticlassClassificationEvaluator
from pyspark.sql.functions import format_number
spark = SparkSession.builder.appName('logit').getOrCreate()
data_raw = spark.sql("SELECT * FROM fft_class_stats")
```


#### Prepare Target (multi-class)
Our target is a text column, but is categorical in nature. So we must convert the `string` column to numeric (`int`).  We did this in the Linear Regression section to use the `Association` column as a _feature_, whereas now we are going to use it as the _target_.  I believe there are specific functions (shortcuts) when the target class in binary instead of multi-class, but I don't know them yet and this target is multi-class.

```python
indexer = StringIndexer(inputCol='Association', outputCol='Association_int')
data = indexer.fit(data_raw).transform(data_raw)

## Output -- we see the newly created Association_int col appended as expected
print(data.show())
+------+--------------+----------+------+-----+-----+-----------+---+---+---+---+---+----+----+----+---+---+---+---+---+---------------+
|Job_ID|           Job|  Identity|Gender|Level| Role|Association|HPm|MPm|PAm|MAm|SPm|Move|Jump| CEV|HPc|MPc|PAc|MAc|SPc|Association_int|
+------+--------------+----------+------+-----+-----+-----------+---+---+---+---+---+----+----+----+---+---+---+---+---+---------------+
|     1|        Squire|    Ramza1|  Male|    1|Story|     Heroes|125|105|111|102|107|   4|   3| 0.1| 11| 11| 50| 48| 95|            2.0|
|     2|        Squire|    Ramza2|  Male|   10|Story|     Heroes|125|105|111|102|107|   4|   3| 0.1| 11| 11| 50| 48| 95|            2.0|
|     3|        Squire|    Ramza3|  Male|   26|Story|     Heroes|125|105|111|102|107|   4|   3| 0.1| 11| 11| 50| 48| 95|            2.0|


## Check the counts of each category -- same as groupby by Association, just that we are now seeing the integer index
data.groupBy(['Association_int']).count().orderBy('Association_int').show()
+---------------+-----+
|Association_int|count|
+---------------+-----+
|            0.0|   52|
|            1.0|   23|
|            2.0|   19|
|            3.0|   13|
|            4.0|   12|
|            5.0|   11|
|            6.0|    8|
|            7.0|    5|
|            8.0|    4|
+---------------+-----+
```

## Prepare Features
We want to grab all the numeric columns to use as features.  If there were categorical columns that we wanted to also use as features, we'd either dummy them (best) or One Hot Encode them (easier) to utilize them as a feature, as we did in the Linear Regression model above.

```python
numeric_cols = [ 'HPm', 'MPm', 'PAm', 'MAm', 'SPm', 'Move', 'Jump', 'CEV', 'HPc', 'MPc', 'PAc', 'MAc', 'SPc']

assembler = VectorAssembler(
    inputCols=numeric_cols,
    outputCol="features")

prepped_data = assembler.transform(data)
```

Remember that Spark models want data in a `[target, features]` two-column format.  Our target for this classification is the categorical column `Association_int` (which we have indexed above).

```python
model_data = prepped_data.select(['Association_int', 'features'])  ## Using the Int version of classes
model_data.show()

+---------------+--------------------+
|Association_int|            features|
+---------------+--------------------+
|            2.0|[125.0,105.0,111...]|
|            2.0|[125.0,105.0,111...]|
|            2.0|[125.0,105.0,111...]|
|            5.0|[130.0,100.0,120...]|
...
```

Now let's build the model and explicitly name the label (target) and feature columns.

```python
logit = LogisticRegression(labelCol='Association_int', featuresCol='features')
```

One thing I like to do is to do a full-fit of the data to see what the ceiling for the model can be.  In this scenario, we give the model _all_ the data instead of doing a train-test split and training on the train data before predicting on the test data.  Here, we fit on the entire dataset and check the results.  These results likely represent a ceiling for the model as currently constructed, since fitting on a training split of data means we are feeding the model less data than the full-fit, so its prediction accuracy will likely diminish or, at best, stay close to the same.

```python
## Full fit for ceiling estimate
mod = logit.fit(model_data)
mod_sum = mod.summary
mod_sum.predictions.show()
## Note this summary output is the same as if we did mod.evaluate(model_data) -- aka on the entire dataset w/o splitting

+---------------+--------------------+--------------------+--------------------+----------+
|Association_int|            features|       rawPrediction|         probability|prediction|
+---------------+--------------------+--------------------+--------------------+----------+
|            2.0|[125.0,105.0,111...]|[-0.9537968456507..]|[0.00425160611744..]|       2.0|     ## Correct prediction
|            2.0|[125.0,105.0,111...]|[-0.9537968456507..]|[0.00425160611744..]|       2.0|     ## Correct prediction
|            2.0|[125.0,105.0,111...]|[-0.9537968456507..]|[0.00425160611744..]|       2.0|     ## Correct prediction
|            5.0|[130.0,100.0,120...]|[-1.2522147530711..]|[0.00282686258202..]|       5.0|     ## Correct prediction
|            5.0|[135.0,100.0,120...]|[-0.7310342260310..]|[0.00401601282345..]|       5.0|     ## Correct prediction
|            5.0|[150.0,100.0,120...]|[-0.0122283696095..]|[0.00278013043173..]|       5.0|     ## Correct prediction
|            5.0|[120.0,100.0,110...]|[-0.8416667596935..]|[0.01182437937391..]|       1.0|     ## Incorrect prediction
|            2.0|[168.0,80.0,120.0..]|[-0.4800775753928..]|[7.03757732159166..]|       5.0|     ## Incorrect prediction
...
```

##### Evaluate Model

```python
for metric in ['f1', 'weightedPrecision', 'weightedRecall', 'accuracy']:
    mc_eval = MulticlassClassificationEvaluator(labelCol='Association_int', predictionCol='prediction', metricName=metric)
    my_eval = mc_eval.evaluate(mod_sum.predictions)
    print(f"{metric}: {my_eval:.3f}")

## Output
f1: 0.783
weightedPrecision: 0.789
weightedRecall: 0.789
accuracy: 0.789
```

So, we see that for the given data and the features we've given the model (note we have done zero feature engineering), we can expect around a prediction to be right about 78% of the time in the best case scenario.  If we get something above this, it should raise a flag that something might be incorrect with our model.  And while we expect results a bit below this for a properly split-and-trained model, if we get something _far_ below it should also raise a flag that something might be wrong, such as improperly balanced classes during the train/test split, or possible poor sampling (splitting) of outlier records which cause the model to over-or-under train for instances which do (or do not) appear in the test set.

Of course, this tells us nothing about the generalizability of our model and doesn't allow us to tune it validly.  So we should do the familiar train/test split to make our model's predictions justifiably testable.

```python
  train_data, test_data = model_data.randomSplit([.7, .3])
  # print(train_data.describe().show())
  # print(test_data.describe().show())
  # print(train_data.groupBy(['Association_int']).count().orderBy('Association_int').show())    ## use these to review for an equal breakdown of classes in both train and test
  # print(test_data.groupBy(['Association_int']).count().orderBy('Association_int').show())     ## use these to review for an equal breakdown of classes in both train and test
  mod_train = logit.fit(train_data)
  preds_test = mod_train.evaluate(test_data)
  preds_test.predictions.show()

+---------------+--------------------+--------------------+--------------------+----------+
|Association_int|            features|       rawPrediction|         probability|prediction|
+---------------+--------------------+--------------------+--------------------+----------+
|            0.0|[75.0,95.0,140.0,..]|[40.5871319541998..]|[0.99999999998396..]|       0.0|
|            0.0|[77.0,140.0,126.0..]|[44.2923161555047..]|[0.99999999999999..]|       0.0|
|            0.0|[82.0,96.0,93.0,1..]|[36.0502246921500..]|[0.99999999999085..]|       0.0|
...
```

And now we can similarly evaluate the prediction results of the properly trained model.

```python
for metric in ['f1', 'weightedPrecision', 'weightedRecall', 'accuracy']:
    mc_eval = MulticlassClassificationEvaluator(labelCol='Association_int', predictionCol='prediction', metricName=metric)
    my_eval = mc_eval.evaluate(preds_test.predictions)
    print(f"{metric}: {my_eval:.3f}")

## output
f1: 0.659
weightedPrecision: 0.693
weightedRecall: 0.643
accuracy: 0.643
```

Well, that's pretty much exactly what we expected -- a diminished predictive ability due to a smaller corpus of data the model got to learn from.  Depending on context, the `f1` metric is probably the most instructive single-value metric and it lost about 12% by going to the train/test split.  This is a sizable loss but not outside the realm of expectation.  




<BR>
<BR>


### Tree Models
_______

We will use the same classification case we used above in Logistic Regression and see if we get any different results using the tree models.  

There will be two tree models used:
+ Random Forest - `RandomForestClassifier`
+ Gradient Boosted Tree - `GBTClassifier`

One note: Gradient Boosted Trees (`GBTClassifier`) is only built for binary classification, currently.

##### Import Objects and Initiate Session
```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("fft").getOrCreate()
from pyspark.ml.classification import RandomForestClassifier, GBTClassifier
from pyspark.ml.evaluation import MulticlassClassificationEvaluator
from pyspark.ml.feature import StringIndexer, VectorAssembler
data_raw = spark.sql("SELECT * FROM fft_class_stats")
```

##### Prepare Data
Our target col (`Association`) is a categorical string, must convert to numeric.

```python
indexer = StringIndexer(inputCol='Association', outputCol='Association_int')
data = indexer.fit(data_raw).transform(data_raw)

numeric_cols = [ 'HPm', 'MPm', 'PAm', 'MAm', 'SPm', 'Move', 'Jump', 'CEV', 'HPc', 'MPc', 'PAc', 'MAc', 'SPc']
assembler = VectorAssembler(inputCols=numeric_cols, outputCol="features")
prepped_data = assembler.transform(data)

model_data = prepped_data.select(['Association_int', 'features'])

train_data, test_data = model_data.randomSplit([.7, .3])
```

##### Instantiate Models
```python
rfc = RandomForestClassifier(numTrees=400, labelCol='Association_int', featuresCol='features')
gbc = GBTClassifier(labelCol='Association_int', featuresCol='features')
rfc_mod = rfc.fit(train_data)
rfc_preds_train = rfc_mod.transform(train_data)
rfc_preds_train.show()
# gbc.fit(train_data) ## Currently only supports Binary Classification

+---------------+--------------------+--------------------+--------------------+----------+
|Association_int|            features|       rawPrediction|         probability|prediction|
+---------------+--------------------+--------------------+--------------------+----------+
|            0.0|[69.0,1.0,70.0,11..]|[383.598039974532..]|[0.95899509993633..]|       0.0|
|            0.0|[75.0,95.0,140.0,..]|[394.631039702990..]|[0.98657759925747..]|       0.0|
|            0.0|[77.0,1.0,160.0,1..]|[384.534443483304..]|[0.96133610870826..]|       0.0|
|            0.0|[77.0,140.0,126.0..]|[393.787342223999..]|[0.98446835555999..]|       0.0|
```

#### Model Evaluation
##### Train Data Evaluation
This is the review of the training data, so this represents a ceiling for prediction.

```python
for metric in ['f1', 'weightedPrecision', 'weightedRecall', 'accuracy']:
    mc_eval = MulticlassClassificationEvaluator(labelCol='Association_int', predictionCol='prediction', metricName=metric)
    my_eval = mc_eval.evaluate(rfc_preds_train)
    print(f"{metric}: {my_eval:.3f}")

## Output
f1: 0.917
weightedPrecision: 0.950
weightedRecall: 0.919
accuracy: 0.919
```

An F1 score of 91.7% is quite strong and is likely unrealistic for test / new data sets.

##### Feature Importances
Tree models allow the use of _feature importances_ to see which features had the biggest impact on prediction.  We can only see the feature importances for the training data because we have to know the correct prediction answers to evaluate which features were most important.

This code works because the feature order is preserved when we created the feature vector with `VectorAssembler`.  In the `VectorAssembler` creation, we passed in a list of numeric features.  That list is, of course, the list of columns used to make predictions in our model, and its order is maintained in the output of the model's `.featureImportances` attribute.  This allows us to pair the feature names with their importances, as shown below.

```python
feat_imps = {k:v for k,v in zip(numeric_cols, rfc_mod.featureImportances)}
for feat, imp in sorted(feat_imps.items(), key=lambda itm: itm[1], reverse=True):
  print(f"{feat}: {100 * imp:.1f}%")

## Output
MAc: 16.8%
MPc: 14.6%
SPc: 9.8%
MAm: 9.6%
HPc: 9.5%
HPm: 7.7%
SPm: 6.7%
MPm: 6.1%
PAc: 6.1%
CEV: 5.9%
PAm: 3.4%
Move: 2.2%
Jump: 1.5%
```

So, the model says that, in general, growth stats (shown by the `c` suffix, representing stat growth per level up) are more important than the multiplier stats (`m` suffix, acute stat multipliers for a single battle).  Specifically, it also says that Magic stats (both Magic Attack, `MAc`, and Magic Points, `MPc`) are more predictive than Physical stats.  The two movement stats are the two least important stats.


Now let's test the model on 'new', unseen data.

##### Test Data Evaluation
```python
rfc_preds_test = rfc_mod.transform(test_data)
# rfc_preds_test.show()

for metric in ['f1', 'weightedPrecision', 'weightedRecall', 'accuracy']:
    mc_eval = MulticlassClassificationEvaluator(labelCol='Association_int', predictionCol='prediction', metricName=metric)
    my_eval = mc_eval.evaluate(rfc_preds_test)
    print(f"{metric}: {my_eval:.3f}")

## Output
f1: 0.600
weightedPrecision: 0.595
weightedRecall: 0.646
accuracy: 0.646
```


<BR>
<BR>


### Clustering (K-Means)
_______

##### Imports
```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("fft").getOrCreate()
from pyspark.ml.clustering import KMeans
from pyspark.ml.feature import StringIndexer, VectorAssembler, StandardScaler
data = spark.sql("SELECT * FROM fft_class_stats")
```


##### Data Prep
For clustering it is advisable to scale the data.

```python
## Even though we can convert Association from string category to numeric category, we don't need it for clustering because it already IS categorical
# indexer = StringIndexer(inputCol='Association', outputCol='Association_int')
# data = indexer.fit(data_raw).transform(data_raw)

## May cluster more cleanly without 'Level' included -- have to test it out to see
numeric_cols = ['Level', 'HPm', 'MPm', 'PAm', 'MAm', 'SPm', 'Move', 'Jump', 'CEV', 'HPc', 'MPc', 'PAc', 'MAc', 'SPc']
assembler = VectorAssembler(inputCols=numeric_cols, outputCol="features")
prepped_data = assembler.transform(data)

## There is no target column b/c clustering is unsupervised learning
model_data = prepped_data.select(['features'])

## Should scale data for clustering models
scaler = StandardScaler(inputCol='features', outputCol='scaled_features')
scaled_data = scaler.fit(model_data).transform(model_data)
```

##### Finding the Optimal Value for `k`
Find optimal k up to K=n by using the lowest "Within-Set Sum of Squared Errors" (WSSSE). Note that since we've scaled the data, the raw value of WSSSE is meaningless with respect to the original features' scales. As k increases, WSSSE usually decreases becase more groups = smaller count per group, so deviation within each group decreases.

As such, best k is determined by the user, usually visually, where the biggest drop in WSSSE occurs (called the "Elbow" method).  I've done this programmatically by finding the biggest pct drop, though this can be imperfect too.

```python
print("Finding best value of k...")
k_dict = {}

for k_val in range(2, 13):
  kmeans = KMeans(featuresCol='scaled_features', k=k_val)
  kmod = kmeans.fit(scaled_data)
  wssse = kmod.computeCost(scaled_data)  
  pct_chg = 100 * ((wssse - k_dict[k_val-1][0]) / k_dict[k_val-1][0]) if k_val > 2 else 0
  k_dict[k_val] = [wssse, pct_chg]

  print(f"{k_val}: {wssse:.1f} - pct_chg: {pct_chg:.1f}")

best_k = sorted(k_dict.items(), key=lambda itm: itm[1][1], reverse=False)[0][0]
print(f"Best k = {best_k}")

## Output
Finding best value of k...
2: 1603.0 - pct_chg: 0.0
3: 1228.2 - pct_chg: -23.4
4: 1340.8 - pct_chg: 9.2
5: 1258.3 - pct_chg: -6.2
6: 904.8 - pct_chg: -28.1
7: 867.7 - pct_chg: -4.1
8: 819.9 - pct_chg: -5.5
9: 740.8 - pct_chg: -9.6
10: 666.8 - pct_chg: -10.0
11: 597.3 - pct_chg: -10.4
12: 535.3 - pct_chg: -10.4
Best k = 6
```

Here's a quick and dirty plot to confirm the values of K via the "Elbow Method."
```python
## Plot WSSSE values to find the "Elbow" drop
import matplotlib.pyplot as plt
plt.scatter(x=list(k_dict.keys()), y=[v[0] for v in k_dict.values()])
plt.grid(True)
plt.show()
```

![Elbow Method for K-Means: 6 has the biggest drop](/images/k_elbow.png)
<!-- ![Elbow Method for K-Means: 6 has the biggest drop](/Users/jpw/Dropbox/Data_Science/jp_notes/images/k_elbow.png) -->


Now that we have the 'best' value for `k`, let's rebuild the model with it.

##### Rebuild model using best k
```python
kmeans = KMeans(featuresCol='scaled_features', k=best_k)
kmod = kmeans.fit(scaled_data)
```

##### Get Centers (Centroids)
The number of arrays in the Centers list should = k
```python
centers = kmod.clusterCenters()
print(centers)

## Output
[array([0.14191717, 3.44369018, 1.18053693, 3.73757024, 2.89410422,
       4.97335366, 3.40029708, 3.74752896, 1.18759368, 3.11141016,
       2.03485015, 3.60898459, 2.41247493, 6.14342917]),
array([1.28845851, 3.87899582, 2.60368301, 3.49917083, 4.25723373,
       5.26666101, 3.88753477, 3.48245984, 1.64561379, 3.1699072 ,
       ...])]
```


##### Look at cluster labels
This is where it gets a bit messy.  Since the Cluster dataframe is separate from the original dataframe, we have to append the `Cluster` column to the original dataframe.  The easiest way is to `join` the two dataframes.  But since there is no mutually shared unique ID column, we have to create one.

There is a caveat to this approach, however, as noted [here](https://stackoverflow.com/questions/47028442/add-column-from-one-dataframe-to-another-dataframe-in-scala/47028782).

> Note that this will not always work (although it will for small dataframes). monotonically_increasing_id only guarantee that the numbers are increasing, it does not guarantee which numbers are used. Hence, the numbers given to the two dataframes can be different.

Our FFT dataframe is small (147 rows), so it will work (I verified it).  That said, this could be a very challenging problem for larger DFs.

```python
from pyspark.sql.functions import monotonically_increasing_id
clusters = kmod.transform(scaled_data)

## Create shared unique ID col
data2 = data.withColumn("join_col", monotonically_increasing_id())
cluster2 = clusters.withColumn("join_col", monotonically_increasing_id())

df_full = data2.join(cluster2.select(['join_col', 'prediction']), on='join_col', how='outer')
df_full = df_full.withColumnRenamed('prediction', 'Cluster')
df_full.select(['join_col', 'Identity', 'Job', 'Association', 'Cluster']).sort('join_col').show()

## Output
+--------+------+-------------+---------+------+-----+-------+-----------+---+---+---+---+---+----+----+----+---+---+---+---+---+-------+
|join_col|Job_ID|          Job| Identity|Gender|Level|   Role|Association|HPm|MPm|PAm|MAm|SPm|Move|Jump| CEV|HPc|MPc|PAc|MAc|SPc|Cluster|
+--------+------+-------------+---------+------+-----+-------+-----------+---+---+---+---+---+----+----+----+---+---+---+---+---+-------+
|       9|    0A|         Duke|     Larg|  Male|    0|  Story|   Lion_War|100|100|100|100|100|   3|   3| 0.1| 11| 11| 50| 50|100|      4|
|      10|    0B|         Duke|  Goltana|  Male|    0|  Story|   Lion_War|100|100|100|100|100|   3|   3| 0.1| 11| 11| 50| 50|100|      4|
```


##### Investigate Clusters In Detail
Look at every row in a given cluster.

```python
for clust in range(best_k):
  df_full.filter(df_full['Cluster'] == clust).show()

##Output
+--------+------+--------------+---------+------+-----+-----+--------------+---+---+---+---+---+----+----+----+---+---+---+---+---+-------+
|join_col|Job_ID|           Job| Identity|Gender|Level| Role|   Association|HPm|MPm|PAm|MAm|SPm|Move|Jump| CEV|HPc|MPc|PAc|MAc|SPc|Cluster|
+--------+------+--------------+---------+------+-----+-----+--------------+---+---+---+---+---+----+----+----+---+---+---+---+---+-------+
|       0|     1|        Squire|   Ramza1|  Male|    1|Story|        Heroes|125|105|111|102|107|   4|   3| 0.1| 11| 11| 50| 48| 95|      1|
|       1|     2|        Squire|   Ramza2|  Male|   10|Story|        Heroes|125|105|111|102|107|   4|   3| 0.1| 11| 11| 50| 48| 95|      1|
```


##### Investigate Cluster Averages
Note the required GroupBy cleanup code and the added Count column.

```python
grp_cols = ['Cluster']
num_cols = ['Level', 'HPm', 'MPm', 'PAm', 'MAm', 'SPm', 'Move', 'Jump', 'CEV', 'HPc', 'MPc', 'PAc', 'MAc', 'SPc']
dfg = df_full.select(grp_cols + num_cols).groupBy(grp_cols).mean()
dfc = df_full.select(grp_cols).groupBy(grp_cols).count()
formatted_cols = [F.format_number(f"avg({c})", 1).cast('float').alias(f'avg_{c}') for c in num_cols]
dfg = dfg.join(dfc, on='Cluster', how='outer')
dfg.select(grp_cols + ['count'] + formatted_cols).sort('avg_SPm', ascending=False).show()


+-------+-----+---------+-------+-------+-------+-------+-------+--------+--------+-------+-------+-------+-------+-------+-------+
|Cluster|count|avg_Level|avg_HPm|avg_MPm|avg_PAm|avg_MAm|avg_SPm|avg_Move|avg_Jump|avg_CEV|avg_HPc|avg_MPc|avg_PAc|avg_MAc|avg_SPc|
+-------+-----+---------+-------+-------+-------+-------+-------+--------+--------+-------+-------+-------+-------+-------+-------+
|      2|   19|     23.7|   95.5|   93.3|  125.3|  111.7|  122.6|     5.2|     5.2|    0.2|    8.6|   19.4|   40.3|   27.0|   90.8|
|      5|   38|      2.0|  114.5|   76.8|  120.6|   99.1|  117.4|     3.6|     3.3|    0.1|    6.2|   30.0|   38.5|    9.7|   86.3|
|      1|   41|     23.0|  135.2|  114.0|  111.1|  108.8|  109.8|     3.7|     3.1|    0.1|    9.8|   10.7|   46.1|   47.8|   99.1|
|      0|   15|      2.5|  120.1|   51.7|  118.7|   73.9|  103.7|     3.2|     3.3|    0.1|    9.6|   23.6|   42.0|   48.7|   94.5|
|      4|   31|      3.5|   85.0|  102.4|   74.0|  106.9|   99.7|     3.1|     3.0|    0.1|   12.1|   11.6|   59.5|   49.5|  100.0|
|      3|    3|      0.0|    0.0|    0.0|    0.0|    0.0|    0.0|     2.7|     2.0|    0.1|    0.0|    0.0|    0.0|    0.0|    0.0|
+-------+-----+---------+-------+-------+-------+-------+-------+--------+--------+-------+-------+-------+-------+-------+-------+
```




### Recommender Systems
This section will be briefer and will use the course-provided movies dataset.  
The two most common types of recommender systems are:

+ Content-based
+ Collaborative Filtering (CF)

__Content-based__ systems focus on the attributes of the items themselves.  If you like these three songs that all have a funky bass line and a horn section, we think you will probably like this new song that has those same qualities.

__Collaborative Filtering__ systems focus on the preferences of the users and employ an approach known as "widom of the crowds," where a large population of users rate items (films, shoes, furniture, etc....) and their ratings are used as guides to make recommendations to other users.  Further data slicing can be done so that users are clustered, such as "one sub-population of people like these five action films, and also like this other, more obscure action film.  You also like those five action films, so it's likely you will also like the more obscure film."

In general Collaborative Filtering tends to be used more initially due to ease of use (it's harder to quantify different features of products, like a film, which is what Content-based systems require).  It also tends to be equal or better in recommendations.  It also has the advantage of being able to learn which features are most important on its own as new recommendations are made and their results are tracked with time.  `spark.ml` uses Alternating Least Squares (ALS) to learn about these latent factors.  Mathematically, we are taking a sparse, high-dimensional matrix of users v. item ratings and, via identification of the latent factors, decomposing it to smaller user factors and item factors.

Ultimately, most real-world recommender systems use a combination of both types, serving to reinforce any recommendation.

Spark MLlib library for Machine Learning provides a Collaborative Filtering implementation by using Alternating Least Squares. The implementation in MLlib has these parameters:

+ numBlocks is the number of blocks used to parallelize computation (set to -1 to auto-configure).
+ rank is the number of latent factors in the model.
+ iterations is the number of iterations to run.
+ lambda specifies the regularization parameter in ALS.
+ implicitPrefs specifies whether to use the explicit feedback ALS variant or one adapted for implicit feedback data.
+ alpha is a parameter applicable to the implicit feedback variant of ALS that governs the baseline confidence in preference observations.

Here's some boilerplate code to get started.

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName('rec').getOrCreate()
from pyspark.ml.evaluation import RegressionEvaluator
from pyspark.ml.recommendation import ALS
data = spark.read.csv('movielens_ratings.csv', inferSchema=True, header=True)

data.describe().show()

## output
+-------+------------------+------------------+------------------+
|summary|           movieId|            rating|            userId|
+-------+------------------+------------------+------------------+
|  count|              1501|              1501|              1501|
|   mean| 49.40572951365756|1.7741505662891406|14.383744170552964|
| stddev|28.937034065088994| 1.187276166124803| 8.591040424293272|
|    min|                 0|               1.0|                 0|
|    max|                99|               5.0|                29|
+-------+------------------+------------------+------------------+
```

Like other models, we will do a train-test data split.  Unlike other models, recommender systems often deal with user subjectivity regarding specific items.  This makes "trusting" the reliability of a recommender system more difficult in some scenarios.  The canonical example is that a user might not like Star Trek just because they like Star Wars, despite the recommender system believing that they should. (I would be one of those users, by the way).  Perhaps more than any other model, recommender systems need large amounts of data (both distinct items and users) to arrive at reliable conclusions.

```python
# Smaller dataset so we will use 0.8 / 0.2
(training, test) = data.randomSplit([0.8, 0.2])

# Build the recommendation model using ALS on the training data
als = ALS(maxIter=5, regParam=0.01, userCol="userId", itemCol="movieId", ratingCol="rating")
model = als.fit(training)

# Evaluate the model by computing the RMSE on the test data
predictions = model.transform(test)
evaluator = RegressionEvaluator(metricName="rmse", labelCol="rating", predictionCol="prediction")
rmse = evaluator.evaluate(predictions)
print("RMSE = " + str(rmse))

## output in number of stars, from 1-5.
RMSE = 1.751
```

The RMSE described our error in terms of the stars rating column.  Ratings range from 1 to 5 stars, meaning that our average recommendation was off by close to 2 stars!.  That's not great.  Part of this is poor result is due to limited data for a challenging task -- human preferences are difficult to predict with high degrees of accuracy.  

So now that we have the model, how would you actually supply a recommendation to a user?
The same way we did with the test data! For example:

```python
single_user = test.filter(test['userId']==11).select(['movieId','userId'])

# User had 10 ratings in the test data set
# Realistically this should be some sort of hold out set!
single_user.show()

## output
+-------+------+
|movieId|userId|
+-------+------+
|      0|    11|
|     13|    11|
|     18|    11|
|     30|    11|
|     66|    11|
|     70|    11|
|     75|    11|
|     78|    11|
|     79|    11|
|     99|    11|
+-------+------+

reccomendations = model.transform(single_user)
reccomendations.orderBy('prediction',ascending=False).show()

## output
+-------+------+----------+
|movieId|userId|prediction|
+-------+------+----------+
|     30|    11|  5.578189|
|     13|    11|  3.257565|
|     70|    11| 2.7580981|
|     99|    11| 1.7420897|
|     18|    11| 1.5150304|
|     75|    11|   1.34218|
|     79|    11| 0.9733073|
|     66|    11| 0.5732717|
|     78|    11| 0.4434041|
|      0|    11|  -1.85298|
+-------+------+----------+
```






<BR>
<BR>




### Natural Language Processing (NLP)
I'm not supremely interested in NLP at the present time, so this section is simply going to be a pasting of Jose's course notes and example code.  NLP has a number of approaches and toolkits.  In Python, NLTK is the _de facto_ library to start with, though I am sure there are now more advanced ones available.

One of the hallmarks of NLP is that text / written data can be much messier than numerical data, and as a result there is frequently significant overhead in preparing the data for modeling.  A good example is "The dog ran to the corner" vs. "Later, the dog RAN to the corner and grabbed his bone."  Depending on how you prepare the data, and which model utilities you use, these two sentences could be considered as identical, highly similar, marginally similar, or completely different.

Examples:  
If you simply analyze for a match on the shortest string possible, "The dog ran to the corner" in our case, then both sentences would be matched as 100% identical since this exact string is in both.  If you don't control for the "RAN" capitalization, then the strings might not be counted as being as similar or even a match at all. If you don't tokenize the words and sort them, then they might not match since the second example begins with "Later, "... you get the idea.  NLP can be very ticky-tacky.

















```python
# For this code along we will build a spam filter! We'll use the various NLP tools we learned about as well as a new classifier, Naive Bayes.

# We'll use a classic dataset for this - UCI Repository SMS Spam Detection: https://archive.ics.uci.edu/ml/datasets/SMS+Spam+Collection

from pyspark.sql import SparkSession
spark = SparkSession.builder.appName('nlp').getOrCreate()
data = spark.read.csv("smsspamcollection/SMSSpamCollection",inferSchema=True,sep='\t')
data = data.withColumnRenamed('_c0','class').withColumnRenamed('_c1','text')
data.show()

## Output
+-----+--------------------+
|class|                text|
+-----+--------------------+
|  ham|Go until jurong p...|
|  ham|Ok lar... Joking ...|
| spam|Free entry in 2 a...|
|  ham|U dun say so earl...|
|  ham|Nah I don't think...|
| spam|FreeMsg Hey there...|
|  ham|Even my brother i...|
|  ham|As per your reque...|
| spam|WINNER!! As a val...|
| spam|Had your mobile 1...|
|  ham|I'm gonna be home...|
| spam|SIX chances to wi...|
| spam|URGENT! You have ...|
|  ham|I've been searchi...|
|  ham|I HAVE A DATE ON ...|
| spam|XXXMobileMovieClu...|
|  ham|Oh k...i'm watchi...|
|  ham|Eh u remember how...|
|  ham|Fine if thats th...|
| spam|England v Macedon...|
+-----+--------------------+
```

##### Clean and Prepare the Data
Create a new length feature:

```python
from pyspark.sql.functions import length
data = data.withColumn('length',length(data['text']))
data.show()

## Output
+-----+--------------------+------+
|class|                text|length|
+-----+--------------------+------+
|  ham|Go until jurong p...|   111|
|  ham|Ok lar... Joking ...|    29|
| spam|Free entry in 2 a...|   155|
|  ham|U dun say so earl...|    49|
|  ham|Nah I don't think...|    61|
| spam|FreeMsg Hey there...|   147|
|  ham|Even my brother i...|    77|
|  ham|As per your reque...|   160|
| spam|WINNER!! As a val...|   157|
| spam|Had your mobile 1...|   154|
|  ham|I'm gonna be home...|   109|
| spam|SIX chances to wi...|   136|
| spam|URGENT! You have ...|   155|
|  ham|I've been searchi...|   196|
|  ham|I HAVE A DATE ON ...|    35|
| spam|XXXMobileMovieClu...|   149|
|  ham|Oh k...i'm watchi...|    26|
|  ham|Eh u remember how...|    81|
|  ham|Fine if thats th...|    56|
| spam|England v Macedon...|   155|
+-----+--------------------+------+


# Pretty Clear Difference
data.groupby('class').mean().show()

+-----+-----------------+
|class|      avg(length)|
+-----+-----------------+
|  ham|71.48663212435233|
| spam|138.6706827309237|
+-----+-----------------+
```

##### Feature Transformations

```python
from pyspark.ml.feature import Tokenizer, StopWordsRemover, CountVectorizer, IDF, StringIndexer

tokenizer = Tokenizer(inputCol="text", outputCol="token_text")
stopremove = StopWordsRemover(inputCol='token_text',outputCol='stop_tokens')
count_vec = CountVectorizer(inputCol='stop_tokens',outputCol='c_vec')
idf = IDF(inputCol="c_vec", outputCol="tf_idf")
ham_spam_to_num = StringIndexer(inputCol='class',outputCol='label')

from pyspark.ml.feature import VectorAssembler
from pyspark.ml.linalg import Vector

clean_up = VectorAssembler(inputCols=['tf_idf','length'],outputCol='features')
```

##### The Model
We'll use Naive Bayes, but feel free to play around with this choice!

```python
from pyspark.ml.classification import NaiveBayes

# Use defaults
nb = NaiveBayes()
```

##### Pipeline

```python
from pyspark.ml import Pipeline

data_prep_pipe = Pipeline(stages=[ham_spam_to_num,tokenizer,stopremove,count_vec,idf,clean_up])

cleaner = data_prep_pipe.fit(data)
clean_data = cleaner.transform(data)
```

##### Training and Evaluation!

```python
clean_data = clean_data.select(['label','features'])
clean_data.show()

+-----+--------------------+
|label|            features|
+-----+--------------------+
|  0.0|(13461,[8,12,33,6.])|
|  0.0|(13461,[0,26,308,.])|
|  1.0|(13461,[2,14,20,3.])|
|  0.0|(13461,[0,73,84,1.])|
|  0.0|(13461,[36,39,140.])|
|  1.0|(13461,[11,57,62,.])|
|  0.0|(13461,[11,55,108.])|
|  0.0|(13461,[133,195,4.])|
|  1.0|(13461,[1,50,124,.])|
|  1.0|(13461,[0,1,14,29.])|
|  0.0|(13461,[5,19,36,4.])|
|  1.0|(13461,[9,18,40,9.])|
|  1.0|(13461,[14,32,50,.])|
|  0.0|(13461,[42,99,101.])|
|  0.0|(13461,[567,1744,.])|
|  1.0|(13461,[32,113,11.])|
|  0.0|(13461,[86,224,37.])|
|  0.0|(13461,[0,2,52,13.])|
|  0.0|(13461,[0,77,107,.])|
|  1.0|(13461,[4,32,35,6.])|
+-----+--------------------+


(training,testing) = clean_data.randomSplit([0.7,0.3])
spam_predictor = nb.fit(training)
test_results = spam_predictor.transform(testing)
test_results.show()

+-----+--------------------+--------------------+--------------------+----------+
|label|            features|       rawPrediction|         probability|prediction|
+-----+--------------------+--------------------+--------------------+----------+
|  0.0|(13461,[0,1,2,14,.])|[-612.34877984332..]|[0.99999999999999..]|       0.0|
|  0.0|(13461,[0,1,5,15,.])|[-742.97388469249..]|[1.0,5.5494439698..]|       0.0|
|  0.0|(13461,[0,1,6,16,.])|[-1004.8197043274..]|[1.0,5.0315468936..]|       0.0|
|  0.0|(13461,[0,1,12,34.])|[-879.22017540506..]|[1.0,1.0023148863..]|       0.0|
|  0.0|(13461,[0,1,15,33.])|[-216.47131414494..]|[1.0,1.1962236837..]|       0.0|
|  0.0|(13461,[0,1,16,21.])|[-673.71050817005..]|[1.0,1.5549413147..]|       0.0|
|  0.0|(13461,[0,1,22,26.])|[-382.58333036006..]|[1.0,1.7564627587..]|       0.0|
|  0.0|(13461,[0,1,25,66.])|[-1361.5572580867..]|[1.0,2.1016772175..]|       0.0|
|  0.0|(13461,[0,1,33,46.])|[-378.04433557629..]|[1.0,6.5844301586..]|       0.0|
|  0.0|(13461,[0,1,156,1.])|[-251.74061863695..]|[0.88109389963478..]|       0.0|
|  0.0|(13461,[0,1,510,5.])|[-325.61601503458..]|[0.99999999996808..]|       0.0|
|  0.0|(13461,[0,1,896,1.])|[-96.594570068189..]|[0.99999996371517..]|       0.0|
|  0.0|(13461,[0,2,3,6,7.])|[-2547.2759643071..]|[1.0,4.6246337876..]|       0.0|
|  0.0|(13461,[0,2,4,6,8.])|[-998.45874047729..]|[1.0,1.0141354676..]|       0.0|
|  0.0|(13461,[0,2,4,9,1.])|[-1316.5960403060..]|[1.0,4.5097599797..]|       0.0|
|  0.0|(13461,[0,2,4,28,.])|[-430.36443439941..]|[1.0,1.3571829152..]|       0.0|
|  0.0|(13461,[0,2,5,9,7.])|[-743.88098750283..]|[1.0,1.2520543556..]|       0.0|
|  0.0|(13461,[0,2,5,15,.])|[-1081.6827153914..]|[1.0,2.5678822847..]|       0.0|
|  0.0|(13461,[0,2,5,25,.])|[-833.43388596187..]|[0.99999999865682..]|       0.0|
|  0.0|(13461,[0,2,5,25,.])|[-492.43437000641..]|[1.0,2.6689876137..]|       0.0|
+-----+--------------------+--------------------+--------------------+----------+

from pyspark.ml.evaluation import MulticlassClassificationEvaluator

acc_eval = MulticlassClassificationEvaluator()
acc = acc_eval.evaluate(test_results)
print("Accuracy of model at predicting spam was: {}".format(acc))
```
Accuracy of model at predicting spam was: __0.9248020435242028__

Not bad considering we're using straight math on text data! Try switching out the classification models! Or even try to come up with other engineered features!


<BR>


#### Tools for NLP

There are lots of feature transformations that need to be done on text data to get it to a point that machine learning algorithms can understand. Luckily, Spark has placed the most important ones in convenient Feature Transformer calls.

Let's go over them before jumping into the project.

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName('nlp').getOrCreate()
```

<BR>

##### Tokenizer
[Tokenization](http://en.wikipedia.org/wiki/Lexical_analysis#Tokenization) is the process of taking text (such as a sentence) and breaking it into individual terms (usually words).  A simple [Tokenizer](api/scala/index.html#org.apache.spark.ml.feature.Tokenizer">) class provides this functionality.  The example below shows how to split sentences into sequences of words.

[RegexTokenizer](api/scala/index.html#org.apache.spark.ml.feature.RegexTokenizer) allows more advanced tokenization based on regular expression (regex) matching. By default, the parameter `pattern` (regex, default: `"\\s+"`) is used as delimiters to split the input text.
 Alternatively, users can set parameter `gaps` to false indicating the regex `pattern` denotes `tokens` rather than splitting gaps, and find all matching occurrences as the tokenization result.

```python
from pyspark.ml.feature import Tokenizer, RegexTokenizer
from pyspark.sql.functions import col, udf
from pyspark.sql.types import IntegerType

sentenceDataFrame = spark.createDataFrame([
    (0, "Hi I heard about Spark"),
    (1, "I wish Java could use case classes"),
    (2, "Logistic,regression,models,are,neat")
], ["id", "sentence"])

sentenceDataFrame.show()

+---+--------------------+
| id|            sentence|
+---+--------------------+
|  0|Hi I heard about ...|
|  1|I wish Java could...|
|  2|Logistic,regressi...|
+---+--------------------+

tokenizer = Tokenizer(inputCol="sentence", outputCol="words")
regexTokenizer = RegexTokenizer(inputCol="sentence", outputCol="words", pattern="\\W")
# alternatively, pattern="\\w+", gaps(False)

countTokens = udf(lambda words: len(words), IntegerType())

tokenized = tokenizer.transform(sentenceDataFrame)
tokenized.select("sentence", "words")    .withColumn("tokens", countTokens(col("words"))).show(truncate=False)

regexTokenized = regexTokenizer.transform(sentenceDataFrame)
regexTokenized.select("sentence", "words")     .withColumn("tokens", countTokens(col("words"))).show(truncate=False)

+-----------------------------------+------------------------------------------+------+
|sentence                           |words                                     |tokens|
+-----------------------------------+------------------------------------------+------+
|Hi I heard about Spark             |[hi, i, heard, about, spark]              |5     |
|I wish Java could use case classes |[i, wish, java, could, use, case, classes]|7     |
|Logistic,regression,models,are,neat|[logistic,regression,models,are,neat]     |1     |
+-----------------------------------+------------------------------------------+------+

+-----------------------------------+------------------------------------------+------+
|sentence                           |words                                     |tokens|
+-----------------------------------+------------------------------------------+------+
|Hi I heard about Spark             |[hi, i, heard, about, spark]              |5     |
|I wish Java could use case classes |[i, wish, java, could, use, case, classes]|7     |
|Logistic,regression,models,are,neat|[logistic, regression, models, are, neat] |5     |
+-----------------------------------+------------------------------------------+------+
```


<BR>

##### Stop Words Removal

[Stop words](https://en.wikipedia.org/wiki/Stop_words) are words which should be excluded from the input, typically because the words appear frequently and don't carry as much meaning.

`StopWordsRemover` takes as input a sequence of strings (e.g. the output of a [Tokenizer](ml-features.html#tokenizer)) and drops all the stop words from the input sequences. The list of stop words is specified by the `stopWords` parameter. Default stop words for some languages are accessible by calling `StopWordsRemover.loadDefaultStopWords(language)`, for which available options are `danish`, `dutch`, `english`, `finnish`, `french`, `german`, `hungarian`, `italian`, `norwegian`, `portuguese`, `russian`, `spanish`, `swedish` and `turkish`.

A boolean parameter `caseSensitive` indicates if the matches should be case sensitive
(false by default).


```python
from pyspark.ml.feature import StopWordsRemover

sentenceData = spark.createDataFrame([
    (0, ["I", "saw", "the", "red", "balloon"]),
    (1, ["Mary", "had", "a", "little", "lamb"])
], ["id", "raw"])

remover = StopWordsRemover(inputCol="raw", outputCol="filtered")
remover.transform(sentenceData).show(truncate=False)

## output
+---+----------------------------+--------------------+
|id |raw                         |filtered            |
+---+----------------------------+--------------------+
|0  |[I, saw, the, red, balloon] |[saw, red, balloon] |
|1  |[Mary, had, a, little, lamb]|[Mary, little, lamb]|
+---+----------------------------+--------------------+
```

<BR>

##### n-grams

An n-gram is a sequence of nn tokens (typically words) for some integer nn. The NGram class can be used to transform input features into nn-grams.

`NGram` takes as input a sequence of strings (e.g. the output of a [Tokenizer](ml-features.html#tokenizer).  The parameter `n` is used to determine the number of terms in each n-gram. The output will consist of a sequence of n-grams where each n-gram is represented by a space-delimited string of _n_ consecutive words.  If the input sequence contains fewer than `n` strings, no output is produced.


```python
from pyspark.ml.feature import NGram

wordDataFrame = spark.createDataFrame([
    (0, ["Hi", "I", "heard", "about", "Spark"]),
    (1, ["I", "wish", "Java", "could", "use", "case", "classes"]),
    (2, ["Logistic", "regression", "models", "are", "neat"])
], ["id", "words"])

ngram = NGram(n=2, inputCol="words", outputCol="ngrams")

ngramDataFrame = ngram.transform(wordDataFrame)
ngramDataFrame.select("ngrams").show(truncate=False)

+------------------------------------------------------------------+
|ngrams                                                            |
+------------------------------------------------------------------+
|[Hi I, I heard, heard about, about Spark]                         |
|[I wish, wish Java, Java could, could use, use case, case classes]|
|[Logistic regression, regression models, models are, are neat]    |
+------------------------------------------------------------------+
```

<br>


#### Feature Extractors
_______

##### TF-IDF
(The `$` signs are for LaTeX rendering, if activated)

[Term frequency-inverse document frequency (TF-IDF)](http://en.wikipedia.org/wiki/Tf%E2%80%93idf) is a feature vectorization method widely used in text mining to reflect the importance of a term to a document in the corpus. Denote a term by _$t$_, a document by _d_, and the corpus by _D_.

Term frequency _$TF(t, d)$_ is the number of times that term _$t$_ appears in document _$d$_, while document frequency _$DF(t, D)$_ is the number of documents that contains term _$t$_. If we only use term frequency to measure the importance, it is very easy to over-emphasize terms that appear very often but carry little information about the document, e.g. "a", "the", and "of". If a term appears very often across the corpus, it means it doesn't carry special information about a particular document.

Inverse document frequency is a numerical measure of how much information a term provides:

$$ IDF(t, D) = \log \frac{|D| + 1}{DF(t, D) + 1} $$

where |D| is the total number of documents in the corpus. Since logarithm is used, if a term
appears in all documents, its IDF value becomes 0. Note that a smoothing term is applied to avoid dividing by zero for terms outside the corpus. The TF-IDF measure is simply the product of TF and IDF:

$$ TFIDF(t, d, D) = TF(t, d) \cdot IDF(t, D). $$


```python
from pyspark.ml.feature import HashingTF, IDF, Tokenizer

sentenceData = spark.createDataFrame([
    (0.0, "Hi I heard about Spark"),
    (0.0, "I wish Java could use case classes"),
    (1.0, "Logistic regression models are neat")
], ["label", "sentence"])

sentenceData.show()

+-----+--------------------+
|label|            sentence|
+-----+--------------------+
|  0.0|Hi I heard about ...|
|  0.0|I wish Java could...|
|  1.0|Logistic regressi...|
+-----+--------------------+


tokenizer = Tokenizer(inputCol="sentence", outputCol="words")
wordsData = tokenizer.transform(sentenceData)
wordsData.show()

+-----+--------------------+--------------------+
|label|            sentence|               words|
+-----+--------------------+--------------------+
|  0.0|Hi I heard about ...|[hi, i, heard, ab...|
|  0.0|I wish Java could...|[i, wish, java, c...|
|  1.0|Logistic regressi...|[logistic, regres...|
+-----+--------------------+--------------------+


hashingTF = HashingTF(inputCol="words", outputCol="rawFeatures", numFeatures=20)
featurizedData = hashingTF.transform(wordsData)
# alternatively, CountVectorizer can also be used to get term frequency vectors

idf = IDF(inputCol="rawFeatures", outputCol="features")
idfModel = idf.fit(featurizedData)
rescaledData = idfModel.transform(featurizedData)

rescaledData.select("label", "features").show()

+-----+--------------------+
|label|            features|
+-----+--------------------+
|  0.0|(20,[0,5,9,17],[0...|
|  0.0|(20,[2,7,9,13,15]...|
|  1.0|(20,[4,6,13,15,18...|
+-----+--------------------+
```

<BR>

#### CountVectorizer
CountVectorizer and CountVectorizerModel aim to help convert a collection of text documents to vectors of token counts. When an a-priori dictionary is not available, CountVectorizer can be used as an Estimator to extract the vocabulary, and generates a CountVectorizerModel. The model produces sparse representations for the documents over the vocabulary, which can then be passed to other algorithms like LDA.

During the fitting process, CountVectorizer will select the top vocabSize words ordered by term frequency across the corpus. An optional parameter minDF also affects the fitting process by specifying the minimum number (or fraction if < 1.0) of documents a term must appear in to be included in the vocabulary. Another optional binary toggle parameter controls the output vector. If set to true all non-zero counts are set to 1. This is especially useful for discrete probabilistic models that model binary, rather than integer, counts.

```python
from pyspark.ml.feature import CountVectorizer

# Input data: Each row is a bag of words with a ID.
df = spark.createDataFrame([
    (0, "a b c".split(" ")),
    (1, "a b b c a".split(" "))
], ["id", "words"])

# fit a CountVectorizerModel from the corpus.
cv = CountVectorizer(inputCol="words", outputCol="features", vocabSize=3, minDF=2.0)

model = cv.fit(df)

result = model.transform(df)
result.show(truncate=False)

+---+---------------+-------------------------+
|id |words          |features                 |
+---+---------------+-------------------------+
|0  |[a, b, c]      |(3,[0,1,2],[1.0,1.0,1.0])|
|1  |[a, b, b, c, a]|(3,[0,1,2],[2.0,2.0,1.0])|
+---+---------------+-------------------------+
```

<BR>

#### Spark Streaming
This is covered in the course, but it was from 2017 and I believe there have been numerous changes, including possibly platform-redefining changes, to how streaming is handled.  Consulting the course materials (`Python-and-Spark-for-Big-Data-master/Spark Streaming/Introduction to Spark Streaming.ipynb`) would be a good start, but it is probable that there will be more current guides on the internet.  

For what it's worth, this old version of Spark Streaming looked like a damn mess.
