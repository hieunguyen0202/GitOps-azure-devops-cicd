# Example Voting App

A simple distributed application running across multiple Docker containers.
- Video demo:

## Architecture Design

![GitOps-azure-devops-cicd drawio](https://github.com/hieunguyen0202/GitOps-azure-devops-cicd/assets/98166568/a206551e-24cb-47ec-a738-0c5aa40fea3b)

## Implemtation

### Continuos Integration
#### Create new project on Azure DevOps

![image](https://github.com/hieunguyen0202/GitOps-azure-devops-cicd/assets/98166568/a6b3533c-6413-4167-8713-959516d730f1)

- Import your repository
  ```
  https://github.com/dockersamples/example-voting-app.git
  ```
- Go go `Container Registry` to create new repo `my-repo-registry` on Azure/GCP cloud platform

#### Set up agent for runnimg job pipeline
- Go to project setting -> Choose `Agent pools` -> Add agent pool with pool type `Self-hosted`
- Click on this agent -> Click `New agent`, you will see some command to setup agent
- SSH to ubuntu virtual machine
  - Update
    ```
    sudo apt update
    ```
  - Run this command

    ```
    mkdir myagent && cd myagent
    ```
  - Download agent

    ```
    wget https://vstsagentpackage.azureedge.net/agent/3.240.1/vsts-agent-linux-x64-3.240.1.tar.gz
    ```
  - Extract agent

    ```
    tar zxvf ~/Downloads/vsts-agent-linux-x64-3.240.1.tar.gz
    ```
  - Run this command to config

    ```
    ./config.sh
    ```

  - For `Enter server URL`, enter `https://dev.azure.com/{NameOfOrganization}`
  - Enter authentication type (press enter for PAT) > Press on Enter
  - For `Enter personal access token`, go to Setting -> Choose `Personal Access Tokens` -> Click on `New Token` with option Full Access and copy this token
  - For `Enter agent pool (press enter for default)` -> azurehost
  - Press Enter for other
  - Finally, run this command
    ```
    ./run.sh
    ```
  - Go to check that

    ![image](https://github.com/hieunguyen0202/GitOps-azure-devops-cicd/assets/98166568/bef77662-8ab2-4e78-919e-53db85aa9ac2)

- Next, install docker

  ```
  sudo apt install docker.io -y
  ```
- Grant azureuser permission for docker

  ```
  sudo usermod -aG docker samelnguyen08
  sudo systemctl restart docker
  ```

#### Create a pipeline
- Create a pipeline for `vote-service`

  ```yaml
  # Docker
  # Build and push an image to Azure Container Registry
  # https://docs.microsoft.com/azure/devops/pipelines/languages/docker
  
  trigger:
   paths:
     include:
       - vote/*
  
  resources:
  - repo: self
  
  variables:
    # Container registry service connection established during pipeline creation
    dockerRegistryServiceConnection: '8b19a049-e8e2-443b-b21e-a1cbae6f9713'
    imageRepository: 'votingapp'
    containerRegistry: 'cicdapprepo.azurecr.io'
    dockerfilePath: '$(Build.SourcesDirectory)/result/Dockerfile'
    tag: '$(Build.BuildId)'
  
  pool:
   name: 'azurehost'
  
  stages:
  - stage: Build
    displayName: Build 
    jobs:
    - job: Build
      displayName: Build
      steps:
      - task: Docker@2
        displayName: Build an image
        inputs:
          containerRegistry: '$(dockerRegistryServiceConnection)'
          repository: '$(imageRepository)'
          command: 'build'
          Dockerfile: 'vote/Dockerfile'
          tags: '$(tag)'
  
  - stage: Push
    displayName: Push 
    jobs:
    - job: Push
      displayName: Push
      steps:
      - task: Docker@2
        displayName: Push an image
        inputs:
          containerRegistry: '$(dockerRegistryServiceConnection)'
          repository: '$(imageRepository)'
          command: 'push'
          tags: '$(tag)'
  
  # - stage: Update
  #   displayName: Update 
  #   jobs:
  #   - job: Update
  #     displayName: Update
  #     steps:
  #     - task: ShellScript@2
  #       inputs:
  #         scriptPath: 'scripts/updateK8sManifests.sh'
  #         args: 'vote $(imageRepository) $(tag)'  
  ```
- Similar pipeline for `vworker-service` and `result-service`

### Continuos Delivery
#### Create kubernetes cluster on Azure/GCP 
- Choose name `gitops-cluster`
- Enable autoscaling node for min `1` and max `2`, enable for IP of pod

#### Connect to Kubernetes cluster
- For Azure Kubernetes services

  ```
  az aks get-credentials --resource-group resourcegroupname --name clustername
  ```
- For Google Kubernetes Engine

  ```
  gcloud container clusters get-credentials CLUSTER_NAME --region=CLUSTER_REGION
  ```
#### Setup ArgoCD on Kubernetes Cluster with namespace argocd
- Refer this [doc](https://argo-cd.readthedocs.io/en/stable/getting_started/) to download ArgoCD
- Do this command

  ```
  kubectl create namespace argocd
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```
- Check status of pod

  ```
  kubectl get pods -n argocd
  ```
- Get password of argoCD, to list secrets

  ```
  kubectl get secrets -n argocd
  ```
- To show password
  ```
  kubectl get secrets argocd-initial-admin-secret -n argocd -o yaml
  ```
  ![image](https://github.com/hieunguyen0202/GitOps-azure-devops-cicd/assets/98166568/62c44333-e5f1-45d9-913c-5bdd65017090)

- To decode this password

  ```
  echo [password] | base64 --decode
  ```
- Expose End-point for ArgoCD, list service

  ```
  kubectl get svc -n argocd
  ```

- Edit change to `NodePort` for `argocd-server`
  ```
  kubectl edit svc argocd-server -n argocd
  ```

- Rememeber this port

  ![image](https://github.com/hieunguyen0202/GitOps-azure-devops-cicd/assets/98166568/9b48205e-c9d5-4b8f-8ed4-ddcd6f5c2a49)

- To get external IP for node

  ```
  kubectl get nodes -o wide
  ```
#### Configure ArgoCD

