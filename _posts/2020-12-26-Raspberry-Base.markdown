---
layout: post
title:  "树莓派搭建集群前期准备"
date:   2020-12-24 13:28:10 +0800
categories: Raspberry
---

# 安装Ubuntu

* 下载镜像
	- [官网镜像](https://cdimage.ubuntu.com/releases/20.10/release/)
	- [清华镜像](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-cdimage/ubuntu/releases/20.10/release/)

* 烧录镜像
```shell
df -h
diskutil list
diskutil unmountDisk /dev/disk5
sudo dd bs=4m if=ubuntu-20.10-preinstalled-server-arm64+raspi.img of=/dev/disk3
diskutil unmountDisk /dev/disk5
```

* 启动树莓派
	- 插入sd卡
	- 连接键盘 屏幕
	- 开机


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
apt list –installed
```


# 安装必备软件

#### Docker
- 安装
```shell
sudo apt install docker.io
docker -v
```

# DB层image构建

* Dockerfile准备
```dockerfile
FROM centos
COPY mongodb-org-4.0.repo /etc/yum.repos.d/mongodb-org-4.0.repo
RUN yum update -y
RUN yum -y install java-1.8.0-openjdk.x86_64
RUN yum install -y mysql.x86_64
RUN yum install -y mysql-server.x86_64
RUN yum install -y python2.x86_64
RUN yum install -y redis.x86_64
COPY privileges.sql /root/privileges.sql
COPY start_up.sh /root/start_up.sh
# 解决没有systemctl的问题:https://github.com/gdraheim/docker-systemctl-replacement
COPY systemctl.py /usr/bin/systemctl
RUN yum -y install mongodb-org
RUN chmod 777 /root/start_up.sh
CMD /root/start_up.sh
EXPOSE 3306 33060 6379 27017
```

* 其他文件
1. mongodb-org-4.0.repo
```
[mngodb-org]
name=MongoDB Repository
baseurl=http://mirrors.aliyun.com/mongodb/yum/redhat/7Server/mongodb-org/4.0/x86_64/
gpgcheck=0
enabled=1
```
2. privileges.sql 
```sql
use mysql;
select host, user from user;
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
update user set host='%' where user='root';
flush privileges;
grant all privileges on *.* to root@'%';
flush privileges;
```
3. start_up.sh
```shell
echo '开始启动mysql....'
systemctl start mysqld.service
echo `systemctl status mysqld.service`
echo '启动mysql完毕....'
echo '开始修改密码....'
mysql < /root/privileges.sql
echo '修改密码完毕....'
echo '开始配置redis....'
sed -i 's/bind 127.0.0.1/#bind 127.0.0.1/' /etc/redis.conf
sed -i 's/protected-mode yes/protected-mode no/' /etc/redis.conf
echo '配置redis完毕....'
echo '开始启动redis....'
systemctl start redis
echo '启动redis完毕....'
echo '开始配置mongoDB....'
sed -i 's/bindIp: 127.0.0.1/bindIp: 0.0.0.0/' /etc/mongod.conf
echo '配置mongoDB完毕....'
echo '开始启动mongod....'
systemctl start mongod.service
echo '启动mongod完毕....'
echo '无聊ping'
ping 127.0.0.1
```

# 登陆指令合集

```shell
mysql -h 127.0.0.1 -P 3307 -u root -p123456
redis-cli -h 127.0.0.1 -p 3312
mongo 127.0.0.1:3313
```

# DOCKER常用指令记录
```shell
// image build
docker image build -t charlie_java_env:0.0.1 .
// image 启动(端口映射, 让外部可访问到docker;)
docker container run -p 3310:3306 -p 3311:33060 -p 3312:6379 -dit --privileged=true charlie_java_env:0.0.1
// 进入容器
docker exec -it <containerid> /bin/bash 
// 容器端口

docker port <containerid>
```

# SHELL常用指令记录
```shell
uname -a               # 查看内核/操作系统/CPU信息
lsb_release -a         # 查看操作系统版本 (适用于所有的linux，包括Redhat、SuSE、Debian等发行版，但是在debian下要安装lsb)   
cat /proc/cpuinfo      # 查看CPU信息
hostname               # 查看计算机名
lspci -tv              # 列出所有PCI设备
lsusb -tv              # 列出所有USB设备
lsmod                  # 列出加载的内核模块
env                    # 查看环境变量
free -m                # 查看内存使用量和交换区使用量
df -h                  # 查看各分区使用情况
du -sh <目录名>        # 查看指定目录的大小
grep MemTotal /proc/meminfo   # 查看内存总量
grep MemFree /proc/meminfo    # 查看空闲内存量
uptime                 # 查看系统运行时间、用户数、负载
cat /proc/loadavg      # 查看系统负载
mount | column -t      # 查看挂接的分区状态
fdisk -l               # 查看所有分区
swapon -s              # 查看所有交换分区
hdparm -i /dev/hda     # 查看磁盘参数(仅适用于IDE设备)
dmesg | grep IDE       # 查看启动时IDE设备检测状况
ifconfig               # 查看所有网络接口的属性
iptables -L            # 查看防火墙设置
route -n               # 查看路由表
netstat -lntp          # 查看所有监听端口
netstat -antp          # 查看所有已经建立的连接
netstat -s             # 查看网络统计信息
ps -ef                 # 查看所有进程
top                    # 实时显示进程状态
w                      # 查看活动用户
id <用户名>            # 查看指定用户信息
last                   # 查看用户登录日志
cut -d: -f1 /etc/passwd   # 查看系统所有用户
cut -d: -f1 /etc/group    # 查看系统所有组
crontab -l             # 查看当前用户的计划任务
chkconfig --list       # 列出所有系统服务
chkconfig --list | grep on    # 列出所有启动的系统服务
```

# Jekyll指令合集

```shell
bundle exec jekyll serve -H 0.0.0.0
```