---
title: 星辰scaleway1C1G1G-alpine
description: 星辰scaleway1C1G1G-alpine
slug: 星辰scaleway1C1G1G-alpine
date: 2024-10-20 00:00:00+0000
image: https://img.543083.xyz/api/cfile/AgACAgUAAxkDAAIBymcZ2gEc5061x-nlL9wtL0TbaIsaAAIlwDEbXNPRVIvESQrMDUDZAQADAgADeAADNgQ
categories:
    - 星辰scaleway1C1G1G-alpine
tags:
    - 星辰scaleway1C1G1G-alpine
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

## <span style="color:red; font-weight:bold;">页面上的所有操作都会收费，如开关机</span>

如果不想用了，命令行栅机：
```
#stop
scw instance server stop 88e2ffa0-3923-42a1-9b3e-bf51774261ea zone=pl-waw-2
#delete
scw instance server delete 88e2ffa0-3923-42a1-9b3e-bf51774261ea zone=pl-waw-2 with-volumes=all
```

## 设置 SSH Key/创建实例
在面板右上角点击头像，选择 SSH Keys，创建新的 SSH Key 以便后续连接服务器
在面板右上角点击 CLI，根据需要的区域，输入以下命令：
```
法国：scw instance server create zone=fr-par-1 root-volume=local:10GB name=fr type=STARDUST1-S ipv6=true ip=none
 
荷兰：scw instance server create zone=nl-ams-1 root-volume=local:10GB name=nl type=STARDUST1-S ipv6=true ip=none
 
波兰：scw instance server create zone=pl-waw-2 root-volume=local:10GB name=pl type=STARDUST1-S ipv6=true ip=none
```
返回服务器信息表示命令执行成功。如果返回各种乱七八糟参数，表示命令输入有误，需重新执行。
- 添加 IPv6/防火墙规则
- 左侧 Instances，点击 Attach flexible IP，创建免费 ipv6
- 左侧 Instances，Security group 选项卡，进入，Rules 选项卡，右侧编辑，添加所有协议的入栈出栈 Accept 规则
- 分离/创建/删除硬盘
- 面板关机：左侧 Instances，进入实例管理面板，右上角开关，关机
分离 10GB 硬盘：实例管理面板，Attached volumes 选项卡，在硬盘右侧三个点选 Detach 解绑
创建 1GB 硬盘：点击 Create Volume 创建 Local Storage，大小 1GB
删除 10GB 硬盘：左侧 Instances，Volumes 选项卡，旧 10GB 硬盘右侧三个点选 Delete 删除
救援模式/连接 SSH
在实例管理面板的 Advanced settings 选项卡中，选中 Use rescue image，保存
面板关机：左侧 Instances，进入实例管理面板，右上角开关，开机
重启后耐心等待 10 分钟，通过创建的 SSH Key 连接实例，执行命令：
```
parted /dev/vda mklabel gpt
wget -qO- https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-virt-3.20.1-x86_64.iso | dd of=/dev/vda
```
## 改硬盘启动
- 面板关机：左侧 Instances，进入实例管理面板，右上角开关，关机
- 更改硬盘启动：实例管理面板，Advanced settings 选项卡，选中 Use local boot，保存，Boot volume 选择 1GB 硬盘，保存，开机
## 进入 Console
- 安装系统前置操作
```
mkdir /media/setup
cp -a /media/vda/* /media/setup
mkdir /lib/setup
cp -a /.modloop/* /lib/setup
/etc/init.d/modloop stop
umount /dev/vda
mv /media/setup/* /media/vda/
mv /lib/setup/* /.modloop/
```
- 安装 Alpine
```
setup-alpine
```
- 按照以下步骤回答安装过程中问题：
输入主机名
done
输入 y，按 i 进入编辑模式，输入以下网络配置（IPv6 地址和网关可以从实例管理面板中获取）：
```
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet dhcp
iface eth0 inet6 static
  address < 你的 IPv6>
  netmask 64
  gateway < 你的 IPv6 网关>
```
- 新建 root 密码
- 输入时区，巴黎 Europe/Paris，阿姆斯特丹 Europe/Amsterdam，华沙 Europe/Warsaw
```
none
skip
no
openssh
yes
none
vda
sys
```
- 报错后 vi /etc/resolv.conf 输入以下内容并保存
```
nameserver 2001:4860:4860::6464
```
- 输入命令启用官方源
```
echo "http://dl-cdn.alpinelinux.org/alpine/latest-stable/main" >> /etc/apk/repositories
echo "http://dl-cdn.alpinelinux.org/alpine/latest-stable/community" >> /etc/apk/repositories
echo "#http://dl-cdn.alpinelinux.org/alpine/edge/main" >> /etc/apk/repositories
echo "#http://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories
echo "#http://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories
```
- 安装启动项
```
apk update
apk add dosfstools
apk add grub-efi
关闭 swap
setup-disk -s 0 
第一问，vda
第二问，sys
第三问，y
```
- reboot 重启
安装常用项
```
apk add sudo curl wget bash tar unzip
```
- 修改 ssh 端口
```
vi /etc/ssh/sshd_config
 
//找到 #Port 22 行，去掉 #，改成想要的端口号，保存
rc-service sshd restart
```
- 开启 warp和bbr，注：fr 给美国 IP，nl 给法国 IP，pl 给波兰 IP
```
wget -N https://gitlab.com/fscarmen/warp/-/raw/main/menu.sh && bash menu.sh [option] [lisence/url/token]


echo "tcp_bbr" >> /etc/modules-load.d/bbr.conf
modprobe tcp_bbr
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```
- reboot 重启
- 验证
lsmod | grep bbr
 
//出现以下内容表示成功：tcp_bbr

> Photo by [Pawel Czerwinski](https://unsplash.com/@pawel_czerwinski) on [Unsplash](https://unsplash.com/)