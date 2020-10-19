---
title: caddy的简易使用
date: 2017-06-30 15:47:20
tags: 
- Linux
categories:
- 开源工具
---

​	caddy是一个开源的，使用golang 编写，支持HTTPS的web服务器，同时也可以作为负载均衡器使用，caddy的使用非常简单，只要一个二进制包就可以执行，不用像nginx 配置起来那么麻烦。

​	 [caddy下载](https://caddyserver.com/download)

​	caddy可以使用命令行参数的方式去执行，也可以指定配置文件去执行。使用命令行去执行，具体的参数名称参考 [caddy 接收参数](https://caddyserver.com/docs/cli)

​        caddy在某个文件夹下直接执行，通过访问http://localhost:2015访问文件下的所有内容。使用起来非常简单。 

### 使用

 通过配置文件来使用caddy，执行是`caddy -conf caddyfile`,  给caddy 配置不同的caddyfile如下

- 使用caddy 负载均衡，配置caddyfile

  ```sh
  :1234
  proxy / ip1:port1,ip2:port2,ip3:port3{
    policy random
    health_check /version
    health_check_interval 30s
    health_check_timeout 30s
  }
  log ／ /var/log/caddy.log {
  	rotate_size    100
  	rotate_age      7
  	rotate_keep     5
  }
  ```

  注：caddy监听本机`1234`端口，分别将请求转发给`ip1:port1,ip2:port2,ip3:port3` 三台机器，policy为负载均衡的策略，random表示从三台机器上随机选择负载，也有其他选项least_conn, round_robin, first, ip_hash, or uri_hash。`health_check`是对三台机器提供的服务进行健康检查，/version 为进行健康检查的接口。每30秒进行一次健康检查，每次检查超过30秒表示失败。如果发现这个接口不能正常返回，则回被caddy标识为不健康的服务，则后续的负载不会到不健康的服务上。配置log记录caddy日志信息

<!-- more -->

- 使用caddy作为静态资源服务器，配置caddyfile

  ```sh
  :1234
  root /opt/test
  browse
  ext .pdf .html .htm 
  ```

   注：caddy监听本机的`1234`端口，指定了访问的根目录为/opt/test,如果/opt/test 目录下没有index.html,那么访问`http://ip:1234` 会将/opt/test目录结构展现到页面中。ext 表示可以忽略文件的后缀

  

更多的配置可以访问 [caddy配置文件](https://caddyserver.com/docs/http-caddyfile)