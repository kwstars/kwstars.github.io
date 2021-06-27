---
title: "UFW"
date: 2020-10-04T18:16:12+08:00
draft: false
tags: ["UFW"]
categories: ["OPS"]
---

## 一、UFW介绍
下文引用自Wikipedia
> UFW全称为**Uncomplicated
> Firewall**[[1]](https://zh.wikipedia.org/wiki/Uncomplicated_Firewall#cite_note-1)，是[Ubuntu](https://zh.wikipedia.org/wiki/Ubuntu
> "Ubuntu")系统上默认的[防火墙](https://zh.wikipedia.org/wiki/%E9%98%B2%E7%81%AB%E5%A2%99
> "防火墙")组件，为了轻量化配置[iptables](https://zh.wikipedia.org/wiki/Iptables
> "Iptables")而开发的一款工具。UFW提供一个非常友好的界面用于创建基于[IPV4](https://zh.wikipedia.org/wiki/IPV4
> "IPV4")，[IPV6](https://zh.wikipedia.org/wiki/IPV6 "IPV6")的防火墙规则。
> 
> UFW的[图形用户界面](https://zh.wikipedia.org/wik应用i/%E5%9C%96%E5%BD%A2%E4%BD%BF%E7%94%A8%E8%80%85%E7%95%8C%E9%9D%A2
> "图形用户界面")叫做“Gufw”。

## 二、Gufw
```bash
sudo apt-get install gufw
```

## 三、命令行
### 3.1 添加规则
```bash
sudo ufw deny 111
sudo ufw allow 80/tcp
sudo ufw allow http/tcp

# To allow connections from an IP address:
sudo ufw allow from 198.51.100.0

# To allow connections from a specific subnet:
sudo ufw allow from 198.51.100.0/24

# To allow a specific IP address/port combination:
sudo ufw allow from 198.51.100.0 to any port 22 proto tcp
```



### 3.2 查看规则

```bash
# 查看简要状态
sudo ufw status

# 显示rule的号码
sudo ufw status numbered

# 显示详细信息
sudo ufw status verbose 
```



### 3.3 更新规则

```bash
sudo ufw update
```



### 3.4 删除规则

```bash
sudo ufw delete RULE_NUM
```

## 四、配置文件
### 4.1 编辑配置文件
```bash
cat /etc/ufw/applications.d/game    
[privoxy]
title=privoxy
description=privoxy
ports=8118/tcp

[nginx]
title=nginx
description=nginx
ports=80/tcp

[supervisor]
title=supervisor
description=supervisor 9001
ports=9001/tcp

[samba_tcp]
title=samba
description=samba 139/tcp,445/tcp
ports=139,445/tcp

[samba_udp]
title=samba
description=samba 137/udp,138/udp
ports=137,138/udp
```



### 4.2 应用规则

规则需要按照名字一个个应用
```bash
sudo ufw allow samba
```



###  4.3 查看应用程序

```bash
# 列出所有可应用的应用程序
sudo ufw app list

# 查看应用的详细信息
sudo ufw app info PROFILE
```



### 4.4 修改规则

```bash
sudo ufw app update PROFILE
```

---
> [How to Configure a Firewall with UFW](https://www.linode.com/docs/security/firewalls/configure-firewall-with-ufw/)
> 
> [Gufw](https://help.ubuntu.com/community/Gufw)
> 
> [How To Set Up a Firewall with UFW on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server)