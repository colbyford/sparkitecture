# Azure SQL Data Warehouse / Synapse

### Set up Azure SQL DW connection parameters

```python
dwDatabase = "<DATABASENAME>" ## The Azure SQL Data Warehouse database name
dwServer = "<DWNAME>.database.windows.net" ## The Azure SQL Server
dwUser = "<USERNAME>" ## The dedicated loading user login 
dwPass = dbutils.secrets.get(scope = "<SECRETNAME>", key = "<KEYNAME>") ## The dediciated loading user login password
dwJdbcPort =  "1433" 
sqlDwUrlSmall = "jdbc:sqlserver://" + dwServer + ":" + dwJdbcPort + ";database=" + dwDatabase + ";user=" + dwUser+";password=" + dwPass
```

### Define a query 

```python
sqlQuery = """
  SELECT *, 'AzureSqlDw' AS SourceSystem
  FROM dbo.<TABLENAME>
"""
```

### Create a Spark DataFrame using the SQL DW data

```python
data = spark.read \
  .format("com.databricks.spark.sqldw") \
  .option("url", sqlDwUrlSmall) \
  .option("tempDir", tempDir) \
  .option("forwardSparkAzureStorageCredentials", "true") \
  .option("query", sqlQuery) \
  .load() \
  .createOrReplaceTempView("<TEMPVIEWNAME>")
  #.write.saveAsTable("<TABLENAME>")
```

