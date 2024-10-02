---
title: 开设lxc玩具机
description: 开设lxc玩具机
slug: 开设lxc玩具机
date: 2024-09-20 00:00:00+0000
image: cover.jpg
categories:
    - 代理ip,lxc
tags:
    - 代理ip,lxc
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

### 在vps上开设虚拟化lxc玩具

参考https://www.spiritlhl.net/guide/incus/incus_install.html
```
curl -L https://raw.githubusercontent.com/oneclickvirt/incus/main/scripts/incus_install.sh -o incus_install.sh && chmod +x incus_install.sh && bash incus_install.sh

开机
./buildone.sh test 1 256 2 20001 20002 20025 500 500 N debian11
```

pve适合kvm虚拟化的vps，incus适合Lxc


> Photo by [Pawel Czerwinski](https://unsplash.com/@pawel_czerwinski) on [Unsplash](https://unsplash.com/)