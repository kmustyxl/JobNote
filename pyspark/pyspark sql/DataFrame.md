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