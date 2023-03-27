---
title: MiBand7-BLE
date: '2023-02-28'
tags: ['Rust', 'Backend', 'Bluetooth']
draft: true
summary: 小米手环7通过Bluetooth Low Energy协议获取数据
images: []
layout: PostSimple
canonicalUrl:
authors: ['default']
---

## 获取 Authkey

- 确保小米手环 7 连接到小米运动健康应用
- 安卓手机在`内部存储/Android/data/com.mi.health/files/log/XiaomiFit.device.log`中搜索`Authkey`即可获得`1ced8f2175f257591bc4a98c980ffce6`

## 下载安装 Wireshark

## 抓取 BLE 数据包

[主流手机获取蓝牙日志](https://blog.csdn.net/rainyLYJ/article/details/128631231)

三星 Note20 Ultra OneUI5.0 日志路径：`内部存储/log/dumpState_N9860ZCU4GVK5_202303010001.zip`

蓝牙数据包日志路径：`dumpState_N9860ZCU4GVK5_202303010001/FS/data/log/bt/btsnoop_hci.log`
