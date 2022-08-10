---
title: "初探 Azure Functions"
subtitle: "Azure Functions"
excerpt: "Clean Code"
layout: post
author: "blueskyson"
header-style: text
tags:
  - azure
  - azure functions
  - csharp
  - dotnet
---

## 簡介

Azure Functions 是一種無伺服器解決方案，其運行的應用程式被稱為 Function App，可讓您編寫較少的程式碼、維護更少的基礎架構並節省成本。Function App 最常用來回應事件，如資料庫變更、IoT 資料流、訊息佇列等。

Azure Functions 相較於其他解決方案有兩項特點:
- 將系統的邏輯實現為 code blocks，每個 code block 稱為 function，您可以在任何時間點執行個別 function 來響應關鍵事件。
- 隨著需求增加，Azure Functions 會根據需求量來啟用盡可能多的資源和 function 實例來滿足需求。隨著需求下降，任何額外的資源和 Function App 實例都會自動下降。

支援的功能有:
- Build a web API
- Process file uploads
- Build a serverless workflow
- Respond to database changes
- Run scheduled tasks
- Create reliable message queue systems
- Analyze IoT data streams
- Process data in real time

在構建 Function 時，您可以使用以下選項和資源：

- **Use your preferred language**: 使用 C#、Java、JavaScript、PowerShell 或 Python 編寫 Function，或透過 custom handler 來使用其他程式語言。
- **Automate deployment**: 除了 Azure 以外，也支援 container 布署、App Service 布署 (例如 Kudu)、透過 Github Actions 布署。
- **Troubleshoot a function**: 使用監控工具和測試策略來深入了解您的應用。
- **Flexible pricing options**: With the Consumption plan, you only pay while your functions are running, while the Premium and App Service plans offer features for specialized needs.

## 開發環境

透過命令列開發：

- 安裝 .NET 6.0 SDK
- 安裝 Azure Functions Core Tools 4.x 版
- Azure CLI 2.4.0 版或更新版本。

安裝完畢後請確認 `az` 和 `func` 指令可以被正常執行，如果失敗請重新安裝或手動將 `az` 和 `func` 的目錄加入環境變數。

檢查各項工具的版本是否夠新：

```non
> func --version
4.0.4670
> dotnet --list-sdks
6.0.301
> az --version
azure-cli                         2.39.0
core                              2.39.0
```

## 建立本機專案

執行 `func init` 命令，在 FunctionApp 的資料夾中建立 .NET 的 Azure Functions 專案：

```non
> func init FunctionApp --dotnet
```

FunctionApp 包含專案的設定檔，其中 `local.settings.json` 可能會包含從 Azure 下載的機密資訊，因此必須將其加入 `.gitignore`。

將 Function 新增至專案，其中 `HttpExample` 是 Function 的名稱，而 `HTTP trigger` 為方法的觸發方式。

```non
> cd FunctionApp
> func new --name HttpExample --template "HTTP trigger" --authlevel "anonymous"
```

執行完上述指令後會自動產生一份 `HttpExample.cs`，這就是 Azure Functions 的主程式。

先在本機執行以下指令啟動 `HttpExample`:

```non
> func start
```

接下來連線至 http://localhost:7071/api/HttpExample
可以看到以下畫面:

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/azure-functions-1.png)

把網址替換為 http://localhost:7071/api/HttpExample?name=xxxx

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/azure-functions-2.png)

## 範例程式碼說明

此範例為 [HTTP triggers and bindings](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-http-webhook) 程式，用於回應使用者給的 Http Request。

基於 C# 的 Functions 又分為 [in-process](https://docs.microsoft.com/en-us/azure/azure-functions/functions-dotnet-class-library)、[isolated process](https://docs.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide)、[Script (.csx)](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-csharp) 三種模式，此範例單純使用 in-process 來開發，不會詳細討論其他模式。

### host.json

host.json 中的設定值適用於 Azure Functions 實例中的所有函數。

```json
{
    "version": "2.0",
    "logging": {
        "applicationInsights": {
            "samplingSettings": {
                "isEnabled": true,
                "excludedTypes": "Request"
            }
        }
    }
}
```

### HttpExample.cs

- 第 14 行: `[FunctionName("HttpExample")]` 定義此 Function 的名稱為 `HttpExample`，這個名稱與類別名稱或檔名無關，可以自由定義。每個要編譯成 Function 的方法 (如範例中的 `Run`) 都必須要有這個 Attribute，否則 `func` 在編譯時就不會產生這個 Function 的 Route。
- 第 16 行: `[HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req` 用來綁定 Http 觸發器的規則，詳見 [Azure Functions HTTP trigger#Attributes](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-http-webhook-trigger?tabs=in-process%2Cfunctionsv2&pivots=programming-language-csharp#attributes)。
  因為 `Route` 參數為 `null`，所以預設會將 `/api/{FunctionName}` 路徑綁定到 `HttpExample`，即 `https://localhost:7071/api/HttpExample`。
- 19 到 29 行: 先解析網址的是否包含 `name` 的鍵值，再以 json 格式解析 Request Body 是否有 `name` 的鍵值，然後產生 `Hello, {name}...` 字串。
- 31 行: `OkObjectResult` 繼承自 `Microsoft.AspNetCore.Mvc.ObjectResult`，如果回應成功，將產生一個 Status 200 OK。

```csharp
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;

namespace FunctionApp
{
    public static class HttpExample
    {
        [FunctionName("HttpExample")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            string name = req.Query["name"];

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            dynamic data = JsonConvert.DeserializeObject(requestBody);
            name = name ?? data?.name;

            string responseMessage = string.IsNullOrEmpty(name)
                ? "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response."
                : $"Hello, {name}. This HTTP triggered function executed successfully.";

            return new OkObjectResult(responseMessage);
        }
    }
}
```

## 創建所需的 Azure 資源

登入 Azure 帳戶:

```non
> az login
```

新增一個 Resource Group:
- `--name`: Resource Group 名稱。
- `--location`: 指定 Resource Group 的區域。可以用 `az account list-locations` 查看可用區域代碼。

```non
> az group create --name Functions-rg --location japanwest
```

在 Resource Group 中創建 general-purpose Storage Account:
- `--name`: Storage Account 名稱。
- `--location`: 指定 Storage Account 的區域。
- `--resource-group`: Resource Group 名稱。
- `--sku`: 選擇 Azure 的定價單位。 

```non
> az storage account create --name jacklin --location japanwest --resource-group Functions-rg --sku Standard_LRS
```

在 Azure 中創建 Function App:
- `--resource-group`: Resource Group 名稱。
- `--consumption-plan-location`: Function App 布署的地理位置。可以用 `az functionapp list-consumption-locations` 查看可用位置。
- `--runtime`: 指定 runtime stack。
- `--functions-version`: Function App 的版本，目前可接受的版本為 `2`、`3`、`4`。
- `--name`: Function App 在 Azure 上的名稱，布署後透過 `https://{name}.azurewebsites.net` 來連上 Function App。
- `--storage-account`: 此 Resource Group 中的 Storage Account 的字串值。若要用別的 Resource Group 的 Storage Account，則是用 Resource ID。

```non
> az functionapp create --resource-group Functions-rg --consumption-plan-location japanwest --runtime dotnet --functions-version 4 --name jacklin-function --storage-account jacklin
```

## 布署到 Azure Functions

透過以下指令布署，其中 `jacklin-function` 為前面步驟中建立的 Azure Function App 的名稱；`--force` 略過布署前的檢查。

```non
> func azure functionapp publish jacklin-function
...
Functions in jacklin-function:
    HttpExample - [httpTrigger]
        Invoke url: https://jacklin-function.azurewebsites.net/api/httpexample
```

接下來連線到以下網址確認 Function App 是否成功執行。

https://jacklin-function.azurewebsites.net/api/httpexample

## 新增其他 Function

將 HttpExample.cs 改寫如下:

- 第 37 行: 定義新的 Function 的名稱為 `MyProduct`。
- 第 39 到 44 行: 定義路徑為 `products/{category:alpha}/{id:int?}`，並且綁定網址與 `MyProductRun` 參數欄位中的 `category` 和 `id`。

```csharp
using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;

namespace FunctionApp
{
    public static class HttpExample
    {
        [FunctionName("HttpExample")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]
                HttpRequest req,
            ILogger log
        )
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            string name = req.Query["name"];
            Console.WriteLine(name);
            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            dynamic data = JsonConvert.DeserializeObject(requestBody);
            name = name ?? data?.name;

            string responseMessage = string.IsNullOrEmpty(name)
                ? "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response."
                : $"Hello, {name}. This HTTP triggered function executed successfully.";

            return new OkObjectResult(responseMessage);
        }

        [FunctionName("MyProduct")]
        public static IActionResult MyProductRun(
            [HttpTrigger(
                AuthorizationLevel.Anonymous,
                "get",
                "post",
                Route = "products/{category:alpha}/{id:int?}"
            )] HttpRequest req,
            string category,
            int? id,
            ILogger log
        )
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            var message = String.Format($"Category: {category}, ID: {id}");
            return (ActionResult)new OkObjectResult(message);
        }
    }
}
```

啟動 Function App:

```non
> func start
```

除了原先的 http://localhost:7071/api/HttpExample?name=xxxx 之外，還有一個新的 Function http://localhost:7071/api/products/xxxx/1 ，你可以嘗試把 `xxxx` 和 `1` 替換為其他值。

```non
> func azure functionapp publish jacklin-function
...
Functions in jacklin-function:
    HttpExample - [httpTrigger]
        Invoke url: https://jacklin-function.azurewebsites.net/api/httpexample1

    MyProduct - [httpTrigger]
        Invoke url: https://jacklin-function.azurewebsites.net/api/products/{category:alpha}/{id:int?}
```

## 刪除所有 Azure 上的資源

```non
> az group delete --name Functions-rg
```