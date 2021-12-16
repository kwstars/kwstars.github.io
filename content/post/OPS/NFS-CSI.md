---
title: "NFS CSI"
date: 2021-12-14T12:11:52+08:00
draft: true
tags: [""]
categories: ["Kubernetes"]
---



```yaml
---
kind: Service
apiVersion: v1
metadata:
  name: nfs-server
  labels:
    app: nfs-server
spec:
  type: ClusterIP  # use "LoadBalancer" to get a public ip
  selector:
    app: nfs-server
  ports:
    - name: tcp-2049
      port: 2049
      protocol: TCP
    - name: udp-111
      port: 111
      protocol: UDP
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nfs-server
  template:
    metadata:
      name: nfs-server
      labels:
        app: nfs-server
    spec:
      nodeSelector:
        "kubernetes.io/os": linux
        "kubernetes.io/hostname": nfs-server # modify node label
      containers:
        - name: nfs-server
          image: itsthenetwork/nfs-server-alpine:latest
          env:
            - name: SHARED_DIRECTORY
              value: "/exports"
          volumeMounts:
            - mountPath: /exports
              name: nfs-vol
          securityContext:
            privileged: true
          ports:
            - name: tcp-2049
              containerPort: 2049
              protocol: TCP
            - name: udp-111
              containerPort: 111
              protocol: UDP
      volumes:
        - name: nfs-vol
          hostPath:
            path: /data/nfs01/  # modify this to specify another path to store nfs share data
            type: DirectoryOrCreate
```



```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nginx
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  mountOptions:
    - hard
    - nfsvers=4.1
  csi:
    driver: nfs.csi.k8s.io
    readOnly: false
    volumeHandle: unique-volumeid  # make sure it's a unique id in the cluster
    volumeAttributes:
      server: nfs-server.default.svc.cluster.local
      share: /
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-nginx
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: pv-nginx
  storageClassName: ""
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-nfs-example
spec:
  containers:
    - image: nginx
      name: nginx
      ports:
        - containerPort: 80
          protocol: TCP
      volumeMounts:
        - mountPath: /var/www
          name: pvc-nginx
  volumes:
    - name: pvc-nginx
      persistentVolumeClaim:
        claimName: pvc-nginx
```



## Storage Class Usage (Dynamic Provisioning)

```yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi
provisioner: nfs.csi.k8s.io
parameters:
  server: nfs-server.default.svc.cluster.local
  share: /
reclaimPolicy: Delete
volumeBindingMode: Immediate
mountOptions:
  - hard
  - nfsvers=4.1
```



```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs-dynamic
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: nfs-csi
```



---

[CSI driver example](https://github.com/kubernetes-csi/csi-driver-nfs/tree/master/deploy/example)

[Set up a NFS Server on a Kubernetes cluster](https://github.com/kubernetes-csi/csi-driver-nfs/blob/master/deploy/example/nfs-provisioner/README.md)
