---
title: 如何使用nginx stream模块自建proxyip
description: 如何使用nginx stream模块自建proxyip
slug: 如何使用nginx stream模块自建proxyip
date: 2024-08-06 00:00:00+0000
image: cover.jpg
categories:
    - proxyip
tags:
    - proxyip
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

## 转发cf域名
https://blog.tanglu.me/cloudflare_proxy_ip/
- 如何避免被扫
- 演示alpine上
1、安装nginx和stream模块
```shell-session
apk add nginx nginx-mod-stream
```
2、添加配置cf.conf
```shell-session
sg-ac:/etc/nginx/stream.d# ls -lrt
total 4
-rw-r--r--    1 root     root           849 Sep  5 01:56 cf.conf
sg-ac:/etc/nginx/stream.d# cat cf.conf 
    # 定义允许的上游服务器
    upstream allowed_backend {
        server 104.18.32.47:443;
    }

    # 定义一个拒绝连接的空上游（可以是无效的 IP 或者本地地址）
    upstream reject_backend {
        server 127.0.0.1:1;  # 无效的服务器地址，用于拒绝连接
    }
    # 使用 map 指令根据 SNI 值选择上游
    map $ssl_preread_server_name $backend {
        # 允许的域名
        worker.543083.xyz allowed_backend;
        blog.543083.xyz allowed_backend;

        # 其他所有 SNI 值都将被拒绝
        default reject_backend;
    }
    # 定义服务器块，监听443端口
    server {
        listen 443;

        # 启用 SSL 预读取功能以获取 SNI 信息
        ssl_preread on;

        # 根据 SNI 选择上游服务器
        proxy_pass $backend;
    }
```
其中allowed_backend是任意的cloudflare cdn节点，可以nslookup chatgpt.com得到一个ip，map中定义允许的反代域名，可以是自己的开了小黄云的cf域名，其他域名禁止访问，这样防止被扫出来当节点使用