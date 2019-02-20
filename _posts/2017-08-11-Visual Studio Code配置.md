---
layout: post
title: 'Visual Studio Code配置'
date: 2017-08-11 12:27:43 +0800
categories: ['编程']
author: Alex
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

# 2 配置

```text
{
    "workbench.iconTheme": "vscode-icons",
    "workbench.colorTheme": "One Dark Pro",


    // 控制字体系列。
    "editor.fontFamily": "Menlo, Monaco, 'Courier New', monospace", // 以像素为单位控制字号。
    "editor.fontSize": 14,
    // 指向 PHP 可执行文件。
    "php.validate.executablePath": "/usr/bin/php",
    "php.executablePath": "/usr/bin/php",
    // PHP 保存时设置文件的格式。格式化程序必须可用，不能自动保存文件，并且不能关闭编辑器。
    "editor.formatOnSave": true,
    // 在默认不支持 Emmet 的语言中启用 Emmet 缩写功能。在此添加该语言与支持 Emmet 的语言之间的映射。
    // 示例: {"vue-html": "html", "javascript": "javascriptreact"}
    "emmet.includeLanguages": {
        "javascript": "javascriptreact"
    },
    // 为指定的语法定义配置文件或使用带有特定规则的配置文件。
    "emmet.syntaxProfiles": {
        "javascript": "jsx"
    },
    "gitlens.advanced.messages": {
        "suppressGitDisabledWarning": true,
        "suppressShowKeyBindingsNotice": true
    },
    "workbench.startupEditor": "welcomePage",
    "workbench.sideBar.location": "left",
    // go
    "go.gopath": "/Users/alex/go",
    "window.zoomLevel": 0,
    "terminal.integrated.rendererType": "dom",
    "explorer.confirmDelete": false,
    "prettier.singleQuote": true,
    "prettier.trailingComma": "es5",
    "explorer.confirmDragAndDrop": false,
    "javascript.updateImportsOnFileMove.enabled": "always",

    "git.enabled": true,
    "extensions.ignoreRecommendations": false,

    // eslint
    "eslint.autoFixOnSave": true,

    "python.linting.pylintUseMinimalCheckers": false,

    // 这个会带起一个 coder helper，占用大量 CPU，温度超高
    "search.followSymlinks": true
}
```

## 3 插件

### 通用插件

- Auto Close Tag
- Atuo Rename Tag
- beautify，格式化代码工具(JSON|JS|HTML|CSS|SCSS，但不要用来格式化 react)
- Bootstrap 3 Sinnpet
- Bracket Pair Colorizer，让括号拥有独立的颜色，易于区分
- ESlint，js 提示
- filesize，在底部显示当前文件信息
- GitLens，git
- HTML CSS Support，css, scss 智能提示
- HTMLHint，html 代码检测
- HTML Snippets，H5 代码片段以及提示
- indenticator: 缩进指示器
- Markdown Preview Enhanced，markdown 实时预览
- mssql，数据库相关
- One Dark Pro，主题
- Path Intellisense，路径补全
- `Prettier`，代码格式化工具 `(会引起单引号变双引号，以及自动去除单参数括号)`
- `Prettier Now`
- Project Manager，多个项目管理工具
- vscode-icons，给 vscode 的资源树目录加上图标

### php 插件

- PHP Debug
- PHP IntelliSense（该插件引起一个 CPU 核 100%使用率）

### node.js/react.js 插件

- cssrem，px 转 rem
- JavaScript (ES6) code snippets，代码提示
- npm Intellisense
- Reactjs code snippets，代码提示
- React Redux ES6 Snippets，代码提示
- React-Native/React/Redux snippets for es6/es7，代码提示
- CSS Modules，对使用了 css modules 的 jsx 标签的类名补全和跳转到定义位置
