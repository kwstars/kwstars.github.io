---
title: "如何在CentOS8上使用Multipass"
date: 2021-12-13T10:23:48+08:00
draft: false
tags: [""]
categories: ["multipass"]
---

## 一、通过Cockpit管理虚拟机

[Using libvirt in Multipass](https://multipass.run/docs/using-libvirt) 在CentOS8没有成功，使用了 **lxd**

```bash
dnf install cockpit cockpit-machines -y
systemctl start cockpit.socket
systemctl enable cockpit.socket
systemctl status cockpit.socket
firewall-cmd --add-service=cockpit --permanent
firewall-cmd --reload
```



要访问 Cockpit Web 控制台，请打开 Web 浏览器并使用以下 URL

```bash
https://FQDN:9090/
OR
https://SERVER_IP:9090/
```



### 设置网桥

![image-20211213105845361](https://i.imgur.com/seirrNb.png)



## 二、安装KVM

### 2.1 安装准备

确认硬件平台支持虚拟化

```bash
grep -e 'vmx' /proc/cpuinfo		#Intel systems
grep -e 'svm' /proc/cpuinfo		#AMD systems
```



确认 KVM 模块已加载到内核中（默认情况已加载）

```bash
$ sudo lsmod | grep kvm
kvm_intel             303104  0
kvm                   798720  1 kvm_intel
irqbypass              16384  1 kvm
```



### 2.2 安装KVM

```bash
dnf module install virt 
dnf install virt-install virt-viewer

# 运行 virt-host-validate 命令来验证主机是否设置为运行 libvirt 管理程序驱动程序。
virt-host-validate

# 开启
systemctl start libvirtd.service
systemctl enable libvirtd.service
systemctl status libvirtd.service
```



## 三、安装multipass

```bash
dnf install epel-release
dnf upgrade

yum install snapd 
systemctl enable --now snapd.socket
snap install multipass lxd
multipass set local.driver=lxd
```



开启一个ubuntu的虚拟机

```bash
# 启动一个名为ubuntu-0的虚拟机
multipass launch -c 2 -m 4G -d 20G --network name=bridge0 -n k8s-node2 lts
```



---

[Installing snap on CentOS](https://snapcraft.io/docs/installing-snap-on-centos)

[How to Install KVM on CentOS/RHEL 8](https://www.tecmint.com/install-kvm-in-centos-8/)

[3 Ways to Create a Network Bridge in RHEL/CentOS 8](https://www.tecmint.com/create-network-bridge-in-rhel-centos-8/)

[How can I redirect storage of Multipass VMs?](https://askubuntu.com/questions/1248169/how-can-i-redirect-storage-of-multipass-vms)

[Moving the data directory of Multipass and Docker](https://lucasroesler.com/2020/12/moving-the-data-directory-of-multipass-and-docker/)
