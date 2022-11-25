---
title: "學習 Ansible"
subtitle: ""
excerpt: "ansible"
layout: post
author: "blueskyson"
header-style: text
tags:
  - devops
  - others
---

## Ansible 簡介

Ansible 是一個容易上手的 IT 自動化引擎，可自動執行雲端配置、應用程式部署和許多其他 IT 需求。存放 yml 描述檔並執行 `ansible-playbook` 的機器稱為 Control Node，遠端的所有參與佈署的機器稱為 Managed Node。Ansible 這個詞起源於科幻小說 *Ender's Game* 中的一種超光速通訊裝置，用來控制遠方的星際戰艦。

![](https://scholarblogs.emory.edu/lits/files/2018/10/ansible-enders.png)

Ansible 比起其他自動化部屬的工具有兩大特點：
- 在遠端只需要 python 環境與 ssh server，不需要其他 agent 也沒有額外的自定義安全基礎設施，因此易於部署。
- 它使用 YAML，以 Ansible Playbook 的形式，允許您以接近英語的方式描述您的自動化作業。

![](https://user-images.githubusercontent.com/582423/166117891-d0d288f6-ada3-467b-ab06-c22992b5bff3.png)

## Ansible 的術語：

- **Modules**
  透過 Ansible 推送到各個節點的腳本稱為 "module"，Ansible 預設會透過 ssh 來執行 module。通常你會透過現有的 module 來達成需求，但必要時也可以自己開發 module，開發的語言有 python、ruby、bash 等等。許多相關的 Module 集合起來會變成 Collection，例如 Azure.Azcollection。
- **Module utilities**
  當多個 module 使用相同的程式碼時，Ansible 將這些函數存儲為 module utilities，以最大限度地減少重複和維護。module utilities 只能用 python 或 powershell 開發。
- **Plugins**
  用來擴充 Ansible 的核心功能，如轉換資料、記錄輸出、連接到 inventory 等。Plugin 與 Module 不同的點在於 Module 定義了一個介面讓使用者自行設定一些參數，再根據輸入來對遠端做相應的操作；Plugin 主要執行在 Control Node，用來處理 Ansible 核心的輸入和輸出。
- **Inventory**
  Ansible 在一個 INI、YAML 等紀錄它管理的 inventory，該文件讓你幫機器的 ip 分組，格式類似：
  ```ini
  [webservers]
  www1.example.com
  www2.example.com
  
  [dbservers]
  db0.example.com
  db1.example.com
  ```
  在上方範例中，`[webservers]` 底下的成員就屬於 `webservers` 群組，此群組的成員可以共享一些設定，或是做其他進階的操作。
  如果您的基礎設施中存在其他來源 (source) 例如 EC2、OpenStack 等，Ansible 也可以連接到該來源，從這些來源中動態的抓取更多 inventory、group、variable 的資訊。
  ![](https://docs.ansible.com/ansible/latest/_images/ansible_basic.svg)
- **Playbooks**
  Playbook 可以精細地編排 (orchestrate) 基礎設設施，並非常詳細地控制一次要處理多少台機器。一個基礎設施可能持續使用很多年，因此 Ansible 盡量不推出新的特殊語法或功能，讓你不用為了維護久遠以前的基礎設施而花很多時間複習 Ansible 語法。
  下方是一個簡單的 playbook：
  ```yml
  ---
  - hosts: webservers
    serial: 5 # update 5 machines at a time
    roles:
    - common
    - webapp
  
  - hosts: content_servers
    roles:
    - common
    - content
  ```
- **Ansible search path**
  如果您編寫自己的 Modules、Plugins 來擴展 Ansible 的核心功能，您可能在 Ansible 控制節點的不同位置有多個名稱相似或相同的檔案。search path 決定了 Ansible 將在指定的 playbook 使用哪些文件。當 Ansible 找到給定運行中包含的每個 playbook 和 role 時，它會將自動將可能有關的目錄附加到搜索路徑中，並自動載入找到的檔案。
  Ansible 按以下幾種狀況自動載入 modules、Module utilities、plugins：
  + 如果您執行 `$ ansible-playbook /path/to/play.yml`，Ansible 會附加這些目錄：
    ```non
    path/to/modules
    path/to/module_utils
    path/to/plugins
    ```
  + 再來，如果 `play.yml` 包含 `- import_playbook: /path/to/subdir/play1.yml`，Ansible 會附加這些目錄：
    ```non
    path/to/subdir/modules
    path/to/subdir/module_utils
    path/to/subdir/plugins
    ```
  + 如果透過 `role` 來執行 `play.yml`，Ansible 會附加這些目錄：
    ```non
    /path/to/roles/myrole/modules
    /path/to/roles/myrole/module_utils
    /path/to/roles/myrole/plugins
    ```
  + 在 `ansible.cfg` 或環境變數中的特定目錄：
    ```non
    DEFAULT_MODULE_PATH
    DEFAULT_MODULE_UTILS_PATH
    DEFAULT_CACHE_PLUGIN_PATH
    ...
    ```
  另外一些關於 Ansible 的設定會依序從以下路徑載入：
  ```non
  ANSIBLE_CONFIG
  ./ansible.cfg
  ~/.ansible.cfg
  /etc/ansible/ansible.cfg
  ```

## 安裝 Ansible

首先準備一個 Linux 系統和 python 環境，透過 pip 安裝：

```non
$ pip install ansible
```

另外有 ansible-lint 會建議 ansible 寫法：

```non
$ pip install ansible-lint
```

## Lab 1: 在 localhost 執行 Hello World

在 hello world 範例中，我只單純在 localhost 上示範一個簡單的 playbook，先不要管 inventory、roles 等用法。首先新增一個 `hellowrold.yml` 並貼上以下內容：

```yml
---
- name: Play with Ansible
  hosts: localhost
  tasks:
  - name: Just execute df -h
    ansible.builtin.command: df -h
    register: output
    changed_when: output.rc == 0

  - name: Show stdout
    ansible.builtin.debug:
      msg: "{{ output.stdout_lines }}"
```

- 第 1 行：`---` 是撰寫 yml 的慣例，表示文件的開頭，不寫這行 Ansible 還是能正常執行。
- 第 2 行：`names` 是這個 play 的名稱，我命名為 `"Play with Ansible"`。在此範例中，`name`、`hosts`、`tasks` 被稱為 [play keywords](https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html#play)，Ansible 讀到這些 keyword 會做相應的動作，是比較需要背誦的部份。
- 第 3 行：`hosts` 是這個 play 的布屬對象 (target)，是以逗點隔開的 hostname 或是 ip 位址。
- 第 4 行：`tasks` 是部屬對象所要執行的任務。
- 第 5 行：自訂任務的名稱。`name`、`register`、`changed_when` 被稱為 [task keywords](https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html#task)。
- 第 6 行：`ansible.builtin.command` 是一個 Ansible 內建的 module，它在所有布屬對象執行一個指令，在範例中我讓 `localhost` 執行 `df -h`。
- 第 7 行：`register` 會接收這個 task 的回傳訊息（即 `df -h` 的 stdout）並指派給 `"output"` 變數。以寫程式類比的話，相當於宣告了一個名為 `output` 的變數來儲存 `df -h` 的輸出。
- 第 8 行：`changed_when` 是特別用來表示 targets 執行完 task 後，狀態有沒有被「改變」，狀態是否被改變的判斷條件是由撰寫 playbook 的人自訂的，在此範例中我定義當 `df -h` 的回傳值為 0 時即代表機器的狀態被改變了。如果這個 task 完全不會影響機器的狀態，可以把這行改為 `change_when: false`。如果不寫這一行，Ansible 也能正常執行，不過 `change_when` 就永遠會是 `true`，而且會被 ansible-lint 要求補上這一行。 
- 第 9 行：第二個任務。
- 第 10 行：用內建的 debug module 來輸出訊息到終端機。
- 第 11 行：輸出 `df -h` 的每一行 stdout。

透過 `$ ansible-lint helloworld.yml` 來檢查語法有無問題。
透過 `$ ansible-playbook helloworld.yml` 來執行 helloworld 輸出如下：

```non
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Play with Ansible] ***************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************
ok: [localhost]

TASK [Just execute df -h] **************************************************************************************************
changed: [localhost]

TASK [Show stdout] *********************************************************************************************************
ok: [localhost] => {
    "msg": [
        "Filesystem      Size  Used Avail Use% Mounted on",
        "tmpfs           2.4G  3.1M  2.4G   1% /run",
        "/dev/nvme1n1p4  232G   35G  185G  16% /",
        "tmpfs            12G  112M   12G   1% /dev/shm",
        "tmpfs           5.0M  4.0K  5.0M   1% /run/lock",
        "tmpfs            12G     0   12G   0% /run/qemu",
        "/dev/nvme0n1p1  256M   60M  197M  24% /boot/efi",
        "tmpfs           2.4G  2.4M  2.4G   1% /run/user/1000",
        "/dev/nvme0n1p5  239G   77G  162G  33% /media/lin/Data"
    ]
}

PLAY RECAP *****************************************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

可以看到 Ansible 成功執行 `df -h` 並輸出結果在終端機上。

## Lab 2: 使用 inventory 連到遠端主機

首先準備兩個 linux 主機，我自己是用 Virtual Box 裝兩個 Alpine-Virt Linux，在虛擬機中安裝 `python3` 並允許 root 連線，然後用 Host-only 網路來模擬遠端主機。另外本機要安裝 `sshpass` 才能進行接下來的操作。

在目錄中新增一個名為 `inventory.ini` 的檔案：

```ini
[vboxhost]
localhost ansible_user=lin ansible_password=mypassword

[vboxservers]
alpine1 ansible_host=192.168.56.101 
alpine2 ansible_host=192.168.56.102

[allservers:children]
vboxservers
vboxhost

[vboxservers:vars]
ansible_user=root
ansible_password=mypassword
```

在這個檔案中我將兩台虛擬機分配在 `vboxservers`、本機分配在 `localservers`。並且透過 `ansible_host`、`ansible_user`、`ansible_password` 來設定 ssh 連線的變數。通常我們不會直接用 `ansible_password` 儲存密碼，而是用 `ansible vault` 或其他第三方軟體來保護密碼。

- 第 8 行透過 `:children` 語法來組合 group，將 `vboxservers` 和 `vboxhost` 共同歸類在 `allservers`。
- 第 12 行透過 `:vars` 來將重複使用的變數值一次指派給整個群組的機器。


透過 `$ ansible-inventory -i inventory.ini --list` 可以確認目前 inventory 的配置：

```json
{
    "_meta": {
        "hostvars": {
            "alpine1": {
                "ansible_host": "192.168.56.101",
                "ansible_password": mypassword,
                "ansible_user": "root"
            },
            "alpine2": {
                "ansible_host": "192.168.56.102",
                "ansible_password": mypassword,
                "ansible_user": "root"
            },
            "localhost": {
                "ansible_password": mypassword,
                "ansible_user": "lin"
            }
        }
    },
    "all": {
        "children": [
            "allservers",
            "ungrouped"
        ]
    },
    "allservers": {
        "children": [
            "vboxhost",
            "vboxservers"
        ]
    },
    "vboxhost": {
        "hosts": [
            "localhost"
        ]
    },
    "vboxservers": {
        "hosts": [
            "alpine1",
            "alpine2"
        ]
    }
}
```

inventory 檔案可以有許多個，執行 `ansible-playbook` 時也可以引入多個 inventory，所以一般會將不同環境用到的設定分別放在不同的 inventory 檔案中。

然後在同樣的目錄中新增一個 `ansible.cfg`，複製以下內容，讓 ansible 登入陌生的機器時自動儲存 finger print：

```ini
[defaults]
host_key_checking = False
```

接下來我將剛剛 helloworld.yml 的第 3 行 `hosts` 改為 `vboxservers`:

```yml
---
- name: Use inventory
  hosts: vboxservers
  tasks:
  - name: Just execute df -h
    ansible.builtin.command: df -h
    register: output
    changed_when: output.rc == 0

  - name: Show stdout
    ansible.builtin.debug:
      msg: "{{ output.stdout_lines }}"
```

此時目錄結構會長這樣：

```non
.
├── ansible.cfg
├── hellowrod.yml
└── inventory.ini
```

最後，透過 `$ ansible-playbook -i inventory.ini useinventory.yml` 來載入 `inventory.ini` 並執行 `helloworld.yml`:

```non
PLAY [Use inventory] *********************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************
[WARNING]: Platform linux on host alpine2 is using the discovered Python interpreter at /usr/bin/python3.10, but future installation of another Python interpreter could
change the meaning of that path. See https://docs.ansible.com/ansible-core/2.13/reference_appendices/interpreter_discovery.html for more information.
ok: [alpine2]
[WARNING]: Platform linux on host alpine1 is using the discovered Python interpreter at /usr/bin/python3.10, but future installation of another Python interpreter could
change the meaning of that path. See https://docs.ansible.com/ansible-core/2.13/reference_appendices/interpreter_discovery.html for more information.
ok: [alpine1]

TASK [Just execute df -h] ****************************************************************************************************************************************************
changed: [alpine2]
changed: [alpine1]

TASK [Show stdout] ***********************************************************************************************************************************************************
ok: [alpine1] => {
    "msg": [
        "Filesystem                Size      Used Available Use% Mounted on",
        "devtmpfs                 10.0M         0     10.0M   0% /dev",
        "shm                     487.9M         0    487.9M   0% /dev/shm",
        "/dev/sr0                149.0M    149.0M         0 100% /media/cdrom",
        "tmpfs                   487.9M    113.6M    374.2M  23% /",
        "tmpfs                   195.2M    120.0K    195.0M   0% /run",
        "/dev/loop0              106.3M    106.3M         0 100% /.modloop"
    ]
}
ok: [alpine2] => {
    "msg": [
        "Filesystem                Size      Used Available Use% Mounted on",
        "devtmpfs                 10.0M         0     10.0M   0% /dev",
        "shm                     487.9M         0    487.9M   0% /dev/shm",
        "/dev/sda3                 5.8G    194.4M      5.3G   3% /",
        "tmpfs                   195.2M    120.0K    195.0M   0% /run",
        "/dev/sda1                88.2M     24.3M     57.0M  30% /boot",
        "tmpfs                   487.9M     96.0K    487.8M   0% /tmp"
    ]
}

PLAY RECAP *******************************************************************************************************************************************************************
alpine1                    : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
alpine2                    : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

可以看到在兩台虛擬機上成功執行 `df -h`。

## Vault

### Lab 3: 加密整份文件

Ansible Vault 可以加密 Ansible 使用的任何結構化檔案，預設的加密方法是 AES256。首先加密剛剛 inventory 練習中的 `helloworld.yml` 和 `inventory.ini`：

```non
$ ansible-vault encrypt helloworld.yml inventory.ini
```

此時將 `helloworld.yml` 和 `inventory.ini` 打開，會看到以下已被加密的內容：

```yml
$ANSIBLE_VAULT;1.1;AES256
376562366632666466386233376...
```

如果想解密檔案，則可以用 `ansible-vault decrypt heelloworld.yml`。
單純想檢視檔案，則可以用 `ansible-vault view heelloworld.yml`。

接下來透過 Ansible Playbook 解密並執行 `helloworld.yml`：

```non
$ ansible-playbook --ask-vault-pass -i inventory.ini helloworld.yml
```

或是先把密碼存在一個檔案，比如 `mypassword`，然後從該檔案讀取密碼來解密：

```non
$ echo 12345678 > mypassword
$ ansible-playbook --vault-password-file mypassword -i inventory.ini helloworld.yml
```

### Lab 4: vault id

當需要加密的內容變多，而且不同環境要用不同的密碼時，可以透過 vault id 來管理這些秘密內容。vault id 只會為加密的內容附上一個標籤，並不會影響加解密的流程，所以用 vault id 加密的內容仍然可以透過前面提到的 `--vault-password-file` 解密。

用 vault id 加密的，但是可以幫助你使用以下選項來設定：

```non
--vault-id label@source ...
```

label 是我們自自訂的標籤，例如 `dev`、`app` 等。source 是讀取密碼的來源，可以從 stdin 輸入（`prompt`）、或是從檔案（`path/to/file`）、或是符合官方規範的腳本（`path/to/client.py`）。

首先透過 vault id 加密剛剛 inventory 練習中的 helloworld.yml 和 inventory.ini，給予 `foo` 標籤並使用 `mypassword` 檔案中的密碼加密：

```non
$ echo 12345678 > mypassword1
$ echo 12341234 > mypassword2
$ ansible-vault encrypt --vault-id foo@mypassword1 inventory.ini
$ ansible-vault encrypt --vault-id bar@mypassword2 helloworld.yml
```

此時將 `inventory.ini` 打開，會看到被加密的內容有一個 `foo` 標籤：

```yml
$ANSIBLE_VAULT;1.2;AES256;foo
633364633161343033343161376666
```

接下來透過 Ansible Playbook 解密各個 vault-id 和密碼檔案並執行 `helloworld.yml`：

```non
$ ansible-playbook --vault-id foo@mypassword1 \
                   --vault-id bar@mypassword2 \
                   -i inventory.ini helloworld.yml
```

或是也可以不用輸入 vault-id 的資訊，讓 ansible 自行嘗試解密：

```non
$ ansible-playbook --vault-id mypassword1 \
                   --vault-id mypassword2 \
                   -i inventory.ini helloworld.yml
```

從這個練習中可以發現，從 stdin 或檔案中讀取多個不同密碼，需要針對不同的密碼一一輸入或是填入 ansible-playbook 指令的 `--vault-id` 欄位，而使用腳本則可以自動根據不同的 label 來挑選正確的密碼。

### Lab 5: 加密單個秘密，並透過 group_var 引入

剛才介紹的方法都是對整個檔案進行加密，但是加密整個檔案後反而對閱讀造成困擾，通常我們只是想把單個字串隱藏起來而已。此時就要用 `encrypt_string`：

```non
$ ansible-vault encrypt_string --vault-id foo@mypassword --name vboxservers_passwd xyz123
```

執行指令後，終端機會輸出被加密後的內容：

```yml
secret_var: !vault |
          $ANSIBLE_VAULT;1.2;AES256;foo
          66613064643635383932333531
          ...
```

將這整串被加密的變數複製進 `./group_var/vboxservers/foo_settings.yml` 如下：

```yml
vboxservers_user: root
secret_var: !vault |
          $ANSIBLE_VAULT;1.2;AES256;foo
          66613064643635383932333531
          ...
```

將 `inventory.ini` 的檔案改寫如下：

```ini
[vboxhost]
localhost ansible_user=root ansible_password=123456

[vboxservers]
alpine1 ansible_host=192.168.56.101 
alpine2 ansible_host=192.168.56.102

[allservers:children]
vboxservers
vboxhost

[vboxservers:vars]
ansible_user={{ vboxservers_user }}
ansible_password={{ vboxservers_passwd }}
```

此時整個專案的目錄會長這樣：

```non
.
├── ansible.cfg
├── group_vars
│   └── vboxservers
│       └── foo_settings.yml
├── helloworld.yml
├── inventory.ini
└── mypassword
```

請注意不管加密或是明文的自訂變數，都一定要放在 `./group_var/vboxservers/` 目錄裡，因為 Ansible 是靠相對路徑來找變數。以這個練習為例，`vboxservers` 群組中的自訂變數會從 `group_vars` 去尋找 `vboxservers` 目錄，而 `foo_settings.yml` 這個檔案可以隨意命名。

最後用以下指令讓 Ansible Playbook 解密並執行 `helloworld.yml`：

```non
$ ansible-playbook --vault-id foo@mypassword -i inventory.ini helloworld.yml
```

## Lab 6: Roles & Ansible Galaxy

Roles 允許您基於已知的目錄結構自動載入相關的變數、檔案、任務和其他 Ansible 物件，將上述物件按照 Role 分組後，您可以輕鬆地重複使用它們並共享到其他 Ansible 專案。

Asible 官方維護了一個 Roles 的市集叫做 Galaxy，在 Galaxy 中提供的 Role 原始碼放在 Github 上。`ansible-galaxy` 可以從 Galaxy 搜尋、安裝、移除 Roles，可以把它想像成 Ansible 用的 pip。

首先新增一個 `requirements.yml`，包含了安裝 nginx 和 dotnet 的 Roles：

```yml
- src: nginxinc.nginx
  version: 0.23.2
- src: andrewrothstein.dotnet
  version: v3.1.0
```

執行以下指令，從 Ansible Galaxy 下載所有相依的 Roles：

```non
$ ansible-galaxy install -r requirements.yml -p roles
```

新增一個 `main.yml`，內容如下：

```yml
---
- name: Setup Nginx using roles
  hosts: alpine1
  become: True
  roles:
    - nginxinc.nginx

- name: Setup dotnet
  hosts: alpine1
  become: True
  roles:
    - role: andrewrothstein.dotnet
      vars:
        dotnet_install_type: sdk
        dotnet_ver: 3.1.402
        dotnet_subdirs:
          linux-x64: 'pr/f01e3d97-c1c3-4635-bc77-0c893be36820/6ec6acabc22468c6cc68b61625b14a7d'
          linux-musl-x64: 'pr/e301fc5c-c8dd-4f8e-94ee-d19f3caf508f/a4191801aeb8cd813cf7057ac4d936a0'
          osx-x64: 'pr/ac399dfa-04e1-49cf-be75-7112a9eec68f/60b1ca435b12e7b8beb6bb39b9cdf1c6'
        dotnet_checksums:
          linux-x64: sha512:42154efb5ad66ae3dcc300b2c0573a9537dd916fc48cbae92885a63a0b6d7f7c3a4366ca2298107783bc1f1913328f35e778dcda378da276cff3b8269495d5be
          linux-musl-x64: sha512:30916407ee1f99c0f1398a45aa1a480b6d75c5e42488c877b7879ea68a03de07b29943e89e9324c3b14df4ca1d2723116a5c4812b2265cbb103488706aa56b70
          osx-x64: sha512:68b5ecc76b588399d4f5bcc123caf0c1c1f26625bf21731737f004886f665ebe6559e9cc77f1265c678508726025f66862fb901ae95a25265bc3da4bed69335f
```

此時整個專案的目錄會長這樣：

```non
.
├── ansible.cfg
├── group_vars
│   └── vboxservers
├── inventory.ini
├── main.yml
├── mypassword
├── requirements.yml
└── roles
    ├── andrewrothstein.dotnet
    ├── andrewrothstein.unarchive-deps
    └── nginxinc.nginx
```

最後執行 Ansible Playbook：

```non
$ ansible-playbook --vault-id mypassword -i inventory.ini main.yml     
```

## 透過 Ansible 創建 Azure 資源

### Lab 7: 創建 Azure VM

按照[使用 Ansible 在 Azure 中建立 Linux 虛擬機器](https://learn.microsoft.com/zh-tw/azure/developer/ansible/vm-configure?tabs=ansible)在 Cloud Shell 或從自己的 Linux 機器創建一個 Azure VM。

注意 `admin_username`、`ssh_public_keys` 要替換成 Control Node 的資訊。

### Lab 8: 創建 Azure Function

```non
$ pip3 install 
```


```yml
---
- name: Create Azure Function
  hosts: localhost
  connection: local
  tasks:
  - name: Create a resource group
    azure_rm_resourcegroup:
      name: myResourceGroup
      location: japaneast

  - name: Create a storage account
    azure_rm_storageaccount:
      resource_group: myResourceGroup
      name: jacktest0101
      type: Standard_LRS

  - name: Create a function app with app settings
    azure_rm_functionapp:
      resource_group: myResourceGroup
      name: jacktestfunc0101
      storage_account: jacktest0101
      app_settings:
        FUNCTIONS_WORKER_RUNTIME: dotnet
```

## 參考資料

- [Ansible architecture](https://docs.ansible.com/ansible/latest/dev_guide/overview_architecture.html)
- [How to build your inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
- [Encrypting content with Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
- [How To Use Ansible Vault to Protect Sensitive Playbook Data](https://www.digitalocean.com/community/tutorials/how-to-use-vault-to-protect-sensitive-ansible-data)
- [Best Practices](https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html)
- [[Ansible Tutorial] - Ansible Galaxy Tutorial](https://www.youtube.com/watch?v=7CGl82eVOO4)
- [azure.azcollection.azure_rm_functionapp module – Manage Azure Function Apps](https://docs.ansible.com/ansible/latest/collections/azure/azcollection/azure_rm_functionapp_module.html)
- [azure.azcollection.azure_rm_storageaccount module – Manage Azure storage accounts](https://docs.ansible.com/ansible/latest/collections/azure/azcollection/azure_rm_storageaccount_module.html)