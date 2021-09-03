---
title: "Docker和docker Compose的安装与配置"
date: 2019-11-03T19:34:51+08:00
draft: false
tags: ["Docker"]
categories: ["OPS"]
---



## 一、安装
### 1.1 安装docker

安装前准备

```bash
# 关闭selinux, 一定要重启 
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config

# CentOS7修改docker源
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
```



**[根据不同的系统版本安装 Docker Engine](https://docs.docker.com/engine/install/centos/)**



### 1.2 安装docekr-compose

```bash
# 安装epel源 
sudo yum install -y epel-release -y   

# 安装pip 
sudo yum install -y python3-pip

# 修改pip源，如果全局生效修改 /etc/pip.conf 
mkdir ~/.pip 
cat > ~/.pip/pip.conf << EOF 
[global]
timeout = 6000
index-url = https://mirrors.aliyun.com/pypi/simple/
trusted-host = mirrors.aliyun.com
EOF

# 安装docker-compose
pip3 install --upgrade pip setuptools docker-compose
```

## 二、配置
### 2.1 使用国内镜像
```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "1m",
    "max-file": "1"
  },
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn",
    "https://reg-mirror.qiniu.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://registry.docker-cn.com"
  ]
}
EOF

# 重启docker
sudo systemctl restart docker 
```



  `"iptables": false` Docker的FORWARD不生成转发规则, **docker容器不能访问外网**

```bash
$ sudo iptables -nvL
Chain DOCKER (2 references)
 pkts bytes target     prot opt in     out     source               destination
```





### 2.2 如果国内不好用就使用代理吧

[**通过systemd控制docker的修改方法**](https://docs.docker.com/config/daemon/systemd/)

```bash
# 创建目录
sudo mkdir -p /etc/systemd/system/docker.service.d

# 写入代理配置
cat > /etc/systemd/system/docker.service.d/http-proxy.conf << EOF
[Service]
Environment="HTTP_PROXY=http://192.168.1.192:8118/"
Environment="HTTPS_PROXY=http://192.168.1.192:8118/"
Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp"
EOF

# 加载配置, 重启docker
sudo systemctl daemon-reload
sudo systemctl restart docker 

# 验证配置
sudo systemctl show --property=Environment docker

# 通过查看日志排错 
sudo journalctl -u docker
```



### 2.3 确认代理是否生效

```bash
$ sudo docker info
......
# 设置代理
 HTTP Proxy: http://192.168.1.192:8118/
 HTTPS Proxy: http://192.168.1.192:8118/
 No Proxy: localhost,127.0.0.1,docker-registry.example.com,.corp
......
 Insecure Registries:
  127.0.0.0/8
 
 # 走国内的
 Registry Mirrors:
  https://dockerhub.azk8s.cn/
  https://reg-mirror.qiniu.com/
  https://docker.mirrors.ustc.edu.cn/
  https://registry.docker-cn.com/
```



### 2.4 Run Docker As Non-root User In Linux

```bash
# 将用户添加到docker组
sudo usermod -aG docker ${USER}

# 打开一个新的shell来更新这个用户的组信息
su - ${USER}
```



## 三、清理image和container

```bash
# 删除 停止的容器，没有引用的网络和images，build cache 
docker system prune -a

# 删除没有运行的容器
docker rm $(docker ps -a -q)
```



## 四、修改docker目录

**查看位置**

```bash
# docker info
......
 ID: XHOE:7QS3:JR4F:G67M:XZWR:CW6L:ZDX3:US4Y:XLTM:UNEJ:SZJT:Y4FB
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
......
```



**修改位置**

```bash
# vim /etc/docker/daemon.json 
{
  "data-root": "/www/docker"
}
```



五、Docker和iptables

所有Docker的iptables规则都被添加到`DOCKER`链中。不要手动操作此表。如果你需要在Docker的规则之前添加规则，请将它们添加到`DOCKER-USER`链中。这些规则会在Docker自动创建的任何规则之前加载。

```bash
# ubuntu安装包并开启启动
$ sudo apt install iptables-persistent -y
$ sudo systemctl enable netfilter-persistent
$ sudo netfilter-persistent -h
Usage: /usr/sbin/netfilter-persistent (start|stop|restart|reload|flush|save)

# 只允许192.168.1.1访问
$ iptables -I DOCKER-USER -i eth0 ! -s 192.168.1.1 -j DROP

# 只允许192.168.1.0/24网段访问
$ iptables -I DOCKER-USER -i eth0 ! -s 192.168.1.0/24 -j DROP

# 允许192.168.1.1-192.168.1.3区间访问
$ iptables -I DOCKER-USER -m iprange -i eth0 ! --src-range 192.168.1.1-192.168.1.3 -j DROP

# 关联主动请求的地址
$ iptables -I DOCKER-USER -i eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT

# 允许相关连接
$ iptables -I DOCKER-USER -i eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
```



要完全防止 Docker 操纵 iptables 策略，请将 /etc/docker/daemon.json 中的 iptables 键设置为 false。这对大多数用户来说是不合适的，因为 iptables 策略需要手动管理。



---
[How To Run Docker As Non-root User In Linux](https://ostechnix.com/how-to-run-docker-as-non-root-user-in-linux/)

[Configure Docker to use a proxy server](https://docs.docker.com/network/proxy/)

[Docker and iptables](https://docker-docs.netlify.app/network/iptables/)

[Saving Iptables Firewall Rules Permanently](https://www.thomas-krenn.com/en/wiki/Saving_Iptables_Firewall_Rules_Permanently)