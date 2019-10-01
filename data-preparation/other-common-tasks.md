# Other Common Tasks

### Rename all columns

```python
column_list = data.columns
prefix = "my_prefix"
new_column_list = [prefix + s for s in column_list]
#new_column_list = [prefix + s if s != "ID" else s for s in column_list] ## Use if you plan on joining on an ID later
 
column_mapping = [[o, n] for o, n in zip(column_list, new_column_list)]

# print(column_mapping)

# data = data.select(list(map(lambda old, new: col(old).alias(new),*zip(*column_mapping))))
```

### Convert PySpark DataFrame to NumPy array

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

