---
title: 从零开始的Fedora
date: '2023-01-03'
tags: ['Linux', 'Fedora']
draft: false
summary: Based on Fedora Linux 37 (Workstation Edition)
images: []
layout: PostSimple
canonicalUrl: https://blog-nat5uk1.vercel.app/blog/fedora
authors: ['default']
---

## 序言

本文旨在记录基于 Fedora Linux 37 (Workstation Edition)的日常配置与优化，会持续更新与各位看官分享。

## 安装

在一众 Linux 发行版中，Fedora 的安装是那么现代又清新脱俗，官方提供了一键镜像制作应用[Fedora Media Writer](https://getfedora.org/workstation/download/)，它甚至同时提供了 Windows 和 MacOS 双平台版本，你只需准备一个 U 盘即可。如果你需要虚拟机安装或者是其他需求，也可以自行下载 ISO 镜像安装。

## 基本配置

假设你已经拥有一个全新的 Fedora，那就开始吧！

### 修改主机名

设置->关于->设备名称

对，就是这么简单，建议使用纯小写字母和数字组合，这样会同时修改主机名中的三个名称：`static` 、`pretty` 和`transient`

```shell
#查看主机名
hostnamectl --static
hostnamectl --transient
hostnamectl --pretty

#分别修改不同主机名
sudo hostnamectl set-hostname --pretty "Emily's 2nd dev laptop"
sudo hostnamectl set-hostname --static emily-dev-2
```

### 添加源

- RPM Fusion

```shell
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

### 重命名用户目录下的文件夹

很不巧当你安装时选择了中文，用户目录下默认会生成中文的文件夹，在终端中操作会很不便，重命名只需 3 步：

- 重命名用户目录下的所有文件夹

  ```shell
  mv 模板 Templates
  mv 视频 Videos
  mv 图片 Pictures
  mv 文档 Documents
  mv 下载 Downloads
  mv 音乐 Music
  mv 桌面 Desktop
  mv 公共 Public
  ```

- 修改~/.config/user-dirs.dirs

  ```shell
  vi ~/.config/user-dirs.dirs
  ```

  ```shell
  XDG_DESKTOP_DIR="$HOME/Desktop"
  XDG_DOWNLOAD_DIR="$HOME/Downloads"
  XDG_TEMPLATES_DIR="$HOME/Templates"
  XDG_PUBLICSHARE_DIR="$HOME/Public"
  XDG_DOCUMENTS_DIR="$HOME/Documents"
  XDG_MUSIC_DIR="$HOME/Music"
  XDG_PICTURES_DIR="$HOME/Pictures"
  XDG_VIDEOS_DIR="$HOME/Videos"
  ```

- 重启

  ```shell
  reboot
  ```

### 添加窗口最大化最小化按钮

- 安装优化组件

  ```SHELL
  sudo dnf install -y gnome-tweaks
  ```

- super->搜索"优化"->窗口标题栏->标题栏按钮

### 安装 AppImageLauncher

由于很多常用应用不会发布 rpm 包，所以通常使用 AppImage 文件安装应用，AppImageLauncher 是很好的工具。

- 下载[appimagelauncher-\*.rpm](https://github.com/TheAssassin/AppImageLauncher/releases/download/v2.2.0/appimagelauncher-2.2.0-travis995.0f91801.x86_64.rpm)

- 安装

  ```shell
  sudo dnf install -y ~/Downloads/appimagelauncher-2.2.0-travis995.0f91801.x86_64.rpm
  ```

### 安装 Typora

- 下载二进制压缩文件

在[中文官网](https://www.typoraio.cn/#linux)下载二进制压缩文件[Typora-linux-x64.tar.gz](https://download2.typoraio.cn/linux/Typora-linux-x64.tar.gz)

- 解压缩并移动到/opt 目录下

  ```shell
  tar -zxvf Typora-linux-x64.tar.gz
  mv ~/Downloads/bin/Typora-linux-x64 /opt
  ```

- 添加快捷方式到菜单

  ```shell
  vi /usr/share/applications/typora.desktop
  ```

  ```shell
  [Desktop Entry]
  Encoding=UTF-8
  Name=Typora
  Exec=/opt/Typora-linux-x64/Typora %U
  Terminal=false
  Type=Application
  Icon=/opt/Typora-linux-x64/resources/assets/icon/icon_256x256.png
  StartupNotify=true
  Categories=Development
  ```

  `Exec`参数结尾加上`%U`，可以把该应用加入打开方式列表

- 添加到环境变量

  ```shell
  sudo vi ~/.zshrc
  ```

  ```shell
  alias typora="/opt/Typora-linux-x64/Typora"
  ```

### 终端美化

- 安装 zsh

  ```shell
  sudo dnf install -y zsh
  ```

- 安装 oh-my-zsh

  ```shell
  sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
  ```

- 安装 zsh 插件

  ```shell
  git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
  git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
  ```

- 配置 oh-my-zsh

  ```shell
  sudo vi ~/.zshrc
  ```

  ```shell
  plugins=(git zsh-syntax-highlighting zsh-autosuggestions)
  ```

- 安装主题[powerlevel10k](https://github.com/romkatv/powerlevel10k)

  - 下载主题仓库

    ```shell
    git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
    ```

  - 下载字体

    - [MesloLGS NF Regular.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS NF Regular.ttf)

    - [MesloLGS NF Bold.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS NF Bold.ttf)

    - [MesloLGS NF Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS NF Italic.ttf)

    - [MesloLGS NF Bold Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS NF Bold Italic.ttf)

  - 在终端【配置文件首选项】中设置字体为 MesloLGS NF

  - 设置主题

    ```shell
    sudo vi ~/.zshrc
    ```

    ```shell
    ZSH_THEME="powerlevel10k/powerlevel10k"
    ```

  - 根据向导配置完成即可

    ```shell
    p10k configure
    ```

### 安装音视频插件

如果有看直播的需求，默认情况下无法从浏览器打开直播流，需要安装插件：

```SHELL
sudo dnf install gstreamer1-plugins-{bad-\*,good-\*,base} gstreamer1-plugin-openh264 gstreamer1-libav --exclude=gstreamer1-plugins-bad-free-devel

sudo dnf install lame\* --exclude=lame-devel

sudo dnf group upgrade --with-optional Multimedia
```

## 开发配置

### 修改 GitHub host

- 删除 host 文件中的 github 行

  ```shell
  sudo sed -i '/github/d' /etc/hosts
  ```

- 借用网络脚本自动选择连通优良的服务器

  ```shell
  sudo bash -c "curl https://gitlab.com/ineo6/hosts/-/raw/master/next-hosts | grep github >> /etc/hosts"
  ```

### 代理设置

- 系统代理

  - 下载[Clash Verge](https://github.com/zzzgydi/clash-verge/releases/download/v1.2.1/clash-verge_1.2.1_amd64.AppImage)，使用 AppImageLauncher 安装

  - 由于 Clash Verge 对 Fedora 的适配不完善，需要手动修改系统代理：`super + a` -> 设置 ->网络 -> 网络代理

- 终端代理

  - ```shell
    sudo vi ~/.zshrc
    ```

    ```shell
    export http_proxy=http://127.0.0.1:7890
    export https_proxy=http://127.0.0.1:7890
    ```

### Android 开发

#### OpenJDK

- 查找包名

  ```shell
  dnf search openjdk
  ```

- 安装最新版本

  ```shell
  sudo dnf install java-latest-openjdk.x86_64
  ```

- 添加到环境变量

  ```shell
  export JAVA_HOME="/usr/lib/jvm/java-19-openjdk-19.0.1.0.10-2.rolling.fc37.x86_64"
  ```

#### Android SNK and NDK

- [官网](https://developer.android.com/studio#command-tools)查找 commandlinetools 对应版本并下载安装

  ```shell
  cd ~/Downloads

  wget https://dl.google.com/android/repository/commandlinetools-linux-9123335_latest.zip

  mv commandlinetools-linux-9123335_latest.zip cmdline-tools.zip
  unzip cmdline-tools.zip
  cd cmdline-tools
  mkdir latest
  mv bin latest/
  mv lib latest/
  mv NOTICE.txt latest/
  mv source.properties latest/
  cd ..
  mkdir ~/.android # You can use another location for your SDK but I prefer using ~/.android
  mv cmdline-tools ~/.android
  ```

- 添加环境变量

  ```shell
  export ANDROID_HOME="$HOME/.android"
  ```

- 使用 sdkmanager 安装 SDK

  ```shell
  cd ~/.android/cmdline-tools/latest/bin
  ./sdkmanager --list #查询对应工具版本
  ./sdkmanager "platforms;android-33" "platform-tools" "ndk;25.1.8937393" "build-tools;33.0.1"
  ```

#### 环境变量

- 添加到`~/.zshrc`

  ```shell
  export JAVA_HOME="/usr/lib/jvm/java-19-openjdk-19.0.1.0.10-2.rolling.fc37.x86_64"
  export ANDROID_HOME="$HOME/.android"
  export NDK_HOME="$ANDROID_HOME/ndk/25.1.8937393"
  ```
