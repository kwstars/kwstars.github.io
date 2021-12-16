---
title: "PV PVC StorageClass关系"
date: 2021-12-14T10:25:36+08:00
draft: true
tags: [""]
categories: ["Kubernetes"]
---



```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-recycler
  namespace: default
spec:
  restartPolicy: Never
  volumes:
  - name: vol
    hostPath:
      path: /any/path/it/will/be/replaced
  containers:
  - name: pv-recycler
    image: "k8s.gcr.io/busybox"
    command: ["/bin/sh", "-c", "test -e /scrub && rm -rf /scrub/..?* /scrub/.[!.]* /scrub/*  && test -z \"$(ls -A /scrub)\" || exit 1"]
    volumeMounts:
    - name: vol
      mountPath: /scrub
```





---

[Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

[PV、PVC、StorageClass讲解](https://www.cnblogs.com/rexcheny/p/10925464.html)
