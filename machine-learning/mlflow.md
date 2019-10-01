# MLflow

{% hint style="danger" %}
Databricks-Specific Functionality
{% endhint %}

Track metrics and parameters

```python
import mlflow

## Log Parameters and Metrics from your normal MLlib run
with mlflow.start_run():
  # Log a parameter (key-value pair)
  mlflow.log_param("alpha", 0.1)

  # Log a metric; metrics can be updated throughout the run
  mlflow.log_metric("AUC", 0.871827)
  mlflow.log_metric("F1", 0.726153)
  mlflow.log_metric("Precision", 0.213873)
```

