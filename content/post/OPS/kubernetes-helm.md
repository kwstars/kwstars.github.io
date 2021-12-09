---
title: "helm"
date: 2021-12-06T20:10:39+08:00
draft: false
tags: [""]
categories: ["Kubernetes"]
---

### helm

```bash
# 添加一个chart库
helm repo add elastic https://helm.elastic.co

# 从chart存储库更新信息
helm repo update 

# 查找chart
helm search repo prometheus

# 下载chart
helm pull elastic/elasticsearch

# 查看历史的发布版本
helm history traefik
```





### Elasticsearch

```bash
helm repo add elastic https://helm.elastic.co

# 从存储库下载 chart
helm pull elastic/elasticsearch

helm upgrade --install --namespace monitoring elasticsearch elastic/elasticsearch -f ./values.yaml --set replicas=1 --set minimumMasterNodes=1
```



### Kibana

```bash
helm upgrade --install --namespace monitoring kibana elastic/kibana 
```



### Fluent

```bash
helm repo add fluent https://fluent.github.io/helm-charts

helm show values fluent/fluent-bit

wget https://raw.githubusercontent.com/fluent/helm-charts/main/charts/fluent-bit/values.yaml

helm upgrade --install --namespace monitoring fluent-bit fluent/fluent-bit
```



### kube-prometheus-stack

```bash
helm upgrade --install --namespace monitoring kube-prometheus-stack prometheus-community/kube-prometheus-stack

# 查看grafana密码
kubectl get secret -n monitoring kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```



### Traefik

```bash
helm upgrade --install traefik --namespace=kube-system traefik/traefik -f ./values-prod.yaml
```



创建一个用于 Dashboard 访问的 IngressRoute 资源清单

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: traefik-dashboard
  namespace: kube-system
spec:
  entryPoints:
  - web
  routes:
  - match: Host(`traefik.linux88.com`)  # 指定域名
    kind: Rule
    services:
    - name: api@internal
      kind: TraefikService  # 引用另外的 Traefik Service
EOF
```



创建kibana、prometheus相关的资源清单

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: monitoringressroute
  namespace: monitoring
spec:
  entryPoints:
    - web
  routes:
  - match: Host(`kibana.linux88.com`)
    kind: Rule
    services:
    - name: kibana-kibana
      port: 5601
  - match: Host(`prom.linux88.com`)
    kind: Rule
    services:
    - name: kube-prometheus-stack-prometheus
      port: 9090
  - match: Host(`grafana.linux88.com`)
    kind: Rule
    services:
    - name: kube-prometheus-stack-grafana
      port: 80
  - match: Host(`alert.linux88.com`)
    kind: Rule
    services:
    - name: kube-prometheus-stack-alertmanager
      port: 9093
 EOF
```



---

[Elastic Stack Kubernetes Helm Charts](https://github.com/elastic/helm-charts)

[Fluent Helm Charts](https://github.com/fluent/helm-charts)

[Fails to send data to ElasticSearch](https://github.com/fluent/fluent-bit/issues/3052)

[Traefik](https://www.qikqiak.com/k8strain2/network/ingress/traefik/)

[Install Prometheus and Grafana in your Kubernetes cluster](https://howchoo.com/kubernetes/install-prometheus-and-grafana-in-your-kubernetes-cluster)

[Capture Traefik Metrics for Apps on Kubernetes with Prometheus](https://traefik.io/blog/capture-traefik-metrics-for-apps-on-kubernetes-with-prometheus/)
