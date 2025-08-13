

---

````markdown
# 🚀 Azure Pipelines Setup with Self-Hosted Agent VM

This guide explains how to set up **Azure Pipelines** for multiple microservices using **Docker** and **Azure Container Registry (ACR)**, along with creating a **Self-Hosted Agent VM**.

---

## 📌 Steps to Create Azure Pipelines

### 1️⃣ Login to Azure DevOps
- Go to [Azure DevOps](https://dev.azure.com/) and log in with your credentials.

---

### 2️⃣ Create a Project
- Click **New Project** → Enter **Project Name** → Choose visibility (**Private** or **Public**).

---

### 3️⃣ Import Code from GitHub
- Navigate to **Repos** → **Import Repository**.
- Provide your GitHub repo URL and click **Import**.

---

### 4️⃣ Set Default Branch to `main`
- Go to **Branches** → Click the **⋮ (three dots)** next to your branch → Select **Set as default branch**.

---

### 5️⃣ Create Separate Pipelines for Each Microservice
Example: **worker**, **result**, **vote**.

- Navigate to **Pipelines** → **Create Pipeline**.
- Select **Azure Repos Git**.
- Choose your repo.
- Select **Docker: Build and push image to Azure Container Registry**.

---

### 6️⃣ Configure Pipeline
- Select **Azure Subscription**.
- Select your **Azure Container Registry (ACR)** for authentication.
- Validate and click **Configure**.

---

## ⚙ Azure Pipeline YAML Example
Below is a sample pipeline for the **result** microservice:

```yaml
trigger:
  paths:
    include:
      - result/*

resources:
- repo: self

variables:
  dockerRegistryServiceConnection: '8d1a20f7-316a-4a5b-b8b7-c84810d2fa5a'
  imageRepository: 'demoapp'
  containerRegistry: 'azuredemoap.azurecr.io'
  dockerfilePath: '$(Build.SourcesDirectory)/result/Dockerfile'
  tag: '$(Build.BuildId)'

pool:
  name: 'azureagent'

stages:
- stage: Build
  displayName: Build
  jobs:
  - job: Build
    displayName: Build
    steps:
    - task: Docker@2
      displayName: Build the image
      inputs:
        containerRegistry: '$(dockerRegistryServiceConnection)'
        repository: '$(imageRepository)'
        command: 'build'
        Dockerfile: 'result/Dockerfile'
        tags: '$(tag)'

- stage: Push
  displayName: Push
  jobs:
  - job: Push
    displayName: Push
    steps:
    - task: Docker@2
      displayName: Push the image
      inputs:
        containerRegistry: '$(dockerRegistryServiceConnection)'
        repository: '$(imageRepository)'
        command: 'push'
        tags: '$(tag)'
````

---

## 🖥 Self-Hosted Agent Setup

### 1️⃣ Create a VM in Azure

* Use Azure Portal or CLI to create a **Linux/Windows VM**.
* Ensure **internet access** is enabled.

---

### 2️⃣ Install Prerequisites on the VM

For Linux:

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install unzip curl -y
```

---

### 3️⃣ Create a Personal Access Token (PAT)

* Go to **User Settings** → **Personal Access Tokens**.
* Click **New Token**.
* Scope: `Agent Pools → Read & Manage`.

---

### 4️⃣ Download & Configure Azure Pipelines Agent

* Go to **Project Settings** → **Agent Pools** → **Default** → **New Agent**.
* Select **OS** (Linux/Windows) → Download.
* Extract the agent:

```bash
mkdir myagent && cd myagent
tar zxvf ~/downloads/vsts-agent-linux-x64-<version>.tar.gz
```

* Configure the agent:

```bash
./config.sh
```

* Start the agent:

```bash
./run.sh
```

*(Optional: Install as a service for automatic start on reboot)*

---

## 📌 Azure Pipelines Structure Overview

```
Pipeline → Stage(s) → Job(s) → Step(s) → Task(s)/Script(s)

🔹 Stage: Logical grouping of jobs (Build, Test, Deploy)
🔹 Job: Runs on an agent (e.g., Ubuntu, Windows)
🔹 Step: Single execution unit in a job
🔹 Task: Predefined action (Docker build, push, etc.)
```

---
