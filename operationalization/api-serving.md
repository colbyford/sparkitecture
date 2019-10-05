# API Serving

## Use MMLSpark

### Load in required libraries

```python
from pyspark.ml.tuning import CrossValidatorModel
from pyspark.ml import PipelineModel
from pyspark.sql.types import IntegerType
from pyspark.sql.functions import col, round

import sys
import numpy as np
import pandas as pd
import mmlspark
from pyspark.sql.functions import col, from_json
from pyspark.sql.types import *
import uuid
from mmlspark import request_to_string, string_to_response
```

### Load in transformation pipeline and trained model

```text
## Load in the transformation pipeline
mypipeline = PipelineModel.load("/mnt/trainedmodels/pipeline/")

## Load in trained model
mymodel = CrossValidatorModel.load("/mnt/trainedmodels/lr")
```

### Define username, key, and IP address

```text
username = "admin"
ip = "10.0.0.4" #Internal IP
sas_url = "" # SAS Token for your VM's Private Key in Blob
```

### Define input schema

```text
input_schema = StructType([
  StructField("id", IntegerType(), True),
  StructField("x1", IntegerType(), True),
  StructField("x2", DoubleType(), True),
  StructField("x3", StringType(), True),
 ])
```

### Set up streaming DataFrame

```python
serving_inputs = spark.readStream.continuousServer() \
                      .option("numPartitions", 1) \
                      .option("name", "http://10.0.0.4:8898/my_api") \
                      .option("forwarding.enabled", True) \
                      .option("forwarding.username", username) \
                      .option("forwarding.sshHost", ip) \
                      .option("forwarding.keySas", sas_url) \
                      .address("localhost", 8898, "my_api") \
                      .load()\
                      .parseRequest(input_schema)

mydataset = mypipeline.transform(serving_inputs)

serving_outputs = mymodel.bestModel.transform(mydataset) \
  .makeReply("prediction")

# display(serving_inputs)

```

### Set up server

```python
server = serving_outputs.writeStream \
    .continuousServer() \
    .trigger(continuous="1 second") \
    .replyTo("my_api") \
    .queryName("my_query") \
    .option("checkpointLocation", "file:///tmp/checkpoints-{}".format(uuid.uuid1())) \
    .start()
```

### Test the webservice

```text
import requests
data = u'{"id":0,"x1":1,"x2":2.0,"x3":"3"}'

#r = requests.post(data=data, url="http://localhost:8898/my_api") # Locally
r = requests.post(data=data, url="http://102.208.216.32:8902/my_api") # Via the VM IP

print("Response {}".format(r.text))
```

{% hint style="warning" %}
You may need to run `sudo netstat -tulpn` if you're running inside Databricks.

Use this command to look for the port that was opened by the server.
{% endhint %}

### Resources:

MMLSpark on GitHub: [https://github.com/Azure/mmlspark](https://github.com/Azure/mmlspark)

