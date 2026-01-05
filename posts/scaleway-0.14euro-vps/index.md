# Scaleway €0.14 IPv6-Only 廉价 KVM VPS


> Credit: 本文参考 [Moohr 的教程](https://www.mhr.hk/posts/alpine-on-scw/)

折腾 #DN42 的过程就是买 VPS 的血泪史（不知道 DN42 可以查看 Lantian 师祖的[这篇文章](https://lantian.pub/article/modify-website/dn42-experimental-network-2020.lantian/)）。DN42 玩家们普遍使用 [Bird](https://bird.network.cz/)，内存占用小，1G 内存的机器整张 Fulltable 绰绰有余。DN42 的网络小的多，节省成本尽量选择小 VPS。本文选择精简的 [Alpine Linux](https://www.alpinelinux.org/)。

此文章最终部署的 VPS 规格为：
- CPU: 1 core
- RAM: 1 GB
- Disk: 1GB
- Visualization: KVM
- System: Alpine Linux
- Location: Amsterdam/Paris/Warsaw
- Network: IPv6(/64), IPv4(w/ Cloudflare Warp)

[!特别注意]
> 无公网 IPv4, 通过 Cloudflare Warp 访问 IPv4 资源

## 下单

我们的目标机型是 STARDUST1-S 型 VPS。

选择类别中的 Development，找到 STARDUST1-S。地区随意，如果缺货换一个地区。下单默认配置：1 CPU、1GB、10GB（硬盘稍后手动删除），系统随便选，SSH Key 自行添加。

![](/images/scaleway-0.14euro-vps/Snipaste_2025-11-26_13-16-07.png)

IPv4 取消勾选（不然每月 €2.92）。

![](/images/scaleway-0.14euro-vps/Snipaste_2025-11-26_13-16-16.png)

此时结算价格为每月预估 €0.97，如果不对请重新检查配置。

![](/images/scaleway-0.14euro-vps/Snipaste_2025-11-26_13-22-58.png)

**或者可以用右上角的 Cli 下单（据说可无视售罄强开）：**

![](/images/scaleway-0.14euro-vps/Snipaste_2025-11-26_13-17-17.png)

法国巴黎可用区1 PAR-1：
```
scw instance server create zone=fr-par-1 root-volume=local:10GB name=fr type=STARDUST1-S ipv6=true ip=none
```
荷兰阿姆斯特丹可用区1 AMS-1：
```
scw instance server create zone=nl-ams-1 root-volume=local:10GB name=nl type=STARDUST1-S ipv6=true ip=none
```
波兰华沙可用区1 WAW-2：
```
scw instance server create zone=pl-waw-2 root-volume=local:10GB name=pl type=STARDUST1-S ipv6=true ip=none
```

# 配置 VPS

默认的 10GB 硬盘对我们来说还是太大了，而且一个月要 €0.86，不符合穷鬼原则。我们删除它再建立一个 1G 小盘，然后再安装系统。

## 配置硬盘

### 删除默认 10G 的 Block Storage

关掉 VPS，进入左侧菜单 Storage ➡️ Block Storage，可用区一般会自动选择，Detach 现有的这块盘然后删除。
 
### 创建 1G 系统盘

左侧菜单 Storage ➡️ Local Storage 创建，选择相同可用区，大小 1GB（酌情增加，€0.03/GB/mo）。

由于面板 Bug，此时是无法 Attach 到机器上的，先创建，然后在菜单 Attach。

## 安装 Alpine

此部分为 ISO 通过 alpine-setup 安装，也可[通过 DD 安装](https://www.mhr.hk/posts/alpine-on-scw/)。

主机设置 Settings Bootmode 为 Use rescue image。

启动 VPS，Rescue 镜像的 Cloudinit 会自动加载之前配置过的密钥，直接连接进去。

创建 GPT 分区
```
parted /dev/vda mklabel gpt
```

下载 ISO 镜像写入分区
```
wget -qO- https://dl-cdn.alpinelinux.org/alpine/v3.22/releases/x86_64/alpine-virt-3.22.0-x86_64.iso | dd of=/dev/vda
```

关机，Bootmode 修改回 Use local boot。开机。

打开 Console(VNC) 用 root 登录（无密码）。

准备安装环境
```
mkdir /media/setup
cp -a /media/vda/* /media/setup
mkdir /lib/setup
cp -a /.modloop/* /lib/setup
/etc/init.d/modloop stop
umount /dev/vda
mv /media/setup/* /media/vda/
mv /lib/setup/* /.modloop/
```

运行安装程序
```
setup-alpine
```

- 配置主机名，自行输入。
- 配置网络，输入 `done`，下一步按 `y` 手动配置。
```
auto eth0
iface eth0 inet6 static
    address <ipv6_address>
    netmask 64
    gateway <ipv6_gateway>
```
- 配置 Root 密码
- 选择时区 `Europe/Paris` `Europe/Amsterdam` `Europe/Warsaw`
- 配置软件包管理器，此时如没有网络可 `skip`
- 安装 boot：依次输入 `vda`、`sys`

此时可能会出现安装失败，找不到 `dosfstools` `grub-efi` 等，先不急。

先配置一下 DNS（如没有），然后配置好官方软件源。

```
echo "http://dl-cdn.alpinelinux.org/alpine/latest-stable/main" >> /etc/apk/repositories
echo "http://dl-cdn.alpinelinux.org/alpine/latest-stable/community" >> /etc/apk/repositories
echo "#http://dl-cdn.alpinelinux.org/alpine/edge/main" >> /etc/apk/repositories
echo "#http://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories
echo "#http://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories
```

安装缺失工具

```
apk update
apk add dosfstools
apk add grub-efi
```

完成安装

```
# 33 MB efi
export BOOT_SIZE=33
# 安装 disk，关闭 swap
setup-disk -s 0
```
依次输入 `vda` `sys` `y` 即完成

如不报错可 reboot

## 网络

可使用 Cloudflare Warp 访问 IPv4。如部署脚本 https://gitlab.com/fscarmen/warp


---

> 作者: [Choco](https://words.choco.pub)  
> URL: https://words.choco.pub/posts/scaleway-0.14euro-vps/  

