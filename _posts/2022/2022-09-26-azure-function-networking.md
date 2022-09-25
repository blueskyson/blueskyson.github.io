---
title: "Azure Functions 網路管理"
subtitle: ""
excerpt: "azure functions network vnet nat gateway"
layout: post
author: "blueskyson"
header-style: text
tags:
  - azure
  - azure functions
  - csharp
  - dotnet
---

# Azure Functions 網路管理

下圖來自 [Azure Functions networking options](https://docs.microsoft.com/en-us/azure/azure-functions/functions-networking-options) 展示不同方案中可以做的網路設定。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/azure-function-network-1.png)

## 限制可以直接存取 Function App 的 IP

在 **Networking** -> **Access Restriction** 設定 IP 及其子網路存取 Function App 的規則。每條規則的 Priority 的值越小，會越先被採納。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/azure-function-network-2.png)

## 使用 VNet 與 NAT Gateway 限制 outbound request 的 IP

Azure Functions 的 App Service Plan 和 Premium Plan 允許開發者使用 VNet 來配置對外 IP，如此一來就方便我們去設定第三方服務的 Allow IP List。

首先登入 Azure 並創建一個測試用的 Resource Group。

```non
> az login
> az group create --name Jack-Test --location japaneast
```

新增一個 Storage Account。

```non
> az storage account create --name jack1 --location japaneast --resource-group Jack-Test --sku Standard_LRS
```

接著參考 [az functionapp plan](https://docs.microsoft.com/en-us/cli/azure/functionapp/plan?view=azure-cli-latest) 新增一個最便宜的 App Service (D1) 方案、用 Windows 系統、命名為 Plan1。

```non
> az functionapp plan create --resource-group Jack-Test --name Plan1 --min-instances 1 --sku D1
```

創建一個 Function App 並套用剛剛新增的 Plan1。

```non
> az functionapp create --resource-group Jack-Test --runtime dotnet --functions-version 4 --name jack-linux --storage-account jack1 --plan Plan1
```

在 Azure Portal 新增一個 VNet。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/azure-function-network-3.png)

啟用 jack-linux 的 VNet Integration，將剛剛創建的 VNet 啟用。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/azure-function-network-4.png)

接著新增一個 Public IP Address。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/azure-function-network-5.png)

新增一個 NAT Gateway，將剛剛的 Function App 所用到的 Subnet 加進去。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/azure-function-network-6.png)

將 Function 推送到 Azure 上。

```non
> func azure functionapp publish jack-nat-function
```

最後在 **Configuration** 設定環境變數 `WEBSITE_VNET_ROUTE_ALL` 的值為 `1`，如此一來 Function App 的 outbound request 就會固定使用剛剛創建的 Public IP Address。