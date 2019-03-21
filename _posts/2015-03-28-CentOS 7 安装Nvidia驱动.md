---
layout: post
title: 'CentOS 7 安装Nvidia驱动'
date: 2015-03-28 20:45:00 +0800
categories: ['编程']
tags: ['centos']
author: Alex
permalink: /centos7-nvidia-driver
---

在`centos7`上安装`nvidia`驱动

# 1 重要说明

**`请先看这里,我自己每次重装linux系统后都是按照这个一步一步来的，都是成功的`**。

但有的人和我说他没成功！！！

我的显卡: gtx970, 官方下载的驱动

# 2 在英伟达官网下载相应驱动

搜索出相应的驱动后，不要直接点，而是右健，Save Link as...
否则，会出现下载半天没动静的情况。
存放的路径上最好不要有中文。
我存放的路径是

```
~/Downloads/NVIDIA-Linux-x86_64-346.47.run
```

# 3 屏蔽默认带有的 nouveau

使用 su 命令切换到 root 用户下: su root
打开 **/lib/modprobe.d/dist-blacklist.conf**

将 nvidiafb 注释掉。

```
#blacklist nvidiafb
```

然后添加以下语句：

```
blacklist nouveau
options nouveau modeset=0
```

# 4 重建 initramfs image 步骤

```
mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
dracut /boot/initramfs-$(uname -r).img $(uname -r)
```

# 5 修改运行级别为文本模式

```
systemctl set-default multi-user.target
```

# 6 重新启动, 使用 root 用户登陆

```
reboot
```

# 7 查看 nouveau 是否已经禁用

```
ls mod | grep nouveau
```

如果没有显示相关的内容，说明已禁用。

# 8 进入下载的驱动所在目录

```
chmod +x NVIDIA-Linux-x86_64-346.47.run
./NVIDIA-Linux-x86_64-346.47.run
```

安装过程中，选择 accept
如果提示要修改 xorg.conf，选择`yes`

# 9 修改运行级别回图形模式

```
systemctl set-default graphical.target
```

# 10 重新启动，OK

在 Applications--Other 可以看见 NVIDIA X Server Settings 菜单。
然后添加以下语句：

```
blacklist nouveau
options nouveau modeset=0
```

## 4 重建 initramfs image 步骤

```
mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
dracut /boot/initramfs-$(uname -r).img $(uname -r)
```

## 5 修改运行级别为文本模式

```
systemctl set-default multi-user.target
```

## 6 重新启动, 使用 root 用户登陆

```
reboot
```

## 7 查看 nouveau 是否已经禁用

```
ls mod | grep nouveau
```

如果没有显示相关的内容，说明已禁用。

## 8 进入下载的驱动所在目录

```
chmod +x NVIDIA-Linux-x86_64-346.47.run
./NVIDIA-Linux-x86_64-346.47.run
```

安装过程中，选择 accept
如果提示要修改 xorg.conf，选择`yes`

## 9 修改运行级别回图形模式

```
systemctl set-default graphical.target
```

## 10 重新启动，OK

在 Applications--Other 可以看见 NVIDIA X Server Settings 菜单。
