---
title: "Rsync"
date: 2019-10-03T10:20:50+08:00
draft: false
tags: ["Rsync"]
categories: ["OPS"]
---

##  一、基本配置

### Server端配置
1. 安装rsync
```bash
yum install -y rsync
```

2. 配置rsync
```bash
# cat /etc/rsyncd.conf
# /etc/rsyncd: configuration file for rsync daemon mode

# See rsyncd.conf man page for more options.

# configuration example:

# uid = nobody
# gid = nobody
# use chroot = yes
# max connections = 4
# pid file = /var/run/rsyncd.pid
# exclude = lost+found/
# transfer logging = yes
# timeout = 900
# ignore nonreadable = yes
# dont compress   = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2

# [ftp]
#        path = /home/ftp
#        comment = ftp export area

[test1]
uid = user1
gid = user1
path = /home/user1/dir
auth users = user1
secrets file = /etc/rsyncd.secrets
list = no
read only = no
write only = no

[test2]
uid = user2
gid = user2
path = /home/user2/dir
auth users = user2
secrets file = /etc/rsyncd.secrets
list = no
read only = no
write only = no
```

3. 为rsync的守护进程添加帐号和密码
```bash
# cat /etc/rsyncd.secrets
user1:password1
user2:password2
```
4. 修改密码文件为root只读
```bash
 chmod 600 /etc/rsyncd.secrets
```

5. 启动rsync的守护进程
```bash
systemctl start rsyncd
systemctl enable rsyncd
systemctl status rsyncd
```

### Client端配置
1. 给客户端添加验证文件
```bash
sudo mkdir /etc/rsyncd
sudo vim /etc/rsyncd/rsyncd.pass
password1
sudo chmod 600 /etc/rsyncd/rsyncd.pass
```
2. 同步数据
```bash
rsync -zvP --password-file=/etc/rsyncd/rsyncd.pass 192.168.1.101::test2 /opt/rsync
```

## 二、使用场景
### 2.1 将服务器上的数据同步到本地

```bash
#!/bin/bash
rsync -avP --exclude='log' --exclude='config' --log-file="transmission1.log" --password-file=/etc/rsyncd/rsyncd.pass 192.168.1.38::test1 /opt/test1/game
rsync -avP --exclude='log' --exclude='config' --log-file="transmission2.log" --password-file=/etc/rsyncd/rsyncd.pass 192.168.1.38::test2 /opt/test2/game

# 将地址修改为本地的地址
cd /opt/lihui1/sgame
sed -i 's#192.168.1.100#127.0.0.1#g' `grep -lr "192.168.1.100" $PWD`

cd /opt/lihui2/sgame
sed -i 's#192.168.1.100#127.0.0.1#g' `grep -lr "192.168.1.100" $PWD`
```



### 2.2 排除目录或文件的方式

[Rsync 秒杀一切备份工具，你能手动屏蔽某些目录吗？](https://mp.weixin.qq.com/s?__biz=MzA4Nzg5Nzc5OA==&mid=2651691358&idx=1&sn=ce0ec367430e93ca82eb593a116c20ce&chksm=8bcb5af7bcbcd3e153831cd9f64d9c63599201fab8ee781e17df8387b3275f8c12126bac864e&mpshare=1&scene=1&srcid=1112oGrt18ZgHQsYPzq3azTG&sharer_sharetime=1605145652974&sharer_shareid=5159710fd0af9e38b08ec96a390754c8&exportkey=AeC8DQfm9i4MqiTlu7yyLdQ%3D&pass_ticket=mAPyJ%2BpMZXyKYmeOwYgYj4ii%2BQnhgppHQDRS%2BcHtrO%2F96Jqtwc6CsS5ebBz53t0q&wx_header=0#rd)



## 三、在容器中运行rsync

[Rsync Daemon with support for multiple RSYNC Modules](https://github.com/jazzdd86/rsyncd)

### 3.1 安装docker和docker-compose

[Docker和docker-compose的安装与配置]({{< ref "post/OPS/Docker和docker-compose的安装与配置.md" >}})



### 3.2 编辑docker-compose.yaml文件

```yaml
version: '3.0'

services:
  rsyncd:
    image: kwstars/rsyncd
    container_name: rsyncd
    ports:
      - "873:873"   
    restart: always
    environment:
      # (optional) - use only if non-default values should be used
      RSYNC_TIMEOUT: 300
      RSYNC_PORT: 873
      RSYNC_MAX_CONNECTIONS: 10
      # (optional) - global username and password
      RSYNC_USERNAME: globaluser
      RSYNC_PASSWORD: password123
      # ID_NAME is the only required parameter for each rsync module
      TEST01_NAME: ngame01
      #TEST01_ALLOW: 192.168.1.0/24
      TEST01_READ_ONLY: "false"
      TEST01_VOLUME: /test01
      #TEST01_USERNAME: test01
      #TEST01_PASSWORD: test01
      NGAME01_UID: 1001 # 指定uid创建用户: useradd -M -s /sbin/nologin -u 1001 test01
      NGAME01_GID: 1001

      TEST02_NAME: ngame02
      TEST02_READ_ONLY: "false"
      TEST02_VOLUME: /test02
      TEST02_UID: 1002 
      TEST02_GID: 1002

    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /data/web/test01/go:/test01:rw
      - /data/web/test02/go:/test02:rw

networks:
  default:
    name: mynetwork
    driver: bridge
    ipam:
      config:
        - subnet: 172.31.255.0/24
          gateway: 172.31.255.1
```



### 3.3 运行容器

进入到docker-compose.yaml所在的目录

```bash
$ sudo docker-compose up -d
```



查容器

```bash
$ sudo docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
9e56a80c0ff0        kwstars/rsyncd      "/server.sh"             2 hours ago         Up 2 hours          0.0.0.0:873->873/tcp       rsyncd
```



### 3.4 检查配置

进入容器查看配置

```bash
$ sudo docker exec -it rsyncd sh
/ # cat /etc/rsyncd.conf 
# GLOBAL OPTIONS
uid = root
gid = root
#incoming chmod = Du=wrx,Dgo=rx,Fu=wr,Fgo=r
use chroot = no
pid file = /var/run/rsyncd.pid
log file = /dev/stdout
timeout = 300
max connections = 10
port = 873

# MODULE OPTIONS
[test02]
    uid = 1002
    gid = 1002
    read only = false
    path = /test02
    comment = test02
    lock file = /var/lock/rsyncd
    list = no
    ignore errors = no
    ignore nonreadable = yes
    transfer logging = yes
    log format = %t: host %h (%a) %o %f (%l bytes). Total %b bytes.
    refuse options = checksum dry-run
    dont compress = *.gz *.tgz *.zip *.z *.rpm *.deb *.iso *.bz2 *.tbz
    exclude =  *.!sync *.swp
    secrets file = /etc/rsyncd.secrets
    auth users = globaluser

# MODULE OPTIONS
[test01]
    uid = 1001
    gid = 1001
    read only = false
    path = /test01
    comment = test01
    lock file = /var/lock/rsyncd
    list = no
    ignore errors = no
    ignore nonreadable = yes
    transfer logging = yes
    log format = %t: host %h (%a) %o %f (%l bytes). Total %b bytes.
    refuse options = checksum dry-run
    dont compress = *.gz *.tgz *.zip *.z *.rpm *.deb *.iso *.bz2 *.tbz
    exclude =  *.!sync *.swp
    secrets file = /etc/rsyncd.secrets
    auth users = globaluser
/ # cat /etc/rsyncd.
rsyncd.conf     rsyncd.secrets
/ # cat /etc/rsyncd.secrets 
globaluser:password123
globaluser:password123
```



### 3.5 在客户端测试

```bash
echo "password123" >> pwsswd
chmod 600 passwd
rsync -avP --password-file=passwd  testdir/ globaluser@192.168.1.100::test01
```



---

