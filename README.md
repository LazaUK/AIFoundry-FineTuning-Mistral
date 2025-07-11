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
- **Azure Subscription**: Your subscription must be **allowlisted** for MaaP preview.
- **Azure AI Project**: Make sure your project is created and accessible;
- **Managed Identity**: [Add a *User-Assigned Managed Identity* to your **AI Hub's workspace**](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-identity-based-service-authentication?view=azureml-api-2&tabs=cli#add-a-user-assigned-managed-identity-to-a-workspace-in-addition-to-a-system-assigned-identity), and provide it with the *Storage Blob Data Contributor* access to **AI Hub's storage account**;
- **Python Packages**: Install the necessary Python packages for interacting with Azure AI Foundry and Entra ID.
``` PowerShell
pip install azure-identity azure-ai-ml
```

> [!WARNING]
> Fine-tuning of non-Azure-OpenAI models on MaaP is currently in **Preview mode**. Your subscription must be allow-listed to access this functionality.

## Step 1: Environment Setup
Set up your environment variables to ensure the provided Jupyter notebook works correctly:

| Variable                   | Description                                 | Example Value                                  |
| :------------------------- | :------------------------------------------ | :--------------------------------------------- |
| `SUBSCRIPTION_ID`          | Your Azure subscription ID.                 | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`         |
| `RESOURCE_GROUP`           | Your Azure ML resource group name.          | `my-resource-group`                            |
| `WORKSPACE_NAME`           | Your Azure ML workspace name.               | `my-ai-workspace`                              |
| `MANAGED_IDENTITY_OBJECTID`| ObjectID of your AI Hub's managed identity. | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`         |

## Step 2: Job Payload Definition
The payload structure allows you to adjust relevant settings for your fine-tuning job, from hyper-parameters like learning rate or number of epochs to the SKU of the target compute instance.

``` Python
payload = {
    "properties": {
        "displayName": DISPLAY_NAME,
        "experimentName": EXPERIMENT_NAME,
        "identity": {
            "identityType": "Managed",
            "objectId": MANAGED_IDENTITY_OBJECTID
        },
        "fineTuningDetails": {
            "hyperParameters": {
                "learning_rate": 5e-6,
                "num_train_epochs": 1,
                "per_device_train_batch_size": 1
            },
            "model": {
                "jobInputType": "mlflow_model",
                "mode": "ReadOnlyMount",
                "uri": MODEL_NAME
            },
            "modelProvider": "Custom",
            "taskType": TASK_TYPE,
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
                GPU, CPU
            ]
        },
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
If successful, you should see output similar to this:
``` JSON
Status Code: 201
Response JSON: {'id': '/subscriptions/xxxxxxxxxxxxx/resourceGroups/xxxxxxxxxxx/providers/Microsoft.MachineLearningServices/workspaces/xxxxxxxxxxx/jobs/mistral-finetuning-job', 'name': 'mistral-finetuning-job', 'type': 'Microsoft.MachineLearningServices/workspaces/jobs', 'properties': {'description': None, 'tags': {}, 'properties': {'PipelineType': 'FineTuning', 'original_model_id': 'azureml://registries/azureml/models/mistralai-Mistral-7B-v01/versions/19', 'azureml.ModelName': 'mistralai-Mistral-7B-v01', 'azureml.PipelineType': 'FineTuning', 'azureml.original_model_id': 'azureml://registries/azureml/models/mistralai-Mistral-7B-v01/versions/19',
.....
'queueSettings': None, 'outputs': {'registered_model': {'description': None, 'uri': None, 'assetName': 'Mistral-7B-v01-Finetune', 'mode': 'ReadWriteMount', 'jobOutputType': 'mlflow_model'}}}, 'systemData': {'createdAt': '2025-05-21T00:20:48.3840162+00:00', 'createdBy': 'Laziz Turakulov', 'createdByType': 'User'}}
```

## Step 4: Monitoring Job Status
You can monitor the job in the Azure AI Foundry portal or poll the job status using the REST API.
![Mistral7b_FT_JobProgress](images/Mistral_FT_JobProgress.png)

## Step 5: Endpoint Deployment (Optional)
Once the job completes successfully, you can register and deploy the model using Azure AI Foundry or the REST API.
