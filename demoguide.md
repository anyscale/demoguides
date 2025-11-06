# Anyscale on AKS Demo Guide

**Audience:** Microsoft Field Engineers
**Goal:** Provide guidance on how to demo Anyscale on AKS functionality

---

## Prerequisites

### Step 1: Request Access
Contact [omar@anyscale.com](mailto:omar@anyscale.com) for access to the demo Anyscale organization.

### Step 2: Login
Navigate to [console.anyscale.com](https://console.anyscale.com) and sign in with your credentials.

![Anyscale Console Login](screenshots/01-console-login.png)

---

## Demo 1: Multi-modal AI

### Preparation

1. **Launch a workspace with Multi-Modal AI template**
   - From the Anyscale console, create a new workspace
   - Select the "Multi-Modal AI" template

   ![Multi-Modal AI Template Selection](screenshots/02-multimodal-template.png)

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

3. **Acquire nodes**
   - Request node provisioning for the updated configuration

4. **Launch the workspace**
   - Start the workspace with the new configuration

### Demo Execution

- **[Optional]** Modify the Batch Inference notebook to use Azure Blob Storage instead of S3
  - This demonstrates cloud-native integration with Azure services
  - Update storage connection strings and authentication as needed

![Multi-Modal AI Workspace Running](screenshots/05-multimodal-workspace.png)

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
   - Navigate to Settings → Access Tokens
   - Create a new token with read permissions
   - Copy the token for the next step

   ![HuggingFace Token Creation](screenshots/08-huggingface-token.png)

4. **Configure environment variables**
   - In the Anyscale workspace settings, navigate to **Dependencies → Environment Variables**
   - Add the following environment variable:
     ```
     HF_TOKEN=<YOUR_HF_TOKEN>
     ```
   - Replace `<YOUR_HF_TOKEN>` with your actual HuggingFace token

   ![Environment Variables Configuration](screenshots/09-env-variables.png)

5. **Acquire nodes**
   - Request node provisioning for the updated configuration

6. **Launch the workspace**
   - Start the workspace with the new configuration

### Demo Execution

- Demonstrate LLM deployment capabilities
- Show model serving and inference functionality
- Highlight integration with AKS and Azure ecosystem

![Deploy LLMs Workspace Running](screenshots/10-deploy-llms-workspace.png)

![LLM Inference Example](screenshots/11-llm-inference.png)

---

## Tips for a Successful Demo

- Ensure nodes are provisioned before the demo to avoid wait times
- Test the workflows in advance to familiarize yourself with the UI
- Prepare talking points about AKS integration benefits
- Have backup examples ready in case of any technical issues
- Emphasize scalability and Azure-native features

---

## Support

For questions or issues, contact [omar@anyscale.com](mailto:omar@anyscale.com)
