---
layout: post
title:  "树莓派搭建集群前期准备"
date:   2020-12-24 13:28:10 +0800
categories: Raspberry
---

# 安装Ubuntu

* 下载镜像
	* [官网镜像](https://cdimage.ubuntu.com/releases/20.10/release/)
	* [清华镜像](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-cdimage/ubuntu/releases/20.10/release/)

* 烧录镜像
```shell
df -h
diskutil list
diskutil unmountDisk /dev/disk5
sudo dd bs=4m if=ubuntu-20.10-preinstalled-server-arm64+raspi.img of=/dev/disk3
diskutil unmountDisk /dev/disk5
```

* 启动树莓派


# 连接wifi
- 连接
```shell
ip a
cd /etc/netplan/
vi 50-cloud-init.yaml
netplan apply
```
- 保持wifi连接
```shell
# 原因：网络静默一段时间后自动断开
ping 192.168.1.1 > /dev/null &
```

# apt-get源调整
```shell
vi /etc/apt/sources.list
# 批量替换 :%s/ports.ubuntu.com/mirrors.aliyun.com/g
sudo apt-get update
```


# 安装必备软件

## 