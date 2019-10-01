# Model Saving and Loading

## Model Saving

### Save model\(s\) to mounted storage 

```python
lrcvModel.save("/mnt/trainedmodels/lr")
rfcvModel.save("/mnt/trainedmodels/rf")
dtcvModel.save("/mnt/trainedmodels/dt")
display(dbutils.fs.ls("/mnt/trainedmodels/"))
```

### Remove a model

Spark MLlib models are actually a series of files in a directory. So, you will need to recursively delete the files in model's directory, then the directory itself.

```python
dbutils.fs.rm("/mnt/trainedmodels/dt", True)
```

## Score new data using a trained model

### Load in required libraries

```text
from pyspark.ml.tuning import CrossValidatorModel
from pyspark.ml import PipelineModel
from pyspark.sql.functions import col, round
from pyspark.sql.types import IntegerType, FloatType
```

### Load in the transformation pipeline

```python
pipeline = PipelineModel.load("/mnt/trainedmodels/pipeline/")
## Fit the pipeline to new data
transformeddataset = pipeline.transform(dataset)
```

### Load in the trained model

```python
model = CrossValidatorModel.load("/mnt/trainedmodels/lr/")
## Score the data using the model
scoreddataset = model.bestModel.transform(transformeddataset)
```

### Remove unnecessary columns from the scored data

```python
## Function to extract probability from array
getprob = udf(lambda v:float(v[1]),FloatType())

## Select out the necessary columns
output = scoreddataset.select(col("ID"),
                              col("label"),
                              col("rawPrediction"),           
                              getprob(col("probability")).alias("probability"),
                              col("prediction"))
```

### 

