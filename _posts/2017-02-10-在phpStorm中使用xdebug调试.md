---
layout: post
title: '在PHPStorm中使用xdebug调试'
date: 2017-02-10 17:16:18 +0800
categories: ['编程']
tags: ['php', 'xdebug']
author: Alex
noexcerpt: 1
---

# 1 环境说明

在 mac 下搭建的 lnmp 环境，可以参考:
[Mac 搭建 lnmp 环境](http://alex-my.xyz/Mac%E6%90%AD%E5%BB%BAlnmp%E7%8E%AF%E5%A2%83)
<br>
nginx 中的网站配置:

```
fastcgi_pass   127.0.0.1:9000
```

环境均使用 brew 安装，其中 xdebug 被安装到:

```
/usr/local/opt/php56-xdebug/xdebug.so
```

php 的配置中也有指向

```
/usr/local/etc/php/5.6/conf.d/ext-xdebug.ini
```

## 2 配置 php.ini

/usr/local/etc/php/5.6/php.ini

```
[xdebug]
xdebug.remote_enable =1
xdebug.remote_handler = "dbgp"
xdebug.remote_host = "localhost"        # 调试的IDE所在的地址
xdebug.remote_port = 9001               # 调试的IDE所用的端口
xdebug.remote_mode = "req"
xdebug.idekey="PHPSTORM"
```

xdebug.remote_mode：

```
req: 在PHP程序开始执行的时候，xdebug与IDE建立连接
jit: 在PHP程序执行到断点处或者遇到Error的时候，xdebug才与IDE建立连接
```

需要注意的是，这里不要再添加以下配置，会出现警告: 已经加载了 xdebug.so

```
zend_extension="/usr/local/opt/php56-xdebug/xdebug.so"
```

还有一个重要的是，如果你用的是 nginx，并且是默认配置，一般 9000 端口都是被使用的。
按照网上其它教程做而 xdebug 无法断点的原因就是使用了以下配置:

```
xdebug.remote_port = 9000
```

重启 php-fpm

```
killall php-fpm
php-fpm -D
```

## 3 配置 phpStorm

打开 phpstorm--Preferences--Languages & Frameworks -- PHP
点击 Debug, 填写以下内容

```
Xdebug -- Debug port: 9001 # 和php.ini中的xdebug.remote_port保持一致
```

打开 Debug--DBGp Proxy 填写以下内容

```
IDE key: phpStorm
Host: localhost         # 要调试的网站地址, 如127.0.0.1, site.com
Port: 80                # 要调试的网站端口
```

- 打开网站工程，IDE 右上角，点击 Edit Configurations..
- 点击弹出框左侧的+号。
- 选择 PHP Web Application
- 此时左侧多了一列 PHP Web Application -- Unnamed (改名为 start)
- 在右侧 -- Configuration -- Server 右侧的 ...
- 在弹出框 Servers 左侧点击+号，填写以下内容

```
Name: start2        # 随意名称
Host: localhost     # 网站地址，与Debug--DBGp Proxy相同
Port: 80            # 网站端口，与Debug--DBGp Proxy相同
Debugger: Xdebug
```

一些就绪后，在 IDE 的右上侧,绿色三角形右侧，有一个臭虫按钮，打好断点，就可以点击使用了

## 4 xdebug 工作原理说明

- IDE 中安装了一个遵循 BGDp 协议的 Xdebug 插件, 称为 xdebug-a
- 调试模式下，IDE 中的 xdebug-a 创建服务，监听端口: 9001(在 phpStorm 中设置的)
- IDE 在当前 url 后面加上了 XDEBUG_SESSION_START 参数
- php 服务器中的 xdebug 模块，称为 xdebug-b, 接收到带有 XDEBUG_SESSION_START 的请求后，会进入到调试模式
- xdebug-b 会以协议(BGDp)向 xdebug-a 的服务建立连接，提供调试服务。
- php.ini 中配置的 xdebug.remote_host:xdebug.remote_port 是 xdebug-a 的地址和端口
  <br>
  xdebug-a 创建服务时，这个端口不能被其它进程占用了。
