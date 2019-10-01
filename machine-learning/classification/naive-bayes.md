# Naïve Bayes

## Setting Up a Naïve Bayes Classifier

{% hint style="info" %}
**Note:** Make sure you have your training and test data already vectorized and ready to go before you begin trying to fit the machine learning model to unprepped data.
{% endhint %}

### Load in required libraries

```python
from pyspark.ml.classification import NaiveBayes
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator
from pyspark.ml.evaluation import BinaryClassificationEvaluator
from pyspark.mllib.evaluation import BinaryClassificationMetrics


# cvModel uses the best model found from the Cross Validation
# Evaluate best model
print('Accuracy:', nbevaluator.evaluate(nbpredictions))
print('AUC:', BinaryClassificationMetrics(nbpredictions['label','prediction'].rdd).areaUnderROC)
```

### Initialize Naïve Bayes object

```python
nb = NaiveBayes(labelCol="label", featuresCol="features")
```

### Create a parameter grid for tuning the model

```python
nbparamGrid = (ParamGridBuilder()
               .addGrid(nb.smoothing, [0.0, 0.2, 0.4, 0.6, 0.8, 1.0])
               .build())
```

### Define how you want the model to be evaluated

```python
nbevaluator = BinaryClassificationEvaluator(rawPredictionCol="rawPrediction")
```

### Define the type of cross-validation you want to perform

```python
# Create 5-fold CrossValidator
nbcv = CrossValidator(estimator = nb,
                      estimatorParamMaps = nbparamGrid,
                      evaluator = nbevaluator,
                      numFolds = 5)
```

### Fit the model to the data

```python
nbcvModel = nbcv.fit(train)
print(nbcvModel)
```

### Score the testing dataset using your fitted model for evaluation purposes

```python
nbpredictions = nbcvModel.transform(test)
```

### Evaluate the model

```python
print('Accuracy:', nbevaluator.evaluate(nbpredictions))
print('AUC:', BinaryClassificationMetrics(nbpredictions['label','prediction'].rdd).areaUnderROC)
```

{% hint style="info" %}
**Note:** When you use the `CrossValidator`function to set up cross-validation of your models, the resulting model object will have all the runs included, but will only use the best model when you interact with the model object using other functions like `evaluate` or `transform`.
{% endhint %}

