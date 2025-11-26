---
author: ["Me"]
title: "Scaleway €0.14 IPv6-only 廉价 KVM VPS"
date: "2025-11-26"
description: "Scaleway VPS 每月只需一块钱，1C1G，不限流量，KVM，还要啥自行车"
summary: "每月只需一块钱，1C1G，不限流量，KVM，还要啥自行车"
tags: ["DN42", "Scaleway", "VPS"]
ShowToc: true
---

> Credit: 本文参考 [Moohr的教程](https://www.mhr.hk/posts/alpine-on-scw/)

折腾 #DN42 的过程就是买 VPS 的血泪史（不知道 DN42 可以查看 Lantian 师祖的[这篇文章](https://lantian.pub/article/modify-website/dn42-experimental-network-2020.lantian/)）。DN42 玩家们普遍使用 [Bird](https://bird.network.cz/)，内存占用小，1G 内存的机器整张 Fulltable 绰绰有余。DN42 的网络小的多，节省成本尽量选择小 VPS。

此文章最终部署的 VPS 规格为：
- CPU: 1 core
- RAM: 1 GB
- Disk: 1GB
- Visualization: KVM
- Location: Amsterdam/Paris/Warsaw
- Network: IPv6(/64), IPv4(w/ Cloudflare Warp)

⚠️特别注意：无公网 IPv4⚠️

# 下单

我们的目标机型是 STARDUST1-S 型 VPS。

选择类别中的 Development，找到 STARDUST1-S。地区随意，如果缺货换一个地区。下单默认配置：1 CPU、1GB、10GB（硬盘稍后手动删除），IPv4 取消勾选（不然每月 €2.92）。



或者可以用右上角的 Cli 下单：

法国巴黎可用区1 PAR-1：
```
scw instance server create zone=fr-par-1 root-volume=local:10GB name=fr type=STARDUST1-S ipv6=true ip=none
```
荷兰阿姆斯特丹可用区1 AMS-1：
```
scw instance server create zone=nl-ams-1 root-volume=local:10GB name=nl type=STARDUST1-S ipv6=true ip=none
```
波兰华沙可用区1 WAW-1：
```
scw instance server create zone=pl-waw-1 root-volume=local:10GB name=pl type=STARDUST1-S ipv6=true ip=none
```

下单后
