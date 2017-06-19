# 公众号web开发过程记录

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