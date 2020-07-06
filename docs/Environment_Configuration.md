# Demo Presenter Azure VM Environment
 - Custom VM Image has been create in the InCycle Labs - MSDN DevTest Subscription
    - Demo-Image-Gallery -> Demo Images -> ASK-Developer-Demo v1.0.0
 - The aks-developer-demo reposiotry has a ARM Template and Parameter file to create the VM using the defined image
 - To create VM environment using Azure CLI
    - Clone Repo locally
    - az group create --location "centralus" --name "aksDeveloperDemo-vm" --tags "Creator=[consultant UPN]"
    - az deployment group create  --name "aks-demo" --resource-group "aksDeveloperDemo-vm" --template-file template.json --parameters "@parameters.json"
    - RDP to created VM using aksdemouser / P2ssw0rd1234 for credentials
 ## VM Configuration
 1. VS Code Installed
    - C\# VS Code Extension installed
    - Docker VS Code Extension installed
    - Azure CLI Extension Installed
 2. Azure CLI Installed
 3. Git Installed
 4. Docker Installed
 5. Powershell 7.0.1 Installed
 6. Postman Installed

# Demo Presenter Requirements
1.  Owner Access to target Azure Subscription
2.  Contributor Access to <https://dev.azure.com/incyclesoftware/AKS%20Enterprise%20Accelerator>
