# Model Evaluation

### Evaluate multiclass classification models

```python
from pyspark.ml.evaluation import MulticlassClassificationEvaluator
from pyspark.mllib.evaluation import MulticlassMetrics

# Evaluate best model
print('Accuracy:', lrevaluator.evaluate(lrpredictions))
lrmetrics = MulticlassMetrics(lrpredictions['label','prediction'].rdd)
print('Confusion Matrix:\n', lrmetrics.confusionMatrix())
print('F1 Score:', lrmetrics.fMeasure(1.0,1.0))
```

### Evaluate binary classification models

```python
for model in ["lrpredictions", "dtpredictions", "rfpredictions", "nbpredictions", "gbpredictions"]:
    df = globals()[model]
    
    tp = df[(df.label == 1) & (df.prediction == 1)].count()
    tn = df[(df.label == 0) & (df.prediction == 0)].count()
    fp = df[(df.label == 0) & (df.prediction == 1)].count()
    fn = df[(df.label == 1) & (df.prediction == 0)].count()
    a = ((tp + tn)/df.count())
    
    if(tp + fn == 0.0):
        r = 0.0
        p = float(tp) / (tp + fp)
    elif(tp + fp == 0.0):
        r = float(tp) / (tp + fn)
        p = 0.0
    else:
        r = float(tp) / (tp + fn)
        p = float(tp) / (tp + fp)
    
    if(p + r == 0):
        f1 = 0
    else:
        f1 = 2 * ((p * r)/(p + r))
    
    print("Model:", model)
    print("True Positives:", tp)
    print("True Negatives:", tn)
    print("False Positives:", fp)
    print("False Negatives:", fn)
    print("Total:", df.count())
    print("Accuracy:", a)
    print("Recall:", r)
    print("Precision: ", p)
    print("F1 score:", f1)
    print('AUC:', BinaryClassificationMetrics(df['label','prediction'].rdd).areaUnderROC)
print("\n")
```

