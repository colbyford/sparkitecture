# Random Forest

## Setting Up a Random Forest Classifier

{% hint style="info" %}
**Note:** Make sure you have your training and test data already vectorized and ready to go before you begin trying to fit the machine learning model to unprepped data.
{% endhint %}

### Load in required libraries

```python
from pyspark.ml.classification import RandomForestClassifier
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator
from pyspark.ml.evaluation import BinaryClassificationEvaluator
```

### Initialize Random Forest object

```python
rf = RandomForestClassifier(labelCol="label", featuresCol="features")
```

### Create a parameter grid for tuning the model

```python
rfparamGrid = (ParamGridBuilder()

               .addGrid(rf.maxDepth, [2, 5, 10])

               .addGrid(rf.maxBins, [5, 10, 20])

               .addGrid(rf.numTrees, [5, 20, 50])
             .build())
```

### Define how you want the model to be evaluated

```python
rfevaluator = BinaryClassificationEvaluator(rawPredictionCol="rawPrediction")
```

### Define the type of cross-validation you want to perform

```python
# Create 5-fold CrossValidator
rfcv = CrossValidator(estimator = rf,
                      estimatorParamMaps = rfparamGrid,
                      evaluator = rfevaluator,
                      numFolds = 5)
```

### Fit the model to the data

```python
rfcvModel = rfcv.fit(train)
print(rfcvModel)
```

### Score the testing dataset using your fitted model for evaluation purposes

```python
rfpredictions = rfcvModel.transform(test)
```

### Evaluate the model

```python
print('Accuracy:', rfevaluator.evaluate(rfpredictions))
print('AUC:', BinaryClassificationMetrics(rfpredictions['label','prediction'].rdd).areaUnderROC)
print('PR:', BinaryClassificationMetrics(rfpredictions['label','prediction'].rdd).areaUnderPR)
```

{% hint style="info" %}
**Note:** When you use the `CrossValidator` function to set up cross-validation of your models, the resulting model object will have all the runs included, but will only use the best model when you interact with the model object using other functions like `evaluate` or `transform`.
{% endhint %}

