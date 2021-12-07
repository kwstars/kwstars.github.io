---
title: "Kubernetes安装Elasticsearch"
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

helm install elasticsearch elastic/elasticsearch -f ./values.yaml --set replicas=1 --set minimumMasterNodes=1
```



### Kibana

```bash
helm install kibana elastic/kibana 
```



### Fluent

```bash
helm repo add fluent https://fluent.github.io/helm-charts

helm show values fluent/fluent-bit

wget https://raw.githubusercontent.com/fluent/helm-charts/main/charts/fluent-bit/values.yaml

helm install fluent-bit fluent/fluent-bit
```





---

[Elastic Stack Kubernetes Helm Charts](https://github.com/elastic/helm-charts)

[Fluent Helm Charts](https://github.com/fluent/helm-charts)

[Fails to send data to ElasticSearch](https://github.com/fluent/fluent-bit/issues/3052)
