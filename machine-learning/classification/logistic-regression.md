# Logistic Regression

## Setting Up a Logistic Regression Classifier

{% hint style="info" %}
**Note:** Make sure you have your training and test data already vectorized and ready to go before you begin trying to fit the machine learning model to unprepped data.
{% endhint %}

### Load in required libraries

```python
from pyspark.ml.classification import LogisticRegression
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator
from pyspark.ml.evaluation import BinaryClassificationEvaluator
from pyspark.mllib.evaluation import BinaryClassificationMetrics
```

### Initialize Logistic Regression object

```python
lr = LogisticRegression(labelCol="label", featuresCol="features")
```

### Create a parameter grid for tuning the model

```python
lrparamGrid = (ParamGridBuilder()
             .addGrid(lr.regParam, [0.01, 0.1, 0.5, 1.0, 2.0])
             .addGrid(lr.elasticNetParam, [0.0, 0.25, 0.5, 0.75, 1.0])
             .addGrid(lr.maxIter, [1, 5, 10, 20, 50])
             .build())
```

### Define how you want the model to be evaluated

```python
lrevaluator = BinaryClassificationEvaluator(rawPredictionCol="rawPrediction", metricName = "areaUnderROC")
```

### Define the type of cross-validation you want to perform

```python
# Create 5-fold CrossValidator
lrcv = CrossValidator(estimator = lr,
                    estimatorParamMaps = lrparamGrid,
                    evaluator = lrevaluator,
                    numFolds = 5)
```

### Fit the model to the data

```python
lrcvModel = lrcv.fit(train)
print(lrcvModel)
```

### Score the testing dataset using your fitted model for evaluation purposes

```python
lrpredictions = lrcvModel.transform(test)
```

### Evaluate the model

```python
print('Accuracy:', lrevaluator.evaluate(lrpredictions))
print('AUC:', BinaryClassificationMetrics(lrpredictions['label','prediction'].rdd).areaUnderROC)
print('PR:', BinaryClassificationMetrics(lrpredictions['label','prediction'].rdd).areaUnderPR)
```

{% hint style="info" %}
**Note:** When you use the `CrossValidator` function to set up cross-validation of your models, the resulting model object will have all the runs included, but will only use the best model when you interact with the model object using other functions like `evaluate` or `transform`.
{% endhint %}

