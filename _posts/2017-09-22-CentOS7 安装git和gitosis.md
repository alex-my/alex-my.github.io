---
layout: post
title: 'CentOS7 安装git和gitosis'
date: 2017-09-22 13:26:32 +0800
categories: ['编程']
tags: ['centos', 'git']
author: Alex
permalink: /centos7-install-git-gitosis
---

后记: 个人后来还是改安装`gitlab`了，2G 内存吃紧，加上`swap`可以使用。在后来，`github`允许不限量创建私有项目了，都是好消息。

# 1 环境说明

- 服务器使用的阿里云, CentOS7.4
- 本机使用的 Mac
- 服务器的 ssh 端口不是默认的 22,而是 8998(实际不是,不想告诉你),安全为主

  - 打开 `/etc/ssh/sshd_config`
  - 找到 **Port 22**
  - 将 22 修改为 8998
  - 重启 ssh 服务: `systemctl restart sshd.service`
  - 在阿里云后台安全组添加 8998
  - 在 firewalld 添加端口 8998

    ```text
    -- 添加
    firewall-cmd --zone=public --add-port=8998/tcp --permanent
    -- 重载
    firewall-cmd --reload
    -- 查看
    firewall-cmd --zone=public --list-ports
    ```

- 多提一句,虽然我们修改了 ssh 端口,但还是可以用类似`namp`工具扫描出来,所以我把`ICMP`也屏蔽了

  ```text
  echo 1 >/proc/sys/net/ipv4/icmp_echo_ignore_all
  ```

  - 如果要恢复,把数字修改为 0 就好
  - 重启就失效了,所以需要添加到开机启动中(rc.local 或者 systemd)
  - 试一试,还能`ping`的通服务器吗？

- 我的机子实际 IP 地址也不是`124.123.122.121`, 随意填的

# 2 安装 git

- 卸载旧的 git

  ```text
  yum remove git
  ```

- 安装依赖包

  ```text
  yum install autoconf curl-devel expat-devel openssl-devel zlib-devel perl-devel
  ```

- 下载源码包

  - 去官网 [git-scm](https://www.kernel.org/pub/software/scm/git/) 拷贝链接
  - 我所使用的链接: https://www.kernel.org/pub/software/scm/git/git-2.9.5.tar.gz
  - 下载到任意位置(装完就删)

    ```text
    wget https://www.kernel.org/pub/software/scm/git/git-2.9.5.tar.gz
    ```

- 编译安装
  ```text
  tar -zxvf git-2.9.5.tar.gz
  cd git-2.9.5
  ./configure
  make
  make install  # 需要root权限
  ```
- 制作软链接到`/usr/bin`

  ```text
  ln -s /usr/local/bin/git /usr/bin/git
  ```

- 查看版本号

  ```text
  git --version
  ```

  - 输出: `git version 2.9.5`

- 删除安装包
  ```text
  rm -rf git-2.9.5 git-2.9.5.tar.gz
  ```

# 3 新建用户 git

- 我们希望有专门的用户来管理 git 服务,而不是使用拥有最高权限的 root 用户
- 新建用户 git,用于运行服务(root 用户执行)

  ```text
  useradd git
  ```

- 为新用户创建密钥等信息(git 用户执行)

  ```text
  su git  # 从root用户转为git
  cd ~    # 实际上为 /home/git
  mkdir .ssh
  chmod 700 .ssh
  cd .ssh
  ssh-keygen -t rsa
  ```

  - 会在`/home/git/.ssh`生成两个文件`id_rsa`,`id_rsa.pub`

# 4 安装 gitosis

- 这是一款非常好用的权限管理工具,具体说明自行搜索
- 安装 python 以及 python 工具,一般默认自带 2.7.5,可以照样再执行一遍

  ```text
  yum install python python-setuptools
  ```

- 下载安装 gitosis

  ```text
  git clone git://github.com/res0nat0r/gitosis.git
  cd gitosis
  python setup.py install
  ```

  - 最后输出 `Finished processing dependencies for gitosis==0.2`表示成功

# 5 配置 gitosis

- 初始化 gitosis

  ```text
  gitosis-init < /home/git/.ssh/id_rsa.pub
  rm -rf /home/git/.ssh/id_rsa.pub # 安全起见,就删了吧
  chmod 755 /home/git/repositories/gitosis-admin.git/hooks/post-update
  ```

- 由于仓库默认就是`/home/git/repositories`,但是常常这个地方硬盘较小,因此我希望能指定到较大空间的硬盘去
- 没有找到如何在`gitosis-init`的时候指定路径的方法,这边就采取妥协的方式: 使用软链接(root 用户执行)

  ```text
  exit # 退出git用户,返回到之前的root用户
  mv /home/git/repositories /data/git # 移动到/data目录下并重命名为git
  ln -s /data/git /home/git/repositories
  chown git:git /data/git
  chown git:git /home/git/repositories
  ```

- 先把权限管理工程克隆下来(管理员/运维人员**以后**可以克隆到自己的电脑上,我这里直接放在了 git 用户这里)(git 用户执行)
  ```text
  su git
  cd ~
  git clone ssh://git@124.123.122.121:8998/gitosis-admin.git git-manage
  ```
  - 如果 ssh 端口是默认的 22，可以使用`git clone git@124.123.122.121:gitosis-admin.git git-manage`
- 在 git-manage 目录中有两个文件
  - gitosis.conf: 配置权限
  - keydir: 存放用户公钥
- 将自己的密钥文件上传到`keydir`目录中,并命名,比如我自己电脑上的公钥命名为`alex.pub`
  ```text
  # linux,mac等用scp命令,服务器ssh端口为8998
  scp -P 8998 id_rsa.pub root@124.123.122.121:/data
  ```
  - 登录服务器将`id_rsa.pub`移动到`/home/git/git-manage/keydir`中,并命名为`alex.pub`
- 配置`gitosis.conf`文件(git 用户执行)

  ```text
  [gitosis]

  [group gitosis-admin]
  members = git@keylala
  writable = gitosis-admin

  [group dev]
  members = alex
  writable = js.keylala
  ```

  - 以上代码中,我们可以看见,只有`git@keylala`用户能够操作`gitosis-admin`项目,运维可以把自己的名称也加到这里
  - members 指的是用户名称,这个用户名称必须与公钥文件的名称相同
  - members 可以配置多个用户,用空格隔开就行
  - writable 用于指定可以写的工程,比如后面我们会创建一个`js.keylala.git`工程
  - 以上的意思是,拥有`alex.pub`这个公钥对应私钥的用户可以操作`js.keylala.git`工程
  - 至于`group dev`什么的都是自己命名的,请随意

- 按照以上改完之后,要记得,`git-manage`是导出的项目工程,我们要用命令推送回仓库中去(git 用户执行)

  ```text
  cd /home/git/git-manage
  git add .
  git commit -m "add new group"
  git push origin master
  ```

  - 新增文件时用`git add .`
  - 如果只是修改,可以不用`git add .`，而是用`git commit -am "change"`一次性搞定

# 6 使用

- 创建项目(git 用户执行)

  ```text
  su git
  cd /data/git
  git init --bare js.keylala.git
  ```

  - 这个时候会有一个`js.keylala.git`文件夹生成

- 在自己的电脑上访问

  ```text
   git clone ssh://git@124.123.122.121:8998/js.keylala.git
  ```

  - 这样就下载到自己的机子上了
  - 如果服务器用的是默认的 22 端口,用以下命令

    ```text
    git clone git@124.123.122.121:js.keylala.git
    ```

- 在自己的电脑上修改并提交

  ```text
  cd js.keylala
  touch readme.md
  git add .
  git commit -m "add new file"
  git push origin master
  ```

- 提交之后，可以在另一个地方再克隆出来看看，是不是有`readme.md`这个文件了
