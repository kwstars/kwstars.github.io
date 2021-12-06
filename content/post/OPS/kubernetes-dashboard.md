---
title: "Kubernetes Dashboard"
date: 2021-12-06T16:28:06+08:00
draft: false
tags: [""]
categories: ["Kubernetes"]
---



## 一、部署Dashboard

### 1.1 部署

```bash
GITHUB_URL=https://github.com/kubernetes/dashboard/releases
VERSION_KUBE_DASHBOARD=$(curl -w '%{url_effective}' -I -L -s -S ${GITHUB_URL}/latest -o /dev/null | sed -e 's|.*/||')
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/${VERSION_KUBE_DASHBOARD}/aio/deploy/recommended.yaml
```



### 1.2 RBAC 配置

创建以下资源清单文件  **dashboard.yaml**

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard 
```



部署管理员用户配置

```bash
kubectl create -f dashboard.admin-user.yml -f dashboard.admin-user-role.yml
```



### 1.3 获取Bearer Tokenl

```bash
kubectl -n kubernetes-dashboard describe secret admin-user-token | grep '^token'
```



## 二、升级Dashboard

```bash
kubectl delete ns kubernetes-dashboard
GITHUB_URL=https://github.com/kubernetes/dashboard/releases
VERSION_KUBE_DASHBOARD=$(curl -w '%{url_effective}' -I -L -s -S ${GITHUB_URL}/latest -o /dev/null | sed -e 's|.*/||')
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/${VERSION_KUBE_DASHBOARD}/aio/deploy/recommended.yaml -f dashboard.yml
```



## 三、删除Dashboard

```bash
kubectl delete ns kubernetes-dashboard
kubectl delete clusterrolebinding kubernetes-dashboard
kubectl delete clusterrole kubernetes-dashboard
```



---

[Deploy and Access the Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

[Kubernetes Dashboard](https://rancher.com/docs/k3s/latest/en/installation/kube-dashboard/)
