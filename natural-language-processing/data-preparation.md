# Text Data Preparation

## Tokenization and Vectorization

### Load in required libraries

```python
from pyspark.ml.feature import RegexTokenizer, StopWordsRemover, CountVectorizer
```

### Remove usernames, dates, links, etc.

```python

def clean_text(c):
  c = lower(c)
  c = regexp_replace(c, "(https?\://)\S+", "") # Remove links
  c = regexp_replace(c, "(\\n)|\n|\r|\t", "") # Remove CR, tab, and LR
  c = regexp_replace(c, "(?:(?:[0-9]{2}[:\/,]){2}[0-9]{2,4})", "") # Remove dates
  c = regexp_replace(c, "@([A-Za-z0-9_]+)", "") # Remove usernames
  c = regexp_replace(c, "[0-9]", "") # Remove numbers
  c = regexp_replace(c, "\:|\/|\#|\.|\?|\!|\&|\"|\,", "") # Remove symbols
  #c = regexp_replace(c, "(@[A-Za-z0-9_]+)|([^0-9A-Za-z \t])|(\w+:\/\/\S+)", "")
  return c

dataset = dataset.withColumn("text", clean_text(col("text")))
```

### RegEx tokenization

```python
regexTokenizer = RegexTokenizer(inputCol="text", outputCol="words", pattern="\\W")
```

### Remove stop words

```python
# Add Stop words
add_stopwords = ["http","https","amp","rt","t","c","the","@","/",":"] # standard web stop words

stopwordsRemover = StopWordsRemover(inputCol="words", outputCol="filtered").setStopWords(add_stopwords)
```

### Count words

```python
# Bag of Words Count
countVectors = CountVectorizer(inputCol="filtered", outputCol="features", vocabSize=10000, minDF=5)
```

### Index strings

```python
# String Indexer
from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorAssembler
label_stringIdx = StringIndexer(inputCol = "class", outputCol = "label")
```

### Create transformation pipeline

```python
from pyspark.ml import Pipeline

pipeline = Pipeline(stages=[regexTokenizer, stopwordsRemover, countVectors, label_stringIdx])

# Fit the pipeline to training documents.
pipelineFit = pipeline.fit(data)
dataset = pipelineFit.transform(data)
```

{% hint style="info" %}
Once the transformation pipeline has been fit, you can use normal [classification algorithms](../machine-learning/classification/) for classifying the text.
{% endhint %}

## Extras

### Get label numbers for each class

```python
from pyspark.sql import *
from pyspark.sql.functions import col
labelset = dataset.select(col("class"),
                          col("label")).distinct()
display(labelset)
```

### Split text body into sentences

```python
from pyspark.sql.types import *
from pyspark.sql.window import *
from pyspark.sql.functions import col, split, explode, row_number
# Split text by sentence and convert to array
array_df = data.withColumn("text", split(col("text"), "\.").cast("array<string>"))
  
# Explode array into separate rows in the dataset
split_df = array_df.withColumn("text", explode(col("text")))\
                   .withColumn("part_number", row_number().over(Window.partitionBy("internet_message_id").orderBy("id")))
data = split_df
display(data)
```

#### Create \`part\_number\` for the split sentences

```python
from pyspark.sql.window import *
from pyspark.sql.functions import row_number

data.withColumn("part_number", row_number().over(Window.partitionBy("body_id").orderBy("id"))).show()
```

