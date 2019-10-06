# Reading and Writing Data

## Reading in Data

### ...from Mounted Storage

```python
dataset = sqlContext.read.format('csv')
                    .options(header='true', inferSchema='true', delimiter= ',') \
                    .load('/mnt/<FOLDERNAME>/<FILENAME>.csv')
```

