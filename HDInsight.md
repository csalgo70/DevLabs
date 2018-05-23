### Authenticate 

**STEP 1**: 

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


**STEP 2**:

POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.HDInsight/clusters/{clusterName}/roles/workernode/resize?api-version=2015-03-01-preview

Authorization header should be "Bearer {value of access_token from previous STEP 1}"

This would return a 202.

**STEP 3**:

GET {url from Azure-AsyncOperation header value of response from previous STEP 2 }

Authorization header should be "Bearer {value of access_token from previous STEP 1}"

You would get the following response and thus know the progress.

{
    "status": "InProgress"
}

