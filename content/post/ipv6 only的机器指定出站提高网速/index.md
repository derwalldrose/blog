---
title: ipv6 only的机器指定出站提高网速
description: ipv6 only的机器指定出站提高网速
slug: ipv6 only的机器指定出站提高网速
date: 2024-09-20 00:00:00+0000
image: cover.jpg
categories:
    - 代理ip,ipv6 only
tags:
    - 代理ip,ipv6 only
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

### ipv6 only

ipv6 only的机器没有v4入口，但是一般自带v4出口，带宽很小，只有v6带宽高
系统默认的解析顺序是ipv6，对于支持ipv6的网站来说速度没有影响，但是对于只有ipv4的网站，走v4速度会很慢

这时我们可以安装warp代理v4流量，然后从warp v6出口，这时速度会有大幅提升
```
wget -N https://gitlab.com/fscarmen/warp/-/raw/main/menu.sh && bash menu.sh [option] [lisence/url/token]
```
选择warp 4
然后修改里面的engpoint为官方默认的
```
root@wawo:~# cat /etc/wireguard/warp.conf
[Interface]
PrivateKey = nAT67OpRgxE4wTBNwAdkz8ydslrp7MCguEb/TlzxD+A=
Address = 172.16.0.2/32
Address = 2606:4700:110:855f:fc54:6680:6544:e68c/128
DNS = 1.1.1.1,8.8.8.8,8.8.4.4,2606:4700:4700::1111,2001:4860:4860::8888,2001:4860:4860::8844
MTU = 1420
PostUp = ip -4 rule add from 10.10.57.99 lookup main
PostDown = ip -4 rule delete from 10.10.57.99 lookup main
PostUp = ip -6 rule add from 2400:c620:2c:a291::1 lookup main
PostDown = ip -6 rule delete from 2400:c620:2c:a291::1 lookup main

#Reserved = [226,135,179]
#Table = off
#PostUp = /etc/wireguard/NonGlobalUp.sh
#PostDown = /etc/wireguard/NonGlobalDown.sh

[Peer]
PublicKey = bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=
AllowedIPs = 0.0.0.0/0
#AllowedIPs = ::/0
Endpoint = engage.cloudflareclient.com:2408
PersistentKeepalive = 30
```


> Photo by [Pawel Czerwinski](https://unsplash.com/@pawel_czerwinski) on [Unsplash](https://unsplash.com/)