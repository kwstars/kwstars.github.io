---
title: "Kubernetes"
date: 2021-11-15T15:20:29+08:00
draft: true
tags: ["Kubernetes"]
categories: ["OPS"]

---

# 5. The Workloads

```bash
kubectl run nginx --image=nginx

# 通过将 --dry-run=client -o yaml 添加到命令中，您可以看到创建相同 Pod 必须编写的 YAML 模板：
kubectl run nginx --image=nginx --dry-run=client -o yaml
```



## Pod Specs

容器字段将更精确地定义和参数化Pod的每个容器，无论它是普通容器（containers）还是init容器（initContainers）。imagePullSecrets字段将有助于从私人注册中心下载容器镜像。

Volumes字段（卷）将定义一个卷的列表，容器将能够装载和共享。

Scheduling字段将帮助你定义最合适的节点来部署Pod，通过标签（nodeSelector）选择节点，直接指定节点名称（nodeName），使用亲和力和容忍度，选择一个特定的调度器（schedulerName），并要求一个特定的运行时类（runtimeClassName）。它们还将被用来确定一个Pod相对于其他Pod的优先级（priorityClassName和优先级）。

生命周期字段将有助于定义一个Pod在终止后是否应该重新启动（restartPolicy），并微调在终止的Pod的容器中运行的进程被杀死的时间（terminalGracePeriodSeconds），或在运行的Pod尚未终止时被停止的时间（activeDeadlineSeconds）。它们也有助于定义一个Pod的准备状态（readinessGates）。

主机名和名称解析字段将有助于定义Pod的主机名（hostname）和部分FQDN（subdomain），在容器的/etc/hosts文件中添加主机（hostAliases），微调容器的/etc/resolv.conf文件（dnsConfig），并定义DNS配置的策略（dnsPolicy）。

主机命名空间字段将有助于说明Pod是否必须使用网络（hostNetwork）、PID（hostPID）和IPC（hostIPC）的主机命名空间，以及容器是否将共享同一（非主机）进程命名空间（shareProcessNamespace）。

服务账户字段对于赋予一个Pod特定的权利非常有用，通过影响它一个特定的服务账户（serviceAccountName）或者通过automountServiceAccountToken禁用默认服务账户的自动挂载。

安全上下文字段（securityContext）有助于在Pod级别定义各种安全属性和通用容器设置。



## Container Specs

与容器运行时相关的字段如下：

- 镜像字段定义了容器的镜像（image）和提取镜像的策略（imagePullPolicy）。

- Entrypoint 字段定义了入口的命令（command）和参数（args）以及其工作目录（workingDir）。

- 端口字段（ports）定义了要从容器中暴露出来的端口列表。

- 环境变量字段有助于定义将在容器中导出的环境变量，可以直接导出（env），也可以通过引用 ConfigMap 或 Secret 值导出（envFrom）。

- 卷字段定义了要挂载到容器中的卷，无论它们是文件系统卷（volumeMounts）还是原始块卷（volumeDevices）。

Kubernetes相关的字段如下：

- 资源字段（resources）有助于定义容器的资源要求和限制。

- 生命周期字段有助于定义生命周期事件的处理程序（lifecycle），参数化终止消息（terminalMessagePath和terminalMessagePolicy），并定义探针来检查容器的有效性（livenessProbe）和就绪性（readinessProbe）。

- 安全上下文字段有助于在容器级别定义各种安全属性和常见的容器设置。

- 调试字段是非常专业的字段，主要用于调试目的（stdin、stdinOnce和tty）。



## Pod Controllers

ReplicaSet: 确保在任何时候都有指定数量的Pod副本在运行。

Deployment: 启用Pod和ReplicaSets的声明式更新。

StatefulSet: 管理Pods和ReplicaSets的更新，照顾到有状态的资源。

DaemonSet: 确保所有或某些节点都在运行一个Pod的副本。

Job: 启动Pod并确保其完成。

CronJob: 在一个基于时间的时间表上创建一个Job。

## ReplicaSet Controller



## Update and Rollback

```bash
kubectl create deployment nginx --image=nginx:1.10

# 子命令 status 为我们提供了部署的状态
$ kubectl rollout status deployment nginx
deployment "nginx" successfully rolled out

# history 子命令为我们提供了部署的修订历史。这里的部署是第一次修订：
$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>

# 我们现在将更新 nginx 的映像以使用 1.11 修订版。一种方法是使用 kubectl set image 命令
$ kubectl set image deployment nginx nginx=nginx:1.11

# 我们可以使用 history 子命令看到 Deployment 处于第二次修订：
$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

# 更改原因默认为空。它可以包含用于通过使用 --record 标志进行部署的命令
$ kubectl set image deployment nginx nginx=nginx:1.12 --record
$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         kubectl set image deployment nginx nginx=nginx:1.12 --record=true

# 或者在发布后设置 kubernetes.io/change-cause 注释
$ kubectl set image deployment nginx nginx=nginx:1.13
$ kubectl annotate deployment nginx kubernetes.io/change-cause="update to revision 1.13" --record=false --overwrite=true
$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         kubectl set image deployment nginx nginx=nginx:1.12 --record=true
4         update to revision 1.13

# 还可以编辑部署的规范：
$ kubectl edit deployment nginx
 36     spec:
 37       containers:
 38       - env:
 39         - name: FOO
 40           value: bar
 41         image: nginx:1.13

# 查看
$ kubectl describe pod -l app=nginx | grep -A1 "Environment:"
    Environment:
      FOO:  bar

# 让我们为此版本设置更改原因并查看历史记录：
$ kubectl annotate deployment nginx kubernetes.io/change-cause="add FOO environment variable" --record=false --overwrite=true
$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         kubectl set image deployment nginx nginx=nginx:1.12 --record=true
4         update to revision 1.13
5         add FOO environment variable

# 现在让我们使用 undo 子命令回滚最后一个 rollout：
$ kubectl rollout undo deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         kubectl set image deployment nginx nginx=nginx:1.12 --record=true
5         add FOO environment variable
6         update to revision 1.13

$ kubectl rollout undo deployment nginx --to-revision=3
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
5         add FOO environment variable
6         update to revision 1.13
7         kubectl set image deployment nginx nginx=nginx:1.12 --record=true

# 最后，您可以验证每个修订版是否存在一个 ReplicaSet：
kubectl get replicaset
NAME               DESIRED   CURRENT   READY   AGE
nginx-5b58f46894   1         1         1       12m
nginx-5b946576d4   0         0         0       17m
nginx-76b5cc478f   0         0         0       10m
nginx-78bfc8f6bc   0         0         0       13m
nginx-7c6bdb84db   0         0         0       6m17s
```



## Deployment Strategies

你在 "部署控制器 "部分已经看到，在改变新旧ReplicaSets的复制数量时，部署控制器提供了不同的策略。

### The Recreate Strategy

最简单的策略是`Recreate`策略：在这种情况下，旧的ReplicaSet将被缩小到零，当这个ReplicaSet的所有Pod都停止后，新的ReplicaSet将被扩大到要求的复制数量。

一些后果如下。

- 在旧的Pods停止和新的Pods开始时，会有一个小的停机时间。
- 不需要额外的资源来并行运行以前和新的Pod。
- 新旧版本将不会同时运行。

### The RollingUpdate Strategy

RollingUpdate 策略是一种更高级的策略，并且是您创建 Deployment 时的默认策略。



## Running Jobs

```bash
kubectl get pods
kubectl get jobs
kubectl get jobs a-job -o yaml
```



## CronJob Controller

# 6. Configuring Applications

一个应用程序可以用不同的方式进行配置。

- 通过向命令传递参数
- 通过定义环境变量

- 使用配置文件

## Arguments to the Command



## Environment Variables

### Declaring Values Directly

#### In Declarative Form

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        env:
        - name: VAR1
          value: "value1"
        - name: VAR2
          value: "value2"
```



#### In Imperative Form

```bash
kubectl create deployment nginx --image=nginx
kubectl set env deployment nginx \
  --env VAR1=value1 \
  --env VAR2=value2
```



### Referencing Specific Values from ConfigMaps and Secrets

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vars
data:
  var1: value1
  var2: value2
---
apiVersion: v1
kind: Secret
metadata:
  name: passwords
stringData:
  pass1: foo
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        env:
        - name: VAR1
          valueFrom:
            configMapKeyRef:
                key: var1
                name: vars
        - name: VAR2
          valueFrom:
            configMapKeyRef:
                key: var2
                name: vars
        - name: PASS1
          valueFrom:
            secretKeyRef:
                key: pass1
                name: passwords
```



请注意，如果在所引用的ConfigMaps或Secrets中找不到所引用的键，则创建部署将失败。如果你想在某个值不存在的情况下创建部署（在这种情况下，相应的环境变量将不会被定义），你可以使用可选字段。

```yaml
- name: PASS2
  valueFrom:
    secretKeyRef:
        key: pass2
        name: passwords
        optional: true
```



#### In Imperative Form

```bash
$ kubectl create configmap vars \
  --from-literal=var1=value1 \
  --from-literal=var2=value2 \
  --from-literal=var3=value3
configmap/vars created

$ kubectl create secret generic passwords \
  --from-literal=pass1=foo \
  --from-literal=pass2=bar
secret/passwords created

$ kubectl create deployment nginx --image=nginx
deployment.apps/nginx created

$ kubectl set env deployment nginx \
  --from=configmap/vars \
  --keys="var1,var2"
deployment.apps/nginx env updated

$ kubectl set env deployment nginx \
  --from=secret/passwords \
  --keys="pass1"
deployment.apps/nginx env updated
```



### Referencing All Values from ConfigMaps and Secrets

#### In Declarative Form

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vars
data:
  var1: value1
  var2: value2
---
apiVersion: v1
kind: Secret
metadata:
  name: passwords
stringData:
  pass1: foo
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app:  nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        envFrom:
        - configMapRef:
            name: vars
        - secretRef:
            name: passwords
        - secretRef:
            name: notfound
            optional: true
```



#### In Imperative Form

```bash
$ kubectl create configmap vars \
  --from-literal=var1=value1 \
  --from-literal=var2=value2
configmap/vars created

$ kubectl create secret generic passwords \
  --from-literal=pass1=foo
secret/passwords created

$ kubectl create deployment nginx --image=nginx
deployment.apps/nginx created

$ kubectl set env deployment nginx \
  --from=configmap/vars
deployment.apps/nginx env updated

$ kubectl set env deployment nginx \
  --from=secret/passwords
deployment.apps/nginx env updated
```



### Referencing Values from Pod Fields

- metadata.name
- metadata.namespace
- metadata.uid
- spec.nodeName
- spec.serviceAccountName
- status.hostIP
- status.podIP

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
                fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
                fieldPath: metadata.namespace
        - name: POD_UID
          valueFrom:
            fieldRef:
                fieldPath: metadata.uid
        - name:   POD_NODENAME
          valueFrom:
            fieldRef:
                fieldPath: spec.nodeName
        - name: POD_SERVICEACCOUNTNAME
          valueFrom:
            fieldRef:
                fieldPath: spec.serviceAccountName
        - name: POD_HOSTIP
          valueFrom:
            fieldRef:
                fieldPath: status.hostIP
        - name: POD_PODIP
          valueFrom:
            fieldRef:
                fieldPath: status.podIP
```



```bahs
kubectl exec -it nginx-xxxxxxxxxx-yyyyy bash -- -c "env | grep POD_"
```



### Referencing Values from Container Resources Fields

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources:
          requests:
            cpu: "500m"
            memory: "500Mi"
          limits:
            cpu: "1"
            memory: "1Gi"
        env:
        - name: M_CPU_REQUEST
          valueFrom:
            resourceFieldRef:
                resource: requests.cpu
                divisor: "0.001"
        - name: MEMORY_REQUEST
          valueFrom:
            resourceFieldRef:
                resource: requests.memory
        - name:  M_CPU_LIMIT
          valueFrom:
            resourceFieldRef:
                resource: limits.cpu
                divisor: "0.001"
        - name: MEMORY_LIMIT
          valueFrom:
            resourceFieldRef:
                resource: limits.memory
```

M_CPU_REQUEST=500

MEMORY_REQUEST=524288000

M_CPU_LIMIT=1000

MEMORY_LIMIT=1073741824



### Configuration File from ConfigMap

以在容器文件系统中挂载 ConfigMap 内容。挂载的 ConfigMap 的每个键/值将是一个文件名及其在挂载目录中的内容。 

例如，您可以以声明形式创建此 ConfigMap：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config
data:
  nginx.conf: |
    server {
      location / {
        root /data/www;
      }
      location /images/ {
        root /data;
      }
    }

# 以命令形式：
$ cat > nginx.conf <<EOF
server {
    location / {
        root /data/www;
    }
    location /images/ {
        root /data;
    }
}
EOF

$ kubectl create configmap config --from-file=nginx.conf
configmap/config created

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas:  1
  selector:
    matchLabels:
      app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        volumes:
        - name: config-volume
          configMap: config
        containers:
        - image: nginx
          name: nginx
          volumeMounts:
          - name: config-volume
            mountPath: /etc/nginx/conf.d/
```



### Configuration File from Secret

```bash
apiVersion: v1
kind: Secret
metadata:
  name: passwords
stringData:
  password: foobar
  
# or imperative form:
$ kubectl create secret generic passwords \
  --from-literal=password=foobar
secret/passwords created

# Then mount the Secret in the container:
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
      - name: passwords-volume
        secret:
          secretName: passwords
          containers:
          - image: nginx
            name: nginx
            volumeMounts:
            - name: passwords-volume
              mountPath: /etc/passwords
```



### Configuration File from Pod Fields

声明性地，可以使用包含 Pod 值的文件挂载卷：

- metadata.name
- metadata.namespace
- metadata.uid
- metadata.labels
- metadata.annotations

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: pod-info
          mountPath: /pod
          readOnly: true
      volumes:
        - name: pod-info
          downwardAPI:
            items:
            - path: metadata/name
              fieldRef:
                fieldPath: metadata.name
            - path: metadata/namespace
              fieldRef:
                fieldPath: metadata.namespace
            - path: metadata/uid
              fieldRef:
                fieldPath: metadata.uid
            - path: metadata/labels
              fieldRef:
                fieldPath: metadata.labels
            - path: metadata/annotations
              fieldRef:
                fieldPath: metadata.annotations
```

As a result, in the container, you can find files in /pod/metadata:

```bash
$ kubectl exec nginx-xxxxxxxxxx-yyyyy bash -- -c \
  'for i in /pod/metadata/*; do echo $i; cat -n $i; echo ; done'
/pod/metadata/annotations
     1  kubernetes.io/config.seen="2020-01-11T17:21:40.497901295Z"
     2  kubernetes.io/config.source="api"
/pod/metadata/labels
     1  app="nginx"
     2  pod-template-hash="789ccf5b7b"
/pod/metadata/name
     1  nginx-xxxxxxxxxx-yyyyy
/pod/metadata/namespace
     1  default
/pod/metadata/uid
     1  631d01b2-eb1c-49dc-8c06-06d244f74ed4
```



### Configuration File from Container Resources Fields

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources:
          requests:
            cpu: "500m"
            memory: "500Mi"
          limits:
            cpu: "1"
            memory: "1Gi"
        volumeMounts:
        - name: resources-info
          mountPath: /resources
          readOnly: true
      volumes:
        - name: resources-info
          downwardAPI:
            items:
            - path: limits/cpu
              resourceFieldRef:
                resource: limits.cpu
                divisor: "0.001"
                containerName: nginx
            - path: limits/memory
              resourceFieldRef:
                resource: limits.memory
                containerName: nginx
            - path: requests/cpu
              resourceFieldRef:
                resource: requests.cpu
                divisor: "0.001"
                containerName: nginx
            - path: requests/memory
              resourceFieldRef:
                resource: requests.memory
                containerName: nginx
```

```bash
$ kubectl exec nginx-85d7c97f64-9knh9 bash -- -c \
  'for i in /resources/*/*; do echo $i; cat -n $i; echo ; done'
/resources/limits/cpu
     1  1000
/resources/limits/memory
     1  1073741824
/resources/requests/cpu
     1  500
/resources/requests/memory
     1  524288000
```



### Configuration File from Different Sources

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: values
data:
  cpu: "4000"
  memory: "17179869184"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources:
          requests:
            cpu: "500m"
            memory: "500Mi"
          limits:
            cpu: "1"
            memory: "1Gi"
        volumeMounts:
        - name: config
          mountPath: /config
          readOnly: true
      volumes:
        - name: config
          projected:
            sources:
            - configMap:
                name: values
                items:
                - key: cpu
                  path: cpu/value
                - key: memory
                  path: memory/value
            - downwardAPI:
                items:
                - path: cpu/limits
                  resourceFieldRef:
                    resource: limits.cpu
                    divisor: "0.001"
                    containerName: nginx
                - path: memory/limits
                  resourceFieldRef:
                    resource: limits.memory
                    containerName: nginx
                - path: cpu/requests
                  resourceFieldRef:
                    resource: requests.cpu
                    divisor: "0.001"
                    containerName: nginx
                - path: memory/requests
                  resourceFieldRef:
                    resource: requests.memory
                    containerName: nginx
```



```bash
$ kubectl exec nginx-7d797b5788-xzw79 bash -- -c \
  'for i in /config/*/*; do echo $i; cat -n $i; echo ; done'
/config/cpu/limits
     1  1000
/config/cpu/requests
     1  500
/config/cpu/value
     1  4000
/config/memory/limits
     1  1073741824
/config/memory/requests
     1  524288000
/config/memory/value
     1  17179869184
```



# 7. Scaling an Application

## Manual Scaling

```yaml
apiVersion: apps/v1 kind:
Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image:  nginx
        name: nginx
```



```bash
$ kubectl create deployment nginx --image=nginx
deployment.apps/nginx created
$ kubectl scale deployment nginx --replicas=4
deployment.apps/nginx scaled
```



## Auto-scaling

HorizontalPodAutoscaler 资源（通常称为 HPA）可用于根据当前副本的 CPU 使用情况自动扩展部署。

```bash
 kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml
```



您必须编辑 metrics-server 部署，以便将 --kubelet-insecure-tls 和 --kubelet-preferred-address-types=InternalIP 标志添加到容器中启动的命令，并添加 hostNetwork: true：

```bash
$ kubectl edit deployment metrics-server -n kube-system
    spec:
      hostNetwork: true ## add this line
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-insecure-tls # add this line
        - --kubelet-preferred-address-types=InternalIP ## add this line
```



您可以使用以下命令检查 v1beta1.metrics.k8s.io APIService 的状态：

```bash
$ kubectl get apiservice v1beta1.metrics.k8s.io -o yaml
[...]
status:
  conditions:
  - lastTransitionTime: "2020-08-15T15:38:44Z"
    message: all checks passed
    reason: Passed
    status: "True"
    type: Available
 
$ kubectl top nodes
NAME         CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
Controller   95m          9%     1256Mi              34%
worker-0     34m          3%     902Mi               25%
worker-1     30m          3%     964Mi               26%
```



您现在可以使用单个副本启动部署。请注意，HPA 需要 CPU 资源请求才能工作：

```bash
$ kubectl create deployment nginx --image=nginx
deployment.apps/nginx created
$ kubectl set resources --requests=cpu=0.05 deployment/nginx
deployment.extensions/nginx resource requirements updated
```



现在为此部署创建一个 HorizontalPodAutoscaler 资源，它将能够以命令形式将部署从一个副本自动扩展到四个副本，CPU 利用率为 5%：

```bash
$ kubectl autoscale deployment nginx \
--min=1 --max=4 --cpu-percent=5
horizontalpodautoscaler.autoscaling/nginx autoscaled

# or in declarative form:
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx
spec:
  minReplicas: 1
  maxReplicas: 4
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  targetCPUUtilizationPercentage: 5
```



要增加当前运行的 Pod 的 CPU 利用率，您可以使用 curl 命令对其发出大量请求：

```bash
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-xxxxxxxxxx-yyyyy   1/1     Running   0          31s

$ kubectl port-forward pod/nginx-xxxxxxxxxx-yyyyy 8084:80
Forwarding from 127.0.0.1:8084 -> 80
and in another terminal:
$ while : ; do curl http://localhost:8084; done

# 同时，您可以使用以下命令跟踪 Pod 对 CPU 的使用情况：
$ kubectl top pods
NAME                     CPU(cores)   MEMORY(bytes)
nginx-xxxxxxxxxx-yyyyy   3m           2Mi

# 一旦 CPU 使用率超过 5%，就会自动部署第二个 Pod：
$ kubectl get hpa nginx
NAME    REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS
nginx   Deployment/nginx   7%/5%     1         4         2

$ kubectl get pods
NAME                     READY   STATUS   RESTARTS   AGE
nginx-5c55b4d6c8-fgnlz   1/1     Running  0          12m
nginx-5c55b4d6c8-hzgfl   1/1     Running  0          81s

# 如果停止 curl 请求并观察创建的 HPA，可以看到在 CPU 利用率再次低下 5 分钟后，副本数将再次设置为 1：
$ kubectl get hpa nginx -w
NAME    REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS AGE
nginx   Deployment/nginx   4%/5%     1         4         2        10m
nginx   Deployment/nginx   3%/5%     1         4         2        10m
nginx   Deployment/nginx   0%/5%     1         4         2        11m
nginx   Deployment/nginx   0%/5%     1         4         1        16m

$ kubectl describe hpa nginx
[...]
Events:
  Type   Reason            Age From                      Message
  ----   ------            --- ----                      -------
  Normal SuccessfulRescale 10m horizontal-pod-autoscaler New size: 2; reason: \
cpu resource utilization (percentage of request) above target
  Normal SuccessfulRescale 2m47s horizontal-pod-autoscaler New size: 1; reason: \
All metrics below target
```



# 8. Application Self-Healing

```bash
# 此处，Pod 已被调度到节点 worker-0 上。 让我们将此节点置于维护模式，看看 Pod 会发生什么：
$ kubectl drain worker-0 --force
node/worker-0 cordoned
WARNING: deleting Pods not managed by ReplicationController, ReplicaSet, Job, Daemon\
Set or StatefulSet: default/nginx
evicting pod "nginx"
pod/nginx evicted
node/worker-0 evicted

$ kubectl get pods
No resources found in default namespace.
```



## Controller to the Rescue



## Liveness Probes

可以为 Pod 的每个容器定义一个活跃度探测器。如果 kubelet 无法成功执行给定次数的探测器，则容器被认为不健康并重新启动到同一个 Pod 中。

liveness 有以下三种可能：

1. Make an HTTP request.
2. Execute a command.
3. Make a TCP connection.

请注意，也可以为一个容器定义一个准备就绪的探针。准备就绪探针的主要作用是表明一个Pod是否准备好为网络请求提供服务。当准备就绪探针成功时，该Pod将被添加到匹配服务的后端列表中。

之后，在容器的执行过程中，如果准备就绪探针失败，该Pod将从服务的后端列表中删除。这对于检测一个容器是否能够处理更多的连接（例如，如果它已经处理了大量的连接）并停止发送新的连接是非常有用的。

### HTTP Request Liveness Probe

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    livenessProbe:
      httpGet:
        scheme: HTTP
        port: 80
        path: /healthz
```



### Command Liveness Probe

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: postgres
spec:
  containers:
  - image: postgres
    name: postgres
    livenessProbe:
      initialDelaySeconds: 10
      exec:
        command:
        - "psql"
        - "-h"
        - "localhost"
        - "-U"
        - "unknownUser"
        - "-c"
        - "select 1"
```



### TCP Connection Liveness Probe

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: postgres
spec:
  containers:
  - image: postgres
    name: postgres
    livenessProbe:
      initialDelaySeconds: 10
      tcpSocket:
        port: 5433
```



## Resource Limits and Quality of Service (QoS) Classes



# 9. Scheduling Pods

## Adding Labels to Nodes

```bash
$ kubectl label node worker-0 disk=ssd
node/worker-0 labeled
$ kubectl label node worker-1 disk=ssd
node/worker-1 labeled
$ kubectl label node worker-2 disk=hdd
node/worker-2 labeled
$ kubectl label node worker-3 disk=hdd
node/worker-3 labeled

$ kubectl label node worker-0 compute=gpu
node/worker-0 labeled
$ kubectl label node worker-2 compute=gpu
node/worker-1 labeled
```



## Adding Node Selectors to Pods

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeSelector:
        disk: ssd
      containers:
      - image: nginx
        name: nginx
```



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeSelector:
        disk: ssd
        compute: gpu
      containers:
      - image: nginx
        name: nginx
```



## Manual Scheduling

记住，Kubernetes调度器会寻找带有空nodeName的Pod，而kubelet组件会寻找带有相关节点名称的Pod。

如果你创建了一个Pod，并在其规格中指定了自己的nodeName，调度器将永远不会看到它，而相关的kubelet将立即采用它。其效果是，Pod将被安排在指定的节点上，不需要调度器的帮助。



## DaemonSets

例如，这里有一个 DaemonSet，它将在标有计算=gpu 的节点上部署一个假设的 GPU 守护程序：

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: gpu-daemon
  labels:
    app: gpu-daemon
spec:
  selector:
    matchLabels:
      app: gpu-daemon
  template:
    metadata:
      labels:
        app: gpu-daemon
    spec:
      nodeSelector:
        compute: gpu
      containers:
      - image: gpu-daemon
        name: gpu-daemon
```



## Static Pods

```bash
http://127.0.0.1:8001/api/v1/nodes/docker-desktop/proxy/configz

$ cat <<EOF | sudo tee /etc/kubernetes/manifests/nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
EOF
```



## Resource Requests

### In Imperative Form

```bash
$ kubectl create deployment nginx --image=nginx
deployment.apps/nginx created
$ kubectl set resources deployment nginx \
  --requests=cpu=0.1,memory=1Gi
deployment.extensions/nginx resource requirements updated
```



### In Declarative Form

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources:
          requests:
            cpu: 100m
            memory: 1Gi
```



## Running Multiple Schedulers

You can get an example scheduler on the feloy/scheduler-round-robin GitHub repository.[1](https://www.neat-reader.com/webapp#Fn1)

这个调度器的代码很简单，不能在生产中使用，但展示了一个调度器的生命周期。

- 监听没有nodeName值的Pods
- 选择一个节点

- 将选定的节点绑定到Pod上

- 发送一个事件



现在你可以创建一个deployment，指定这个特定的调度器。

```bash
$ kubectl logs scheduler-round-robin-xxxxxxxxxx-yyyyy -f
found 2 nodes: [worker-0 worker-1]

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      schedulerName: scheduler-round-robin
      containers:
      - image: nginx
        name: nginx
```



## Examine Scheduler Events

```bash
$ kubectl describe pod nginx-xxxxxxxxxx-yyyyy
[...]
Events:
  Type    Reason     Age   From                    Message
  ----    ------     ----  ----                    -------
  
$ kubectl get events | grep Scheduled
0s          Normal    Scheduled                 pod/nginx-554b9c67f9-snpkb      \
          Successfully assigned default/nginx-554b9c67f9-snpkb to worker-0
```



# 10. Discovery and Load Balancing

## Services

```bash
$ kubectl create deployment nginx --image=nginx
deployment.apps/nginx created
$ kubectl expose deployment nginx --port 80
service/webapp exposed
$ kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
webapp       ClusterIP   10.97.68.130   <none>        80/TCP    5s
```



在声明形式中，您可以使用此模板创建服务：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp
spec:
  ports:
  - port: 80
  selector:
    app: nginx
```



现在，我们可以尝试部署另一个 Pod，连接到它，并尝试与这个 nginx Pod 通信：

```bash
$ kubectl run \
  --image=busybox box \
   sh -- -c 'sleep $((10**10))'
pod/box created
$ kubectl exec -it box sh
/ # wget http://webapp -q -O -
<!DOCTYPE html>
<html> [...]
</html>
/ # exit
```



## Service Types

### ClusterIP

```bash
<name>.<namespace>.svc.cluster.local

$ kubectl exec -it box cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
```



### NodePort

如果你想从集群外部访问一个服务，你可以使用NodePort类型。除了创建一个 ClusterIP，这将在集群的每个节点上分配一个端口（默认范围为 30000-32767），该端口将被路由到 ClusterIP。



### LoadBalancer

如果你想从云环境之外访问一个服务，你可以使用LoadBalancer类型。除了创建一个NodePort，它还会创建一个外部负载平衡器（如果你使用一个受管理的Kubernetes集群，如Google GKE、Azure AKS、Amazon EKS等），它通过NodePort路由到ClusterIP。



### ExternalName

这是一种特殊类型的服务，不使用选择器字段，而是使用DNS CNAME记录，将服务重定向到外部DNS名称。



## Ingress

通过一个LoadBalancer服务，你可以访问你的应用程序的一个微服务。如果你有几个应用程序，每个都有几个访问点（至少是前端和API），你将需要保留大量的负载均衡器，这可能是非常昂贵的。

一个Ingress相当于一个Apache或nginx虚拟主机；它允许将几个微服务的访问点复用到一个负载均衡器中。选择是根据请求的主机名和路径进行的。

你必须在你的集群中安装一个Ingress控制器，以便使用Ingress资源。

![img](http://dockone.io/uploads/article/20200215/06b9ed6127d7c96401693beec8ff827f.png)

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.34.1/deploy/static/
provider/baremetal/deploy.yaml

$ kubectl get services -n ingress-nginx
NAME           TYPE      CLUSTER-IP      EXTERNAL-IP    PORT(S)  \
   AGE
ingress-nginx  NodePort  10.100.169.243  <none>     80:32351/TCP,443:31296/TCP\
   9h
$ HTTP_PORT=32351
$ HTTPS_PORT=31296
```



```bash
$ kubectl create deployment webapp --image=httpd
deployment.apps/webapp created
$ kubectl expose deployment webapp --port 80
service/webapp exposed
$ kubectl apply -f - <<EOF
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: webapp-ingress
spec:
  backend:
    serviceName: webapp
    servicePort: 80
EOF
```



```bash
$ kubectl create deployment echo --image=kennship/http-echo
deployment.apps/echo created
$ kubectl expose deployment echo --port 3000
service/echo exposed
$ kubectl delete ingress webapp-ingress
ingress.extensions "webapp-ingress" deleted
$ kubectl apply -f - <<EOF
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: plex-ingress
spec:
  rules:
  - host: webapp.com
    http:
      paths:
      - path: /
        backend:
          serviceName: webapp
          servicePort: 80
  - host: echo.com
    http:
      paths:
      - path: /
        backend:
          serviceName: echo
          servicePort: 3000
EOF
```



### HTTPS and Ingress

```bash
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -out echo-ingress-tls.crt \
    -keyout echo-ingress-tls.key \
    -subj "/CN=echo.com/O=echo-ingress-tls"
[...]
$ kubectl create secret tls echo-ingress-tls \
    --key echo-ingress-tls.key \
    --cert echo-ingress-tls.crt
secret/echo-ingress-tls created
Then add a section to the Ingress resource:
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: plex-ingress
spec:
  tls:
  - hosts:
    - echo.com
    secretName: echo-ingress-tls
  rules:
  - host: webapp.com
    http:
      paths:
      - path: /
        backend:
          serviceName: webapp
          servicePort: 80
  - host: echo.com
    http:
      paths:
      - path: /
        backend:
          serviceName: echo
          servicePort: 3000
```



# 11. Security

## Authentication

### Normal User Authentication

普通用户不受 Kubernetes API 管理。您必须有一个外部系统来管理用户及其凭据。可以通过不同的方法处理普通用户的身份验证：

- Client Certificate
- HTTP Basic Auth
- Bearer Token
- Authentication proxy



### Service Account Authentication

#### Service Account Outside the Cluster



## Authorization

The following modules are available:

- ABAC: Attribute-based access control
- RBAC: Role-based access control
- Webhook: HTTP callback mode
- Node: Special-purpose module for kubelets
- AlwaysDeny: Blocks all requests, for testing purpose
- AlwaysAllow: To completely disable authorization



## Working with Private Docker Registries



## Pre-pulling Images on Nodes



## Giving Credentials to kubelet

# 14. Observability

## Debugging at the Kubernetes Level

```bash

kubectl describe pods nginx-d9bc977d8-h66wf 

kubectl get events -w
```



## Debugging Inside Containers

```bash
kubectl exec nginx-d9bc977d8-h66wf -- ls /usr/share/nginx/html

kubectl exec -it nginx-d9bc977d8-h66wf -- bash
```



## Debugging Services

```bash
kubectl create deployment nginx --image=nginx
kubectl scale deployment/nginx --replicas=2
kubectl expose deployment nginx --port=80

kubectl get endpoints nginx

kubectl describe service nginx

kubectl get pods -l app=nginx -o wide
```



### Logging
```bash
kubectl logs nginx

kubectl logs -l app=nginx --prefix

--follow (-f for short) allows to follow the stream; use Ctrl-C to stop.
--previous (-p for short) allows to view the logs of the previous containers, which is useful when a container crashed and you want to see the error that made it crash.
--container=name (-c name for short) shows the logs of a specific container, and --all-containers shows the logs of all the containers.
--timestamps displays timestamps of logs at the beginning of lines.
```



## Logging at the Node Level

日志存储路径 `/var/log/pods`

### Cluster-Level Logging with a Node Logging Agent

使用syslog, StackDriver, Elastic, fluentd, etc

### Using a Sidecar to Redirect Logs to stdout

如果你的应用程序不能将日志输出到stdout或stderr，而只能输出到文件，你可以运行一个sidecar容器，它将读取这些日志文件，并将其流向自己的stdout。这样一来，日志在节点层面上就可用了，可以用kubectl logs来探索，也可以用logging agent来导出。



## Monitoring

```bash
kubectl top nodes

kubectl top pods
```



### Monitoring with Prometheus

