---
layout: post
title: 'Visual Studio Code配置'
date: 2017-08-11 12:27:43 +0800
categories: ['编程']
author: Alex
noexcerpt: 1
---

## 1 typings

- 智能提示工具
- 如果未安装 node, 请先安装 nvm(node 版本管理工具)
- 安装 typings
  ```
  npm install -g typings
  ```
- 更新 typings

  ```
  npm update -g typescript
  ```

- 安装提示补全，需要进入到项目根目录下，安装完后，会在根目录下出现一个 typings 的文件夹
- 安装 node.js 补全

  ```
  typings install dt~node --global
  ```

- 安装 koa 补全

  ```
  `typings install dt~koa`
  ```

- 开启智能提示

  - 方法 1 在需要智能提示的文件顶部添加提示信息文件所在的目录，注意文件之间的相对位置

    ```javascript
    /// <reference path="./typings/index.d.ts" />
    ```

  - 方法 2 在项目的根目录下添加一个`jsconfig.json`文件，内容可以参考[JavaScript in VS Code](https://code.visualstudio.com/docs/languages/javascript)

    ```javascript
    {
        "compilerOptions": {
            "target": "ES6"
        },
        "exclude": [
            "node_modules",
            "**/node_modules/*"
        ]
    }
    ```

    - 对提示有特殊要求的自行参看文档，没有特殊要求的就用上面的这个。

# 2 配置与插件

- 该部分内容写在了另一篇文章中《Mac 软件安装》，该文还未上传
