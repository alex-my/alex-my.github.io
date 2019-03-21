---
layout: post
title: 'logrotate切割nginx日志'
date: 2017-06-26 12:35:00 +0800
categories: ['编程']
tags: ['后端']
author: Alex
permalink: /logrotate-cut-nginx-log
---

记录使用`logrorate`来切个日志的过程

# 1 配置

- 使用系统自带的 logrorate 来切个 nginx 日志，位于**/usr/sbin/logrotate**
- 假设服务器上有两个网站的 nginx 配置分别如下:

  - 去除其它配置信息，只保留了日志相关
  - A 网站

  ```
  ...
  access_log  /data/logs/a.com/access.log;
  error_log   /data/logs/a.com/error.log;
  ...
  ```

  - B 网站

  ```
  ...
  access_log  /data/logs/b.com/access.log;
  error_log   /data/logs/b.com/error.log;
  ...
  ```

- 在**/etc/logrotate.d/**下创建一个配置文件 nginx, 内容如下:

  ```
  # 这里可以添加你想切个的目录，也可以直接使用正则表达式
  /data/logs/a.com/*.log
  /data/logs/b.com/*.log
  {
  	daily
  	rotate 30
  	missingok
  	dateext
  	compress
  	delaycompress
  	notifempty
  	sharedscripts
  	postrotate
  	    if [ -f /usr/local/nginx/nginx.pid ]; then
  	        kill -USR1 `cat /usr/local/nginx/nginx.pid`
  	    fi
  	endscript
  }
  ```

  - 需要注意的是你们的 nginx.pid 位置，不一定是在**/usr/local/nginx/nginx.pid**

- 配置说明

      |         配置         |                               说明                               |
      | -------------------- | ---------------------------------------------------------------- |
      | daily                | 指定转储周期为每天                                               |
      | weekly               | 指定转储周期为每周                                               |
      | monthly              | 指定转储周期为每月                                               |
      | rotate               | 转储次数，超过将会删除最老的那一个                               |
      | missingok            | 忽略错误，如“日志文件无法找到”的错误提示                         |
      | dateext              | 切换后的日志文件会附加上一个短横线和YYYYMMDD格式的日期           |
      | compress             | 通过gzip 压缩转储旧的日志                                        |
      | delaycompress        | 当前转储的日志文件到下一次转储时才压缩                           |
      | notifempty           | 如果日志文件为空，不执行切割                                     |
      | sharedscripts        | 只为整个日志组运行一次的脚本                                     |
      | prerotate/endscript  | 在转储以前需要执行的命令可以放入这个对，这两个关键字必须单独成行 |
      | postrotate/endscript | 在转储以后需要执行的命令可以放入这个对，这两个关键字必须单独成行 |

# 2 测试

- 执行以下命令进行测试

```
logrotate -vf /etc/logrotate.d/nginx
```

- 然后到相应的日志目录下查看 （/data/logs/a.com/, /data/logs/b.com/）
- 应该会有类似以下的文件:
  - access.log
  - access.log-20170626
  - error.log
  - error.log-20170626

# 3 添加定时任务

- 每日 0 点执行脚本

  - 在终端运行 crontab -e
  - 插入以下语句

  ```
  0 0 * * * /usr/sbin/logrotate -vf /etc/logrotate.d/nginx
  ```
