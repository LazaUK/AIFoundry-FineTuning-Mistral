# Fine-Tuning Mistral-7B in Azure AI Foundry

This guide walks you through the process of fine-tuning a model using **Azure AI Foundry**. It covers setting up prerequisites, configuring your environment, submitting a fine-tuning job using Managed Identity, and deploying the model.

> [!NOTE]
> The Python code and training datasets in this repo are adapted from Microsoft's [Azure Machine Learning examples](https://github.com/Azure/azureml-examples/tree/main/sdk/python/jobs/finetuning) repo.

## 📑 Table of Contents
- [Prerequisites](#-prerequisites)
- [Environment Setup](#-environment-setup)
- [Job Payload Definition](#-job-payload-definition)
- [Job Submission](#-job-submission)
- [Monitoring Job Status](#-monitoring-job-status)
- [Endpoint Deployment](#-endpoint-deployment-optional)

## ✅ Prerequisites
Before you begin, ensure you have the following:
- **Azure Subscription**: Your subscription must be allowlisted for MaaP (Model-as-a-Platform) preview.
- **Azure AI Project**: Make sure your project is created and accessible.
- **Managed Identity**:
    * Create a **User-Assigned Managed Identity**.
    * Assign the **Storage Blob Data Contributor** role to this identity on the **hub storage account**.
    * Attach the identity to your Azure AI Project.
- **Python Packages**: Ensure you have the necessary Python packages installed for interacting with Azure AI Foundry. (Specific package names would be added here if provided in the original context.)

## ⚙️ Environment Setup
Set up your environment variables to make provided Jupyter notebook work:

| Variable                  | Description                                      |
| ------------------------- | ------------------------------------------------ |
| `subscription_id`         | Azure subscription ID.                           |
| `resource_group`          | Azure ML resource group name.                    |
| `workspace_name`          | Azure ML workspace name.                         |
| `model_registry`          | Azure ML model registry name.                    |

## 🧱 Job Payload Definition
(Details on how to define the job payload would be added here if provided in the original context.)

## 🚀 Job Submission
(Details on how to submit the job would be added here if provided in the original context.)

## 📊 Monitoring Job Status
You can monitor the job in the Azure AI Studio portal or poll the job status using the REST API.

## 🌐 Endpoint Deployment (Optional)
Once the job completes successfully, you can register and deploy the model using Azure AI Studio or the REST API. Please note that deployment steps are not yet fully supported via Foundry REST for all models.
