---
title: git服务器搭建
categories:
  - 工具
date: 2024-05-06 21:40:24
tags:
---

## 1. 安装git
**安装相关包**
```
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-devel
```
**安装git**
```
yum install git
```

## 2. 创建用户组和用户
```
groupadd git
useradd git -g git
```

## 3. 导入公钥
将需要访问git的用户公钥导入到/home/git/.ssh/authorized_keys文件中
```
cd /home/git/
mkdir .ssh
chmod 755.ssh
touch .ssh/authorized_keys
chmod 644.ssh/authorized_keys
```

## 4. 初始化git仓库
选择一个目录作为git的仓库，如/home/repos 创建目录并分配权限，初始化一个名为test的空仓库，修改test仓库的权限。
```
cd /home
mkdir repos
chown git:git repos/
cd repos
git init --bare test.git
chown -R git:git test.git
```

## 5. 本地clone
```
git clone git@<your server address>:/home/repos/test.git/
```