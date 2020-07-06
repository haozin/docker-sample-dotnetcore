# .NET Core Application Overview
This is the basic default .NET Core API Weather Forecsast and is used as the base for application discussion in moving to Containers and K8s.<br>
First part of the dmeo is to showcase building and running the app using native .NET Core outisde of a container.

1. Open VS Code and a PowerShell / Cmd Terminal Window
    - CTRL+SHIFT+`
2. Using VS Code Clone aks-developer-demo Repository
    - cd F:\repos
    - git clone https://incyclesoftware@dev.azure.com/incyclesoftware/AKS%20Enterprise%20Accelerator/_git/aks-developer-demo aks-dev-demo
3. Using VS Code
    - Open 'aks-dev-demo' Folder of Weather Forecast application
    - Walk thru weather application
    - Briefly go over simple web api application
4. Using VS Code - Open PowerShell / Cmd Terminal Window
    - CTRL+SHIFT+`
5. VS Code Terminal - Build Application
    - dotnet build weatherforecast/weather.csproj
    - Validate application builds without errors
6. Open Postman and the AKS Developer Demo Collection
    - Import 'AKS Deveoper Demo.postman_collection.json' from the 'environment' folder of the repo
    - Open 'Local Test - .NET Core'
7. VS Code Terminal - Run Application
   - dotnet run --project weatherforecast/weather.csproj --configuration debug --launch-profile Weather
8.  In Postman - Run 'Local Test - .NET Core'
    -   Review JSON results
9.  VS Code Terminal - Stop Application

# Containerize .NET Core Application
This part of the demo is to demonstrate the basic steps of building and running a Docker container for<br>
the Weather Forecast API. VS Code is used to auto generate a docker file so time should be taken to briefly<br>
discuss the key elements in the docker file once generated.

1. VS Code - Create Docker File
    - CTRL+SHIFT+P - Bring up VS Code Command Palette
    - Search for 'Docker:Add Docker Files to Workspace' Command
    - Execute Command - Docker:Add Docker Files to Workspace
    - Select '.NET: ASP.NET Core' as the Platform
    - Select 'Linux' as the Platform
    - Select 'No' to option Docker Compose files
    - Enter '5000' as the Application Ports
2. Add Environment variable to Docker Image
    - VS Code - Modify Docker file
    - Add to Base Image
        - 'ENV ASPNETCORE_URLS=http://*:5000'
    - Modify Line 8 of Docker File
        - 'COPY ["Weather.csproj", "."]'
    - Modify Line 9 of Docker File
        - 'RUN dotnet restore "Weather.csproj"'
    - Modify Line 11 of Docker File
        - 'WORKDIR "/src/" '
    - Save
3.  Build Docker Image
    - In VS Code Terminal - Get list of current docker images
        - docker images
    - Notice WeatherForecast images does not exist yet
    - In VS Code Terminal - Build Docker Image
        - docker build -t weatherforecast-image weatherforecast/.
    - In VS Code Terminal - List images
        - docker images
    - Notice new 'weatherforecast-image' is available
4.  Create Docker Container
    - In VS Code Terminal - Create Docker Container
        - docker create --name myweather --publish 5000:5000/tcp weatherforecast-image
    - In CS Code Terminal - List all docker containers
        - docker ps -a
5.  Start Weatherforecast Docker container
    - In VS Code Terminal - Start Docker Container
        - docker start myweather
    - In VS Code Terminal - Verify running container
        - docker ps
        - Verify container myweather is running
    - In Postman - Verify App output
        - Run 'Local Test - .NET Core'
        - Verify JSON results
6.  Stop Weatherforecast Docker container
    - In VS Code Terminal - Docker Stop
        - docker stop myweather
    - In VS Code Terminal - Delete container
        - docker rm myweather

# Publish Docker Image to Azure Container Registry
This portion of the demo is to showcase creating an Azure Container Registry that contains<br>
the developer create docker images and which the K8s cluster will pull images from for deployment.<br>
The steps will demonstrate creating the ACR and pushing the local docker image into the created ACR.

1. Login into Azure
    - In VS Code Terminal
        - az login
        - az account set --subscription 'InCycle Labs - MSDN DevTest'
2. Create Azure Resource Group
    - In VS Code Terminal
        - az group create --name aksDeveloperDemo --location centralus --tags Creator=ConsultantName
3. Create Azure Container Registry
    - In VS Code Terminal
        - az acr create --resource-group aksDeveloperDemo --name acrDeveloperDemo --sku Basic
        - az acr login --name acrDeveloperDemo
4. Get Azure Container Registry Login Server
    - In VS Code Terminal
        - az acr list --resource-group aksDeveloperDemo --query "[].{acrLoginServer:loginServer}" --output table
5. Tag Local Weatherforecast Docker image
    - In VS Code Terminal
        - docker tag weatherforecast-image acrdeveloperdemo.azurecr.io/weatherforecast:v1
        - docker images
6. Push Image to Azure Container Registry
    - In VS Code Terminal
        - docker push acrdeveloperdemo.azurecr.io/weatherforecast:v1
        - az acr repository list --name acrDeveloperDemo --output table
        - az acr repository show-tags --name acrDeveloperDemo --repository weatherforecast --output table

# Create a Kubernetes Cluster
This portion of the demo is to showcase the steps needed to create a basic Azure Kubernetes Cluster<br>
Make sure that you have 'Owner' permissions on the target subscription because the CLI command created<br>
necessary RBAC security Service Principal to connect the the ACR.  In order to set permissions you must be an  'owner'

1. Modify application manifest
    - In VS Code Terminal
        - az aks create --resource-group aksDeveloperDemo --name aksDeveloperDemo --node-count 2 --generate-ssh-keys --attach-acr acrDeveloperDemo
2. Install Kubernetes CLI
    - In VS Code Terminal
        - az aks install-cli
3. Connect to Azure Kubernetes Cluster
    - In VS Code Terminal
        - az aks get-credentials --resource-group aksDeveloperDemo --name aksDeveloperDemo
        - kubectl get nodes

# Deploy Container Image to Cluster
This portion of the demo focuses on deploying a docker image stored in ACR using a single<br>
AKS YAML mainifest file.

1. Create Azure Kubernetes Cluster
    - In VS Code Editor
        - Open 'manifest/weatherforecast.deployment.yml'
        - Review key elements of the yaml file and tie back to presentation of Pods and Services
        - Validate container and image values are correct for the ACR being used
2. Deploy Weatherforecast
    - In VS Code Terminal
        - kubectl apply -f ../manifest/weatherforecast.deployment.yml
3. Monitor Deployment
    - In VS Code Terminal
        - kubectl get service weatherforecast-api --watch
        - CTRL-C (Stop watch when service ready)
4. Validate Weatherforecast Deployment
    - In Postman - Verify App output
        - Open 'AKS Cluster - .NET Core' and modify IP Address from the deployment
        - Run 'AKS Cluster - .NET Core'
        - Verify JSON results

# Scale Weatherforecast
In this portion of the demo focus is towards being able to quickly scale the number of pods that the<br>
Weatherforecast service is running in.

1. Get Weatherforecast pods
    - In VS Code Editor
        - kubectl get pods
        - Verify number of pods corresponds to yaml configuration file
2. Increase pod count
    - In VS Code Terminal
        - kubectl scale --replicas=5 deployment/weatherforecast-api
        - kubctl get pods
        - Validate pod count increased

# Modify .NET Core Application
This final portion of the demo showcases the ability to locally edit the weather forecast service<br>
and quickly deploy it without impacting the current running service.

1. VS Code Editor - Modify WeatherController.cs
    - Change number of forecasts returned to 2 (line 30)
    - Save
2. In VS Code Terminal - Build Application
    - dotnet build
    - Validate application builds without errors
4. Build Docker Image
    - In VS Code Terminal - Build Docker Image
        - docker build -t weatherforecast-image Weatherforecast/.
    - In VS Code Terminal - List images
        - docker images
        - Notice new 'weatherforecast-image' is available
5.  Run Docker Container - Run Application as a Docker Container
    - Is VS Code Terminal
        - docker run --name myweather --publish 5000:5000/tcp --rm -it weatherforecast-image
    - In Postman - Verify App output
        - Run 'Local Test - .NET Core'
        - Verify JSON results
    - In VS Code Terminal
        - CTRL+C to stop application
7. Tag Weatherforecast Docker image
    - In VS Code Terminal
        - docker tag weatherforecast-image acrdeveloperdemo.azurecr.io/weatherforecast:v2
        - docker images
8. Push Image to Azure Container Registry
    - In VS Code Terminal
        - docker push acrdeveloperdemo.azurecr.io/weatherforecast:v2
        - az acr repository list --name acrDeveloperDemo --output table
        - az acr repository show-tags --name acrDeveloperDemo --repository weatherforecast --output table
9. Deploy Updated Weatherforecast
    - In VS Code Terminal - Monitor Deployment
        - Open second PowerShell terminal to monitor doployment
        - kubectl get pods --watch
        - CTRL+C when deployment is complete
    - In VS Code Terminal - Create Update
        - kubectl set image deployment weatherforecast-api weatherforecast=acrdeveloperdemo.azurecr.io/weatherforecast:v2
        - kubectl get pods
10. Validate Weatherforecast Deployment
    - In VS Code Terminal
        - kubectl get service weatherforecast-api
    - In Postman - Verify App output
        - Open 'AKS Cluster - .NET Core' and modify IP Address from the deployment
        - Run 'AKS Cluster - .NET Core'
        - Verify JSON results return 2 forecasts
