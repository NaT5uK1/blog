---
title: shadowsocks-rust搭建与部署
date: '2022-08-14'
tags: ['Proxy', 'Shadowsocks']
draft: false
summary: ssserver安装配置与自启动服务配置 on Ubuntu Minimal 22.04
images: []
layout: PostSimple
canonicalUrl: https://blog-nat5uk1.vercel.app/blog/shadowsocks-rust-configure
authors: ['default']
---

## 优化网络性能

- 增加打开的文件描述符的限制以处理数千级并发的 TCP 连接

```shell
vim /etc/security/limits.conf
```

```
* soft nofile 51200
* hard nofile 51200

# for server running in root:
root soft nofile 51200
root hard nofile 51200
```

```shell
ulimit -n 51200
```

- 调整内核参数

shadowsocks 调整参数的原则是：

1. 尽快重用端口和连接
2. 尽可能大地扩大队列和缓冲区
3. 选择大延迟和高吞吐量的 TCP 拥塞算法

```shell
vim /etc/sysctl.conf
```

```
fs.file-max = 51200

net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 4096

net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_mem = 25600 51200 102400
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_congestion_control = hybla
```

重新加载配置

```shell
sysctl -p
```

## 安装

```shell
wget https://github.com/shadowsocks/shadowsocks-rust/releases/download/v1.14.3/shadowsocks-v1.14.3.x86_64-unknown-linux-gnu.tar.xz
tar -xf shadowsocks-v1.14.3.x86_64-unknown-linux-gnu.tar.xz
cp ssserver /usr/local/bin
chmod a+x /usr/local/bin/ssserver
```

## shadowsocks server 配置

```shell
mkdir /etc/shadowsocks
vim /etc/shadowsocks/config.json
```

```
{
    "server": "::",
    "server_port":8888,
    "method":"chacha20-ietf-poly1305",
    "password":"pw"
}
```

- 测试服务可用性：

```
ssserver -c /etc/shadowsocks/config.json
```

## 配置服务单元文件

- 创建单元文件

```shell
vim /etc/systemd/system/shadowsocks-server.service
```

```
[Unit]
Description=shadowsocks-rust service
After=network.target

[Service]
Type=simple
ExecStart=ssserver -c /etc/shadowsocks/config.json
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=ssserver
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

- 启用服务

```shell
systemctl enable /etc/systemd/system/shadowsocks-server.service
```

- 使系统重新加载单元文件

```shell
sysctl --system
```

- 开启服务

```shell
systemctl start shadowsocks-server
```

- 查看服务状态

```shell
systemctl status shadowsocks-server
```

- 关闭服务

```shell
systemctl stop shadowsocks-server
```
