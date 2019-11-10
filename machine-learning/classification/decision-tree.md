# Decision Tree

## Setting Up a Decision Tree Classifier

{% hint style="info" %}
**Note:** Make sure you have your training and test data already vectorized and ready to go before you begin trying to fit the machine learning model to unprepped data.
{% endhint %}

### Load in required libraries

```python
from pyspark.ml.classification import DecisionTreeClassifier
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator
from pyspark.ml.evaluation import BinaryClassificationEvaluator
```

### Initialize Decision Tree object

```python
dt = DecisionTreeClassifier(labelCol="label", featuresCol="features")
```

### Create a parameter grid for tuning the model

```python
dtparamGrid = (ParamGridBuilder()
             .addGrid(dt.maxDepth, [2, 5, 10])
             .addGrid(dt.maxBins, [10, 20])
             .build())
```

### Define how you want the model to be evaluated

```python
dtevaluator = BinaryClassificationEvaluator(rawPredictionCol="rawPrediction")
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
print('Accuracy:', dtevaluator.evaluate(dtpredictions))
print('AUC:', BinaryClassificationMetrics(dtpredictions['label','prediction'].rdd).areaUnderROC)
print('PR:', BinaryClassificationMetrics(dtpredictions['label','prediction'].rdd).areaUnderPR)
```

{% hint style="info" %}
**Note:** When you use the `CrossValidator` function to set up cross-validation of your models, the resulting model object will have all the runs included, but will only use the best model when you interact with the model object using other functions like `evaluate` or `transform`.
{% endhint %}

