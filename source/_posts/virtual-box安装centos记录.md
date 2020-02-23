---
title: virtual box安装centos记录
date: 2020-02-23 12:39:01
tags: ["开发"]
---

不想在本机器上安装数据库等复杂的环境，所以下载了一个virtual box，安装centos，在虚拟环境中安装一系列运行环境，在本机开发。开篇记录一下。
命令汇总：
```bash
# 网络配置
ip a
nmcli d
nmtui
vi /etc/sysconfig/network-scripts/ifcfg-[网卡名]
service network restart
yum install net-tools
```
<!--more-->

+ 安装virtualbox。
```bash
brew cask install virtualbox
```

+ 下载centos
centos有多种版本，minimal dvd stream。这里选择centos 7 minimal版本。作为一个服务器来说，下载minimal自己装软件是最适合的。

+ centos配置网络
    minimal网络是需要自己手工配置的，否则网络是不通的，可以输入```nmcli d```进行查看
    {% asset_img nmcli.png %}

    这时候发现我们使用的虚拟机网卡不通，这时候有两种方式，一个是手动修改网卡的配置文件
    ```bash
    vi /etc/sysconfig/network-scripts/ifcfg-[网卡名]
    ```
    改动前和改动后见下面的截图
    {% asset_img ip_script.png %}

    或者使用带ui的方法，```nmtui```，点击空格为check
    {% asset_img nmtui.png %}
    不论那种方法，最后记得```service network restart```
    然后就发现网络连接上了，使用```ip a```进行状态查看（现在ifconfig还没有安装）
    最后，可以使用yum进行安装和更新了 ```yum install net-tools```


