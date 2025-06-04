---
title: "自架 Ubuntu 24 Kubernetes 叢集"
subtitle: ""
excerpt: "ubuntu 24 Kubernetes k8s vm"
layout: post
author: "blueskyson"
header-style: text
tags:
  - linux
---

在本教學中，我們將會在 Ubuntu 24.04 Server 上安裝並架設一個四節點的 Kubernetes 叢集（包含一個控制節點與三個工作節點）。

Kubernetes 叢集由控制節點（Control Plane）與工作節點（Worker Nodes）所組成。控制節點負責整體叢集的管理與調度，而工作節點則實際執行應用程式容器（Pods）。本篇將逐步帶你完成所有必要設定，從系統安裝、節點配置，到基本應用部署。

首先，我們需要準備兩台 Ubuntu 24.04 的 VM，作為 Master Node 與 Worker Node。每台 VM 至少需要 2GB 的記憶體與 2 個 CPU 核心。

| 節點類型 | CPU | 記憶體 | 磁碟空間 | OS | NAT DHCP IP |
| -------- | --- | ------ | -------- | -- | -------- |
| Master | 2   | 2GB    | 15GB     | Ubuntu 24.04 | 192.168.122.11 |
| Worker | 2   | 2GB    | 15GB     | Ubuntu 24.04 | 192.168.122.12 |

## Master Node 設定

### 系統更新與基本設定

首先登入 Master Node 並更新系統套件：

```non
$ sudo apt update
```

把 swap 關閉：

```non
$ sudo swapoff -a
```

啟用 Kernal IP Forwarding：

```non
$ echo "net.ipv4.ip_forward=1" | sudo tee -a  /etc/sysctl.conf
$ sudo sysctl -p
```

啟用 overlay 與 br_netfilter 模組，並用 `lsmod` 確認模組已載入：

```non
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

$ sudo modprobe overlay
$ sudo modprobe br_netfilter
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf 
net.bridge.bridge-nf-call-iptables = 1 
net.bridge.bridge-nf-call-ip6tables = 1 
net.ipv4.ip_forward = 1 
EOF

$ sudo sysctl -p
$ lsmod | grep br_netfilter 
$ lsmod | grep overlay
```

### 安裝 kubelet kubeadm kubectl

參考 [官方文件](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) 透過 APT 安裝 Kubernetes 工具。（筆者當下用的版本為 V1.33，後續版本的 `curl -fsSL` 網址可能有差異，建議去官往查看完整指令，不要直接複製貼上）

```non
$ sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
$ sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list
```

接著更新 APT 並安裝各個工具：

```non
$ sudo apt-get update
$ sudo apt-get install -y kubelet kubeadm kubectl
```

### 安裝 containerd

使用 containerd 作為容器執行環境。參考[官方文件](https://docs.docker.com/engine/install/ubuntu/) 加入 Docker 的 APT 存儲庫：

```non
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
$ sudo chmod a+r /etc/apt/keyrings/docker.asc
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

接著更新 APT 並安裝 containerd：

```non
$ sudo apt-get update
$ sudo apt-get install -y containerd.io
```

編輯 `/etc/containerd/config.toml` 檔案，啟用 `SystemdCgroup` 並註解掉 `disabled_plugins = ["cri"]`：

```non
$ sudo vim /etc/containerd/config.toml
```

```toml
# disabled_plugins = ["cri"]
SystemdCgroup = true
```

重新啟動 containerd 服務：

```non
$ sudo systemctl restart containerd
```

### kubeadm 初始化

使用 `kubeadm` 初始化 Kubernetes 叢集。`--pod-network-cidr` 用來劃出 Pod 網路的 IP 範圍，控制平面會自動為每個節點分配一個 CIDR（Classless Inter-Domain Routing 區塊）。萬一 `10.100.0.0/26` 剛好跟你的網路設定衝突，請選擇其他 CIDR 範圍。 

```non
$ sudo kubeadm init --pod-network-cidr=10.100.0.0/16
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

> 如果此時透過 `sudo crictl ps` 指令查看容器狀態，發現 etcd、kube-apiserver 等一直重啟，可以試試把 CgroupV2 改成 CgroupV1。實際錯誤原因我不清楚，但這是我在 qemu/kvm 虛擬機上遇到的問題。
> 
> ```non
> $ sudo vim /etc/default/grub
> ```
>
> 修改 `GRUB_CMDLINE_LINUX_DEFAULT`
>
> ```
> GRUB_CMDLINE_LINUX_DEFAULT="systemd.unified_cgroup_hierarchy=0"
> ```
>
> 更新 GRUB 並重新啟動 VM。
> 
> ```non
> $ sudo update-grub
> ```

最後一步，安裝 Flannel 作為 Pod 網路插件。Flannel 是一個輕量級的網路解決方案，適合用於 Kubernetes 叢集。

```non
$ kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

## Worker Node 設定

參考前面 Master Node 的步驟，更新系統、關閉 swap、啟用 Kernal IP Forwarding、安裝 kubelet、kubeadm、kubectl 以及 containerd，但是不要做 `kubeadm init` 的步驟。

在 Master Node 上執行以下指令，取得加入叢集的指令：

```non
$ kubeadm token create --print-join-command
```

在 Worker Node 上執行上述指令，即可加入叢集。

## 驗證叢集狀態

在 Master Node 上執行以下指令，檢查叢集狀態：

```non
$ kubectl get nodes
```