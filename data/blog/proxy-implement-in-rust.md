---
title: Rust：从零开始的代理编写
date: '2022-08-13'
tags: ['Rust', 'Proxy']
draft: true
summary: 使用Rust编写具有混淆功能的网络代理
images: []
layout: PostSimple
canonicalUrl: https://blog-nat5uk1.vercel.app/blog/proxy-implement-in-rust
authors: ['default']
---

## 代理流程

通常情况下，客户端发起一个网络请求与接收服务器的响应都需要经过防火墙检查，如图：

![base](https://github-page-img.oss-cn-hongkong.aliyuncs.com/img/base network.png)

防火墙采用黑名单模式，只有命中黑名单规则才会被防火墙拦截请求或响应，即不命中黑名单规则就可以使防火墙放行。

> 黑名单规则：可能是 ip 地址或域名，也可能是数据包流量特征

所以需要满足两点即可成功建立代理：

- 客户端和服务器的 ip 和域名都未处于黑名单规则中

- 混淆数据包，使防火墙无法判断流量特征

下图表示代理流程：

![proxy](https://github-page-img.oss-cn-hongkong.aliyuncs.com/img/proxy network.png)

1. 客户端请求转发给代理客户端

2. 代理客户端加密请求，发送给代理服务器
3. 防火墙审查请求通过，放行该请求
4. 代理服务器解密请求，向目标服务器发送真实请求
5. 目标服务器返回响应给代理服务器
6. 代理服务器加密响应，发送给代理客户端
7. 防火墙审查响应通过，放行该响应
8. 代理客户端解密响应，转发给客户端

## 代理服务工作原理

### 代理客户端

本质上是一个运行在客户端的程序，它会创建一个流量转发服务并绑定到监听端口，所有网络请求都会先发送到代理客户端，经过代理客户端对数据包的加密后，封装为 socks5 协议的数据包再发送给代理服务器。

### 代理服务器

本质上是一个运行在服务器的程序，监听所有来自代理客户端的请求，反解 socks5 协议的数据包获得目标服务器以及原数据，再将原数据包发送给目标服务器。

当真正的目标服务器返回了数据时，代理服务器会把返回的数据加密后封装为 socks5 数据包转发给对应的代理客户端，代理客户端收到数据再解密后，转发给本机的应用。

## Socks5 协议

> **SOCKS**是一种网络传输协议，主要用于客户端与外网服务器之间通讯的中间传递。SOCKS 是"SOCKet Secure"的缩写。根据[OSI 模型](https://zh.m.wikipedia.org/wiki/OSI模型)，SOCKS 是[会话层](https://zh.m.wikipedia.org/wiki/会话层)的协议，位于[表示层](https://zh.m.wikipedia.org/wiki/表示层)与[传输层](https://zh.m.wikipedia.org/wiki/传输层)之间。
