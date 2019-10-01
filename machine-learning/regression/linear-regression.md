# Linear Regression

## Setting Up Linear Regression

{% hint style="info" %}
**Note:** Make sure you have your training and test data already vectorized and ready to go before you begin trying to fit the machine learning model to unprepped data.
{% endhint %}

### Load in required libraries

```python
from pyspark.ml.regression import LinearRegression
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator
from pyspark.ml.evaluation import RegressionEvaluator
```

### Initialize Linear Regression object

```python
lr = LinearRegression(labelCol="label", featuresCol="features")
```

### Create a parameter grid for tuning the model

```python
lrparamGrid = (ParamGridBuilder()
             .addGrid(lr.regParam, [0.001, 0.01, 0.1, 0.5, 1.0, 2.0])
             #  .addGrid(lr.regParam, [0.01, 0.1, 0.5])
             .addGrid(lr.elasticNetParam, [0.0, 0.25, 0.5, 0.75, 1.0])
             #  .addGrid(lr.elasticNetParam, [0.0, 0.5, 1.0])
             .addGrid(lr.maxIter, [1, 5, 10, 20, 50])
             #  .addGrid(lr.maxIter, [1, 5, 10])
             .build())
```

### Define how you want the model to be evaluated

```python
lrevaluator = RegressionEvaluator(predictionCol="prediction", labelCol="label", metricName="rmse")
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

### Get model information

```python
lrcvSummary = lrcvModel.bestModel.summary
print("Coefficient Standard Errors: " + str(lrcvSummary.coefficientStandardErrors))
print("P Values: " + str(lrcvSummary.pValues)) # Last element is the intercept
```

### Score the testing dataset using your fitted model for evaluation purposes

```python
lrpredictions = lrcvModel.transform(test)
```

### Evaluate the model

```python
print('RMSE:', lrevaluator.evaluate(lrpredictions))
```

{% hint style="info" %}
**Note:** When you use the `CrossValidator` function to set up cross-validation of your models, the resulting model object will have all the runs included, but will only use the best model when you interact with the model object using other functions like `evaluate` or `transform`.
{% endhint %}

