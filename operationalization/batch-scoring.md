# Batch Scoring

{% hint style="info" %}
This section is designed for use with a data orchestration tool that can call and execute Databricks notebooks. For more information on how to set up Azure Data Factory, see: [https://docs.microsoft.com/en-us/azure/data-factory/transform-data-using-databricks-notebook](https://docs.microsoft.com/en-us/azure/data-factory/transform-data-using-databricks-notebook).
{% endhint %}

### Create date parameter

```python
dbutils.widgets.text("varReportDate", "19000101")
ReportDate = dbutils.widgets.get("varReportDate")
print(ReportDate)
```

### Connect to storage

```python
storage_account_name = "mystorage"
storage_account_access_key = ""

file_location = "wasbs://<container>@mystorage.blob.core.windows.net/myfiles/data_" + ReportDate + ".csv"
file_type = "csv"

spark.conf.set(
  "fs.azure.account.key."+storage_account_name+".blob.core.windows.net",
  storage_account_access_key)
```

### Define input schema

```python
from pyspark.sql.types import *

schema = StructType([
    StructField("ReportingDate", DateType(), True),
    StructField("id", StringType(), True),
    StructField("x1", IntegerType(), True),
    StructField("x2", DoubleType(), True)
])
```

### Read in new data

```python
dataset = spark.read\
               .format(file_type)\
               .option("header", "true")\
               .schema(schema)\
               .load(file_location)

## You can avoid defining a schema by having spark infer it from your data
## This doesn't always work and can be slow
#.option("inferSchema", "true")

## Fill in na's, if needed
# dataset = dataset.na.fill(0)
display(dataset)
```

### Load in transformation pipeline and model

```python
from pyspark.ml.tuning import CrossValidatorModel
from pyspark.ml import PipelineModel
from pyspark.sql.types import IntegerType
from pyspark.sql.functions import col, round
from pyspark.ml.regression import GeneralizedLinearRegressionModel

mypipeline = PipelineModel.load("/mnt/trainedmodels/pipeline/")
mymodel = CrossValidatorModel.load("/mnt/trainedmodels/lr")
```

### Score data using the model

```python
## Transform new data using the pipeline
mydataset = mypipeline.transform(dataset)
## Score new data using a trained model
scoreddataset = mymodel.bestModel.transform(mydataset)

output = scoreddataset.select(col("id"),
                              col("ReportingDate"),
                              col("prediction").alias("MyForecast"))
display(output)
```

### Write data back out to storage

```python
fileloc = "/mnt/output" + str(ReportDate) #+ ".csv"
output.write.mode("overwrite").format("com.databricks.spark.csv").option("header","true").csv(fileloc)
```

