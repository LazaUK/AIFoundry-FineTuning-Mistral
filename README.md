# Fine-Tuning Mistral-7B in Azure AI Foundry

This guide walks you through the process of fine-tuning Mistral 7B AI model in **Azure AI Foundry**. It covers steps of checking prerequisites, configuring your environment and submitting a fine-tuning job on MaaP (Model-as-a-Platform) using Managed Identity.

> [!NOTE]
> The Python code and training datasets in this repo are adapted from Microsoft's [Azure Machine Learning examples](https://github.com/Azure/azureml-examples/tree/main/sdk/python/jobs/finetuning) repo.

## 📑 Table of Contents
- [Prerequisites](#prerequisites)
- [Step 1: Environment Setup](#step-1-environment-setup)
- [Step 2: Job Payload Definition](#step-2-job-payload-definition)
- [Step 3: Job Submission](#step-3-job-submission)
- [Step 4: Monitoring Job Status](#step-4-monitoring-job-status)
- [Step 5: Endpoint Deployment](#step-5-endpoint-deployment-optional)

## Prerequisites
Before you begin, ensure you have the following:
- **Azure AI Project**: Make sure your project is created and accessible;
- **Managed Identity**: If using a *User-Assigned Managed Identity*, ensure that it has the *Storage Blob Data Contributor* RBAC role assigned to access the **AI Hub's storage account**;
- **Python Packages**: Install the necessary Python packages for interacting with Azure AI Foundry and Entra ID.
``` PowerShell
pip install azure-identity azure-ai-ml
```

> [!WARNING]
> Fine-tuning of non-Azure-OpenAI models on MaaP is currently in Preview mode. Your subscription must be allow-listed to access this functionality.

## Step 1: Environment Setup
Set up your environment variables to make provided Jupyter notebook work:

| Variable                     | Description                                      |
| ---------------------------- | ------------------------------------------------ |
| `SUBSCRIPTION_ID`            | Azure subscription ID.                           |
| `RESOURCE_GROUP`             | Azure ML resource group name.                    |
| `WORKSPACE_NAME`             | Azure ML workspace name.                         |
| `MANAGED_IDENTITY_OBJECTID`  | ObjectID of AI Hub's managed identity.           |

## Step 2: Job Payload Definition
The payload structure allows you to adjust relevant settings of finetuning job: from hyper-parameters like learning rate or number of epochs to SKU of the SKU of the target compute instance.

``` Python
payload = {
    "properties": {
        "displayName": JOB_NAME,
        "experimentName": EXPERIMENT_NAME,
        "identity": {
            "identityType": "Managed",
            "objectId": MANAGED_IDENTITY_OBJECTID
        },
        "fineTuningDetails": {
            "hyperParameters": {
                "learning_rate": 0.00002,
                "num_train_epochs": 1,
                "per_device_train_batch_size": 1
            },
            "model": {
                "jobInputType": "mlflow_model",
                "mode": "ReadOnlyMount",
                "uri": MODEL_NAME
            },
            "modelProvider": "Custom",
            "taskType": "TextCompletion",
            "trainingData": {
                "jobInputType": "uri_file",
                "mode": "ReadOnlyMount",
                "uri": data_asset_id
            }
        },
        "jobType": "FineTuning",
        "outputs": {
            "registered_model": {
                "assetname": FT_MODEL_NAME,
                "jobOutputType": "mlflow_model"
            }
        },
        "resources": {
            "instanceTypes": [
                INSTANCE_SKU
            ]
        }
    }
}
```

## Step 3: Job Submission
Once you authenticate and retrieve the Entra ID token, you fine-tuning job can be submitted through the REST API:
``` Python
response = requests.put(
    endpoint,
    headers = headers,
    json = payload
)

print("Status Code:", response.status_code)

try:
    print("Response JSON:", response.json())
except Exception:
    print("Response Text:", response.text)
```
If successful, you should see an update in your notebook cell, similar to this:
``` Python
Status Code: 201
Response JSON: {'id': '/subscriptions/xxxxxxxxxxxxx/resourceGroups/xxxxxxxxxxx/providers/Microsoft.MachineLearningServices/workspaces/xxxxxxxxxxx/jobs/mistral-finetuning-job', 'name': 'mistral-finetuning-job', 'type': 'Microsoft.MachineLearningServices/workspaces/jobs', 'properties': {'description': None, 'tags': {}, 'properties': {'PipelineType': 'FineTuning', 'original_model_id': 'azureml://registries/azureml/models/mistralai-Mistral-7B-v01/versions/19', 'azureml.ModelName': 'mistralai-Mistral-7B-v01', 'azureml.PipelineType': 'FineTuning', 'azureml.original_model_id': 'azureml://registries/azureml/models/mistralai-Mistral-7B-v01/versions/19',
.....
'queueSettings': None, 'outputs': {'registered_model': {'description': None, 'uri': None, 'assetName': 'Mistral-7B-v01-Finetune', 'mode': 'ReadWriteMount', 'jobOutputType': 'mlflow_model'}}}, 'systemData': {'createdAt': '2025-05-21T00:20:48.3840162+00:00', 'createdBy': 'Laziz Turakulov', 'createdByType': 'User'}}
```

## Step 4: Monitoring Job Status
You can monitor the job in the Azure AI Studio portal or poll the job status using the REST API.
![Mistral_FT_JobProgress](images/Mistral_FT_JobProgress.png)

## Step 5: Endpoint Deployment (Optional)
Once the job completes successfully, you can register and deploy the model using Azure AI Studio or the REST API. Please note that deployment steps are not yet fully supported via Foundry REST for all models.
