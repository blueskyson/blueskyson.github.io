---
title: "在 Azure Functions 使用 Key Vault"
subtitle: ""
excerpt: "azure functions key vault"
layout: post
author: "blueskyson"
header-style: text
tags:
  - azure
  - azure functions
  - csharp
  - dotnet
---

## 建立一個 Key Vault

首先在 Azure 建立 Key Vault 命名為 jack-keyvault，裡面有:

| Secret  | Value       |
| ------- | ----------- |
| TestKey | Hello World |

如下圖：

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/azure-function-key-vault-1.png)

## 在本機測試 Function App

在本機創建一個 Function App：

```non
> func init KeyVaultFunction --dotnet
```

安裝 Key Vault 的相依套件：

```non
> cd KeyVaultFunction
> dotnet restore
> dotnet add package Azure.Identity
> dotnet add package Azure.Security.KeyVault.Secrets
```

然後新增一個 Http-Triggered 的 Function，我將其命名為 `HttpTrigger.cs` 並貼上以下程式碼：

```csharp
using System;
using Azure.Security.KeyVault.Secrets;
using Azure.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;

namespace KeyVaultFunction
{
    public class HttpTrigger
    {
        [FunctionName("HttpTrigger")]
        public IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]
                HttpRequest req,
            ILogger log
        )
        {
            try {
                string keyVaultUrl = Environment.GetEnvironmentVariable("KEY_VAULT_URL")!;
                string secretName = Environment.GetEnvironmentVariable("SECRET_NAME")!;

                var client = new SecretClient(new Uri(keyVaultUrl), new DefaultAzureCredential());
                KeyVaultSecret secret = client.GetSecret(secretName);
                log.LogInformation($"Successfully get Key Vault from: {keyVaultUrl}. Secret name: {secretName}");

                return new OkObjectResult(secret.Value);
            }
            catch (Exception ex)
            {
                log.LogInformation($"Exception occurred. Source: {ex.Source}. Message: {ex.Message}");
                return new BadRequestObjectResult($"Exception occurred. Source: {ex.Source}. Message: {ex.Message}");
            }
        }
    }
}
```

當使用者觸發這個 Function 後，這份程式碼會從 `local.settings.json` 中讀取 `KEY_VAULT_URL` 和 `SECRET_NAME` 的值，然後請求 Key Vault 回傳該秘密的值，最後顯示把結果透過 `OkObjectResult` 傳回給使用者。

接下來在 local.settings.json 貼上 `KEY_VAULT_URL` 以及 `SECRET_NAME`：

```json
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "FUNCTIONS_WORKER_RUNTIME": "dotnet",
        "KEY_VAULT_URL": "https://jack-keyvault.vault.azure.net/",
        "SECRET_NAME": "TestKey"
    }
}
```

然後就可以在本地測試了。先登入 Azure 來讓 `SecretClient` 可以驗證本機的身分，然後執行 Function App：

```
> az login
> func start
```

測試 Function App 能否取得 secret：

```
> curl http://localhost:7071/api/HttpTrigger
Hello World
```

## 在 Azure 測試 Function App

首先創建一個 Function App，我將其命名為 Jack1，然後啟用它的 Identity，然後按下 Save:

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/azure-function-key-vault-2.png)

在 Configuration 填上剛剛在 `local.settings.json` 出現的 `KEY_VAULT_URL` 以及 `SECRET_NAME`，然後按下 Save:

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/azure-function-key-vault-3.png)


然後回到 jack-keyvault 新增一個 Access Polocy，然後按下 Save，讓 Jack1 可以取得秘密資料:

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2022/azure-function-key-vault-4.png)

然後將 Function App 推送到 Azure 上:

```non
> func azure functionapp publish Jack1
```

然後透過瀏覽器開啟 https://jack1.azurewebsites.net/api/httptrigger 即可看到 Hello World 字串。
