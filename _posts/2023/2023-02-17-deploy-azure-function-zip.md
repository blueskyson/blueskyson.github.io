---
title: "透過 Zip 佈署 Azure Functions"
subtitle: ""
excerpt: "azure functions zip deploy"
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

## 建立本機專案

用以下指令在當前目錄中建立一個 Http Trigger 的 Function App。

```non
func init FunctionApp --name FunctionApp --worker-runtime dotnet
cd FunctionApp
func new --name HttpExample --template "HTTP trigger" --authlevel "anonymous"
```

測試執行 Function App：

```non
func start
```

## 透過 Zip 部屬

測試完成後，先透過 dotnet cli 將 Function App 打包成 binary code 到 `./publish` 目錄中：

```non
dotnet publish -o publish
cd ./publish
zip -r publish.zip *
```

注意一定要進到 `./publish` 目錄中打包，不然 Azure 會抓不到檔案，並且請慎選 zip 軟體，像 winrar 或其他 windows 的軟體可能會因為用反斜線來儲存路徑，導致 Azure 一樣抓不到檔案，建議用 linux 平台的 zip 來打包，我自己實測 Info-Zip 是可以用的。

接下來創建一個 App Service Plan 的 Azure Function，透過以下指令部屬到 Azure：

```non
export RG_NAME=<your resource group name>
export APP_NAME=<your function app name>

az login
az functionapp deployment source config-zip -g $RG_NAME -n $APP_NAME --src ./publish.zip
az functionapp config appsettings set -g $RG_NAME -n $APP_NAME --settings WEBSITE_RUN_FROM_PACKAGE=1
```

`WEBSITE_RUN_FROM_PACKAGE` 必須設為 1 才能夠讓 Function App 透過 zip 來部屬。

部屬時有機會遇到 `HTTPSConnectionPool(host='cybersaas-crm-scheduler-999i.scm.azurewebsites.net', port=443): Read timed out. (read timeout=3)`，此時可以用

```non
az webapp deployment source config-zip -g $RG_NAME -n $APP_NAME --src ./publish.zip
```

來替代。（參考 https://github.com/Azure/azure-cli/issues/13655）