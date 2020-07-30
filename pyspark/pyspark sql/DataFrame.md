### agg(*exprs)
> *Aggregate on the entire DataFrame without groups (shorthand for df.groupBy.agg()).*
```python
>>> df.agg({"age": "max"}).collect()
[Row(max(age)=5)]
>>> from pyspark.sql import functions as F
>>> df.agg(F.min(df.age)).collect()
[Row(min(age)=2)]
```
### alias(alias)
> *Returns a new DataFrame with an alias set.*
```python
>>> from pyspark.sql.functions import *
>>> df_as1 = df.alias("df_as1")
>>> df_as2 = df.alias("df_as2")
>>> joined_df = df_as1.join(df_as2, col("df_as1.name") == col("df_as2.name"), 'inner')
>>> joined_df.select("df_as1.name", "df_as2.name", "df_as2.age").collect()
[Row(name=u'Bob', name=u'Bob', age=5), Row(name=u'Alice', name=u'Alice', age=2)]
```
### approxQuantile(col, probabilities, relativeError)
> 
    Parameters:	
    col – the name of the numerical column

    probabilities – a list of quantile probabilities Each number must belong to [0, 1]. For example 0 is the minimum, 0.5 is the median, 1 is the maximum.
    
    relativeError – The relative target precision to achieve (>= 0). If set to zero, the exact quantiles are computed, which could be very expensive. Note that values greater than 1 are accepted but give the same result as 1.
    
    Returns:	
    the approximate quantiles at the given probabilities

```python
>>> df.approxQuantile("age",[0.1,0.9],0.3)
[9.0,27.0]
```
### coalesce(numPartitions)
> *Returns a new DataFrame that has exactly numPartitions partitions.*
```python
>>> df.coalesce(1).rdd.getNumPartitions()
1
```
### corr(col1, col2, method=None)
> *Calculates the correlation of two columns of a DataFrame as a double value.*
> Parameters:	
    col1 – The name of the first column
    col2 – The name of the second column
    method – The correlation method. Currently only supports “pearson”
### cov(col1, col2)
> *Calculate the sample covariance for the given columns, specified by their names, as a double value*
> Parameters:	
    col1 – The name of the first column
    col2 – The name of the second column
### createOrReplaceTempView(name)
> *Creates or replaces a temporary view with this DataFrame.*
```python
>>> df.createOrReplaceTempView("people")
>>> df2 = df.filter(df.age > 3)
>>> df2.createOrReplaceTempView("people")
>>> df3 = spark.sql("select * from people")
>>> sorted(df3.collect()) == sorted(df2.collect())
True
>>> spark.catalog.dropTempView("people")
```
### createTempView(name)
> *Creates a temporary view with this DataFrame.*
```python
>>> df.createTempView("people")
>>> df2 = spark.sql("select * from people")
>>> sorted(df.collect()) == sorted(df2.collect())
True
>>> df.createTempView("people")  
Traceback (most recent call last):
...
AnalysisException: u"Temporary table 'people' already exists;"
>>> spark.catalog.dropTempView("people")
```
### cube(*cols)
> *Create a multi-dimensional cube for the current DataFrame using the specified columns, so we can run aggregation on them.*
```python
>>> df.cube("name", df.age).count().orderBy("name", "age").show()
+-----+----+-----+
| name| age|count|
+-----+----+-----+
| null|null|    2|
| null|   2|    1|
| null|   5|    1|
|Alice|null|    1|
|Alice|   2|    1|
|  Bob|null|    1|
|  Bob|   5|    1|
+-----+----+-----+
```
### describe(*cols)
> *Computes statistics for numeric columns.*
>
    This include count, mean, stddev, min, and max. If no columns are given, this function computes statistics for all numerical columns.
```python
>>> df.describe().show()
+-------+------------------+
|summary|               age|
+-------+------------------+
|  count|                 2|
|   mean|               3.5|
| stddev|2.1213203435596424|
|    min|                 2|
|    max|                 5|
+-------+------------------+
>>> df.describe(['age', 'name']).show()
+-------+------------------+-----+
|summary|               age| name|
+-------+------------------+-----+
|  count|                 2|    2|
|   mean|               3.5| null|
| stddev|2.1213203435596424| null|
|    min|                 2|Alice|
|    max|                 5|  Bob|
+-------+------------------+-----+
```
### distinct()
> *Returns a new DataFrame containing the distinct rows in this DataFrame.*
```python
>>> df.distinct().count()
2
```
### drop(col)
> *Returns a new DataFrame that drops the specified column.*
```python
>>> df.drop('age').collect()
[Row(name=u'Alice'), Row(name=u'Bob')]
```
```python
>>> df.drop(df.age).collect()
[Row(name=u'Alice'), Row(name=u'Bob')]
```
```python
>>> df.join(df2, df.name == df2.name, 'inner').drop(df.name).collect()
[Row(age=5, height=85, name=u'Bob')]
```
### dropDuplicates(subset=None)
>*Return a new DataFrame with duplicate rows removed, optionally only considering certain columns.*

> drop_duplicates() is an alias for dropDuplicates().
```python
>>> from pyspark.sql import Row
>>> df = sc.parallelize([ \
...     Row(name='Alice', age=5, height=80), \
...     Row(name='Alice', age=5, height=80), \
...     Row(name='Alice', age=10, height=80)]).toDF()
>>> df.dropDuplicates().show()
+---+------+-----+
|age|height| name|
+---+------+-----+
|  5|    80|Alice|
| 10|    80|Alice|
+---+------+-----+
```
```python
>>> df.dropDuplicates(['name', 'height']).show()
+---+------+-----+
|age|height| name|
+---+------+-----+
|  5|    80|Alice|
+---+------+-----+
```
### dropna(how='any', thresh=None, subset=None)
>*Returns a new DataFrame omitting rows with null values. DataFrame.dropna() and DataFrameNaFunctions.drop() are aliases of each other.*

>
    Parameters:	
    how – ‘any’ or ‘all’. If ‘any’, drop a row if it contains any nulls. If ‘all’, drop a row only if all its values are null.
    thresh – int, default None If specified, drop rows that have less than thresh non-null values. This overwrites the how parameter.
    subset – optional list of column names to consider.
```python
>>> df4.dropna().show()
+---+------+-----+
|age|height| name|
+---+------+-----+
| 10|    80|Alice|
+---+------+-----+
```
### dtypes
Returns all column names and their data types as a list.
```python
>>> df.dtypes
[('age', 'int'), ('name', 'string')]
```
### fillna(value, subset=None)
>*Replace null values, alias for na.fill(). DataFrame.fillna() and DataFrameNaFunctions.fill() are aliases of each other.*
>
    Parameters:	
    value – int, long, float, string, or dict. Value to replace null values with. If the value is a dict, then subset is ignored and value must be a mapping from column name (string) to replacement value. The replacement value must be an int, long, float, or string.
    subset – optional list of column names to consider. Columns specified in subset that do not have matching data type are ignored. For example, if value is a string, and subset contains a non-string column, then the non-string column is simply ignored.
```python
>>> df4.fillna(50).show()
+---+------+-----+
|age|height| name|
+---+------+-----+
| 10|    80|Alice|
|  5|    50|  Bob|
| 50|    50|  Tom|
| 50|    50| null|
+---+------+-----+
```
```python
>>> df4.fillna({'age': 50, 'name': 'unknown'}).show()
+---+------+-------+
|age|height|   name|
+---+------+-------+
| 10|    80|  Alice|
|  5|  null|    Bob|
| 50|  null|    Tom|
| 50|  null|unknown|
+---+------+-------+
```
### filter(condition)
>Filters rows using the given condition.
>where() is an alias for filter().
```python
>>> df.filter(df.age > 3).collect()
[Row(age=5, name=u'Bob')]
>>> df.where(df.age == 2).collect()
[Row(age=2, name=u'Alice')]
```
```python
>>> df.filter("age > 3").collect()
[Row(age=5, name=u'Bob')]
>>> df.where("age = 2").collect()
[Row(age=2, name=u'Alice')]
```
### first()
> Returns the first row as a Row.
```python
>>> df.first()
Row(age=2, name=u'Alice')
```
### foreach(f)
> Applies the f function to all Row of this DataFrame.
> This is a shorthand for df.rdd.foreach().
```python
>>> def f(person):
...     print(person.name)
>>> df.foreach(f)
```
### foreachPartition(f)
> Applies the f function to each partition of this DataFrame.
> This a shorthand for df.rdd.foreachPartition().
```python
>>> def f(people):
        for person in people:
...         print(person.name)
>>> df.foreachPartition(f)
```
### groupBy(*cols)
> Groups the DataFrame using the specified columns, so we can run aggregation on them. See GroupedData for all the available aggregate functions.

> groupby() is an alias for groupBy().

>Parameters:	cols – list of columns to group by. Each element should be a column name (string) or an expression (Column).
```python
>>> df.groupBy().avg().collect()
[Row(avg(age)=3.5)]
>>> sorted(df.groupBy(df.name).agg({"age":"mean"}).collect())
[Row(name=u'Alice', avg(age)=2.0), Row(name=u'Bob', avg(age)=5.0)]
>>> sorted(df.groupBy(df.name).avg().collect())
[Row(name=u'Alice', avg(age)=2.0), Row(name=u'Bob', avg(age)=5.0)]
>>> sorted(df.groupBy(["name",df.age]).count().collect())
[Row(name=u'Alice', age=2, count=1), Row(name=u'Bob', age=5, count=1)]
```

### head(n=None)
> Returns the first n rows.

> Note that this method should only be used if the resulting array is expected to be small, as all the data is loaded into the driver’s memory.

> Parameters:	n – int, default 1. Number of rows to return.
> Returns:	If n is greater than 1, return a list of Row. If n is 1, return a single Row.
```python
>>> df.head()
Row(age=2, name=u'Alice')
>>> df.head(1)
[Row(age=2, name=u'Alice')]
```