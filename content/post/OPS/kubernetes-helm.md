---
title: "通过helm安装chart"
date: 2021-12-06T20:10:39+08:00
draft: false
tags: [""]
categories: ["Kubernetes"]
---



## Helm

### Elasticsearch

```bash
helm repo add elastic https://helm.elastic.co

# 从存储库下载 chart
helm pull elastic/elasticsearch

helm install --namespace monitoring elasticsearch elastic/elasticsearch -f ./values.yaml --set replicas=1 --set minimumMasterNodes=1
```



### Kibana

```bash
helm install --namespace monitoring kibana elastic/kibana 
```



### Fluent

```bash
helm repo add fluent https://fluent.github.io/helm-charts

helm show values fluent/fluent-bit

wget https://raw.githubusercontent.com/fluent/helm-charts/main/charts/fluent-bit/values.yaml

helm install --namespace monitoring fluent-bit fluent/fluent-bit
```



### kube-prometheus-stack

```bash
helm install kube-prometheus-stack --namespace monitoring prometheus-community/kube-prometheus-stack

# 查看grafana密码
kubectl get secret  prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```



### traefik

```bash
helm install traefik traefik/traefik
```



---

[Elastic Stack Kubernetes Helm Charts](https://github.com/elastic/helm-charts)

[Fluent Helm Charts](https://github.com/fluent/helm-charts)

[Fails to send data to ElasticSearch](https://github.com/fluent/fluent-bit/issues/3052)
