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

# 添加用户
adduser dev0
passwd dev0
chmod -v u+w /etc/sudoers
vi /etc/sudoers
    #最后一行添加
dev0 ALL=(ALL) PASSWD:ALL
chmod -v u+w /etc/sudoers
```
<!--more-->

+ 安装virtualbox。
```bash
brew cask install virtualbox
```

+ 下载centos
centos有多种版本，minimal dvd stream。这里选择centos 7 minimal版本。作为一个服务器来说，下载minimal自己装软件是最适合的。

+ centos配置网络
注意使用桥接模式，nat模式下的虚拟机连接不上，跟virtualbox的网络设计有关？ // todo
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

+ 为系统添加用户
来源：https://blog.csdn.net/bug4pie/article/details/79761443
```bash
adduser dev0
passwd dev0
chmod -v u+w /etc/sudoers
vi /etc/sudoers
#最后一行添加
dev0 ALL=(ALL) PASSWD:ALL
chmod -v u+w /etc/sudoers
#查看系统全部用户
cat /etc/passwd
```

+ [安装docker](https://docs.docker.com/install/linux/docker-ce/centos/#prerequisites)
```bash
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
# 进行测试
sudo docker run hello-world
```

+ 安装开发用的容器
[安装mysql docker](https://hub.docker.com/_/mysql?tab=description)
```bash
docker pull mysql
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag -p 3306:3306
docker exec -it some-mysql bash
```

+ 防火墙配置
（centos7 默认使用firewalld，而非iptables）
```bash
sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
sudo firewall-cmd --zone=public --add-masquerade --permanent #打开ip伪装 待详细了解 todo
sudo firewall-cmd --reload
```

+ 在宿主机telnet 虚拟机3306端口，通。

至此在mac上安装虚拟机，在虚拟机中安装docker，并使用docker下载开发环境，在mac上开发完整走通。

我觉得再简单一点应该是，mac上安装docker，配置dockfile，直接启动docker作为部署环境比较简单。后续再了解。//todo