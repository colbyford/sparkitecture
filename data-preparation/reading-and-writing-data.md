# Reading and Writing Data

## Reading in Data

### ...from Mounted Storage

```python
dataset = sqlContext.read.format('csv') \
                    .options(header='true', inferSchema='true', delimiter= ',') \
                    .load('/mnt/<FOLDERNAME>/<FILENAME>.csv')

## or spark.read.format('csv')...
## Formats: json, parquet, jdbc, orc, libsvm, csv, text, avro
```

### ...when Schema Inference Fails

```python
from pyspark.sql.types import *

schema = StructType([StructField('ID', IntegerType(), True),
                     StructField('Value', DoubleType(), True),
                     StructField('Category', StringType(), True),
                     StructField('Date', DateType(), True)])

dataset = sqlContext.read.format('csv') \
                    .schema(schema) \
                    .options(header='true', delimiter= ',') \
                    .load('/mnt/<FOLDERNAME>/<FILENAME>.csv')
```

## Writing out Data

```python
df.coalesce(1) \
   .write.format("com.databricks.spark.csv") \
   .option("header", "true") \
   .save("file.csv")
```

## Other Resources

Apache Spark Data Sources Documentation: [https://spark.apache.org/docs/latest/sql-data-sources.html](https://spark.apache.org/docs/latest/sql-data-sources.html)

