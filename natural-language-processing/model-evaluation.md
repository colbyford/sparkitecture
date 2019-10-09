# Model Evaluation

### Multiclass classification evaluator

```python
from pyspark.mllib.evaluation import MulticlassMetrics
evaluator = MulticlassClassificationEvaluator(predictionCol="prediction")

for model in ["lrpredictions", "nbpredictions", "rfpredictions"]:
    
    df = globals()[model]
    ########################################
    # Compute raw scores on the test set
    predictionAndLabels = df.select("prediction", "label").rdd

    # Instantiate metrics object
    metrics = MulticlassMetrics(predictionAndLabels)

    # Overall statistics
    precision = metrics.precision()
    recall = metrics.recall()
    f1Score = metrics.fMeasure()
    print("Summary Stats for: ", model)
    #print(metrics.confusionMatrix())
    print("Accuracy = %s" % evaluator.evaluate(df))
    print("Precision = %s" % precision)
    print("Recall = %s" % recall)
    print("F1 Score = %s" % f1Score)

    # Weighted stats
    #print("Weighted recall = %s" % metrics.weightedRecall)
    #print("Weighted precision = %s" % metrics.weightedPrecision)
    #print("Weighted F(1) Score = %s" % metrics.weightedFMeasure())
    #print("Weighted F(0.5) Score = %s" % metrics.weightedFMeasure(beta=0.5))
    #print("Weighted false positive rate = %s" % metrics.weightedFalsePositiveRate)
    print("\n")
```

