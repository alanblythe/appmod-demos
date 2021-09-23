# appmod-demos

This repository is a companion to a presentation given the Cognizant organization on Sept. 23 2021 entitled "*Modern Application development – Container services, AKS and Functions*"  

## Intitial Setup and Prerequisites

### Visual Studio Code
Install Visual Studio Code from the following link: https://code.visualstudio.com/Download

Install the following VsCode extensions:  
-  Azure Account
-  Azure Tools
-  C#
  
### Azure CLI  
For Windows Choclatey is the easiest way to install, follow these instructions to install Choclatey, https://chocolatey.org/install  

Execute the following script in Windows Powershell or Windows Powershell ISE.  
`Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))`

Ensure choco was added to your User or system path, if not add.  
`C:\ProgramData\chocolatey\bin`  
  
Open a command prompt and Install the Azure CLI  
`choco install azure-cli`

In advanced settings update your user PATH environment variable to include:  
`C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\wbin`  
After completing keep the dialog open as you will be adding more here.  

### Donet CLI  
To install the dotnet CLI, install .Net Core from https://docs.microsoft.com/en-us/dotnet/core/install/windows?tabs=net50  

Ensure the dotnet CLI is added to your user or system PATH  
`C:\Program Files (x86)\dotnet\`  

### Docker Desktop  

Windows: https://www.docker.com/products/docker-desktop  
Install docker desktop, this will also require the Windows Subsystem for Linx (WSL2) to be installed.  
This will most likely require a restart.  

Mac: https://hub.docker.com/editions/community/docker-ce-desktop-mac?utm_source=docker&utm_medium=webreferral&utm_campaign=dd-smartbutton&utm_location=header

### Docker CLI
When installing Docker Desktop the Docker CLI will also be installed

### Kubectl  
Follow the instructions at https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/ to install  
Add the following to your User PATH environment variable  
`%USERPROFILE%\.azure-kubectl`  
`%USERPROFILE%\.azure-kubelogin`  

### Azure Functions Core Tools
Follow the instructions at: https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=v3%2Cwindows%2Ccsharp%2Cportal%2Cbash%2Ckeda#v2 to install the Azure Functions Core Tools
Ensure your User or system PATH is updated with `C:\Program Files\Microsoft\Azure Functions Core Tools\`  

## Restart all terminals and VsCode instances
This ensures that your PATH environment variable is correct
## Azure
First, ensure you have an subscription that you can deploy your resources to  
Ensure you can login to Azure using `az login`  
Use `az account show --output table` to find what your active directory and subscription is.  
  
List your subscriptions with:  
`az account list --output table`  
Set your active subscription with:    
`az account set --subscription "My Demos"`  

Read more here: https://docs.microsoft.com/en-us/cli/azure/manage-azure-subscriptions-azure-cli  
Look at your registered providers with:  
Mac/Linux: `az provider list --output table | grep " Registered"`  
Windows: `az provider list --output table | findstr /C:" Registered"`  
Ensure the following providers are registered:  
- Microsoft.Storage
- Microsoft.OperationsManagement
- Microsoft.OperationalInsights
- Microsoft.ContainerService
- Microsoft.ManagedIdentity
- microsoft.insights 
- Microsoft.ContainerRegistry
- Microsoft.Compute
- Microsoft.Network

If not registered register with:  
 `az provider register --namespace {provider-string}`  
e.g. `az provider register --namespace Microsoft.Network`  
## Containers: Docker and Azure Container Registry 
Ensure that docker cli is working
1. open a new terminal
2. run `docker --version`
3. if not working follow the docker desktop install instructions  

Instructions  
1. Open VsCode  
2. Create a folder/directory and use VsCode Open Folder  
3. Open a new terminal from the terminal menu
4. Your terminal should be in the folder you opened, if not cd to it
5. Run `dotnet new webapi -o forecast.api -n forecast.api`
6. cd to the new directory `cd forecast.api`
7. Terminal: run `vscode .`, this will open vscode to the new application directory
8. VsCode Command Pallette (ctl+shift+P): find `add docker files to workspace` and click
9. A new dockerfile should be added to your directory, view to ensure it looks correct
10. Terminal: `docker image list`, no images should exist
11. VsCode Command Pallette (ctl+shift+P): `Docker Images: Build Image`
12. Terminal: Run `docker image list`, you should now see your image
13. Terminal:  
    `docker run -p 5000:5000 --name forecastapi forecastapi`  
14. Menu/Terminal->New Terminal: call api with curl  
    `curl http://localhost:5000/WeatherForecast`
15. Ensure you see a response from the API with weather forecast data
16. Terminal: show running containers
    `docker ps`
17. Terminal: stop running container  
    `docker stop forecastapi`  
18. Create ACR in the the terminal with az cli  
    Create a resource group  
    `az group create --location eastus --resource-group rg-ipldemo`
20. From here on you will use a name for your ACR such as: `{YourInitials}IplDemo` JklAcrDemo to ensure no DNS conflicts
20. Terminal: Create an Azure Container Registry resource  
    `az acr create --location eastus --name {YourAcrName} --resource-group rg-ipldemo --sku basic`
21. Terminal: Update Docker CLI config to include our new ACR repository `az acr login --name {YourAcrName}`
22. Terminal: publish our local image to ACR  
	`docker tag forecastapi:latest {YourAcrName}.azurecr.io/forecastapi:latest`  
	`docker push {YourAcrName}.azurecr.io/forecastapi:latest`  
23. Navigate to the Azure Portal -> All Resources
24. Find your ACR
25. Select `repositories` in the left menu
26. View your published image


## Azure Kubernetes Service (AKS)
1. Create a new AKS cluster  
    `az group create --name rg-k8s --location eastus`
    `az aks create --resource-group rg-k8s --name aks-ipldemo --node-count 1 --enable-addons monitoring --generate-ssh-keys`
    `az aks get-credentials --resource-group rg-k8s --name aks-ipldemo`    
2. Give AKS access to ACR to pull images  
   `az aks update -n aks-ipldemo -g rg-k8s --attach-acr {YourAcrName}`
3. Azure portal: Find your cluster, namespaces menu, see there are no namespaces
4. VsCode: review `forecast.yml` from the same directory as this Readme  
   Notice the service, deployment, and loadbalancer
5. VsCode->Terminal: View running pods  
    `kubectl get pods`
7. VsCode->Terminal: view k8s namespaces
   `kubectl get namespaces`
10. VsCode: Open Folder, folder that contains the forecast.yml file
11. VsCode->Terminal: Create our k8s resources
    `kubectl apply -f forecast.yml`
12. Azure portal: Find and view your AKS cluster  
    Click workloads and now see a running pod, forecast, browse around and you can more details
14. Azure portal: View the updated load balancer  
    Find your resource group such as: `MC_rg-k8s_aks-ipldemo_eastus`  
    Click the Load Balancer  
    Click the Load Balancing Rules menu  
    Click the one load balancing rule and notice it load  balacing port 80


## Azure Functions and dotnet CLI
1. Terminal: Create a new Azure function shell project  
    `func init forecast.func --dotnet`
2. Terminal:  
   `cd forecast.func`  
   `code .`  
3. Terminal: create a new HTTP triggered function in your project  
   `func new --name weatherforecast --template "HTTP trigger" --authlevel "anonymous"`
4. VsCode: View the created files in the File Explorer
5. VsCode: Copy the contents of azurefuncforecast.cs to `weatherforecast.cs` file
6. VsCode: F5 to debug the project
7. Ctrl+Click the URL in the output window to show the forecast data
8. Terminal: Create a new Azure Function resource (Watch for replacement {your initials})  
	`az group create --name rg-azurefunc --location eastus`  
	`az storage account create --name stazurefunc{your initials} --location eastus --resource-group rg-azurefunc --sku Standard_LRS`  
	`az functionapp create --resource-group rg-azurefunc --consumption-plan-location eastus   --runtime dotnet --functions-version 3 --name forecastfunc --storage-account stazurefunc{your initials}`
8. Terminal: Publish your file to the Azure Function
	`func azure functionapp publish forecastfunc`
9. hit the url in the output window
10. go to the azure portal
11. view the function app in All Resources  
	show the function, click it  
	show various aspects of the function  
	cant be modified since its deployed with a package  

## Clean Up

1. delete resource group  
   `az group delete --resource-group rg-ipldemo`
2. delete k8s deployment + service  
    `kubectl delete deployment forecast`
    `kubectl delete service forecast`
3. delete images from acr  
   `docker rmi $(docker images -q) -f`
   `docker system prune`
4. Delete Azure Function  
   `az group delete --resource-group rg-azurefunc`
5. Delete AKS cluster  
   `az group delete --resource-group rg-k8s`

