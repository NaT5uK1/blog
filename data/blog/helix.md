---
title: Helix基础安装与使用
date: '2022-08-15'
tags: ['Editor', 'Tools', 'Terminal']
draft: false
summary: A post-modern modal text editor built by rust
images: []
layout: PostSimple
canonicalUrl: https://blog-nat5uk1.vercel.app/blog/helix
authors: ['default']
---

## 安装

```shell
brew install helix
```

## 基础操作

```shell
#跟着教程先试试吧
hx --tutor
```

- 命令(Command)模式：按`:`进入

  - 写入(保存)：`:` `w` `Enter`

  - 退出：`:` `q` `Enter`

- 插入(Insert)模式：

  - 在所选字符前插入：`i`
  - 在所选字符后追加：`a`
  - 在行首插入：`I`
  - 在行尾追加：`A`
  - 修改选中部分：`c`
  - 在下一行插入新行：`o`
  - 在上一行插入新行：`O`

- 普通(Normal)模式：按`ecs`进入

  - 光标移动：`h` `j` `k` `l`

  - 删除字符：按`d`即可删除光标所在位置的单字符或删除已选中的部分

  - 单词跳转并选择：

    - `w`：光标跳至下一个单词词首，并选中从光标到下一个单词词首的部分

    - `e`：光标跳至当前词尾，并选中光标到词尾的部分

    - `b`：光标跳至当前词首，并选中光标到词首的部分

    - > Tips：在包含空格的句子中，使用`e` `b`连按或者`b` `e`连按来选择不带空格的单词，通常配合`c`修改很有用

    - `x`：选中光标所在行，光标跳至行尾；多次使用同时选择所有选中行

    - `;`：取消选择

  - 撤销：`u`

  - 重做：`U`

  - 多光标：

    - `C`：在下一行当前光标对应位置新增一个光标，多次使用叠加
    - `,`：保留一个光标位置

  - 查找并选择：

    - 范围精确查找：

      1. 先选择查找范围

      2. `s`进入 select 框

      3. 输入查找内容，支持正则表达式，`Enter`查找结果被选中，非正则忽略大小写

    - 文件内精确查找：

      - `/`：从光标向前查找整个文件，进入 search 框，`Enter`确认搜索内容
      - `shift-/`即`?`：从光标向后查找整个文件，进入 search 框，`Enter`确认搜索内容
      - `n`：跳转到下一个匹配项
      - `N`：跳转到上一个匹配项

    - 快捷查找：
      - `f<ch>`：光标向后跳转至下一个所输入字符，并选择包括该字符的区间
      - `t<ch>`：光标向后跳转至下一个所输入字符，并选择不包括该字符的区间
      - `F<ch>`：光标向前跳转至上一个所输入字符，并选择包括该字符的区间
      - `T<ch>`：光标向前跳转至上一个所输入字符，并选择不包括该字符的区间

  - 复制：`y`，`space-y`复制到系统剪切板

  - 粘贴：`p`，`space-p`粘贴到系统剪切板

  - 大小写转换：

    - `： 选中部分变为小写

    - `~`：切换选中部分的大小写

    - > Tips：对 macos 下标准键盘`alt`键的识别有问题，不能单纯使用 alt-`变为大写，可以先变小写再切换一次

  - 宏指令：

    - `Q`：记录指令/退出记录
    - `q`：在光标位置重复宏记录指令

  - 保存位置跳转(jumplist)：

    - `ctrl-s`：记录当前位置

    - `ctrl-i`： 跳转到 jumplist 的下一个位置

    - `ctrl-o`： 跳转到 jumplist 的上一个位置

    - > Tips：与记录位置无关，只与记录顺序有关，内部维护了一个线性表来存储位置

  - 合并行：
    - `J`：将下一行合并到光标所在行行尾
    - 配合`x`选中多行，按`J`合并选中行为一行
  - 水平缩进：
    - `<`：向左缩进光标所在行，位移量 4 空格
    - `>`：向右缩进光标所在行，位移量 1 空格
  - 跳转：
    - `gg`：跳转首行
    - `ge`：跳转末行
    - `gh`：跳转行首
    - `gl`：跳转行尾
    - `<num>gg`跳转到 num 行
    - `gd`：跳转到当前函数/类/接口...实现
    - `gn`：跳转到下一个打开的 buffer
    - `gp`：跳转到上一个打开的 buffer
  - 多功能键`space`：
    - `space-f`：打开文件选择器
    - `space-F`：在当前目录打开文件选择器
    - `space-a`：代码提示
    - `space-d`：Debug
    - `space-w`：进入多窗口选项
      - `space-ww`：跳转到下一个窗口
      - `space-ws`：水平分屏
      - `space-wv`：垂直分屏
      - `space-wt`：切换分屏方向
      - `space-wq`：关闭当前窗口
      - `space-wh`：跳转到左边窗口，`jkl`相同
      - `space-wns`：以水平分屏的方式打开一个新的 buffer
      - `space-wnv`：以垂直分屏的方式打开一个新的 buffer

## 配置

自定义配置文件路径：`~/.config/helix/config.toml`

### 主题

主题配置文件路径：`~/.config/helix/themes/`

- 使用[官方提供的主题文件](https://github.com/helix-editor/helix/tree/master/runtime/themes)

```shell
wget -P ~/.config/helix/themes/ https://raw.githubusercontent.com/helix-editor/helix/master/runtime/themes/tokyonight.toml

hx ~/.config/helix/config.toml
```

```toml
theme = "tokyonight"
```

回到 Normal 模式，`:config-reload`命令重新加载配置

- 自定义主题

  留给读者自行探索吧
