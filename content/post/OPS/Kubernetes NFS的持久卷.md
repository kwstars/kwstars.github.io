---
title: "Kubernetes"
date: 2021-12-12T15:20:29+08:00
draft: true
tags: ["Kubernetes"]
categories: ["OPS"]

---



## 一、安装NFS服务器

安装nfs服务器

```bash
# nfs server for centos8
dnf install nfs-utils

$ sudo cat > /etc/exports << EOF
/data/k8s_data/		192.168.0.100/24(rw,sync,no_root_squash,no_all_squash,no_acl)
EOF

# 启动nfs
systemctl start nfs-server.service
systemctl enable nfs-server.service
systemctl status nfs-server.service

# 检查nfs
$ showmount -e 127.0.0.1
Export list for 127.0.0.1:
/data/k8s_data 192.168.0.100/24
```



### 二、各个节点需要安装nfs客户端

```bash
# nfs clinet for ubuntu
sudo apt install nfs-common
```



## 三、创建PV，PVC

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
  labels:
    release: stable
spec:
  capacity:
    storage: 1024Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path:  "/data/k8s_data"
    server: 192.168.0.100
    readOnly: false
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  storageClassName: nfs
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1024Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
          volumeMounts:
            - mountPath: "/usr/share/nginx/html"
              name: nginx-pv-storage
      volumes:
        - name: nginx-pv-storage
          persistentVolumeClaim:
            claimName: nfs-pvc
```



```bash
$ kubectl get pods                                                                                                                                                                                                           
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-85694dcf79-nq4sn   1/1     Running   0          40m
nginx-deployment-85694dcf79-xm47j   1/1     Running   0          40m

$ kubectl exec -it nginx-deployment-85694dcf79-nq4sn -- bash
echo 111 >> /usr/share/nginx/html/1.txt

$ kubectl exec -it nginx-deployment-85694dcf79-nq4sn -- cat /usr/share/nginx/html/1.txt
111
```

