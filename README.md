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

Next open the training notebook
---

## Demo 2: Deploy LLMs

### Prerequisites

#### 1. Configure Compute

- Terminate the workspace if it is already running.
- Navigate to compute configuration settings.
- Change the head node from `2CPU-8GB` to `8CPU-32GB`.
- Change the worker nodes from `Auto-select workers` to `2 × A100`.

![A100 Node Configuration](screenshots/07-a100-nodes.png)

#### 2. Create a HuggingFace Token

If you don't already have a HuggingFace account, sign up at https://huggingface.co/join. You must verify your email address before you can create tokens.

Follow these steps to generate an access token:

1. Log in to https://huggingface.co.
2. Click your profile avatar in the top-right corner and select **Settings**.
3. In the left sidebar, click **Access Tokens**.
4. Click the **+ Create new token** button.
5. Give the token a descriptive name (e.g., `anyscale-demo`).
6. Set the role to **Read** — this is sufficient for downloading gated models. (Use **Write** only if you need to push models or datasets.)
7. Click **Create token**.
8. A pop-up will display your new token. Click **Copy** to copy it to your clipboard immediately — you will not be able to view the full token value again after closing this dialog.
9. Store the token in a secure location (e.g., a password manager). Do not commit it to version control.

![HuggingFace Token Creation](screenshots/08-huggingface-token.png)

**Gated model access:** The Llama 3.1 model used in this demo is a gated model. Before your token will work, you must also visit the model page at https://huggingface.co/meta-llama/Meta-Llama-3.1-8B-Instruct, review the license, and click **Request access**. Approval is usually granted within minutes.

**Security tip:** Create a separate token for each use case (local dev, CI/CD, demo). This way you can revoke one token without affecting others. You can manage and delete tokens at any time from the **Access Tokens** settings page.

#### 3. Set the HuggingFace Token (required in two places)

| Location | How to set |
|----------|-----------|
| **Workspace** | Anyscale → Dependencies → Environment Variables → add `HF_TOKEN=<YOUR_HF_TOKEN>` |
| **Shell / deployment** | export in the terminal or pass `--env HF_TOKEN=...` when deploying |

#### 4. Prepare the Source Files

Before launching the workspace, create or update the following files inside `small-size-llm/`.

**serve_llama_3_1_8b.py**

```python
# serve_llama_3_1_8b.py
from ray.serve.llm import LLMConfig, build_openai_app

llm_config = LLMConfig(
    model_loading_config=dict(
        model_id="my-llama-3.1-8b",
        # For gated models, HF_TOKEN must be provided via env.
        model_source="meta-llama/Meta-Llama-3.1-8B-Instruct",  # <-- CHANGED: was "unsloth/Meta-Llama-3.1-8B-Instruct", now uses official gated Meta repo
    ),
    accelerator_type="A100",
    deployment_config=dict(
        autoscaling_config=dict(min_replicas=1, max_replicas=2),
    ),
    engine_kwargs=dict(max_model_len=8192, gpu_memory_utilization=0.78),
)

app = build_openai_app({"llm_configs": [llm_config]})
```

**Why `gpu_memory_utilization=0.78`?**

vLLM may attempt to use all reported GPU memory. Some GPU features (ECC, driver/firmware overhead, MIG/vGPU) reserve memory and can cause OOMs. Setting this value to `0.78` leaves safe headroom (roughly 100% − 12% ECC − 10% headroom ≈ 78%).

**service.yaml**

```yaml
name: deploy-llama-3-8b
image_uri: anyscale/ray-llm:2.50.1-py311-cu128  # <-- NOTE: pre-built image, use containerfile: ./Dockerfile for custom
compute_config:
  auto_select_worker_config: true  # <-- NOTE: service auto-selects workers; workspace uses manual 2×A100 config separately
  head_node:
    instance_type: 8CPU-32GB
working_dir: .
cloud:
applications:
  - import_path: serve_llama_3_1_8b:app
```

**Note on `auto_select_worker_config`:** The service YAML uses `auto_select_worker_config: true`, which lets Anyscale pick the best worker type automatically. The manual `2 × A100` configuration you set in Step 1 applies to the workspace only; the service manages its own workers independently.

**notebook.ipynb**

Open `small-size-llm/notebook.ipynb` and replace any occurrence of `accelerator_type="L4"` with `accelerator_type="A100"`.
<!-- CHANGED: must match the A100 GPU configured in Step 1 -->

### 5. Launch the Workspace and Verify It Is Running

Launch the workspace with the new compute and environment settings. Before proceeding, confirm the workspace and all nodes are up:

**Via the Anyscale UI:**

- Open the workspace in the Anyscale Console.
- Check the workspace status indicator in the top-right corner — it should show **Running**.
- Click the status badge to open the **Cluster Panel** and verify that the head node (`8CPU-32GB`) and worker nodes (`2 × A100`) are all provisioned and healthy.

**Via the CLI:**

```bash
# Check workspace status by name
anyscale workspace status --name <YOUR_WORKSPACE_NAME>

# Or wait until the workspace reaches RUNNING state (times out after 30 min by default)
anyscale workspace wait --name <YOUR_WORKSPACE_NAME>
```

**Tip:** If nodes take longer than expected to provision, check the Cluster Panel for pending or failed nodes. Common causes include insufficient cloud quota or unavailable instance types in your region.

### 6. Deploy the Service

Open a terminal inside the Anyscale workspace (either the built-in web terminal or VS Code integrated terminal). Navigate to the `small-size-llm/` directory, export your HuggingFace token, and deploy:

```bash
cd small-size-llm/
export HF_TOKEN=<YOUR_HF_TOKEN>
anyscale service deploy -f service.yaml --env HF_TOKEN=$HF_TOKEN
```

Quick verification that the token is set:

```bash
echo "$HF_TOKEN"
```

Once the service status shows **Running**, follow `small-size-llm/notebook.ipynb` to validate the deployed endpoint.

### Tips for a Successful Demo

- **Provision nodes early.** Spin up the workspace well before the demo to avoid wait times.
- **Dry-run the full workflow in advance** so you are familiar with the UI and any potential issues.
- **Keep the HF token secure.** Only provide it via workspace environment variables or the `--env` deploy flag.
- **Prepare backup examples** in case of technical issues.
- **Highlight production benefits** such as AKS integration, scalability, and Azure-native features.

Support
For questions or issues, contact ms-field-collab@anyscale.com