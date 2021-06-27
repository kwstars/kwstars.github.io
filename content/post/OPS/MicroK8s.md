---
title: "MicroK8s"
date: 2020-11-18T09:46:33+08:00
draft: false
tags: ["Kubernetes"]
categories: ["OPS"]
---





## 一、安装MicroK8s

### 1.1 安装

```bash
# 安装MicroK8s
sudo snap install microk8s --classic --channel=1.19

# 查所有MicroK8s的版本
snap info microk8s

# 加入用户组并对会话刷新
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
su - $USER

# 防火墙设置
sudo ufw allow in on cni0 && sudo ufw allow out on cni0
sudo ufw default allow routed

# 检查状态
microk8s status

# 将microk8s.kubectl别名为kubectl
sudo snap alias microk8s.kubectl kubectl

# 启动相关的add-on
microk8s.enable dashboard dns
```



### 1.2 一些排错命令

```bash
# 检查
microk8s inspect

# Ubuntu的日志
sudo snap logs -f microk8s

# 添加代理来下载k8s.gcr.io/pause镜像
vim /var/snap/microk8s/current/args/containerd-env
HTTP_PROXY=http://192.168.1.192:8118
HTTPS_PROXY=http://192.168.1.192:8118

# 修改register
vim /var/snap/microk8s/current/args/containerd-template.toml
......
    [plugins.cri.registry]
      [plugins.cri.registry.mirrors]
        [plugins.cri.registry.mirrors."docker.io"]
          endpoint = ["https://registry-1.docker.io"]
        [plugins.cri.registry.mirrors."localhost:32000"]
          endpoint = ["http://localhost:32000"]
......

# 检查node节点
$ microk8s.kubectl get nodes 

# 查看所有的pods
$ microk8s.kubectl get pods -A
```



## 二、部署nginx

```bash
# 部署nginx容器
microk8s.kubectl create deployment nginx --image=nginx

# 查看容器
microk8s.kubectl get pods

# 查看容器的一些信息
microk8s.kubectl describe pod POD_NAME
 
# 进入容器
microk8s.kubectl exec -it POD_NAME -- sh
```



---

[番外篇：使用MicroK8s](https://www.jianshu.com/p/8350d49c99cf)

[Introduction to MicroK8s](https://microk8s.io/docs)