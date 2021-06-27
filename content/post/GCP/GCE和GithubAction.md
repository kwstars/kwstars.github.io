---
title: "GCE 和 Github Action"
date: 2021-03-21T10:34:27+08:00
draft: false
tags: ["GCE"]
categories: ["GCP"]
---

通过 Github Action 部署应用, 此方法可能有些权限问题。

## 一、进入GCP的 IAM & Admin

### 1.1 在`Service Accounts`中创建一个用于github action的帐号

![image-20210325105046682](https://i.imgur.com/ChfuppX.png)

### 1.2 将github帐号添加到默认的`xxxxxxxxxx-compute@developer.gserviceaccount.com`帐号中

![image-20210325110039269](https://i.imgur.com/Xzimap8.png)

### 1.3 在`IAM`中将github帐号添加到`Compute Instance Admin (v1)`

![image-20210325104123904](https://i.imgur.com/cFA9cGQ.png)

### 1.4 创建key

![image-20210325110459156](https://i.imgur.com/6Cs4qIe.png)



## 二、Github Action配置

### 2.1 编写 Github Action

进入Github账户 -> Action -> New workflow 

```yam
name: GCE Deploy

on:
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest    
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@master
        with:
          project_id: ${{ secrets.GCP_PROJECT_ID }}
          service_account_key: ${{ secrets.GCP_SA_KEY }}
          export_default_credentials: true

#       - name: Use gcloud CLI
#         run: |
#           gcloud info
#           gcloud compute instances list

      - name: Copy files to GCE
        run: gcloud compute scp --port=${{ secrets.PORT }} --zone=us-central1-f --recurse SOURCE_FILES INSTANCE_NAME:TARGET_DIR_NAME
```



### 2.2 添加 secrets

Setting -> Secrets 

![image-20210325111135794](https://i.imgur.com/mx8o1BM.png)

### 2.3 手动run

![image-20210325111318854](https://i.imgur.com/8godmO0.png)

---

[(gcloud.beta.compute.scp) Could not add SSH key to instance metadata permissions](https://stackoverflow.com/questions/56049804/gcloud-beta-compute-scp-could-not-add-ssh-key-to-instance-metadata-permissions)

[setup-gcloud GitHub Action](https://github.com/google-github-actions/setup-gcloud)

[gcloud compute scp](https://cloud.google.com/sdk/gcloud/reference/compute/scp)