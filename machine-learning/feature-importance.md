# Feature Importance

### Extract important features using Gini

```python
## Based on: https://www.timlrx.com/2018/06/19/feature-selection-using-feature-importance-score-creating-a-pyspark-estimator/
import pandas as pd

def ExtractFeatureImportance(featureImp, dataset, featuresCol):
    list_extract = []
    for i in dataset.schema[featuresCol].metadata["ml_attr"]["attrs"]:
        list_extract = list_extract + dataset.schema[featuresCol].metadata["ml_attr"]["attrs"][i]
    varlist = pd.DataFrame(list_extract)
    varlist['score'] = varlist['idx'].apply(lambda x: featureImp[x])
    return(varlist.sort_values('score', ascending = False))
  
  
# ExtractFeatureImportance(model.stages[-1].featureImportances, dataset, "features")
dataset_fi = ExtractFeatureImportance(model.bestModel.featureImportances, dataset, "features")
dataset_fi = sqlContext.createDataFrame(dataset_fi)
display(dataset_fi)
```

### Extract important features using p-values

```python
## Based on: https://stackoverflow.com/questions/42935914/how-to-map-features-from-the-output-of-a-vectorassembler-back-to-the-column-name
lrm = model.stages[-1]
## Transform the data:
transformed =  model.transform(df)
#Extract and flatten ML attributes:
from itertools import chain

attrs = sorted(
    (attr["idx"], attr["name"]) for attr in (chain(*transformed
        .schema[lrm.summary.featuresCol]
        .metadata["ml_attr"]["attrs"].values())))
# and map to the output:

[(name, lrm.summary.pValues[idx]) for idx, name in attrs]
# [(name, lrm.coefficients[idx]) for idx, name in attrs]
```

### Extract coefficients from a model

```python
import pandas as pd

featurelist = pd.DataFrame(dataset.schema["features"].metadata["ml_attr"]["attrs"]["binary"]+dataset.schema["features"].metadata["ml_attr"]["attrs"]["numeric"]).sort_values("idx")
featurelist["Coefficient"] = pd.DataFrame(model.bestModel.coefficients.toArray())
featurelist = sqlContext.createDataFrame(featurelist)

display(featurelist)
```

