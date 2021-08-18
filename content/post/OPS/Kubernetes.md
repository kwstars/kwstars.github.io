---
title: "Kubernetes"
date: 2021-08-17T15:03:58+08:00
draft: true
tags: ["Kubernetes"]
categories: ["Kubernetes"]
---



## Prepare

### containerd

```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```



### docker

安装

```bash
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update -y
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```



配置

```bash
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "storage-driver": "overlay2",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
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
```



启动

```bash
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## Kubernetes

### [Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

Debian-based distributions

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

# 国内需要代理
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
# 下载后拷贝到对应目录
sudo cp apt-key.gpg /usr/share/keyrings/kubernetes-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
# 上面修改为
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# 安装
sudo apt-get update
sudo apt install -y kubelet=1.22.0-00 kubeadm=1.22.0-00 kubectl=1.22.0-00

# Only for CentOS
yum install -y kubelet-1.22.0 kubeadm-1.22.0 kubectl-1.22.0

sudo apt-mark hold kubelet kubeadm kubectl
```



### [Creating a cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

下载k8s安装所需的镜像，**docker需要设置代理**，文章在文下的引用中

```bash
kubeadm config images list

kubeadm config images pull --kubernetes-version=v1.22.0
```



启动kubelet并忽略swap错误

```bash
# 让kubelet忽略swap开启的错误
echo 'Environment="KUBELET_EXTRA_ARGS=--fail-swap-on=false"' >> /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

sudo systemctl daemon-reload
sudo systemctl enable kubelet
```



安装ControlPlaneEndpoint

```bash
sudo kubeadm init --kubernetes-version=v1.22.0 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap --control-plane-endpoint=k8s.linux88.com
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.8.31:6443 --token w9u3rd.ydfy41425fk408tk \
	--discovery-token-ca-cert-hash sha256:07928ff5fa21cfeee2c6b82ad3efe84c8f45e730b6861b42b2989d327c9ba1f5 

```



验证安装

```bash
$ kubectl get pods -A                                          
NAMESPACE     NAME                         READY   STATUS              RESTARTS   AGE
kube-system   coredns-558bd4d5db-xbf65     0/1     ContainerCreating   0          54s
kube-system   coredns-558bd4d5db-xhnrt     0/1     ContainerCreating   0          54s
kube-system   etcd-mi                      1/1     Running             0          61s
kube-system   kube-apiserver-mi            1/1     Running             0          61s
kube-system   kube-controller-manager-mi   1/1     Running             0          61s
kube-system   kube-proxy-wj94j             1/1     Running             0          55s
kube-system   kube-scheduler-mi            1/1     Running             0          61s

$ sudo ss -tunlop |grep kube                                                                                         
tcp    LISTEN  0       4096              127.0.0.1:37307          0.0.0.0:*      users:(("kubelet",pid=1708929,fd=11))                                          
tcp    LISTEN  0       4096              127.0.0.1:10248          0.0.0.0:*      users:(("kubelet",pid=1708929,fd=31))                                          
tcp    LISTEN  0       4096              127.0.0.1:10249          0.0.0.0:*      users:(("kube-proxy",pid=1710415,fd=19))                                       
tcp    LISTEN  0       4096              127.0.0.1:10257          0.0.0.0:*      users:(("kube-controller",pid=1708273,fd=7))                                   
tcp    LISTEN  0       4096              127.0.0.1:10259          0.0.0.0:*      users:(("kube-scheduler",pid=1708379,fd=7))                                    
tcp    LISTEN  0       4096                      *:10250                *:*      users:(("kubelet",pid=1708929,fd=32))                                          
tcp    LISTEN  0       4096                      *:6443                 *:*      users:(("kube-apiserver",pid=1708216,fd=7))                                    
tcp    LISTEN  0       4096                      *:10256                *:*      users:(("kube-proxy",pid=1710415,fd=12))   
```



Installing a Pod network add-on

```bash
# 2选1
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```



验证

```bash
$ kubectl get pods -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-58497c65d5-2rd2b   1/1     Running   0          27s
kube-system   calico-node-lpw94                          1/1     Running   0          27s
kube-system   coredns-558bd4d5db-xbf65                   1/1     Running   0          113s
kube-system   coredns-558bd4d5db-xhnrt                   1/1     Running   0          113s
kube-system   etcd-mi                                    1/1     Running   0          2m
kube-system   kube-apiserver-mi                          1/1     Running   0          2m
kube-system   kube-controller-manager-mi                 1/1     Running   0          2m
kube-system   kube-proxy-wj94j                           1/1     Running   0          114s
kube-system   kube-scheduler-mi                          1/1     Running   0          2m
```







---

[白话 Kubernetes Runtime](https://zhuanlan.zhihu.com/p/58784095)

[Container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)

[Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

[Docker HTTP/HTTPS proxy](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)

