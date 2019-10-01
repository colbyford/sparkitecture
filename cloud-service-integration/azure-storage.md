# Azure Storage

{% hint style="danger" %}
Databricks-Specific Content
{% endhint %}

## Mounting Blob Storage

Once you create your blob storage account in Azure, you will need to grab a couple bits of information from the Azure Portal before you mount your storage.



Once you have the required bits of information, you can use the following code to mount the storage location inside the Databricks environment

```python
dbutils.fs.mount(
  source = "wasbs://<CONTAINERNAME>@<STORAGEACCOUNT>.blob.core.windows.net",
  mount_point = "/mnt/<FOLDERNAME>/",
  extra_configs = {"fs.azure.account.key.<STORAGEACCOUNT>.blob.core.windows.net":"<KEYGOESHERE>"})
```

### Resources:

* To learn how to create an Azure Storage service, visit [https://docs.microsoft.com/en-us/azure/storage/](https://docs.microsoft.com/en-us/azure/storage/)

