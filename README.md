# Azure DevOps Assignment 🚀

This project demonstrates the implementation of Azure DevOps practices including work item queries, dashboards, pipeline variables, variable/task groups, service connections, self-hosted agents, approvals, and CI/CD pipelines for various deployment targets.

---


## 📊 Tasks

| Task No. | Task Description                                             | Status        |
|----------|--------------------------------------------------------------|---------------|
| 1        | Work Items Dashboard & Queries                               | ✅ Completed  |
| 2        | Use of Pipeline Variables                                     | ✅ Completed  |
| 3        | Variable & Task Groups with Scoped Stages                    | ✅ Completed  |
| 4        | Create and Use Service Connection                            | ✅ Completed  |
| 5        | Create Linux/Windows Self-Hosted Agent                       | ✅ Completed  |
| 6        | Add Pre & Post Deployment Approvals                          | ✅ Completed  |
| 7        | CI/CD: Docker Build & Push to ACR, Deploy to AKS             | ✅ Completed  |
| 8        | CI/CD: Docker Build & Push to ACR, Deploy to ACI             | ✅ Completed  |
| 9        | CI/CD: .NET App Build and Deploy to Azure App Service        | ✅ Completed  |
| 10       | CI/CD: React App Build and Deploy to Azure Virtual Machine   | ✅ Completed  |

---

## 📌 1. Work Items Dashboard & Queries

- Created custom work item queries using `Boards > Queries` for:
  - Active bugs
  - Tasks assigned to me
- Pinned queries to a shared dashboard for tracking.



## ⚙️ 2. Pipeline Variables

Used pipeline-level and stage-level variables in YAML.

```yaml
variables:
  imageName: myapp
  tag: $(Build.BuildId)
steps:
- script: echo "Building $(imageName):$(tag)"
```

## 📦 3. Variable & Task Groups
Created variable groups in Library > Variable Groups

Scoped them to specific pipeline stages.

```yaml
variables:
- group: BuildSecrets

stages:
- stage: Build
  variables:
    - group: BuildSecrets
- stage: Deploy
  variables:
    - group: DeploySecrets
```


### 🔐 4. Service Connection
Created Azure Resource Manager and Docker Registry (ACR) service connections from Project Settings > Service Connections

Used them in pipelines to push images and deploy.


## 🖥️ 5. Self-Hosted Agent
Configured a Linux self-hosted agent with the following:

```bash
wget https://vstsagentpackage.azureedge.net/agent/3.225.0/vsts-agent-linux-x64-3.225.0.tar.gz
tar zxvf vsts-agent-linux-x64-3.225.0.tar.gz
./config.sh --url https://dev.azure.com/ORG --auth pat --token YOUR_PAT
./svc.sh install
./svc.sh start
```


## ⏱️ 6. Pre & Post Deployment Approvals
Added manual approvers in Release Pipelines

Configured both pre-deployment and post-deployment conditions per environment.

## 🐳 7. CI/CD: Docker → ACR → AKS
```yaml
trigger:
- main

variables:
  imageName: myapp

stages:
- stage: Build
  jobs:
  - job: DockerBuild
    steps:
    - task: Docker@2
      inputs:
        containerRegistry: 'acr-connection'
        repository: '$(imageName)'
        command: 'buildAndPush'
        Dockerfile: '**/Dockerfile'
        tags: |
          latest
          $(Build.BuildId)

- stage: Deploy
  jobs:
  - job: AKSDeploy
    steps:
    - task: Kubectl@1
      inputs:
        kubernetesServiceEndpoint: 'aks-connection'
        command: apply
        arguments: -f manifests/deployment.yaml
```

## 🚀 8. CI/CD: Docker → ACR → ACI
```yaml

- task: AzureCLI@2
  inputs:
    azureSubscription: 'azure-service-connection'
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      az container create \
        --resource-group my-rg \
        --name myapp \
        --image myregistry.azurecr.io/myapp:latest \
        --dns-name-label myappdemo \
        --ports 80
```

## 🧩 9. CI/CD: .NET → Azure App Service
```yaml

trigger:
- main

pool:
  vmImage: 'windows-latest'

variables:
  buildConfiguration: 'Release'

steps:
- task: DotNetCoreCLI@2
  inputs:
    command: 'publish'
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'

- task: AzureWebApp@1
  inputs:
    azureSubscription: 'azure-service-connection'
    appType: 'webApp'
    appName: 'dotnet-app'
    package: '$(Build.ArtifactStagingDirectory)/**/*.zip'
```

## 🌐 10. CI/CD: React → Azure VM (via SSH)
```yaml

trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '18.x'

- script: |
    npm install
    npm run build

- task: CopyFilesOverSSH@0
  inputs:
    sshEndpoint: 'vm-ssh-connection'
    sourceFolder: 'build'
    contents: '**'
    targetFolder: '/var/www/html'
```
---
