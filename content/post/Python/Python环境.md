---
title: "Python环境"
date: 2019-12-28T20:37:25+08:00
draft: false
tags: ["Python"]
categories: ["Python"]
---

## 一、pyenv

### 1.1 pyenv 基于 Github Checkout 安装

1. git clone pyenv的代码

```bash
git clone https://github.com/pyenv/pyenv.git ~/.pyenv 
```

2. 定义环境变量`PYENV_ROOT`

```bash
# For bash:
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile

# For Ubuntu Desktop:
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc

# For Zsh:
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc

# For Fish shell:
set -Ux PYENV_ROOT $HOME/.pyenv
set -Ux fish_user_paths $PYENV_ROOT/bin $fish_user_paths
```

3. 将`pyenv init`添加到shell中以启用`shims`和`autocompletion`功能。请确保将`eval "$(pyenv init -)"`放在shell配置文件的末尾，因为它在初始化期间会操纵PATH。

```bash
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bash_profile
```

Zsh note: Modify your `~/.zshrc` file instead of `~/.bash_profile`.

fish note: Use `pyenv init - | source` instead of `eval (pyenv init -)`.

Ubuntu and Fedora note: Modify your `~/.bashrc` file instead of `~/.bash_profile`.

4. 刷新shell

```bash
exec "$SHELL"
```

5. 安装编译环境

[Install Python build dependencies](https://github.com/pyenv/pyenv/wiki#suggested-build-environment) 

6. 将Python版本安装到`$(pyenv root)/version`中。例如，要下载并安装Python 2.7.8，请运行：

```bash
pyenv install 3.9.1
```



### 1.2 Upgrading

```bash
cd $(pyenv root)
git pull
```



### 1.3 Uninstalling pyenv

```bash
# 禁用
pyenv init

# 删除
rm -rf $(pyenv root)
```



### 1.4 解决下载python慢

```bash
mkdir ~/.pyenv/cache
version="3.9.1"; echo $version; wget "https://mirrors.huaweicloud.com/python/$version/Python-$version.tar.xz" -P ~/.pyenv/cache/; pyenv install $version 
```



### 1.5 查看当前安装的版本

```bash
pyenv versions
```



## 二、pyenv-virtualenv

### 2.1 下载插件 

```bash
# 下载
git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv

# 添加到环境变量
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc

# 刷新shell
exec "$SHELL"
```



### 2.2 安装一个版本

```bash
# 指定版本或使用系统的版本
pyenv virtualenv [-f|--force] [VIRTUALENV_OPTIONS] [version] <virtualenv-name>
pyenv virtualenv 3.9.1 mypython
```



### 2.3 激活或停用pyenv virtualenv

```bash
pyenv activate <name>
pyenv deactivate
```



---

[Simple Python Version Management: pyenv](https://github.com/pyenv/pyenv#basic-github-checkout)

[pyenv](https://github.com/pyenv/pyenv)

[pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv)