# Login to Azure Container Registry
 

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/9dfc454a-fb10-4c34-819a-0fc413af46a6" />


<img width="428" height="282" alt="image" src="https://github.com/user-attachments/assets/87c5322b-59c7-4f74-9775-c137877037a7" />


### Azure Container Registry:
One Azure Registry can have multiple repositories -- and one repo can have multiple tags.


### Install Azure CLI
Login to Azure ( make sure correct tenant and subscription)

```bash
az login
az login --tenant
az account list --output table
az account set --subscription <Sub name or id>
```

```bash
az acr login -n <ACR name>
```



## Push/pull image now from/to ACR:


You can use Docker commands to push/pull :

```bash
docker push <image name:tag>
docker pull <image name:tag>
```

### You can also use az cli to build/push/pull:

```bash
az acr build -t <imageName> -r <acrName> .                                                     :  Builds a Docker image from a Dockerfile and pushes it to the specified registry. 
az acr repository list --name <acrName> --output table                                         : Lists all repositories in the specified registry. 
az acr repository show-tags --name <acrName> --repository <repositoryName> --output table      : Shows tags for a specific repository within the registry. 
```

Tagging:
  
  Tag a local image with the registry's login server and repository name for pushing. 

```bash
docker tag <imageName>:<tag> <acrLoginServer>/<imageName>:<tag>    
```



azurecr.io --> its a domain for ACR endpoint

 **Example**:
   https://mytestacr.azurecr.io







   
