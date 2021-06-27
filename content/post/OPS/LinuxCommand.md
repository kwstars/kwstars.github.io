---
title: "Linux Commands"
date: 2017-11-06T10:57:38+08:00
draft: false
tags: ["BASH"]
categories: ["OPS"]
---



## 查找替换多个文件

```bash
grep -rl "old_string" . | xargs sed -i 's/old_string/new_string/g'
```

- **grep -rl**: 递归搜索，仅打印包含“ old_string”的文件
- **xargs**: 接受`grep`命令的输出，并使其成为下一个命令（即`sed`命令）的输入
- **sed -i ‘s/old_string/new_string/g’:** new_string替换old_string



## 清空docker的log

```bash
cat /dev/null > /var/lib/docker/containers/CONTAINER_ID/CONTAINER_ID-json.log

cat /dev/null >  $(docker inspect --format='{{.LogPath}}'  CONTAINER_ID)
```



## 查找文件夹中最的的文件或目录

```bash
find /var/lib/docker/ -name "*.log" -exec ls -sh {} \; | sort -h -r | head -20

du -aSh /var/lib/docker/ | sort -h -r | head -n 10
```



## 下载并解压

```bash
# wget
wget -c https://studygolang.com/dl/golang/go1.15.6.linux-amd64.tar.gz  -O - | tar -xz -C /

# curl
curl -SL https://studygolang.com/dl/golang/go1.15.6.linux-amd64.tar.gz | tar -xvz -C /
```



## SSH

[SSH Port Forwarding Example](https://www.ssh.com/ssh/tunneling/example)

[玩转SSH端口转发](https://blog.fundebug.com/2017/04/24/ssh-port-forwarding/)

local -> forwarder -> server

### Local Forwarding

一般来讲，云主机的防火墙默认只打开了22端口，如果需要访问6379端口的话，需要修改防火墙。为了保证安全，防火墙需要配置允许访问的IP地址。但是，本地公网IP通常是网络提供商动态分配的，是不断变化的。这样的话，防火墙配置需要经常修改，就会很麻烦。

```bash
# ssh -L [<local host>:]<local port>:<remote host>:<remote port> <SSH hostname>
# 将server(192.168.1.123)的6379端口 转发到 本地的18080端口
ssh -L 127.0.0.1:18080:192.168.1.123:6379 FORWARDER_HOST
```

### Remote Forwarding

本地主机运行了一个服务，端口为18080，远程云主机需要访问这个服务。

```bash
# ssh -R [<local host>:]<local port>:<remote host>:<remote port> <SSH hostname>
# 将本地的6379端口，转发到PROXY上的18080端口
ssh -R 18080:127.0.0.1:6379 FORWARDER_HOST  

# 默认情况下，OpenSSH仅允许PROXY连接转发的端口。但是，可以使用服务器配置文件sshd_config中的GatewayPorts选项进行控制。可以使用以下替代方法：
GatewayPorts yes

# 客户端可以指定一个IP地址，允许从该IP地址连接到端口。语法如下：在此示例中，仅允许从IP地址52.194.1.73到端口8080的连接。
GatewayPorts clientspecified
ssh -R 52.194.1.73:8080:localhost:80 host147.aws.example.com

# 案例: 将公司内的 clinet1 和 clinet2 端口映射到 FORWARDER_HOST(公网主机)上的 2221, 2222端口
ssh -R 2221:clinet1:22 -R 2222:clinet1:22 FORWARDER_HOST
```

### Dynamic Port Forwarding

1. 把远端ssh服务器当作了一个安全的代理服务器 
2. 远程云主机运行了多个服务，分别使用了不同端口，本地主机需要访问这些服务。

```bash
# ssh -D [<local host>:]<local port> <SSH hostname>
ssh -D localhost:44444 root@REMOTE_HOST
```

本地设置socket代理 `127.0.0.1:4444`，就可以通过REMOTE_HOST访问了

### Chain port forwarding

家中`A电脑` 通过 云主机B 访问 公司`C电脑`

```bash
# 在C主机通过远程端口转发登陆云主机B
ssh -R localhost:2000:localhost:3000 root@103.59.22.17

# 在A电脑通过本地端口转发登录云主机B
ssh -L localhost:3000:localhost:2000 root@103.59.22.17

# 在A电脑通过 3000 端口访问
```



### SSHkey相关

#### ssh key 创建 

```bash
ssh-keygen -t rsa -f ~/.ssh/[KEY_FILENAME] -C [USERNAME]
chmod 400 ~/.ssh/[KEY_FILENAME]
```

- `[KEY_FILENAME]`：您要用于SSH密钥文件的名称。例如，文件名`my-ssh-key`生成一个名为的私钥文件`my-ssh-key`和一个名为的公钥文件`my-ssh-key.pub`。
- `[USERNAME]`：连接到虚拟机的用户名。例如， `cloudysanfrancisco@gmail.com`或`cloudysanfrancisco`。

#### ssh key 修改私钥密码

```
ssh-keygen -p

ssh-keygen -p -f ~/.ssh/id_dsa
```



#### ssh key 修改私钥comment

```bash
ssh-keygen -c -C "my new comment" -f ~/.ssh/my_ssh_key
```



## curl

```bash
# 通过本地的socks5代理访问google
curl -so /dev/null -w "%{http_code}" google.com -x socks5://127.0.0.1:1080
```



## dd
```bash
# 先跳过 skip*bs 大小的内容，然后复制 count*bs 大小的内容过来用 grep 查询。
dd if=FILE bs=1024 skip=1000 count=2000 | grep SEARCH_CONTENT
```



### 查看当前的网络流量

```bash
# 查看总的流入和流出流量
iftop

# 查看对应应用接收和发送的流量
nethogs
```



### 查看文件句柄

```bash
ulimit -a

# 查看系统打开句柄最大数量
cat /proc/sys/fs/file-max

# 查看打开句柄总数
lsof | awk '{print $2}' | wc -l

# 根据打开文件句柄的数量降序排列，其中第二列为进程ID：
lsof | awk '{print $2}' | sort | uniq -c | sort -nr | more

# 修改打开数
vim /etc/security/limits.conf 
root soft nofile 65535
root hard nofile 65535
* soft nofile 65535
* hard nofile 65535
```



## tcpdump

```bash
tcpdump -n -S -i eth0 host www.baidu.com and tcp port 80
```



## ip

[Linux ip Command Examples](https://www.cyberciti.biz/faq/linux-ip-command-examples-usage-syntax/)

![webwxgetmsgimg](https://i.imgur.com/KqAb9LA.jpg)