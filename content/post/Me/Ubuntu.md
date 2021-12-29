---
title: "Ubuntu"
date: 2020-08-10T15:38:51+08:00
draft: true
tags: ["Me"]
categories: ["OPS"]
---

### 1. 安装火焰截图

[flameshot install](https://github.com/lupoDharkael/flameshot#installation)

```bash
sudo apt install flameshot -y

frlameshot config

# 程序快速启动gui
flameshot gui
```

![image-20200801221817231](https://i.imgur.com/xGtvB6Y.png)



### 2. 新建桌面图标

```bash
tee ~/.local/share/applications/tim.desktop << EOF
[Desktop Entry]
Encoding=UTF-8
Name=Tim
Exec=/home/kira/下载/TIM-x86_64.AppImage
Icon=/usr/local/src/tim.png
Terminal=false
Type=Application
Categories=Internet;
EOF
```



### 3. 安装zsh

```bash
git clone https://github.com/ohmyzsh/ohmyzsh.git ~/.oh-my-zsh

sudo apt install zsh -y

chsh -s $(which zsh)

ln -s /opt/docker/zsh/.zshrc ~/.zshrc
ln -s /opt/docker/zsh/robbyrussell.zsh-theme ~/.oh-my-zsh/themes/robbyrussell.zsh-theme

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

sudo apt-get install autojump

```



### 4. SSH 和 Git 配置

```bash
ln -s /opt/docker/ssh/config ~/.ssh/config 
ln -s /opt/docker/git/.gitconfig ~/.gitconfig
```



### 5. 安装google拼音输入法

 [https://github.com/junxnone/Linux/issues/57](https://github.com/junxnone/Linux/issues/57)



### 6. 取色软件
[Gpick](http://www.gpick.org/)



### 7. 修改面板的上下左右
[如何在xfce4中启用自然滚动？](https://www.it-swarm.dev/zh/xubuntu/%E5%A6%82%E4%BD%95%E5%9C%A8xfce4%E4%B8%AD%E5%90%AF%E7%94%A8%E8%87%AA%E7%84%B6%E6%BB%9A%E5%8A%A8%EF%BC%9F/961376604/)
```bash
tee /etc/X11/Xsession.d/80synaptics << EOF
synclient VertScrollDelta=-58
synclient HorizScrollDelta=-58
EOF
```



### 8.GIF工具

```bash
sudo add-apt-repository ppa:peek-developers/stable
sudo apt update
sudo apt install peek
```



### 9. image Writer

```bash
balenaEtcher	
```



### 10. 安装chrome

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb

sudo dpkg -i google-chrome-stable_current_amd64.deb
```



### 11. Top bar

```bash
sudo apt install indicator-multiload gnome-system-monitor -y
```



### 12. 快捷键占用

[Navigation Back and Forward not working at Intellij IDEA](https://stackoverflow.com/questions/33801165/navigation-back-and-forward-not-working-at-intellij-idea/33809139)

```bash
gsettings set org.gnome.desktop.wm.keybindings switch-to-workspace-left "['']"
 gsettings set org.gnome.desktop.wm.keybindings switch-to-workspace-right "['']"
```


### 13. [gnome-terminal的alt+1不能使用](https://askubuntu.com/questions/1147053/alt-key-stopped-working-in-gnome-terminal-after-upgrade-to-ubuntu-19-04)

 配置文件首选项->启用助记键

![image-20210406164758047](https://i.imgur.com/dAUuPtz.png)

### 14. onedrive

```bash
# 安装
sudo add-apt-repository ppa:yann1ck/onedrive
sudo apt-get update
sudo apt install onedrive

# 修改配置文件
cat .config/onedrive/config                                                                                                  
sync_dir="/data/onedrive"
# skip_file = "~*|.~*|*.tmp"
# monitor_interval = "300"
skip_dir = "图片|应用|文档|桌面|裁剪图"
# log_dir = "/var/log/onedrive/"

# 启动
systemctl --user enable onedrive
systemctl --user start onedrive

# 同步
onedrive

# 错误查看日志
journalctl --user-unit=onedrive -f
```



15. ### TigerVNC

[How to Install TigerVNC Server on Ubuntu 20.04](https://atetux.com/how-to-install-tigervnc-server-on-ubuntu-20-04)

```bash
sudo apt install tigervnc-standalone-server -y
sudo adduser atetux
vncpasswd
vncserver -localhost no
```

