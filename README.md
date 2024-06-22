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


### Continuos Delivery

