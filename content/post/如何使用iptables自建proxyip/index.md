---
title: 如何使用iptables自建proxyip
description: 如何使用iptables自建proxyip
slug: 使用iptables自建proxyip
date: 2024-08-06 00:00:00+0000
image: cover.jpg
categories:
    - proxyip
tags:
    - proxyip
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

## 1. 安装 iptables： ##
 - Ubuntu / Debian
```shell-session
apt update && apt install sudo -y && sudo apt install iptables iptables-persistent -y
```
 - CentOS
```shell-session
yum update -y && yum install sudo -y && sudo yum install iptables iptables-services  -y
```
 - Alpine
```shell-session
apk update && apk add sudo bash iptables
```
如果提示失败可以尝试添加DNS后再重新运行以上命令

```shell-session
echo -e "nameserver 1.1.1.1\nnameserver 8.8.8.8\nnameserver 2606:4700:4700::1111\nnameserver 2001:4860:4860::8888" | tee /etc/resolv.conf 
```
## 2. 修改 sysctl 配置： ##
启用 IPv4 和 IPv6 的流量转发。
```shell-session
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding=1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
接下来不会提供 IPv4 的部署方法，如果你有能力，可以自己修改命令去实现，如果你不会，说明你还不明白这个操作以为着什么，所以请不要自建 IPv4 的 ProxyIP！

## 3. 设置 ip6tables 转发 IPv6 流量规则： ##
先获取转发目标IP地址，推荐自行 解析 Cloudflare CDN 地址获得 CFIP，执行nslookup chat.openai.com，例如返回内容如下
```shell-session
root:~# nslookup chat.openai.com
Server:         1.1.1.1
Address:        1.1.1.1#53

Non-authoritative answer:
chat.openai.com canonical name = chat.openai.com.cdn.cloudflare.net.
Name:   chat.openai.com.cdn.cloudflare.net
Address: 104.18.37.228
Name:   chat.openai.com.cdn.cloudflare.net
Address: 172.64.150.28
Name:   chat.openai.com.cdn.cloudflare.net
Address: 2606:4700:4400::6812:25e4
Name:   chat.openai.com.cdn.cloudflare.net
Address: 2606:4700:4400::ac40:961c
如果你的 VPS 只有 IPv6 出口，可以执行如下命令

sudo ip6tables -t nat -A PREROUTING -p tcp --dport 443 -j DNAT --to-destination [2606:4700:4400::ac40:961c]:443
sudo ip6tables -t nat -A POSTROUTING -p tcp -d 2606:4700:4400::ac40:961c --dport 443 -j MASQUERADE
```
通过以上步骤，你应该能够成功地在只有 IPv6 的 443 端口的流量转发到指定的 IPv6 地址 2606:4700:4400::ac40:961c。

你可以使用以下命令来查看 ip6tables 中的当前规则，确认规则已正确添加：

```shell-session
sudo ip6tables -t nat -L -v -n
```
## 4. 保存 ip6tables 规则： ##
要确保在重启后规则依然有效，你需要将规则保存到文件中，并设置系统在启动时加载这些规则。

保存规则：

```shell-session
sudo ip6tables-save > /etc/iptables/rules.v6
```
## 5. 重启系统或者手动加载规则 ##
你可以重启系统来验证规则是否被正确加载，或者手动加载保存的规则：

```shell-session
sudo ip6tables-restore < /etc/iptables/rules.v6
```
## 6. 后悔药 ##
清除现有的转发规则

```shell-session
sudo ip6tables -t nat -F
```

> Photo by [Pawel Czerwinski](https://unsplash.com/@pawel_czerwinski) on [Unsplash](https://unsplash.com/)