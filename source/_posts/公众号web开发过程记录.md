---
title: 公众号开发前端
date: 2017-06-19 11:33:30
tags: 前端
---

上一篇讲了微信公众号后端的开发，这一篇是讲讲前端页面的开发。前端的技术栈用的是react。这里介绍了在开发过程中遇到的一些问题。以及一些知识点的介绍

以下步骤的前提是你已经安装了node，才能继续往下走。

## 创建一个node工程

​	初始化一个工程，并进行一些基本信息填写。执行`npm init`，根据提示填写信息，最后会生成一个package.json 文件。这样一个node工程初始化完成了。以下package.json 文件是我修改过的，默认生成的文件和这个是有差异的。主要讲解几个字段，main:表示入口文件，scripts：主要是脚本命令，执行npm start 的时候，它会执行scripts 下start 后的命令，dependencies：是工程依赖的一些库，devDependencies：主要是开发的时候使用的一些库，生产的时候不需要使用。

<!-- more -->

```json
{
  "name": "doudou-client",
  "version": "1.0.0",
  "description": "wechat public",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "webpack-dev-server -d --hot --inline --progress --colors --host 0.0.0.0 --port 8080",
    "dev": "webpack -d --watch"
  },
  "author": "cocoon",
  "license": "ISC",
  "dependencies": {
    
  },
  "devDependencies": {
    "webpack": "^1.12.2",
    "webpack-dev-server": "^1.12.1"
  }
}
```

​	有一个package.json  文件还不能使用，接下来是编写app.js文件，也就是入口文件，我们就简易的写一个demo文件吧！我们要使用react进行开发我们就得安装reactjs 相关的依赖，执行以下命令来添加reactjs 依赖的库,和开发时使用的一些开发库。如：webpack

 ```sh
npm install --save react react-dom
npm install --save-dev css-loader
npm install --save-dev style-loader
npm install --save-dev babel-loader babel-core babel-preset-es2015 babel-preset-react
npm install --save-dev webpack webpack-dev-server
 ```

创建一个app.js文件

```jsx
import React, { Component } from 'react';
import ReactDOM from 'react-dom';

class App extends Component {
    render() {
           return (
               <div>hello wechat test</div>
           );
       }
}
ReactDOM.render((
       <App/>
), document.getElementById('container'));
```

​	除了js文件之外，我们还需要一个 html文件，来加载这些js文件。

```html
<!doctype html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no"/>
    <link rel="stylesheet" href="src/css/fontAwesome/css/font-awesome.min.css">

    <link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/slick-carousel/1.6.0/slick.min.css" />
    <link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/slick-carousel/1.6.0/slick-theme.min.css" />

    <title>微信公众号网页</title>
    <script>
        function setHeight() {
            console.log(screen.height)
            document.body.style.height = screen.height + 'px';
        }
    </script>
</head>
<body style="width: 100%;" onload="setHeight()">
<div id="container" id="container" style="height: 100%"></div>
<script src="./assets/vendor.bundle.js"></script>
<script src="./assets/bundle.js"></script>
</body>
</html>
```

​	发现我们的html页面中并没有加载app.js文件，是因为我们通过webpack 来打包js文件，css文件，图片等。html页面加载的是我们打包之后的文件。所以我们在工程目录下创建一个webpack.config.js文件，来配置webpack一些打包的配置项

```js
var webpack = require('webpack');
var path = require('path');
var autoprefixer = require('autoprefixer');
var HtmlWebpackPlugin = require('html-webpack-plugin');
var ExtractTextPlugin = require('extract-text-webpack-plugin');
var OpenBrowserPlugin = require('open-browser-webpack-plugin');

module.exports = {
    entry: {
        js: ['./app.js'],
        vendor: ['react', 'classnames', 'react-router', 'react-dom', 'react-addons-css-transition-group']
    },
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: './bundle.js'
    },
    devServer:{
        proxy:{
            '/api':{
                target:'http://localhost:5000',
                secure: false,
                changeOrigin: true,
                pathRewrite: {'^/api': ''}
            }
        },
    },
    module: {
        loaders: [
            {
                test: /\.js[x]?$/,
                exclude: /node_modules/,
                loader: 'babel',
                query: {
                    cacheDirectory: true,
                    presets: ['es2015', 'react']
                }
            }, {
                test: /\.less$/,
                loader: 'style!css!postcss!less'
            }, {
                test: /\.css/,
                loader: ExtractTextPlugin.extract('style', 'css', 'postcss')
            }, {
                test: /\.(png|jpg|svg)$/,
                loader: 'url?limit=25000'
            }
        ]
    },
    postcss: [autoprefixer],
    plugins: [
        new webpack.DefinePlugin({
            DEBUG: process.env.NODE_ENV !== 'production'
        }),
        new ExtractTextPlugin('weui.min.css'),
        new webpack.optimize.CommonsChunkPlugin('vendor', 'vendor.bundle.js'),
        
        new HtmlWebpackPlugin({
            template: path.join(__dirname, './index.html')
        }),
    ]
};

```

​	更多的webpack学习，请移步[webpack学习入门](https://zhaoda.gitbooks.io/webpack/content/)



整个工程demo在GitHub上[wechat_public_template](https://github.com/Cocoon-break/wechat_public_template)

## 常规问题

- 网页无法适应全屏？

  在开发过程中发现body标签高度始终无法全屏，及时设置高度为100%，也无法生效。

   ```html
  <!--1.添加移动适配设置 -->
  <meta name="viewport"
            content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no"/>

  <!--2.添加js动态的把body高度设置为，设备屏高-->
  <script>
          function setHeight() {
              document.body.style.height = screen.height + 'px';
          }
  </script>

  <!--3.在body onload 时执行js-->
  <body style="width: 100%;" onload="setHeight()"></body>
   ```

  ​

## 开源库使用时遇到的问题

- 在使用react-weui 发现样式无法生效

  ```js
  import 'react-weui/lib/react-weui.min.css';//导入对应样式文件
  ```

- 在使用fontawesome 时，发现图标无法显示

   ```html
   <!--在html中把样式文件加上-->
   <link rel="stylesheet" href="src/css/fontAwesome/css/font-awesome.min.css">
   ```

- 开发时想通过fetch直接请求服务端接口，解决跨域问题

  ```js
  //package.json 使用webpack-dev-server 启动开发环境，重点是-d
  "scripts":{
    "start": "webpack-dev-server -d --hot --inline --progress --colors --host 0.0.0.0 --port 8080",
  }

  //webpack.config.js 配置代理
    module.exports = {
      //其他配置省略
      devServer:{
          proxy:{
              '/api':{
                  target:'http://localhost:5000',
                  secure: false,
                  changeOrigin: true,
                  pathRewrite: {'^/api': ''}
              }
          },
      },
    }
    
  //使用时代码
    fetch('/api/show_operation/1')
              .then(response=>response.json())
              .then(result=>console.log(result))
    
  //说明：通过proxy配置可以最终发出的请求路径为http://localhost:500/show_operation/1
  ```

  ​

