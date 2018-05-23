## Using Resource Manager REST API to perform some of the HDInsight actions

### Authenticate 

POST https://login.microsoftonline.com/{tenant-id}/oauth2/token

Content-Type: application/x-www-form-urlencoded

In the Body provide the following as x-www-form-urlencoded

| Key | Value
|---- |----
| grant_type | client_credentials
| client_id | service_principal_id
| resource | https://management.azure.com/
| client_secret | service_principal_key

### How to resize cluster using Resource Manager REST API

**STEP 1**:

POST 

https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.HDInsight/clusters/{clusterName}/roles/workernode/resize?api-version=2015-03-01-preview

Authorization header should be "Bearer {value of access_token from previous **Authenticate** }"

This would return a 202.

**STEP 2**:

GET {url from Azure-AsyncOperation header value of response from previous STEP 1 }

Authorization header should be "Bearer {value of access_token from previous **Authenticate**}"

You would get the following response and thus know the progress.

{
    "status": "InProgress"
}

### How to execute action script using Resource Manager REST API

**STEP 1**

POST

https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.HDInsight/clusters/{clusterName}/executeScriptActions?api-version=2015-03-01-preview

Authorization header should be "Bearer {value of access_token from previous **Authenticate** }"

Content-Type: application/json

Body should follow the following json schema

```json
{  
  "scriptActions": [  
    {  
      "name": "ScaleUpCluster",  
      "uri": "https://{storageContainerName}.blob.core.windows.net/cchiq-dataplane/ScaleUpCluster.bash",  
      "roles": [  
        "headnode"
      ]  
    }
  ],  
  "persistOnSuccess": false  
}
```
