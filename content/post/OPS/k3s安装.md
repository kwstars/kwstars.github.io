---
title: "K3S安装"
date: 2021-12-06T10:32:06+08:00
draft: false
tags: [""]
categories: ["K3S"]
---

## 一、安装multipass

[Install Multipass](https://multipass.run/)



## 二、在win10启动一个虚拟机

1. 创建bridge网桥

![image-20211202133625245](https://i.imgur.com/VFRdDlv.png)



2. 安装ubuntu虚拟机

```powershell
# 查看支持的镜像
PS C:\> multipass find
Image                       Aliases           Version          Description
core                        core16            20200818         Ubuntu Core 16
core18                                        20200812         Ubuntu Core 18
18.04                       bionic            20211129         Ubuntu 18.04 LTS
20.04                       focal,lts         20211129         Ubuntu 20.04 LTS
21.04                       hirsute           20211130         Ubuntu 21.04
21.10                       impish            20211103         Ubuntu 21.10
appliance:adguard-home                        20200812         Ubuntu AdGuard Home Appliance
appliance:mosquitto                           20200812         Ubuntu Mosquitto Appliance
appliance:nextcloud                           20200812         Ubuntu Nextcloud Appliance
appliance:openhab                             20200812         Ubuntu openHAB Home Appliance
appliance:plexmediaserver                     20200812         Ubuntu Plex Media Server Appliance
anbox-cloud-appliance                         latest           Anbox Cloud Appliance
minikube                                      latest           minikube is local Kubernetes

# 安装一个20.04的ubuntu
PS C:\> multipass launch --bridged --name k3s-server lts
```



## 三、安装k3s服务端

kubeconfig文件会写入**/etc/rancher/k3s/k3s.yaml**

```bash
$ curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--advertise-address 192.168.0.100 --disable traefik" sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.21.5+k3s2 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.21.5+k3s2/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.21.5+k3s2/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Skipping /usr/local/bin/ctr symlink to k3s, command exists in PATH at /bin/ctr
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-agent-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s-agent.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s-agent.service
[INFO]  systemd: Enabling k3s-agent unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s-agent.service → /etc/systemd/system/k3s-agent.service.
[INFO]  systemd: Starting k3s-agent
```



## 四、安装k3s node节点

K3S_TOKENT为服务节点的**/var/lib/rancher/k3s/server/node-token**文件中

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--node-ip 192.168.0.15 --flannel-iface eth1" K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```



---

[K3s - Lightweight Kubernetes](https://rancher.com/docs/k3s/latest/en/)

[How to Use Flags and Environment Variables](https://rancher.com/docs/k3s/latest/en/installation/install-options/how-to-flags/)
