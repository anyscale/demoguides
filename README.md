# Anyscale on AKS Demo Guide

**Audience:** Microsoft Field Engineers
**Goal:** Provide guidance on how to demo Anyscale on AKS functionality

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Demo 1: Multi-modal Batch Inference](#demo-1-multi-modal-batch-inference)
  - [Preparation](#preparation)
  - [Demo Execution](#demo-execution)
- [Demo 2: Deploy LLMs](#demo-2-deploy-llms)
  - [Preparation](#preparation-1)
  - [Demo Execution](#demo-execution-1)
- [Tips for a Successful Demo](#tips-for-a-successful-demo)
- [Support](#support)

---

## Prerequisites

### Step 1: Request Access
Contact [ms-field-collab@anyscale.com](mailto:ms-field-collab@anyscale.com) for access credentials to the demo Anyscale organization.

### Step 2: Login
Navigate to [console.anyscale.com](https://console.anyscale.com) and sign in with your credentials.

![Anyscale Console Login](screenshots/01-console-login.png)

---

## Demo 1: Multi-modal Batch Inference

### Preparation

1. **Launch a workspace with Multi-Modal AI template**
   - From the Anyscale console, create a new workspace (Create from template)
   - Select the "Multi-Modal AI" template

   ![Multi-Modal AI Template Selection](screenshots/02-multimodal-template.png)
   - Launch the template

2. **Modify compute configuration**
   - **Terminate the workspace** (if already running)
   - Navigate to compute configuration settings

   ![Compute Configuration Settings](screenshots/03-compute-config.png)

   - **Change the head node:**
     - From: `2CPU-8GB`
     - To: `8CPU-32GB`
   - **Change the worker nodes:**
     - From: `Auto-select workers`
     - To: `4 x T4` GPUs

   ![Head and Worker Node Configuration](screenshots/04-node-selection.png)

3. **Modify the container image**
   - Select image "anyscale/ray:2.49.1-py312-cu128"
4. **Re-launch the workspace**
   - Start the Workspace with the new configuration
   - Observe Workspace logs for successful launch. Below is an example:
   ![Example Worskpace Logs](screenshots/04b-sample-cluster-logs.png)

### Demo Execution

- Navigate to the VSCode interface (not VSCode Desktop)
- Access notebooks/01-Batch-Inference.ipynb
- **[Optional]** Modify the Batch Inference notebook to use a shared storage mount
In the start of the section "Data ingestion" replace the code with a reference to S3 to:

```
# Load data.
ds = ray.data.read_images(
    "/mnt/shared_storage/doggos-dataset/train", 
    include_paths=True, 
    shuffle="files",
)
ds.take(1)
```
- Run through the notebook until the section "Monitoring and Debugging" in the notebook

---

## Demo 2: Deploy LLMs

### Preparation

1. **Launch a workspace with Deploy LLMs template**
   - From the Anyscale console, create a new workspace
   - Select the "Deploy LLMs" template

   ![Deploy LLMs Template Selection](screenshots/06-deploy-llms-template.png)

2. **Modify compute configuration**
   - **Terminate the workspace** (if already running)
   - Navigate to compute configuration settings
   - **Change the head node:**
     - From: `2CPU-8GB`
     - To: `8CPU-32GB`
   - **Change the worker nodes:**
     - From: `Auto-select workers`
     - To: `2 x L4` nodes

   ![L4 Node Configuration](screenshots/07-l4-nodes.png)

3. **Set up HuggingFace token**
   - Sign in to [HuggingFace](https://huggingface.co/) (create an account if required)
   - Navigate to Profile → Access Tokens
   - Create a new token with read permissions
   - Copy the token for the next step

   ![HuggingFace Token Creation](screenshots/08-huggingface-token.png)

4. **Configure environment variables**
   - In the Anyscale workspace settings, navigate to **Dependencies → Environment Variables**
   - Edit to Add the following environment variable:
     ```
     HF_TOKEN=<YOUR_HF_TOKEN>
     ```
   - Replace `<YOUR_HF_TOKEN>` with your actual HuggingFace token

   ![Environment Variables Configuration](screenshots/09-env-variables.png)

5. **Launch the workspace**
   - Start the workspace with the new configuration

### Demo Execution

- Follow the instructions in the notebook small-size-llm/notebook.ipynb

---

## Tips for a Successful Demo

- Ensure nodes are provisioned before the demo to avoid wait times
- Test the workflows in advance to familiarize yourself with the UI
- Prepare talking points about AKS integration benefits
- Have backup examples ready in case of any technical issues
- Emphasize scalability and Azure-native features

---

## Support

For questions or issues, contact [ms-field-collab@anyscale.com](mailto:ms-field-collab@anyscale.com)
