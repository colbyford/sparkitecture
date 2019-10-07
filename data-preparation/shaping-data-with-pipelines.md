# Shaping Data with Pipelines

### Load in required libraries

```python
from pyspark.ml import Pipeline
from pyspark.ml.feature import OneHotEncoder, OneHotEncoderEstimator, StringIndexer, VectorAssembler
```

### Define which columns are numerical versus categorical \(and which is the label column\)

```python
label = "dependentvar"
categoricalColumns = ["col1",
                     "col2"]

numericalColumns = ["num1",
                    "num2"]

#categoricalColumnsclassVec = ["col1classVec",
#                              "col2classVec"]
categoricalColumnsclassVec = [c + "classVec" for c in categoricalColumns]
```

### Set up stages

```text
stages = []
```

### Index the categorical columns and perform One Hot Encoding

One Hot Encoding will convert a categorical column into multiple columns for each class. \(This process is similar to dummy coding.\)

```python
for categoricalColumn in categoricalColumns:
  print(categoricalColumn)
  ## Category Indexing with StringIndexer
  stringIndexer = StringIndexer(inputCol=categoricalColumn, outputCol = categoricalColumn+"Index").setHandleInvalid("skip")
  ## Use OneHotEncoder to convert categorical variables into binary SparseVectors
  encoder = OneHotEncoder(inputCol=categoricalColumn+"Index", outputCol=categoricalColumn+"classVec")
  ## Add stages
  stages += [stringIndexer, encoder]
```

### Index the label column and perform One Hot Encoding

```python
## Convert label into label indices using the StringIndexer
label_stringIndexer = StringIndexer(inputCol = label, outputCol = "label").setHandleInvalid("skip")
stages += [label_stringIndexer]
```

{% hint style="info" %}
 **Note:** If you are preparing the data for use in regression algorithms, there's no need to One Hot Encode the label column \(since it should be numerical\).
{% endhint %}

### Assemble the data together as a vector

This step transforms all the numerical data along with the encoded categorical data into a series of vectors using the `VectorAssembler` function.

```python
assemblerInputs = categoricalColumnsclassVec + numericalColumns
assembler = VectorAssembler(inputCols = assemblerInputs, outputCol="features")
stages += [assembler]
```

### Set up the transformation pipeline using the stages you've created along the way

```python
prepPipeline = Pipeline().setStages(stages)
pipelineModel = prepPipeline.fit(dataset)
dataset = pipelineModel.transform(dataset)
```

## Pipeline Saving and Loading

Once your transformation pipeline has been creating on your training dataset, it's a good idea to save these transformation steps for future use. For example, we can save the pipeline so that we can equally transform new data before scoring it through a trained machine learning model. This also helps to cut down on errors when using new data that has classes \(in categorical variables\) or previously unused columns.

### Save the transformation pipeline

```python
pipelineModel.save("/mnt/<YOURMOUNTEDSTORAGE>/pipeline")
display(dbutils.fs.ls("/mnt/<YOURMOUNTEDSTORAGE>/pipeline"))
```

### Load in the transformation pipeline

```python
from pyspark.ml import PipelineModel
pipelineModel = PipelineModel.load("/mnt/<YOURMOUNTEDSTORAGE>/pipeline")
dataset = pipelineModel.transform(dataset)
display(dataset)
```

