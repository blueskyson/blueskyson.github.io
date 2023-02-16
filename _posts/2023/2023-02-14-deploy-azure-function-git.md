---
title: "透過 Local Git 佈署 Azure Functions"
subtitle: ""
excerpt: "azure functions local git deploy"
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

## 透過 User scope 部屬

測試完成後，先創建一個 App Service Plan 的 Azure Function，透過以下指令部屬到 Azure：

```non
export RG_NAME=<your resource group name>
export APP_NAME=<your function app name>

az login
az functionapp deployment source config-local-git --resource-group $RG_NAME --name $APP_NAME
```

此時會得到一個連結類似如下：

```non
{
  "url": "https://jacklin@jack123.scm.azurewebsites.net/jack123.git"
}
```

這個 URL 就是 Azure 上的 Git Repository URL，將他複製下來。

這時請先到 Azure Portal 上設定使用者的密碼：

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/azure-function-git-user.png)


然後再用以下指令將本地的 Function App 推送到這個 Git Repository 中。在終端機會提示使用者輸入剛剛設定的密碼，驗證完成後才能推送到遠端部屬：

```non
git push -f https://jacklin@jack123.scm.azurewebsites.net/jack123.git master:master
Password for 'https://jacklin@jack123.scm.azurewebsites.net':
```

驗證完成後遠端會傳來以下訊息，表示在 Azure 上正在編譯程式碼：

```non
Enumerating objects: 10, done.
Counting objects: 100% (10/10), done.
Delta compression using up to 16 threads
Compressing objects: 100% (8/8), done.
Writing objects: 100% (10/10), 3.69 KiB | 3.69 MiB/s, done.
Total 10 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Deploy Async
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id '9f991b44d8'.
remote: PreDeployment: context.CleanOutputPath False
remote: PreDeployment: context.OutputPath /home/site/wwwroot
remote: Repository path is /home/site/repository
remote: Running oryx build...
remote: Operation performed by Microsoft Oryx, https://github.com/Microsoft/Oryx
remote: You can report issues at https://github.com/Microsoft/Oryx/issues
remote: 
remote: Oryx Version: 0.2.20220812.1, Commit: cdf6b1bef165d05b94830e963646495967d938f4, ReleaseTagName: 20220812.1
remote: 
remote: Build Operation ID: |+XRCL91nA20=.d1126c1d_
remote: Repository Commit : 9f991b44d88cb4f924467091a9d0a361652eeccc
remote: 
remote: Detecting platforms...
remote: Detected following platforms:
remote:   dotnet: 6.0.13
remote: Version '6.0.13' of platform 'dotnet' is not installed. Generating script to install it...
remote: 
remote: Using intermediate directory '/tmp/8db0fe0aaf41a8a'.
remote: 
remote: Copying files to the intermediate directory...
remote: Done in 0 sec(s).
remote: 
remote: Source directory     : /tmp/8db0fe0aaf41a8a
remote: Destination directory: /tmp/build/expressbuild
remote: 
remote: 
remote: Downloading and extracting 'dotnet' version '6.0.405' to '/tmp/oryx/platforms/dotnet/6.0.405'...
remote: .................
remote: Downloaded in 22 sec(s).
remote: Verifying checksum...
remote: Extracting contents...
remote: ..................
remote: performing sha512 checksum for: dotnet...
remote: Done in 48 sec(s).
remote: 
remote: 
remote: Using .NET Core SDK Version: 6.0.405
remote: ...............................................
remote: 
remote: Welcome to .NET 6.0!
remote: ---------------------
remote: SDK Version: 6.0.405
remote: 
remote: Telemetry
remote: ---------
remote: The .NET tools collect usage data in order to help us improve your experience. It is collected by Microsoft and shared with the community. You can opt-out of telemetry by setting the DOTNET_CLI_TELEMETRY_OPTOUT environment variable to '1' or 'true' using your favorite shell.
remote: 
remote: Read more about .NET CLI Tools telemetry: https://aka.ms/dotnet-cli-telemetry
remote: 
remote: ----------------
remote: Installed an ASP.NET Core HTTPS development certificate.
remote: To trust the certificate run 'dotnet dev-certs https --trust' (Windows and macOS only).
remote: Learn about HTTPS: https://aka.ms/dotnet-https
remote: ----------------
remote: Write your first app: https://aka.ms/dotnet-hello-world
remote: Find out what's new: https://aka.ms/dotnet-whats-new
remote: Explore documentation: https://aka.ms/dotnet-docs
remote: Report issues and find source on GitHub: https://github.com/dotnet/core
remote: Use 'dotnet --help' to see available commands or visit: https://aka.ms/dotnet-cli
remote: --------------------------------------------------------------------------------------
remote:   Determining projects to restore...
remote:   Restored /tmp/8db0fe0aaf41a8a/FunctionApp.csproj (in 46.08 sec).
remote: 
remote: Publishing to directory /tmp/build/expressbuild...
remote: 
remote: ............................................
remote: MSBuild version 17.3.2+561848881 for .NET
remote:   Determining projects to restore...
remote:   All projects are up-to-date for restore.
remote:   FunctionApp -> /tmp/8db0fe0aaf41a8a/bin/Release/net6.0/FunctionApp.dll
remote:   FunctionApp -> /tmp/build/expressbuild/
remote: Preparing output...
remote: 
remote: Removing existing manifest file
remote: Creating a manifest file...
remote: Manifest file created.
remote: Copying .ostype to manifest output directory.
remote: 
remote: Done in 153 sec(s).
remote: Writing the artifacts to a Zip file
remote: Running post deployment command(s)...
remote: 
remote: Generating summary of Oryx build
remote: Deployment Log file does not exist in /tmp/oryx-build.log
remote: The logfile at /tmp/oryx-build.log is empty. Unable to fetch the summary of build
remote: Triggering recycle (preview mode disabled).
remote: Deployment successful. deployer =  deploymentPath =
remote: Deployment Logs : 'https://jack123.scm.azurewebsites.net/newui/jsonviewer?view_url=/api/deployments/9f991b44d88cb4f924467091a9d0a361652eeccc/log'
To https://jack123.scm.azurewebsites.net/jack123.git
 * [new branch]      master -> master
```

如此一來便完成 Local Git 部屬。Azure 上預設執行的 Git Branch 預設為 `master`，所以要透過 `my_branch:master` 將本地的分支推送到 `master` 分支上。

如果 Git 根目錄中沒有 `.csproj` 檔，則需要額外新增一個 `.deployment` 檔案，內容如下：

```non
[config]
project = path/to/FunctionApp.csproj
```

## 透過 Application Scope Credentials 部屬

```non
export RG_NAME=<your resource group name>
export APP_NAME=<your function app name>

az login
az functionapp deployment source config-local-git --resource-group $RG_NAME --name $APP_NAME
az functionapp deployment list-publishing-credentials --resource-group $RG_NAME --name $APP_NAME --query scmUri --output tsv
```

此時會得到一個連結類似如下：

```non
https://$jack123:txe7nidoc6epafaCeeq2kKYpNidfrStvTwzdlM6FkpT4j11sERTEj826RNM0@jack123.scm.azurewebsites.net
```

將他複製下來，然後再用以下指令將本地的 Function App 推送到這個 Git Repository 中，注意你的 Git 遠端連結應該是 https://...scm.azurewebsites.net/APP_NAME.git 的形式，並且 user name 會有 `$` 字元，所以在 bash 中應該要把網址用 `''` 括起來才能正常執行：

```non
git push -f 'https://$jack123:txe7nidoc6epafaCeeq2kKYpNidfrStvTwzdlM6FkpT4j11sERTEj826RNM0@jack123.scm.azurewebsites.net/jack123.git' master:master
```

如此一來便完成 Local Git 部屬。這個方式比較適合自動化，因為 Function App 被創建時就會自動產生 Application Scope Credentials 的帳號密碼而且可以直接透過 az cli 產出連線網址。
