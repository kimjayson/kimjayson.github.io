---
layout: post
title: Ubuntu 使用笔记
categories: Linux
description: 使用 Ubuntu 遇到一些问题，笔记在此备忘。
keywords: Linux, Ubuntu
---

使用 Ubuntu 过程中遇到的问题及解决方案。

## 使用 git pull 遇到问题

提示

```
Agent admitted failure to sign using the key.
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

解决方法：

```sh
ssh-add ~/.ssh/id_rsa
```

## 安装和配置 JDK

在 Terminal 运行：

```sh
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer

sudo vim /etc/profile

export JAVA_HOME=/usr/lib/jvm/java-8-oracle
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```

## 安装 SVN 图形前端 RabbitVCS

在 Terminal 运行：

```sh
sudo add-apt-repository ppa:rabbitvcs/ppa
sudo apt-get update
sudo apt-get install rabbitvcs-nautilus3 rabbitvcs-thunar rabbitvcs-gedit rabbitvcs-cli
```

## 创建 eclipse 快捷方式

/usr/share/applications 里新建 Eclipse.desktop，填如下内容：

```
[Desktop Entry]
Name=Eclipse
Comment=Launch Eclipse
Exec=/home/mzlogin/android/eclipse/eclipse
Icon=/home/mzlogin/android/eclipse/icon.xpm
StartupNotify=true
Terminal=false
Type=Application
```

## VirtualBox 里 Ubuntu 分辨率无法调整

Ubuntu 14.04 LTS 在 VirtualBox 中刚安装完时，分辨率只有 640\*480 一种选项，无法调整。

解决方法：

1. 打开 xdiagnose

   ![](/images/posts/linux/xdiagnose.png)

2. 勾选 Debug 下的所有选项

   ![](/images/posts/linux/xdiagnose-2.png)

3. 重启

4. 安装增强功能

   ![](/images/posts/linux/install-additions.png)

   然后：

   ```
   cd /media/<username>/VBOXADDITIONS_X.X.XX_XXXXX
   sudo ./VBoxLinuxAdditions.run
   ```

   （注意把 username 替换成自己的，VBOXADDITIONS 后面的 X 换成具体版本号）

## 与 Win7 共享 SSH key

如下步骤适用于在 Ubuntu 上使用从 Win7 拷贝的 SSH key，反之应该也一样能用。

创建 ~/.ssh 目录，确认其权限为 0700，将 Windows %userprofile%/.ssh 下的 id\_rsa 和 id\_rsa.pub 文件拷贝到 ~/.ssh 目录下，权限分别改为 0600 和 0644。

```sh
mzlogin@ubuntu:~$ ll ~/.ssh
total 20
drwx------  2 mzlogin mzlogin 4096 Jun 22 01:03 ./
drwxr-xr-x 20 mzlogin mzlogin 4096 Jun 22 01:02 ../
-rw-------  1 mzlogin mzlogin 1679 Jun 21 05:17 id_rsa
-rw-r--r--  1 mzlogin mzlogin  399 Jun 21 05:17 id_rsa.pub
```

然后

```sh
ssh-add ~/.ssh/id_rsa
```
