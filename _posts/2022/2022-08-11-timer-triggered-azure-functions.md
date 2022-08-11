---
title: "初探 Timer-Triggered 的 Azure Functions"
subtitle: ""
excerpt: "azure functions timer triggered ncrontab"
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

- 安裝 .NET 6.0 SDK
- 安裝 Azure Functions Core Tools 4.x 版
- Azure CLI 2.4.0 版或更新版本。
- 安裝 Azurite

安裝完畢後請確認 `az`、`func` 和 `azurite` 指令可以被正常執行，如果失敗請重新安裝或手動將 `az` 和 `func` 的目錄加入環境變數。

檢查各項工具的版本是否夠新：

```non
> func --version
4.0.4670
> dotnet --list-sdks
6.0.301
> az --version
azure-cli                         2.39.0
core                              2.39.0
> azurite -h
```

## 建立本機專案

首先啟動 Azure Storage 的模擬器，因為 

執行 `func init` 命令，在 CronApp 的資料夾中建立 .NET 的 Azure Functions 專案：

```non
> func init CronApp --dotnet
```

將基於 `Timer trigger` 的 Function 新增至專案：

```non
> cd CronApp
> func new --name CronApp --template "Timer trigger"
```

`Timer trigger` 依賴 [NCrontab](https://github.com/atifaziz/NCrontab) 套件，這個套件需要透過 `Azure Storage` 儲存一個檔案當作 lock，確保當 Function App 擴展到多個實例時，只使用唯一一個 timer 實例。

此外，如果兩個 Function App 共享相同的 identifying configuration，並且個別使用一個 timer，則只有一個 timer 會運作。

所以接下來先啟動 Azurite 以在本機模擬 Azure Storage:

```non
> azurite --queueHost 127.0.0.1 --location azurite_workspace
```

在上述指令中透過 `--location` 選項，Azurite 所產生的暫存檔會放在 `azurite_workspace` 中。

在執行程式之前，修改一下 `CronApp.cs` 的規則使其每 5 秒觸發一次。
- 第 11 行: 把預設的字串改為 `"*/5 * * * * *"`。`TimerTriggerAttribute` 的原始碼在 [WebJobs.Extensions/Extensions/Timers/TimerTriggerAttribute.cs](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/dev/src/WebJobs.Extensions/Extensions/Timers/TimerTriggerAttribute.cs)。

```csharp
using System;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;

namespace CronApp
{
    public class CronApp
    {
        [FunctionName("CronApp")]
        public void Run([TimerTrigger("*/5 * * * * *")]TimerInfo myTimer, ILogger log)
        {
            log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
        }
    }
}
```

執行程式:

```non
> func start
```

每 5 秒 CronApp 會輸出一條訊息:

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/timer-trigger1.png)

## 佈署到 Azure

流程與 [初探 Azure Functions](https://blueskyson.github.io/2022/08/10/azure-functions-get-started/) 一樣:

```non
> az login
> az group create --name Functions-rg --location japanwest
> az storage account create --name jacklin --location japanwest --resource-group Functions-rg --sku Standard_LRS
> az functionapp create --resource-group Functions-rg --consumption-plan-location japanwest --runtime dotnet --functions-version 4 --name jacklin-function --storage-account jacklin
> func azure functionapp publish jacklin-function
```

佈署到 Azure 後透過 log stream 查看 CronApp 是否有被觸發:

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/timer-trigger2.png)

## 注意事項

NCrontab 已經兩年沒有新的 commit 了，但是 WebJobs.Extensions/Extensions/Timers 還持續有在更新，所以 Azure 團隊應該有在持續維護。