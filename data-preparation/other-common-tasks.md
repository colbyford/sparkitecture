# Other Common Tasks

### Split Data into Training and Test Datasets

```text
train, test = dataset.randomSplit([0.75, 0.25], seed = 1337)
```

## Rename all columns

```python
column_list = data.columns
prefix = "my_prefix"
new_column_list = [prefix + s for s in column_list]
#new_column_list = [prefix + s if s != "ID" else s for s in column_list] ## Use if you plan on joining on an ID later
 
column_mapping = [[o, n] for o, n in zip(column_list, new_column_list)]

# print(column_mapping)

# data = data.select(list(map(lambda old, new: col(old).alias(new),*zip(*column_mapping))))
```

## Convert PySpark DataFrame to NumPy array

```text
## Convert `train` DataFrame to NumPy
pdtrain = train.toPandas()
trainseries = pdtrain['features'].apply(lambda x : np.array(x.toArray())).as_matrix().reshape(-1,1)
X_train = np.apply_along_axis(lambda x : x[0], 1, trainseries)
y_train = pdtrain['label'].values.reshape(-1,1).ravel()

## Convert `test` DataFrame to NumPy
pdtest = test.toPandas()
testseries = pdtest['features'].apply(lambda x : np.array(x.toArray())).as_matrix().reshape(-1,1)
X_test = np.apply_along_axis(lambda x : x[0], 1, testseries)
y_test = pdtest['label'].values.reshape(-1,1).ravel()

print(y_test)
```

## Call Cognitive Service API using PySpark

### Create \`chunker\` function

The cognitive service APIs can only take a limited number of observations at a time \(1,000, to be exact\) or a limited amount of data in a single call. So, we can create a `chunker` function that we will use to split the dataset up into smaller chunks.

```python
## Define Chunking Logic
import pandas as pd
import numpy as np
# Based on: https://stackoverflow.com/questions/25699439/how-to-iterate-over-consecutive-chunks-of-pandas-dataframe-efficiently
def chunker(seq, size):
    return (seq[pos:pos + size] for pos in range(0, len(seq), size))
```

### Convert Spark DataFrame to Pandas

```python
## sentiment_df_pd = sentiment_df.toPandas()
```

### Set up API requirements

```python
# pprint is used to format the JSON response
from pprint import pprint
import json
import requests

subscription_key = '<SUBSCRIPTIONKEY>'
endpoint = 'https://<SERVICENAME>.cognitiveservices.azure.com'
sentiment_url = endpoint + "/text/analytics/v2.1/sentiment"
headers = {"Ocp-Apim-Subscription-Key": subscription_key}
```

### Create DataFrame for incoming scored data

```python
from pyspark.sql.types import *

sentiment_schema = StructType([StructField("id", IntegerType(), True),
                               StructField("score", FloatType(), True)])

sentiments_df = spark.createDataFrame([], sentiment_schema)

display(sentiments_df)
```

### Loop through chunks of the data and call the API

```python
for chunk in chunker(sentiment_df_pd, 1000):
  print("Scoring", len(chunk), "rows.")
  sentiment_df_json = json.loads('{"documents":' + chunk.to_json(orient='records') + '}')
  
  response = requests.post(sentiment_url, headers = headers, json = sentiment_df_json)
  sentiments = response.json()
  # pprint(sentiments)
  
  sentiments_pd = pd.read_json(json.dumps(sentiments['documents']))
  sentiments_df_chunk = spark.createDataFrame(sentiments_pd)
  sentiments_df = sentiments_df.unionAll(sentiments_df_chunk)
  
display(sentiments_df)
sentiments_df.count()
```

### Write the results out to mounted storage

```python
sentiments_df.coalesce(1).write.csv("/mnt/textanalytics/sentimentanalysis/")
```

## Find All Columns of a Certain Type

```python
import pandas as pd
def get_nonstring_cols(df):
    types = spark.createDataFrame(pd.DataFrame({'Column': df.schema.names, 'Type': [str(f.dataType) for f in df.schema.fields]}))
    result = types.filter(col('Type') != 'StringType').select('Column').rdd.flatMap(lambda x: x).collect()
    return result
    
get_nonstring_cols(df)
```

## Change a Column's Type

```python
from pyspark.sql.types import *
from pyspark.sql.functions import col

df = df..withColumn('col1', col('col1').cast(IntegerType()))
```

## Generate StructType Schema Printout \(Manual Execution\)

```python
## Fill in list with your desired column names
cols = ["col1", "col2", "col3"]
i = 1

for col in cols:
    if i == 1:
        print("schema = StructType([")
        print("\tStructField('" + col +  "', StringType(), True),")
    
    elif i == len(cols):
        print("\tStructField('" + col +  "', StringType(), True)])")
        
    else:
        print("\tStructField('" + col +  "', StringType(), True),")
    
    i += 1
    
## Once the output has printed, copy and paste into a new cell
## and change column types and nullability
```

## Generate StructType Schema from List \(Automatic Execution\)

```python
"""
Struct Schema Creator for PySpark

[<Column Name>, <Column Type>, <Column Nullable>]

Types:  binary, boolean, byte, date,
        double, integer, long, null,
        short, string, timestamp, unknown
"""
from pyspark.sql.types import *

## Fill in with your desired column names, types, and nullability
cols = [["col1", "string", False],
        ["col2", "date", True],
        ["col3", "integer", True]]

## Loop to build list of StructFields
schema_set = ["schema = StructType(["]

for i, col in enumerate(cols):
    colname = col[0]
    coltype = col[1].title() + "Type()"
    colnull = col[2]
    
    if i == len(cols)-1:
        iter_structfield = "StructField('" + colname +  "', " + coltype + ", " + str(colnull) + ")])"
    else:
        iter_structfield = "StructField('" + colname +  "', " + coltype + ", " + str(colnull) + "),"
    
    schema_set.append(iter_structfield)

## Convert list to single string
schema_string = ''.join(map(str, schema_set))

## This will execute the generated command string
exec(schema_string)
```

