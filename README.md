# Azure App Service Github Action Template Project

## Prerequisites

* Azure CLI

## Github User Setup

The following steps only need to be performed at the initial creation of the Github repository. 

1. Create a personal access token (classic) with the `repo` and `read:packages` scopes. For more information, see "[Creating a personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)."

## Azure App Service Setup

The following steps only need to be performed at the initial creation of the App Service.


```bash
APP_NAME="myapp112358" # must be globally unique
RESOURCE_GROUP="myapp-rg"
PLAN_NAME="myapp-plan"
DOCKER_REGISTRY_SERVER_URL="https://ghcr.io"
DOCKER_REGISTRY_SERVER_USERNAME="mygithubuser"
DOCKER_REGISTRY_SERVER_PASSWORD="mygithubsecret"

# authenticate to tenant in Azure
az login --tenant ${TENANT_ID}

# create resource group (once)
az group create --name ${RESOURCE_GROUP} --location eastus

# Valid SKUs
# B1, B2, B3, D1, F1, FREE, I1, I1v2, I2, I2v2, I3, I3v2, I4v2, I5v2, I6v2,
# P0V3, P1MV3, P1V2, P1V3, P2MV3, P2V2, P2V3, P3MV3, P3V2, P3V3, P4MV3, 
# P5MV3, S1, S2, S3, SHARED, WS1, WS2, WS3

az appservice plan create \
  --name ${PLAN_NAME} \
  --resource-group ${RESOURCE_GROUP} \
  --is-linux --sku B1
  
az webapp create \
  --name ${APP_NAME} \
  --plan ${PLAN_NAME} \
  --resource-group ${RESOURCE_GROUP} \
  --deployment-container-image-name nginx:latest
  
az webapp config appsettings set \
     --name ${APP_NAME} \
     --resource-group ${RESOURCE_GROUP} \
     --settings \
     DOCKER_REGISTRY_SERVER_URL=${DOCKER_REGISTRY_SERVER_URL} \
     DOCKER_REGISTRY_SERVER_USERNAME=${DOCKER_REGISTRY_SERVER_USERNAME} \
     DOCKER_REGISTRY_SERVER_PASSWORD=${DOCKER_REGISTRY_SERVER_PASSWORD}

# Get Deployment Profile for AZURE_WEBAPP_PUBLISH_PROFILE secret
az webapp deployment list-publishing-profiles \
  --name ${APP_NAME} \
  --resource-group ${RESOURCE_GROUP}\
  --xml --output tsv > ${APP_NAME}.PublishSettings

```

## Github Actions Setup

The following steps only need to be performed at the initial creation of the Github repository. These settings configure github actions to automatically create the docker container and deploy to Azure.

1. In the Github repository settings, create secret named "AZURE_WEBAPP_PUBLISH_PROFILE"
2. Copy contents of the `${APP_NAME}.PublishSettings` file created above to the secret.
3. Update/verify the application name within `.github/workflows/azure-app-service.yml`
4. Commit changes to repository. 

## Updating Application/Configuration

To make changes to the application, perform the following. 

1. Create new git branch with required changes. This can include changes to the source code or changes to configuration.
2. Once changes to branch have been completed and validated, create new git pull request to merge branch with "main".
3. Merging to the "main" branch or commiting directly to it will engage the Github actions to deploy the updated application. 
4. The application or config changes will be live once the application service is restarted in Azure. 

# License

MIT License

Copyright (c) 2023 Nick Marus

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
