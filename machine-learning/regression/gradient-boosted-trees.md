# Gradient-Boosted Trees

## Setting Up Gradient-Boosted Tree Regression

{% hint style="info" %}
**Note:** Make sure you have your training and test data already vectorized and ready to go before you begin trying to fit the machine learning model to unprepped data.
{% endhint %}

### Load in required libraries

```python
from pyspark.ml.regression import GBTRegressor
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator
from pyspark.ml.evaluation import RegressionEvaluator
```

### Initialize Gradient-Boosted Tree object

```python
gb = GBTRegressor(labelCol="label", featuresCol="features")
```

### Create a parameter grid for tuning the model

```python
gbparamGrid = (ParamGridBuilder()
             .addGrid(gb.maxDepth, [2, 5, 10])
             .addGrid(gb.maxBins, [10, 20, 40])
             .addGrid(gb.maxIter, [5, 10, 20])
             .build())
```

### Define how you want the model to be evaluated

```python
gbevaluator = RegressionEvaluator(predictionCol="prediction", labelCol="label", metricName="rmse")
```

### Define the type of cross-validation you want to perform

```python
# Create 5-fold CrossValidator
gbcv = CrossValidator(estimator = gb,
                      estimatorParamMaps = gbparamGrid,
                      evaluator = gbevaluator,
                      numFolds = 5)
```

### Fit the model to the data

```python
gbcvModel = gbcv.fit(train)
print(gbcvModel)
```

### Score the testing dataset using your fitted model for evaluation purposes

```python
gbpredictions = gbcvModel.transform(test)
```

### Evaluate the model

```python
print('RMSE:', gbevaluator.evaluate(gbpredictions))
```

{% hint style="info" %}
**Note:** When you use the `CrossValidator` function to set up cross-validation of your models, the resulting model object will have all the runs included, but will only use the best model when you interact with the model object using other functions like `evaluate` or `transform`.
{% endhint %}

