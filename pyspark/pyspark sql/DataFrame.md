### agg(*exprs)
> *Aggregate on the entire DataFrame without groups (shorthand for df.groupBy.agg()).*
```python
>>> df.agg({"age": "max"}).collect()
[Row(max(age)=5)]
>>> from pyspark.sql import functions as F
>>> df.agg(F.min(df.age)).collect()
[Row(min(age)=2)]
```
  