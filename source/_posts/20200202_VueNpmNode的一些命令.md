---
title: Vue Npm Node 的一些命令
date: 2021-02-02 12:11:57
categories:
  - Vue
tags:
  - Vue
---

Vue Npm Node 的一些命令
<!--more-->

#### 查看node版本命令：
> node -v

#### 使用npm安装webpack(-g代表全局安装) 3.6.0版本 命令：
> npm install webpack@3.6.0 -g

#### 查看webpack版本命令：
> webpack --version

#### webpack打包js的命令：
> webpack ./src/main.js ./dist/bundle.js

#### 初始化项目的npm环境命令：
> npm init

#### 安装项目的依赖包命令：
> npm install

#### 本地安装开发依赖的webpack命令(在当前项目文件夹下执行)：
> npm install webpack@3.6.0 --save-dev

#### npm安装开发依赖的css-loader命令：
> npm install --save-dev css-loader@2.0.2

#### npm安装开发依赖的style-loader命令：
> npm install style-loader@0.23.1 --save-dev

#### npm安装开发依赖的less-loader less命令：
> npm install --save-dev less-loader@4.1.0 less@3.9.0

#### npm安装开发依赖的url-loader命令：
> npm install --save-dev url-loader@1.1.2

#### npm安装开发依赖的file-loader命令：
> npm install --save-dev file-loader@3.0.1

#### npm安装开发依赖的ES6语法处理转成ES5的babel-loader命令：
> npm install --save-dev babel-loader@7.1.5 babel-core@6.26.3 babel-preset-es2015@6.24.1

#### npm安装全局vue命令：
> npm install vue@2.5.21 --save

#### npm安装开发依赖的vue-loader vue-template-compile命令：
> npm install --save-dev vue-loader@15.4.2 vue-template-compiler@2.5.21

#### 添加版权信息在webpack.config.js中加入：
```
const webpack = require('webpack')
和
plugins: [
    new webpack.BannerPlugin('最终版权归aaa所有')
]
```

#### 打包html的plugin：
```
1）npm install html-webpack-plugin@3.2.0 --save-dev
2）删掉webpack.config.js中的：publicPath: 'dist/'
3）删掉html中的：<script src="./dist/bundle.js"></script>
4）webpack.config.js中加：
    const HtmlWebpackPlugin = require('html-webpack-plugin')
    plugins: [
        new HtmlWebpackPlugin({
            template: 'index.html'
        })
    ]
```

#### js压缩的plugin：
```
1）npm install uglifyjs-webpack-plugin@1.1.1 --save-dev
2）webpack.config.js中加：
    const UglifyjsWebpackPlugin = require('uglifyjs-webpack-plugin')
    plugins: [
        new UglifyjsWebpackPlugin()
    ]
```

#### 搭建本地服务器
```
1）npm install webpack-dev-server@2.9.3 --save-dev
2）package.json中加：
    "scripts": {
        "dev": "webpack-dev-server --open"
    },
3）控制台启动使用：npm run dev
```

#### 配置文件分离：
```
1）npm install webpack-merge@4.1.5 --save-dev
2）prod.config.js和dev.config.js中加：
    const webpackMerge = require('webpack-merge')
    const baseConfig = require('./base.config')
    module.exports = webpackMerge(baseConfig, {
        ......
    })
3）package.json中加：
    "scripts": {
        "build": "webpack --config ./build/prod.config.js",
        "dev": "webpack-dev-server --open --config ./build/dev.config.js"
    },
4）重新运行：npm run build 或 npm run dev
```

#### 安装Vue脚手架：
> npm install -g @vue/cli

#### 如果想使用2.x版本的Vue脚手架：
> npm install -g @vue/cli-init

#### Vue CLI2初始化项目：
> vue init webpack my-project

#### Vue CLI3初始化项目：
> vue create my-project
