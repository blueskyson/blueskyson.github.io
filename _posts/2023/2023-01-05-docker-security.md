---
title: "Docker Security 筆記"
subtitle: ""
excerpt: "docker security 掃描 scan trivy snyk osv-scanner inspec"
layout: post
author: "blueskyson"
header-style: text
tags:
  - docker
---

## Docker Security

參考來源：[https://www.prplbx.com/resources/blog/docker-part1/](https://www.prplbx.com/resources/blog/docker-part1/)

儘管 Docker 很受歡迎，仍有許多軟體開發人員避免使用它，因為他們認為 docker 安全性很弱。 Docker 容器包括執行檔、設定檔、相關依賴項、主機環境和網路配置。其中每一個都是淺在的弱點。例如舊函式庫中的漏洞會對整個 docker 容器構成威脅。由於 Docker 容器中有許多不同的技術，安全管理也會變得很複雜。以下為 Docker 的安全實踐：

### 1. 避免用 root 執行

58% 的 docker 鏡像是使用 root 權限運行，攻擊者可以透過 root 訪問所有資訊並可以直接控制硬體，因此 docker 中的 process 最好以最小權限創建的使用者執行。例如以下範例：

```dockerfile
FROM ubuntu:20.04
RUN apt-get update -y && \
apt-get install -y python3-pip python3-dev
USER test
CMD test test.py
```

### 2. 使用信任來源的 Base Image、為自己打包的映像檔簽名

您應該只從受信任的存儲庫中提取基本 Docker 映像檔，或是檢查 checksums 和數位簽名以驗證鏡像的真實性和完整性。

### 3. 更新 Docker 映像檔的作業系統、軟體

我們每天都會遇到數百個與作業系統和函式庫相關的 CVE，維持 Docker 鏡像為最新並創建自動化的更新機制很重要。

### 4. 關閉未使用的連接埠

為測試而開放的連接埠（如 SSH）或尚未確定使用目的的連接埠應在生產環境中關閉。

### 5. 不要用明文儲存機敏資訊

寫死的（API 密鑰、AWS 憑據等）為攻擊者敞開大門。您的 Docker 映像檔不應包含與任何環境（開發、測試、生產等）相關的任何 hardcoded 的機敏資訊。您應該通過環境變數（使用 -e 選項）或使用密碼管理器獲取憑據。

### 6. 定期掃描 Docker 映像檔

有很多簡單易用的 Docker 容器的開源工具掃描工具，它們透過 [Common Vulnerabilities and Exposures](http://cve.mitre.org/cve/) (CVE)、[Center for Internet Security](https://www.cisecurity.org/) (CIS) 數據庫來檢測。

## Docker 掃描工具

參考來源：[https://www.prplbx.com/resources/blog/docker-part2/](https://www.prplbx.com/resources/blog/docker-part2/)

### 1. Docker Bench for Security

Git: [https://github.com/docker/docker-bench-security](https://github.com/docker/docker-bench-security)

由 Docker 官方維護的專案，透過 shell script 或是 container 執行掃描，掃瞄範圍涵蓋 Containerd、Docker Daemon、所有他找的到的 Dockerfile、Image、Container。也可以只選擇掃描 image 和 container 就好。

```bash
git clone [https://github.com/docker/docker-bench-security.git](https://github.com/docker/docker-bench-security.git)
cd docker-bench-security
sudo sh docker-bench-security.sh                 # 方法 1
docker-compose run --rm docker-bench-security    # 方法 2
```

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/dockersec1.png)

檢測項目包含：
1. **Host Configuration:**
    - 透過 auditd 來稽核 /etc、/run、/var、/usr/bin 中的 Docker 設定檔與執行檔，確保所有 docker 操作可以被收集、儲存與查閱 (audit 可能會大量產生 log file，須注意清除 log 的機制或考慮將主要系統與 log 掛載在不同硬碟分區)
    - 檢查 Docker 是否為最新版本。
2. **Docker daemon configuration:**
    - user namespace
    - 開啟 docker cli 驗證
    - 啟用 live restore
    - 禁止要求新的權限
    - ...
3. **Docker daemon configuration files:** 檢查 socket、cert、config 權限。
4. **Container Images and Build File:**
    - 檢查 Dockerfile 中是否透過 root 身分進行操作
    - Dockerfile 中必須包含 [HEALTHCHECK](https://docs.docker.com/engine/reference/builder/#healthcheck) 關鍵字。
5. **Container Runtime:** 
    - 不要共享 network namespace (它允許容器訪問 host os 上的網路服務如 D-bus，因此，容器可能會做一些意想不到的事情，比如關閉 Docker)
    - 限制容器記憶體大小 (防止 denial of service)
    - 設定容器的 CPU Priority
    - 容器的 root filesystem 必須為 readonly、新增獨立的 volume 來寫入程式的資料 (防止容器被竄改)
    - 不要讓容器可以透過 0.0.0.0 連到 host os (無顯著危險，但建議這樣做)
    - 確認 MaximumRetryCount 為 5 (防止 denial of service on the host)
    - 設定容器的 PIDs cgroup limit (防止 fork bomb)
    - ...
6. **Docker Security Operations:** Ensure image sprawl is avoided
7. **Docker Docker Swarm Configuration:** 有使用 Docker Swarm 叢集才需要注意

各個細項都對應一個 CIS Docker 的編號，解決方法可以參考 Tenable 的 [CIS Docker](https://tenable.com/audits/items/search?q=CIS+Docker&sort=&page=1)。

如果上述檢查太繁瑣，可以透過以下方法挑選特定想要檢查的規則：

```bash
# Only check 2.2
sh docker-bench-security.sh -c check_2_2
# Check all except 2.2
sh docker-bench-security.sh -e check_2_2
# Only check rules of Docker Enterprise license
sh docker-bench-security.sh -c docker_enterprise_configuration
# Only check 4.x and 5.x
sh docker-bench-security.sh -c container_images,container_runtime
# Only check 4.x and 5.x except 4.5
sh docker-bench-security.sh -c container_images,container_runtime -e check_4_5
```

另外也可以選擇特定的 image 來掃描：

```bash
# Only include specific image
sh docker-bench-security.sh -i <image name>
```

也可以在產生掃描報告後，印出各個弱點的解決方法的摘要：

```bash
sh docker-bench-security.sh -p
```

### 2. InSpec

InSpec 是 Docker 映像檔的開源測試框架，由 [Chef](https://www.chef.io/) 維護。您可以根據各種標準的 baseline 檢測 Docker 容器的狀態，他們提供的 baseline 有 Linux baseline、CIS docker benchmark、nginx-baseline、ssh-baseline 等，詳細列表可以在[這裡](https://github.com/dev-sec)找到。

```bash
sudo dpkg -i /path/to/inspec.deb
git clone https://github.com/dev-sec/linux-baseline
cd linux-baseline
# 檢測容器的 os
inspec exec https://github.com/dev-sec/linux-baseline -t docker://<container hash>
# 檢測 host os
inspec exec https://github.com/dev-sec/linux-baseline 
```

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/dockersec2.png)

linux baseline 執行後產生的報表主要著重在各個 /etc 目錄中的設定檔的權限，以及建議 Kernel Parameter 的設定值。

另外也可以用以下指令執行 cis docker benchmark，他檢查的規則跟先前使用的 docker-bench-security.sh 一樣。

```
git clone https://github.com/dev-sec/cis-docker-benchmark.git
cd cis-docker-benchmark
inspec exec https://github.com/dev-sec/cis-docker-benchmark.git -t docker://<container hash>
```

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/dockersec3.png)

### 3. Trivy

Trivy 是 Aqua Security 開源專案，用於掃描 docker 映像檔的漏洞，Trivy 還可以掃描目錄和 git repo，它可以在幾分鐘內檢測到 CVE 漏洞。

```bash
sudo apt install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt update
sudo apt install trivy

# 方法一
trivy image --clear-cache
trivy image <registry>/<image name>:<tag>

# 方法二
docker run aquasec/trivy
```

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/dockersec4.png)

由上圖可以得知掃描報告列出：linux 系統套件 (apt、bash、libc 等)、程式相依套件（dotnet library、package.json、Pipfile 等） 的漏洞並指出該把函式庫更新到哪個版本才修復該漏洞。其餘 trivy 主要用法：

- 掃描當前目錄和子目錄：
    ```bash
    trivy fs .
    ```
- 掃描 git repo：
    ```bash
    trivy repo https://github.com/knqyf263/trivy-ci-test
    ```
- 只掃描特定漏洞（漏洞種類有 `vuln,config,secret,license`，預設值為 vuln,secret）、（漏洞等級有 `UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL`，預設為全部）：
    ```bash
    trivy image <image name> --security-checks vuln,confi
    trivy image <image name> --severity HIGH,CRITICAL
    ```
- 掃描檔案系統根目錄：
    ```bash
    # 方法一
    trivy rootfs /path/to/rootfs

    # 方法二：Standalone mode: 在容器裡執行 trivy rootfs
    docker run --rm -it alpine:3.11
    / # curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
    / # trivy rootfs /

    # 方法三：Client/Server mode: 從 host os 觸發容器內的 agent 進行掃描
    trivy server
    trivy rootfs --server http://localhost:4954 --severity CRITICAL /tmp/rootfs
    ```
- Custom [Policies](https://aquasecurity.github.io/trivy/v0.35/docs/misconfiguration/custom/)：
    可以搭配 terrafrom、Docker、RBAC 等，透過 yml、tf、Dockerfile 等格式自訂規則，如果不符合某些安全條件，就會中止佈署程序。
    ```go
    package user.terraform.ID007

    __rego_metadata__ := {
        "id": "ID007",
        "title": "ASG desires too much capacity",
        "severity": "MEDIUM",
        "type": "Terraform Plan Check",
    }

    __rego_input__ := {"selector": [{"type": "json"}]}

    deny[msg] {
        resource := input.planned_values.root_module.resources[_]
        resource.type == "aws_autoscaling_group"
        resource.values.desired_capacity > 10

        msg = sprintf("ASG '%s' desires too much capacity", [resource.name])
    }
    ```

### 4. Snyk

[https://snyk.io/learn/docker-security-scanning/](https://snyk.io/learn/docker-security-scanning/)

Snyk 是開發者資安平台，提供 IDE、GitHub、Docker、ACR 等的整合工具，在 Docker 中，Snyk 被整合在 `docker scan` 中。ACR 整合的文件在[這邊](https://docs.snyk.io/products/snyk-container/image-scanning-library/acr-image-scanning)，他可以從 snyk 提供的 web 界面查看存放在 ACR 的映像檔的漏洞。

```bash
# Snyk requires DockerHub account or Snyk account
docker scan --login
docker scan <image name> --file=<docker file>
```

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/dockersec5.png)

他掃描的弱點大部分來自 CVE。

除了用整合的 `docker scan` 以外，也可以用 snyk cli 來掃描，掃描結果是一樣的。

```bash
snyk container test <image name>
```

Pricing: DockerHub 帳號每個月 10 次免費、Snyk 免費帳號每個月 100 次免費

### 5. Microsoft Defender for Containers

Powerd by Qualys。

Pricing：[https://azure.microsoft.com/zh-tw/pricing/details/defender-for-cloud/](https://azure.microsoft.com/zh-tw/pricing/details/defender-for-cloud/)

詳見官網：[https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-containers-vulnerability-assessment-azure](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-containers-vulnerability-assessment-azure)

啟用方式：
- [https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-containers-vulnerability-assessment-azure](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-containers-vulnerability-assessment-azure)
- [https://learn.microsoft.com/en-us/azure/defender-for-cloud/kubernetes-workload-protections](https://learn.microsoft.com/en-us/azure/defender-for-cloud/kubernetes-workload-protections)
- [https://www.youtube.com/watch?v=IHNdJnDttoU](https://www.youtube.com/watch?v=IHNdJnDttoU)

首先啟用訂用帳戶的 Microsoft Defender for Cloud，然後進入 Microsoft Defender for Cloud，進到 Environment settings -> (Azure > Tenant Root Group > subscription name) -> (Containers 右側 Partial Settings) 啟用 Defender DaemonSet

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/dockersec6.png)


然後推送任意映像檔到 ACR，等一段時間後即可從 [Container registry images should have vulnerability findings resolved](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/dbd0cb49-b563-45e7-9724-889e799fa648) 看到掃描報告。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/dockersec7.png)

### 6. OSV-Scanner

Google 維護的掃描工具，可以掃描 Docker image、依賴套件的 Lock File、sbom ([Software Bill of Materials](https://en.wikipedia.org/wiki/Software_supply_chain))。

```bash
# install golang
sudo apt install golang-go
export GOPATH="$HOME/go"
PATH="$GOPATH/bin:$PATH"

# install osv-scanner
go install github.com/google/osv-scanner/cmd/osv-scanner@v1

# scan an image
osv-scanner --docker <image name>:<tag>
```

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/dockersec8.png)

他的預設報告格式是 table 如上圖，另外也可以用 `--json` 格式。

## 掃描工具比較



|                     | Docker Bench for Security                                       | InSpec                                | Trivy                                                               | Snyk                                                                                             | Microsoft Defender for Containers | OSV-Scanner                    |
| ------------------- |:--------------------------------------------------------------- | ------------------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | --------------------------------- | ------------------------------ |
| 掃瞄範圍            | host 稽核、host 設定值、Docker 設定檔、container runtime 設定值 | host 設定值、container runtime 設定值 | 系統函式庫、dotnet lib                                              | 系統函式庫                                                                                       | 系統函式庫                        | 系統函式庫                     |
| 主要弱點資料庫      | CIS                                                             | CIS                                   | CVE、DLA                                                            | CVE、CWE                                                                                         | CVE                               | CVE                            |
| 報告格式            | text                                                            | text                                  | table、json、sarif、cyclonedx、spdx、spdx-json、github、cosign-vuln | text、json                                                                                       | Azure Portal                      | table、json                    |
| 報告內容            | Kernel Parameter、檔案權限等                                    | Kernel Parameter、檔案權限等          | 弱點版本、修復版本                                                  | 弱點版本、修復版本、有些會在網頁顯示 no fix in debian 8 ...                                      | 弱點版本、修復版本                | 弱點版本、在網頁中顯示修復版本 |
| 費用                | 免費                                                            | 免費                                  | 免費                                                                | 5 人 Team 方案 $125/month                                                                        | $6.8693/vCore/month               | 免費                           |
| 依賴                | git                                                             | 官網下載 deb 檔、git                  | 官網的 PPA                                                          | 已經內建在 `docker scan`，但第一次執行時需要同意他的 license，並且需要透過網址或 token 登入 Snyk | 無                                | golang                         |
