---
title: "透過 Azure Container Registry 佈署 Azure Functions"
subtitle: ""
excerpt: "azure functions container registry identity"
layout: post
author: "blueskyson"
header-style: text
tags:
  - azure
  - azure functions
  - csharp
  - dotnet
---

## 開發環境

透過命令列開發：

- .NET 6.0 SDK
- Azure Functions Core Tools 4.x 版
- Azurite 儲存體模擬器
- Azure CLI 2.4.0 版或更新版本。
- Docker

## 建立本機專案

用以下指令在當前目錄中建立一個 Timer Trigger 的 Function，並產生預設的 Dockerfile。

```non
func init --name ContainerApp --worker-runtime dotnet --docker
func new --name TimerTriggerExample --template "timer trigger"
```

先執行 Azurite 然後測試執行 Function App：

```non
azurite -l azurite_workspace
func start
```

測試完成後，打包成鏡像檔並執行，注意用 docker 執行時，需要令 `AzureWebJobsStorage` 為 `UseDevelopmentStorage=true` 並且透過 Azurite 來模擬儲存體。將你的 Azure Container Registry 的名稱填入下方指令區塊中的 `ACR_NAME` 環境變數：

```non
export ACR_NAME=<your azure container registry name>

docker build -t ${ACR_NAME}.azurecr.io/container-app:latest .
docker run --name container-app -p 8000:80 --network host \
    -e AzureWebJobsStorage="UseDevelopmentStorage=true" ${ACR_NAME}.azurecr.io/container-app:latest
```

特別注意目前 Azure Functions 的容器無法直接在 Mac M1 上執行。

順帶一提，Azurite 也有提供 Docker 鏡像，可以透過以下指令直接執行：

```non
docker run --name azurite -p 10000:10000 -p 10001:10001 -p 10002:10002 -d mcr.microsoft.com/azure-storage/azurite
```

測試完本地的鏡像檔後，將鏡像推送到 Azure Container Registry 中：

```non
az login
az acr login -n ${ACR_NAME}
docker push ${ACR_NAME}.azurecr.io/container-app:latest
```

接下來可以透過兩種方法來透過 Azure Container Registry 佈署 Azure Functions：

## Method 1: Admin Credentials

給 Functon App 一個 Container Registry 的帳號密碼，讓 Function App 透過這組帳號密碼來登入 Container Registry 並拉取鏡像檔。

首先在 Azure Portal 中創建一個資源群組，將此資源群組的名稱填入下方指令區塊中的 `RG_NAME` 環境變數，將 Azure Function 所需的資源創建、佈署在這個資源群組中：

```non
export ACR_NAME=<azure container registry name>
export RG_NAME=<function app's resource group>
export APP_NAME=<function app name>
export STORAGE_NAME=<storage account name>

# 建立儲存體帳號
az storage account create --name ${STORAGE_NAME} --resource-group ${RG_NAME} --sku Standard_LRS --kind StorageV2

# 建立 App Service Plan
az appservice plan create --name PlanB1 --resource-group ${RG_NAME} --is-linux --sku B1

# 獲取 ACR 的 username
az acr credential show -n ${ACR_NAME} --query username --output tsv

# 獲取 ACR 的 password
az acr credential show -n ${ACR_NAME} --query passwords[0].value --output tsv

# 建立/重新佈署 Function App，一次將所有必要資訊設定好
az functionapp create --name ${APP_NAME} --storage-account ${STORAGE_NAME} --resource-group ${RG_NAME} \
    --plan PlanB1 --functions-version 4 \
    --deployment-container-image-name ${ACR_NAME}.azurecr.io/container-app:latest \
    --docker-registry-server-password <acr password> \
    --docker-registry-server-user <acr username>
```

## Method 2: Managed Identity

為 Function App 註冊一個 Identity，讓 Container Registry 給予它 "AcrPull" 的權限。

```non
export ACR_NAME=<azure container registry name>
export RG_NAME=<function app's resource group>
export APP_NAME=<function app name>
export STORAGE_NAME=<storage account name>

# 建立儲存體帳號
az storage account create --name ${STORAGE_NAME} --resource-group ${RG_NAME} --sku Standard_LRS --kind StorageV2

# 建立 App Service Plan
az appservice plan create --name PlanB1 --resource-group ${RG_NAME} --is-linux --sku B1

# 建立一個 Function App 並指定 image 的位址
az functionapp create --name ${APP_NAME} --storage-account ${STORAGE_NAME} --resource-group ${RG_NAME} \
    --plan PlanB1 --functions-version 4 \
    --deployment-container-image-name ${ACR_NAME}.azurecr.io/container-app:latest

# 為 Function App 註冊一個 Identity 並取得 principalId
az functionapp identity assign --resource-group ${RG_NAME} --name ${APP_NAME} --query principalId --output tsv

# 取得 subscriptionId
az account show --query id --output tsv

# 給予 Function App "AcrPull" 權限
az role assignment create --assignee <principal-id> \
    --scope /subscriptions/<subscription-id>/resourceGroups/<container registry resource group>/providers/Microsoft.ContainerRegistry/registries/${ACR_NAME} \
    --role "AcrPull"

# 設定必須透過 Identity 驗證 Container Registry 拉取權限
az resource update --ids /subscriptions/<subscription-id>/resourceGroups/${RG_NAME}/providers/Microsoft.Web/sites/${APP_NAME}/config/web --set properties.acrUseManagedIdentityCreds=True

# 佈屬 Function App
az functionapp config container set --name ${APP_NAME} --resource-group ${RG_NAME} \
    --docker-custom-image-name ${ACR_NAME}.azurecr.io/container-app:latest \
    --docker-registry-server-url https://${ACR_NAME}.azurecr.io
```

此時終端機會顯示 "No credential was provided to access Azure Container Registry. Trying to look up..."，azure 將自動套用當前的 identity 去登入 acr。