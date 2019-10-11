# Structured Streaming

## Read in Streaming Data

### Reading JSON files from storage

```python
from pyspark.sql.types import *

inputPath = "/mnt/data/jsonfiles/"

# Define your schema if it's known (rather than relying on Spark to infer the schema)
jsonSchema = StructType([StructField("time", TimestampType(), True),
                         StructField("id", IntegerType(), True),
                         StructField("value", StringType(), True)])

streamingInputDF = spark.readStream \
                        .schema(jsonSchema) \
                        .option("maxFilesPerTrigger", 1) \ # Treat a sequence of files as a stream by picking one file at a time
                        .json(inputPath)
```

### References

* Databricks Structured Streaming: [https://docs.databricks.com/spark/latest/structured-streaming/index.html](https://docs.databricks.com/spark/latest/structured-streaming/index.html)

