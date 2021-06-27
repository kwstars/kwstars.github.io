---
title: "PicGo-Core 和 Typora"
date: 2020-11-10T15:26:31+08:00
draft: false
tags: ["PicGo-Core", "Typora"]
categories: ["OPS"]
---



系统：Ubuntu 20.04 LTS

## 一、PicGo-Core
### 1.1 安装npm
```bash
sudo apt install npm -y
```

### 1.2 切换npm的源
```bash
sudo npm install -g cnpm --registry=https://registry.npm.taobao.org
```

### 1.3 安装PicGo-Croe
```bash
sudo cnpm install picgo -g
```

### 1.4 生成配置文件
[Imgur图床的配置方法](https://picgo.github.io/PicGo-Doc/zh/guide/config.html#imgur%E5%9B%BE%E5%BA%8A)

**PicGo-Core通过命令行生成配置**
```bash
picgo set uploader
? Choose a(n) uploader imgur
? clientId: xxxxxxxxxx
? proxy: http://127.0.0.1:8118
[PicGo SUCCESS]: Configure config successfully!
$ cat ~/.picgo/config.json 
{
  "picBed": {
    "uploader": "smms",
    "current": "smms",
    "imgur": {
      "clientId": "xxxxxxxxxx",
      "proxy": "http://127.0.0.1:8118"
    }
  },
  "picgoPlugins": {}
}
```

**由于生成的config.json有问题需要修改成**
```bash
$ tee ~/.picgo/config.json << EOF
{
  "picBed": {
    "uploader": "imgur",
    "current": "imgur",
    "imgur": {
      "clientId": "xxxxxxxxxx",
      "proxy": "http://127.0.0.1:8118"
    }
  },
  "picgoPlugins": {}
}
EOF
```

## 二、Typora
### 2.1 修改Typora的配置
![image-20200801210445878](https://i.imgur.com/YLhtd9j.png)

### 2.2 测试可以正常上传
![image-20200801210516934](https://i.imgur.com/PjUbnOG.png)

 **由于访问没有走代理，所有图片上传以后国内是无法查看的**

---
[淘宝 NPM 镜像](https://developer.aliyun.com/mirror/npm)
[PicGo-Core 配置文件](https://picgo.github.io/PicGo-Core-Doc/zh/guide/config.html)