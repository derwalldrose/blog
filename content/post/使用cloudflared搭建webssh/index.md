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

### 下载

```bash
curl -L 'https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64' -o /usr/local/bin/cloudflared
```
1. 登录认证
```
cloudflared tunnel login
```
2. 创建域名，得到隧道ID
```
cloudflared tunnel route dns ssh awssgssh.543083.xyz
```
3. 修改配置文件，添加 SSH 代理
```
cat > /root/.cloudflared/config.yml << EOF
tunnel: e7425241-3827-4e3a-830e-dde066ae6915
credentials-file: /root/.cloudflared/e7425241-3827-4e3a-830e-dde066ae6915.json
protocol: h2mux
ingress:
  # 第五个，反代 SSH 服务
  - hostname: awssgssh.543083.xyz
    service: ssh://localhost:1522
  # 最后记得添加一个默认 404
  - service: http_status:404
EOF
```
4. 验证
```
cloudflared tunnel ingress validate
cloudflared tunnel ingress rule https://awssgssh.543083.xyz
```
5. 启动测试
```
cloudflared tunnel --config ~/.cloudflared/config.yml run e7425241-3827-4e3a-830e-dde066ae6915
```
6. 安装系统服务
```
cloudflared --config /root/.cloudflared/config.yml service install
```
7. 配置页面 application
参考：使用 Cloudflare Tunnels 通过 Web SSH 访问服务器
参考：https://blog.hellowood.dev/posts/%E4%BD%BF%E7%94%A8cloudflare-tunnels%E9%80%9A%E8%BF%87web-ssh%E8%AE%BF%E9%97%AE%E6%9C%8D%E5%8A%A1%E5%99%A8/

> Photo by [Pawel Czerwinski](https://unsplash.com/@pawel_czerwinski) on [Unsplash](https://unsplash.com/)