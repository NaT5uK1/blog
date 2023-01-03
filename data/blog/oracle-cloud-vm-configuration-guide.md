---
title: Oracle Cloud VM Configuration Guide
date: '2022-08-10'
tags: ['Linux', 'VPS', 'Ubuntu']
draft: false
summary: Based on Ubuntu 22.04
images: []
layout: PostSimple
canonicalUrl: https://blog-nat5uk1.vercel.app/blog/react-performance-optimization
authors: ['default']
---

## 创建实例

更改默认镜像为`ubuntu20.04`，用户名默认为`ubuntu`

若创建其他镜像，用户名默认为`opc`

## 添加 SSH 密钥对

### 生成密钥对

1. 在【实例】-【资源】-【控制台连接】中，【创建本地连接】
2. 默认选择【为我生成密钥对】
3. 【保存私有秘钥】【保存公共秘钥】
4. 将私钥和公钥重命名为`oracle` `oracle.pub`（任意合法文件名即可）

### 管理密钥对

1. 将`oracle` `oracle.pub`复制到`~/.ssh`中

2. 为`oracle`添加权限：

   ```shell
   chmod 400 ~/.ssh/oracle
   ```

3. 由于通常情况下本机已有名为`id_rsa` `id_rsa.pub`的密钥对存在，所以需要配置多秘钥管理：

   ```SHELL
   cd ~/.ssh
   vim config
   ```

   `~/.ssh/config` 配置如下：

   ```SHELL
   Host host
       HostName host
       IdentityFile ~/.ssh/oracle

   Host github.com
       HostName github.com
       User git
       IdentityFile ~/.ssh/id_rsa
   ```

   - 上面`host`需替换为自己的服务器 ip 或域名

### 上传公钥

1. 在【实例】-【资源】-【控制台连接】中，【创建本地连接】
2. 选择【上载公共秘钥文件(.pub)】
3. 将`oracle.pub`拖入秘钥框
4. 【创建控制台连接】

## 连接 SSH 服务

```shell
ssh ubuntu@host
```

- 提示将 host 加入`known_hosts`中按 y

## 防止 SSH 会话超时

`/etc/ssh/sshd_config`

```shell
ClientAliveInterval 120  // 超时时间，10s
ClientAliveCountMax 720  // 超时次数，0次
```

如果客户端处于非活动状态 120 秒，这将使服务器向客户端发送一个空数据包，共发送 720 次。 如果服务端向客户端发送消息达到此阈值，SSHD 将断开客户端的连接，所以 timeout interval = ClientAliveInterval \* ClientAliveCountMax

- 重启 sshd.service

```shell
sudo systemctl restart sshd.service
```

## 更新 apt

```shell
sudo apt update
```

## 安装图形界面 xfce4

```
sudo apt install xfce4 xfce4-goodies
```

选择`gdm3`

## 创建 VNC 连接

### 服务端

1. 安装 tightvncserver

```shell
sudo apt install tightvncserver
```

2. 启动 vncserver 并设置连接密码

```shell
vncserver
```

```shell
# Output
You will require a password to access your desktops.

Password:
Verify:

# 此处务必选择n，否则无法执行更改操作
Would you like to enter a view-only password (y/n)? n
xauth:  file /home/ubuntu/.Xauthority does not exist

New 'X' desktop is hostname:1

Creating default startup script /home/ubuntu/.vnc/xstartup
Starting applications specified in /home/ubuntu/.vnc/xstartup
Log file is /home/ubuntu/.vnc/hostname:1.log
```

3. 需要修改 vnc 连接密码时（可选）:

```shell
vncpasswd
```

4. 需要停止 vncserver 以完成后续配置

```shell
vncserver -kill :1
```

```shell
# Output
Killing Xtightvnc process ID 29053
```

5. 备份`~/.vnc/xstartup`

```shell
mv ~/.vnc/xstartup ~/.vnc/xstartup.bak
```

6. 修改`~/.vnc/xstartup`

```shell
vim ~/.vnc/xstartup
```

添加如下内容：

```shell
#!/bin/bash
xrdb $HOME/.Xresources
startxfce4 &
```

7. 为`xstartup`添加权限

```shell
chmod +x ~/.vnc/xstartup
```

8. 重启 vnc 服务

```shell
vncserver -localhost
```

```shell
# output
New 'X' desktop is hostname:1

Starting applications specified in /home/ubuntu/.vnc/xstartup
Log file is /home/ubuntu/.vnc/hostname:1.log
```

### 客户端

1. 创建一个 SSH 隧道用于安全地连接到 vncserver

```shell
ssh -L 59000:localhost:5901 -C -N -l ubuntu host
```

此时保持客户端 shell 会话

2. MacOS 下，`cmd+space`搜索 `Screen sharing.app`并打开，连接 ip 地址为`localhost:59000`，输入 vnc 连接密码，显示出 xfce4 图形界面即表示成功，Windows 下可以使用`PuTTY`

## 将 VNC 配置为系统服务

### 服务端

1. 添加 vncserver 服务配置文件

```shell
sudo vim /etc/systemd/system/vncserver@.service
```

添加以下配置：

```shell
[Unit]
Description=Start TightVNC server at startup
After=syslog.target network.target

[Service]
Type=forking
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu

PIDFile=/home/ubuntu/.vnc/%H:%i.pid
ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1
ExecStart=/usr/bin/vncserver -depth 24 -geometry 1280x800 -localhost :%i
ExecStop=/usr/bin/vncserver -kill :%i

[Install]
WantedBy=multi-user.target
```

2. 重新加载单元文件

```shell
sudo systemctl daemon-reload
```

3. 启动单元文件

```shell
sudo systemctl enable vncserver@1.service
```

4. 如果 vncsever 在运行，停止当前服务

```shell
vncserver -kill :1
```

5. 启动 vncserver 服务

```shell
sudo systemctl start vncserver@1
```

6. 验证服务启动状态

```shell
sudo systemctl status vncserver@1
```

```shell
# output
vncserver@1.service - Start TightVNC server at startup
     Loaded: loaded (/etc/systemd/system/vncserver@.service; enabled; vendor pre>
     Active: active (running) since Sat 2022-04-02 20:21:30 UTC; 4s ago
    Process: 29620 ExecStartPre=/usr/bin/vncserver -kill :1 > /dev/null 2>&1 (co>
    Process: 29624 ExecStart=/usr/bin/vncserver -depth 24 -geometry 1280x800 -lo>
   Main PID: 29632 (Xtightvnc)
```

## 更新

```shell
sudo apt update
```

## Firewall

- 查看防火墙状态

```shell
sudo ufw status

Status: inactive	#表示防火墙未开启
```

- 禁用/启用防火墙

```shell
sudo ufw disable
sudo ufw enable
```

- 添加规则

```shell
sudo ufw allow 22/tcp
sudo ufw deny 8080/tcp
```

## iptables

- 开放所有端口

```shell
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -F
```

## 终端美化

- 安装 Zsh

```shell
sudo apt install zsh
```

- 查询 Zsh 版本

```shell
zsh --version
```

- 安装 oh-my-zsh

```shell
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

Do you want to change your default shell to zsh?[Y/n] y
```

- 安装 zsh 插件

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

- 配置 oh-my-zsh

```shell
sudo vim ~/.zshrc
```

```shell
#~/.zshrc
ZSH_THEME="agnoster"
plugins=(git zsh-syntax-highlighting zsh-autosuggestions)
```

```shell
source ~/.zshrc #应用配置
```

## Web Server

### Nginx

- 安装 Nginx

```shell
sudo apt install nginx
```

- 查询服务状态

```shell
sudo systemctl status nginx
```

- 创建自定义配置

```shell
sudo vim /etc/nginx/conf.d/webserver.conf
```

以`*.conf`结尾才能被主配置文件识别，或者手动修改`/etc/nginx/nginx.conf`中的`http`的`include`字段

```
server {
        listen       80;
        server_name  your_domain;
        location / {
            root   /usr/share/nginx/dist;
            index  index.html index.htm;
        }
}
```

> 记得域名解析

- 上传服务文件

```shell
#本机上传到临时目录
scp -r ./dist/ ubuntu@your_ip:/tmp

#远程主机移动到nginx目录
sudo mv /tmp/dist /usr/share/nginx
```

- 删除默认配置

```shell
sudo rm /etc/nginx/sites-enabled/default
```

- 重新加载

```shell
sudo nginx -s reload
```

- 测试服务

```shell
curl 127.0.0.1
```

## Python3

- 版本查询

```shell
python3 -V
```

- 安装 pip

```shell
sudo apt install python3-pip
```

## OpenSSL

- 更新

```shell
sudo apt upgrade openssl
```

- 解决 error while loading shared libraries: libssl.so.1.1: cannot open shared object file: No such file or directory

```shell
wget https://www.openssl.org/source/openssl-1.1.1o.tar.gz
tar -xzf openssl-1.1.1o.tar.gz
cd openssl-1.1.1o
./config
make
make test
sudo make install

sudo find / -name libssl.so.1.1
sudo ln -s /usr/local/lib/libssl.so.1.1 /usr/lib/libssl.so.1.1
sudo find / -name libcrypto.so.1.1
sudo ln -s /usr/local/lib/libcrypto.so.1.1 /usr/lib/libcrypto.so.1.1
```

## GCC

- 安装

```shell
sudo apt install build-essential
```

## 时区

- 查看时区

```shell
date -R
```

- 更改时区

```shell
sudo cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## V2ray 一键脚本

```shell
bash <(curl -s -L https://raw.githubusercontent.com/233boy/v2ray/master/install.sh)
```
