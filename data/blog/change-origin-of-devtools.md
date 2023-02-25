---
title: 换源速查手册
date: '2022-08-13'
tags: ['Tools', 'Linux', 'MacOS']
draft: false
summary: 各种开发工具换源合集（持续更新中）
images: []
layout: PostSimple
canonicalUrl: https://blog-nat5uk1.vercel.app/blog/change-origin-of-devtools
authors: ['default']
---

# NPM/PNPM

1. npm 配置

```shell
#官方源
npm config set registry https://registry.npmjs.org

#阿里源
npm config set registry https://registry.npmmirror.com
```

2. 第三方工具**nrm**

```shell
npm i nrm -g
nrm ls

  npm ---------- https://registry.npmjs.org/
  yarn --------- https://registry.yarnpkg.com/
  tencent ------ https://mirrors.cloud.tencent.com/npm/
  cnpm --------- https://r.cnpmjs.org/
  taobao ------- https://registry.npmmirror.com/
  npmMirror ---- https://skimdb.npmjs.com/registry/

nrm use taobao
```

# Homebrew for MacOS

- 添加环境变量

```shell
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"
```

- 更换仓库上游

```shell
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"
for tap in core cask{,-fonts,-drivers,-versions} command-not-found; do
    brew tap --custom-remote --force-auto-update "homebrew/${tap}" "https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-${tap}.git"
done
brew update
```

# Git

```shell
#添加代理
git config --global https.proxy http://127.0.0.1:7890
git config --global https.proxy https://127.0.0.1:7890
git config --global http.proxy socks5://127.0.0.1:7890
git config --global https.proxy socks5://127.0.0.1:7890

#取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

# pip

```shell
#阿里源
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/

#清华源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple/
```

# Rustup

```shell
#清华源
echo 'export RUSTUP_DIST_SERVER=https://mirrors.tuna.tsinghua.edu.cn/rustup' >> ~/.zshrc
```

# Cargo

`~/.cargo/.config`

```shell
[source.crates-io]
replace-with = 'tuna'

[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"
```

# apt for Ubuntu 22.04 LTS

1. 修改`/etc/apt/sources.list`

```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
```

2. **sed**命令

```shell
sudo sed -i "s@http://.*archive.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list
sudo sed -i "s@http://.*security.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list
```
