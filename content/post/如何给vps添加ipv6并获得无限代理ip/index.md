---
title: 如何给vps添加ipv6并获得无限代理ip
description: 如何给vps添加ipv6并获得无限代理ip
slug: 如何给vps添加ipv6并获得无限代理ip
date: 2024-09-20 00:00:00+0000
image: cover.jpg
categories:
    - 代理ip
tags:
    - 代理ip
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

### Hurricane Electric (简称：HE) 隧道服务简介

Hurricane Electric (简称：HE) 是一家位于美国的全球互联网服务提供商。该公司运营了世界上以对等数最大 IPv6 网络，同时也提供免费的 IPv6 隧道服务，其隧道服务可以追溯到 2001 年。虽然经过多年的发展 IPv6 已经相当普及，但依然还是有部分 VPS 商家由于各种各样的原因没有给 VPS 标配 IPv6 地址，有的需要加钱、有的甚至不给加钱。如果此时有访问 IPv6 网络的需求，就可以接入 HE Tun­nel Bro­ker 提供的 IPv6 隧道免费给 IPv4 VPS 主机添加公网 IPv6 地址来获得 IPv6 网络的访问能力。

## HE IPv6 隧道的使用场景与局限性

### 使用场景：

- 给 VPS 服务器、家用设备接入 IPv6 公网访问能力
- 使 IPv4 VPS 可访问 IPv6 网络，比如给 IPv6 Only VPS 做 SSH 跳板
- HE IPv6 地址可绕过部分网站的 IP 访问限制：
  - 解锁 Netflix 非自制剧 (已失效)
  - 解锁 ChatGPT 访问限制 (只验证可访问，未验证是否稳定不封号)

### 局限性：

- 无法使用 HE 提供的 IPv6 公网 IP 接入 Cloudflare CDN 。
- 滥用严重，IP 评分机构标记为高风险。使用 HE IPv6 访问网站会遇到以下问题：
  - 部分网站为了防止资源滥用，可能会限制访问、频繁弹人机验证、无法注册。
  - 银行、购物等安全要求高的网站可能会判定为欺诈行为、导致砍单，严重会导致封号。

## HE Tunnel Broker 设置教程

### 创建 Tunnel Broker IPv6 隧道

1. 注册 Tunnel Broker 账号。
2. 点击左侧的 `Create Regular Tunnel`（创建常规隧道）。
3. 输入 VPS 的公网 IP 地址。
4. 根据 VPS 的位置选择一个合适的节点。
5. 页面拉到最下方，点击 `Create Tunnel`（创建隧道）。

在 `Tunnel Details` 页面可以看到创建的 IPv6 隧道的详细信息，其中 `Client IPv6 Address` 是申请到的公网 IPv6 地址。

### 获取配置示例

在 `Tunnel Details` 页面有个 `Example Configuration` 选项卡，在这里你可以选择合适的配置示例。就比如这里有 De­bian/​Ubuntu 的 `interfaces` 配置文件示例：

```bash
# /etc/network/interfaces.d/he-ipv6

auto he-ipv6
iface he-ipv6 inet6 v4tunnel
        address 2001:xxx:xxxx:xxxx::2
        netmask 64
        endpoint 216.66.84.46
        local 233.xxx.xxx.233
        ttl 255
        gateway 2001:xxx:xxxx:xxxx::1
EOF
```

TIPS: 如果是 NAT VPS 或 VPC 内网方案则需要将`local`字段后面的公网 IP 替换为内网 IP 。获取命令：

```bash
ip route get 8.8.8.8 | grep -oP 'src \K\S+'
```

### 启用 IPv6 隧道

1. 安装网络工具包

```bash
sudo apt update
sudo apt install net-tools iproute2 -y
```

2. 启动 he-ipv6 网络接口

```bash
sudo ifup he-ipv6
```

TIPS：若提示 `ifup: unknown interface he-ipv6` ，则添加 `source /etc/network/interfaces.d/*`到`/etc/network/interfaces`文件：

```bash
echo 'source /etc/network/interfaces.d/*' >>/etc/network/interfaces
```

3. 启用后执行 `ifconfig` 命令，这时应该有一个 he-ipv6 接口，类似下面这样：

```bash
he-ipv6: flags=209<UP,POINTOPOINT,RUNNING,NOARP>  mtu 1480
          inet6 2001:xxx:xxxx:xxxx::2  prefixlen 64  scopeid 0x0<global>
          inet6 fe80::xxxx:xxxx  prefixlen 64  scopeid 0x20<link>
          sit  txqueuelen 1000  (IPv6-in-IPv4)
          RX packets 11605  bytes 3127821 (3.1 MB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 13811  bytes 2403522 (2.4 MB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

若没有生效可以尝试重启网络：

```bash
sudo systemctl restart networking
```

### 其它操作

#### 检测 IPv6 支持

执行 `ping6 google.com` 命令，能 ping 通说明 VPS 已经支持 IPv6 网络了。

#### NAT VPS 的额外设置

IPv4 NAT VPS 除了前面提到的替换 IP 操作以外，可能还需要一些额外的设置，否则可能还是无法访问 IPv6 网络。

1. 配置防火墙允许 41 端口入站：

```bash
ufw allow 41
```

2. 添加相关的路由规则：

```bash
route -A inet6 add ::/0 dev he-ipv6
```

#### DNS 设置优化

TIPS: 以下设置仅针对操作系统 DNS ，代理软件应单独设置内置的 DNS 和 IP 分流策略，具体参考相关软件文档中的 DNS 和路由部分。

##### 选择合适的 DNS 解析服务器

因为通过 IPv6 隧道去请求可能会拖慢 DNS 解析速度，所以一般不建议在系统中使用 IPv6 地址的 DNS。

编辑 `/etc/resolv.conf` 文件，更改 DNS 解析服务器为支持查询 AAAA 记录的 DNS 服务器，比如 Google Public DNS：

```bash
nameserver 8.8.8.8
nameserver 8.8.4.4
```

##### 优先使用 IPv4 网络

默认情况下 IPv6 网络优先级会高于 IPv4 ，为了防止 IPv6 隧道拖慢 VPS 的正常网速，可以设置优先使用 IPv4 网络。同时也能减轻了对 HE Tun­nel Bro­ker 节点的网络压力，合理使用宝贵的免费资源。

编辑 `/etc/gai.conf` 文件，在末尾添加下面这行配置：

```bash
precedence  ::ffff:0:0/96   100
```

一键添加命令如下：

```bash
echo 'precedence  ::ffff:0:0/96   100' | sudo tee -a /etc/gai.conf
```

完事执行 `curl ip.p3terx.com` 命令，显示 VPS 的 IPv4 地址则代表成功。

### 删除 HE IPv6 隧道

不想用了，或者想使用其它方式访问 IPv6 网络时，记得先删除。

1. 停用隧道：

```bash
sudo ifdown he-ipv6
```

2. 删除 he-ipv6 网络接口配置文件（若没有删除重启后会自动启用）：

```bash
sudo rm -f /etc/network/interfaces.d/he-ipv6
```

### 无限ipv6代理 Http Proxy IPv6 Pool

Make every request from a separate IPv6 address.

https://zu1k.com/posts/tutorials/http-proxy-ipv6-pool/

## Tutorial

Assuming you already have an entire IPv6 subnet routed to your server, for me I purchased [Vultr's server](https://www.vultr.com/?ref=9039594-8H) to get one.

Get your IPv6 subnet prefix and interface name, for me is `2001:19f0:6001:48e4::/64` and `enp1s0`.

```sh
$ ip a
......
2: enp1s0: <BROADCAST,MULTICAST,ALLMULTI,UP,LOWER_UP> mtu 1500 qdisc fq state UP group default qlen 1000
    ......
    inet6 2001:19f0:6001:48e4:5400:3ff:fefa:a71d/64 scope global dynamic mngtmpaddr 
       valid_lft 2591171sec preferred_lft 603971sec
    ......
```

Add route via default internet interface

```sh
ip route add local 2001:19f0:6001:48e4::/64 dev enp1s0
```

Open `ip_nonlocal_bind` for binding any IP address:

```sh
sysctl net.ipv6.ip_nonlocal_bind=1
```

For IPv6 NDP, install `ndppd`:

```sh
apt install ndppd
```

then edit `/etc/ndppd.conf`:


```conf
route-ttl 30000

proxy <INTERFACE-NAME> {
    router no
    timeout 500
    ttl 30000

    rule <IP6_SUBNET> {
        static
    }
}
```
(edit the file to match your configuration)

Restart the service:
```sh
service ndppd restart
```


Now you can test by using `curl`:

```sh
$ curl --interface 2001:19f0:6001:48e4::1 ipv6.ip.sb
2001:19f0:6001:48e4::1

$ curl --interface 2001:19f0:6001:48e4::2 ipv6.ip.sb
2001:19f0:6001:48e4::2
```

Great!

Finally, use the http proxy provided by this project:

```sh
$ while true; do curl -x http://127.0.0.1:51080 ipv6.ip.sb; done
2001:19f0:6001:48e4:971e:f12c:e2e7:d92a
2001:19f0:6001:48e4:6d1c:90fe:ee79:1123
2001:19f0:6001:48e4:f7b9:b506:99d7:1be9
2001:19f0:6001:48e4:a06a:393b:e82f:bffc
2001:19f0:6001:48e4:245f:8272:2dfb:72ce
2001:19f0:6001:48e4:df9e:422c:f804:94f7
2001:19f0:6001:48e4:dd48:6ba2:ff76:f1af
2001:19f0:6001:48e4:1306:4a84:570c:f829
2001:19f0:6001:48e4:6f3:4eb:c958:ddfa
2001:19f0:6001:48e4:aa26:3bf9:6598:9e82
2001:19f0:6001:48e4:be6b:6a62:f8f7:a14d
2001:19f0:6001:48e4:b598:409d:b946:17c
```


> Photo by [Pawel Czerwinski](https://unsplash.com/@pawel_czerwinski) on [Unsplash](https://unsplash.com/)