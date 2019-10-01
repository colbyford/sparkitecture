# Decision Tree

## Setting Up Decision Tree Regression

{% hint style="info" %}
**Note:** Make sure you have your training and test data already vectorized and ready to go before you begin trying to fit the machine learning model to unprepped data.
{% endhint %}

### Load in required libraries

```python
from pyspark.ml.regression import DecisionTreeRegressor
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator
from pyspark.ml.evaluation import RegressionEvaluator
```

### Initialize Decision Tree object

```python
dt = DecisionTreeRegressor(labelCol="label", featuresCol="features")
```

### Create a parameter grid for tuning the model

```python
dtparamGrid = (ParamGridBuilder()
             .addGrid(dt.maxDepth, [2, 5, 10, 20, 30])
             #.addGrid(dt.maxDepth, [2, 5, 10])
             .addGrid(dt.maxBins, [10, 20, 40, 80, 100])
             #.addGrid(dt.maxBins, [10, 20])
             .build())
```

### Define how you want the model to be evaluated

```python
dtevaluator = RegressionEvaluator(predictionCol="prediction", labelCol="label", metricName="rmse")
```

### Define the type of cross-validation you want to perform

```python
# Create 5-fold CrossValidator
dtcv = CrossValidator(estimator = dt,
                      estimatorParamMaps = dtparamGrid,
                      evaluator = dtevaluator,
                      numFolds = 5)
```

### Fit the model to the data

```python
dtcvModel = dtcv.fit(train)
print(dtcvModel)
```

### Score the testing dataset using your fitted model for evaluation purposes

```python
dtpredictions = dtcvModel.transform(test)
```

### Evaluate the model

```python
print('RMSE:', dtevaluator.evaluate(dtpredictions))
```

{% hint style="info" %}
**Note:** When you use the `CrossValidator` function to set up cross-validation of your models, the resulting model object will have all the runs included, but will only use the best model when you interact with the model object using other functions like `evaluate` or `transform`.
{% endhint %}

