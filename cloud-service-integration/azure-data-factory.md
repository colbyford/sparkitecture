# Azure Data Factory

## Transformation with Azure Databricks

Using Azure Databricks with Azure Data Factory, notebooks can be run from an end-to-end pipeline that contains the **Validation**, **Copy data**, and **Notebook** activities in Azure Data Factory.

* **Validation** ensures that your source dataset is ready for downstream consumption before you trigger the copy and analytics job.
* **Copy data** duplicates the source dataset to the sink storage, which is mounted as DBFS in the Azure Databricks notebook. In this way, the dataset can be directly consumed by Spark.
* **Notebook** triggers the Databricks notebook that transforms the dataset. It also adds the dataset to a processed folder or Azure SQL Data Warehouse.

### Import a notebook for Transformation

To import a **Transformation** notebook to your Databricks workspace:

1. Sign in to your Azure Databricks workspace, and then select **Import**.  Your workspace path can be different from the one shown, but remember it for later.
2. Select **Import from: URL**. In the text box, enter `https://adflabstaging1.blob.core.windows.net/share/Transformations.html`.
3. Now let's update the **Transformation** notebook with your storage connection information.

   In the imported notebook, go to **command 5** as shown in the following code snippet.

   * Replace with your own storage connection information.
   * Use the storage account with the `sinkdata` container.

   ```text
   ## Supply storageName and accessKey values  
   storageName = ""  
   accessKey = ""  

   ## Attempt to Mount Data Factory Data in Azure Storage
   dbutils.fs.mount(
       source = "wasbs://sinkdata\@"+storageName+".blob.core.windows.net/",  
       mount_point = "/mnt/Data Factorydata",  
       extra_configs = {"fs.azure.account.key."+storageName+".blob.core.windows.net": accessKey})  
   ```

4. Generate a **Databricks access token** for Data Factory to access Databricks.

   1. In your Databricks workspace, select your user profile icon in the upper right.
   2. Select **User Settings**. 
   3. Select **Generate New Token** under the **Access Tokens** tab.
   4. Select **Generate**.

   _Save the access token_ for later use in creating a Databricks linked service. The access token looks something like `dapi32db32cbb4w6eee18b7d87e45exxxxxx`.

### How to use this template

1. Go to the **Transformation with Azure Databricks** template and create new linked services for following connections.
   * **Source Blob Connection** - to access the source data.

     For this exercise, you can use the public blob storage that contains the source files. Reference the following screenshot for the configuration. Use the following **SAS URL** to connect to source storage \(read-only access\):

     `https://storagewithdata.blob.core.windows.net/data?sv=2018-03-28&si=read%20and%20list&sr=c&sig=PuyyS6%2FKdB2JxcZN0kPlmHSBlD8uIKyzhBWmWzznkBw%3D`

   * **Destination Blob Connection** - to store the copied data.

     In the **New linked service** window, select your sink storage blob.

   * **Azure Databricks** - to connect to the Databricks cluster.

     Create a Databricks-linked service by using the access key that you generated previously. You can opt to select an _interactive cluster_ if you have one. This example uses the **New job cluster** option.
2. Select **Use this template**. You'll see a pipeline created.

### Pipeline introduction and configuration

In the new pipeline, most settings are configured automatically with default values. Review the configurations of your pipeline and make any necessary changes.

1. In the **Validation** activity **Availability flag**, verify that the source **Dataset** value is set to `SourceAvailabilityDataset` that you created earlier.
2. In the **Copy data** activity **file-to-blob**, check the **Source** and **Sink** tabs. Change settings if necessary.
   * **Source** tab 
   * **Sink** tab 
3. In the **Notebook** activity **Transformation**, review and update the paths and settings as needed.

   **Databricks linked service** should be pre-populated with the value from a previous step, as shown: 

   To check the **Notebook** settings:

   1. Select the **Settings** tab. For **Notebook path**, verify that the default path is correct. You might need to browse and choose the correct notebook path.
   2. Expand the **Base Parameters** selector and verify that the parameters match what is shown in the following screenshot. These parameters are passed to the Databricks notebook from Data Factory.

4. Verify that the **Pipeline Parameters** match what is shown in the following screenshot: 
5. Connect to your datasets.

{% hint style="info" %}
In below datasets, the file path has been automatically specified in the template. If any changes required, make sure that you specify the path for both container and directory in case any connection error.
{% endhint %}

* **SourceAvailabilityDataset** - to check that the source data is available.
* **SourceFilesDataset** - to access the source data.
* **DestinationFilesDataset** - to copy the data into the sink destination location. Use the following values:
  * **Linked service** - `sinkBlob_LS`, created in a previous step.
  * **File path** - `sinkdata/staged_sink`.

1. Select **Debug** to run the pipeline. You can find the link to Databricks logs for more detailed Spark logs.

   You can also verify the data file by using Azure Storage Explorer.

{% hint style="info" %}
For correlating with Data Factory pipeline runs, this example appends the pipeline run ID from the data factory to the output folder. This helps keep track of files generated by each run. 
{% endhint %}

### Next steps

* [Introduction to Azure Data Factory](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/data-factory/introduction.md)
* [Transformation with Azure Databricks](https://docs.microsoft.com/en-us/azure/data-factory/solution-template-databricks-notebook)
* [Run a Databricks notebook with the Databricks Notebook Activity in Azure Data Factory](https://docs.microsoft.com/en-us/azure/data-factory/transform-data-using-databricks-notebook)

