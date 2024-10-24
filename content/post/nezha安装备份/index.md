---
title: nezha安装备份
description: nezha安装备份
slug: nezha安装备份
date: 2024-10-20 00:00:00+0000
image: https://img.543083.xyz/api/cfile/AgACAgUAAxkDAAIBymcZ2gEc5061x-nlL9wtL0TbaIsaAAIlwDEbXNPRVIvESQrMDUDZAQADAgADeAADNgQ
categories:
    - nezha
tags:
    - nezha
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---


## 1、dashboard备份

备份/opt/nezha下的的目录和nezha.sh
新机器上执行nezha.sh，启动dashboard

## 2、dashboard取消8008开放端口,使用1panel转发
```
cat docker-compose.yaml 
version: "3.3"

services:
  dashboard:
    image: ghcr.io/naiba/nezha-dashboard
    restart: always
    volumes:
      - ./data:/dashboard/data
      - ./static-custom/static:/dashboard/resource/static/custom:ro
      - ./theme-custom/template:/dashboard/resource/template/theme-custom:ro
      - ./dashboard-custom/template:/dashboard/resource/template/dashboard-custom:ro
    ports:
      - 127.0.0.1:8008:80
      - 5555:5555

```