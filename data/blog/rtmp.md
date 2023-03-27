---
title: 搭建个人rtmp推流服务
date: '2023-03-27'
tags: ['nginx', 'rtmp', 'ubuntu']
draft: false
summary:
images: []
layout: PostSimple
canonicalUrl: https://blog-nat5uk1.vercel.app/blog/rtmp
authors: ['default']
---

## 安装

使用`nginx`作为服务器，借助`libnginx-mod-rtmp`模块完成 rtmp 推流

```shell
sudo apt update
sudo apt install nginx libnginx-mod-rtmp
```

## 配置 Nginx

```shell
sudo vi /etc/nginx/nginx.conf
```

```nginx
rtmp {
        server {
                listen 1935;
                chunk_size 4096;
                allow publish all;

                application live {
                        live on;
                        record off;
                }
        }
}
```

## 防火墙端口开放

使用 VPS 服务商提供的防火墙，去对应页面开放`1935`端口

默认`ufw`是关闭状态

```shell
#ubuntu开放1935端口
sudo ufw allow 1935/tcp
```

## OBS 推流设置

服务器：rtmp://x.x.x.x/live

推流码：obs_stream

理论上可以自定义任何推流码，只需客户端更改 path 即可

## 播放

使用任何支持 rtmp 协议的播放器均可

流地址：rtmp://x.x.x.x/live/obs_stream
