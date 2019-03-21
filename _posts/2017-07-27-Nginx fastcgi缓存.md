---
layout: post
title: 'Nginx fastcgi缓存'
date: 2017-07-27 13:42:57 +0800
categories: ['编程']
tags: ['nginx', '后端']
author: Alex
permalink: /nginx-fastcgi-cache
---

`fastcgi_cache`缓存`fastcgi`生成的内容，很多情况是`php`生成的动态的内容，少了`nginx`与`php`的通信的次数，更减轻了`php`和数据库(`mysql`)的压力

# 1 `fastcgi_cache`与`proxy_cache`区别

- 网上找了很多资料，说的大同小异。
- proxy_cache 主要用于反向代理时，对后端内容源服务器进行缓存。
- fastcgi_cache 用于缓存 fastcgi 生成的内容。
- fastcgi_cache is related to the FastCGI backend protocol. It caches output from FastCGI connected backends.
- proxy_cache is related to backends that use HTTP as the backend protocol, and it caches output from HTTP connected backends.
- 如果这是一个反向代理服务器，使用 proxy_cache 进行缓存，减少代理服务器与源服务器的通信次数。
- 这里将使用 fastcgi_cache, 减少 nginx 与 php 的通信次数。

# 2 `fastcgi_cache`相关指令

- 资料链接 [http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html)
- `fastcgi_cache`
  - Syntax: fastcgi_cache zone / off;
  - Default: fastcgi_cache off;
  - Context: http, server, location
  - 定义一个用于缓存的区域
- `fastcgi_cache_key`
  - Syntax: fastcgi_cache_key string;
  - Default: —
  - Context: http, server, location
  - md5 这个参数后得到缓存文件名。
- `fastcgi_cache_min_uses`
  - Syntax: fastcgi_cache_min_uses number;
  - Default: fastcgi_cache_min_uses 1;
  - Context: http, server, location
  - 定义了请求多少次之后会被缓存
- `fastcgi_cache_methods`
  - Syntax: fastcgi_cache_methods GET / HEAD / POST ...;
  - Default: fastcgi_cache_methods GET HEAD;
  - Context: http, server, location
  - 用于指定缓存哪些方法，默认是不包含 POST 请求的。
- `fastcgi_cache_path`
  - Syntax: fastcgi_cache_path path [levels=levels][use_temp_path=on/off] keys_zone=name:size [inactive=time][max_size=size] [manager_files=number][manager_sleep=time] [manager_threshold=time][loader_files=number] [loader_sleep=time][loader_threshold=time] [purger=on/off][purger_files=number] [purger_sleep=time][purger_threshold=time];
  - Default: —
  - Context: http
  - 设置缓存路径和其它参数
    - path: 缓存路径
    - levels: 设置缓存目录层数，最多 3 层。假设 proxy_cache_key md5 后的值为 b7f54b2df7773722d382f4809d65029c
      - 第一层目录名称取值的最后一位。
      - 第二层目录名称取值的倒数二到三位。
      - 第三次目录名称取值的倒数四到六位
      - levels=1:2, 则目录为 /xx/cache/c/29/b7f54b2df7773722d382f4809d65029c
      - levels=1:2:3, 则目录为 /xx/cache/c/29/650/b7f54b2df7773722d382f4809d65029c
  - **可以用多条命令设置多个缓存磁盘，这样一台机器上的多个网站可以根据`keys_zone`存放在不同的地方。**
- `fastcgi_cache_valid`
  - Syntax: fastcgi_cache_valid [code ...] time;
  - Default: —
  - Context: http, server, location
  - 对于不同的状态码设置不同的缓存时间

# 3 设置示例

- 在 nginx 的目录下有一个文件 nginx.conf, 而各个网站的具体配置则在 sites-enabled 中
- nginx.conf

  ```
  worker_processes  2;
  error_log   /usr/local/var/logs/nginx/error.log debug;
  pid        /usr/local/var/run/nginx.pid;
  worker_rlimit_nofile 1024;

  events {
      worker_connections  1024;
  }
  ```


    http {
        include       mime.types;
        default_type  application/octet-stream;

        log_format main '$remote_addr - $remote_user [$time_local] '
            '"$request" $status $body_bytes_sent '
            '"$http_referer" "$http_user_agent" '
            '"$http_x_forwarded_for" $host $request_time $upstream_response_time $scheme '
            '$cookie_evalogin';

        access_log  /usr/local/var/logs/access.log  main;

        sendfile        on;
        keepalive_timeout  65;
        port_in_redirect off;

        # 内存缓存大小为500M，硬盘缓存大小为50G，缓存超过30分钟没有人访问就会被删除(根据需要自行设置)
        fastcgi_cache_path /Users/alex/WWW/nginx_cache levels=1:2 keys_zone=cache_one:500m inactive=30m max_size=50g;
        # 临时文件存放, 没有设置层级
        fastcgi_temp_path /Users/alex/WWW/nginx_temp;

        fastcgi_cache_key "$scheme$host$request_method$request_uri";

        fastcgi_ignore_headers Cache-Control Expires Set-Cookie;

        # 导入各个网站的详细配置
        include /usr/local/etc/nginx/sites-enabled/*.conf;
    }
    ```

- 示例网站 alex-my.xyz 的配置 (位于 sites-enabled 中)

  ```
  server {
      charset utf-8;
      client_max_body_size 128M;
      listen 80;

      server_name alex-my.xyz;
      root        '/Users/alex/WWW/alex_my/web/';
      index       index.php;

      access_log  /Users/alex/WWW/logs/alex_my.com/access.log;
      error_log   /Users/alex/WWW/logs/alex_my.com/error.log;

      # 0 缓存, 非0 不缓存
      set $skip_cache 1;
      # 动态查询不缓存
      # if ($query_string != "") {
      #     set $skip_cache 1;
      # }
      # fastcgi_cache_methods 默认不缓存POST

      # 只缓存指定的界面，比如请求的地址中含有summary则缓存
      # 请搜索 nginx $request_uri 和 nginx location 来了解相关信息
      # PHP可以通过打印$_SERVER['REQUEST_URI']来获取信息。
      if ($request_uri ~* "index/summary|level|role|online|recharge|vip|consumption") {
          set $skip_cache 0;
      }
  ```


        location / {
            # Redirect everything that isn't a real file to index.php
            try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \.php$ {
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_pass   127.0.0.1:9000;
            try_files $uri =404;

            # 缓存
            fastcgi_cache_bypass $skip_cache;
            fastcgi_no_cache $skip_cache;
            add_header X-Cache "$upstream_cache_status From $host";
            # fastcgi_cache的值注意和fastcgi_cache_path中的key_zone相同
            fastcgi_cache cache_one;
            fastcgi_cache_valid 200 301 302 30m;
        }

        location ~ /\.(ht|svn|git) {
            deny all;
        }
    }
    ```

- 执行后刷新主页发现，在 cache 目录下有文件生成，在这个缓存有效期间，修改主页内容也不会展示出来。
- 为了方便测试，我在主页添加了 `echo date('Y-m-d H:i:s');`, 同时将 nginx.conf 的`fastcgi_cache_path`的`inactive`设置为 1 分钟。

# 4 缓存清理

- 方法 1
  - 使用`ngx_cache_purge`模块。
- 方法 2
  - 使用`fastcgi_cache_valid`对各种响应设置为不同的有效时长。
- 方法 3
  - 根据`fastcgi_cache_path`生成规则，删除指定文件。
- 方法 4
  - 直接删除缓存文件夹。

# 5 权限区分

- 其实这个缓存更适合无关用户状态且改变很少的内容，比如你的文章。
- 如果与用户状态有关，就会发生内容串了的问题。
- 可以借助修改`fastcgi_cache_key`来区分。
- 方法 1: 使用 ip 区分

  - 该方法不安全，会被造假。
  - 修改`fastcgi_cache_key`,添加`remote_addr`。

    ```
    fastcgi_cache_key "$scheme$host$request_method$request_uri$remote_addr;
    ```

  - 还有端口，`$remote_port`。

# 6 参考

- [http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html)
