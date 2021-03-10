---
title: supervisor
date: 2017-04-28 11:24:12
index_img:
- /images/shell/002_supervisor_1.jpg
tags: 
- Python
categories:
- 开源工具
---

###  简介

Supervisor 是一个客户端/服务器系统，允许其用户监视和控制类似UNIX的操作系统上的多个进程。说的简单一点就是可以将supervisor 来启动/停止子进程。子进程退出了，supervisor 可以将子进程重新拉起。

### 介绍

具体的安装方式参考[官网](http://supervisord.org/installing.html)

#### 组成

supervisor工具分两个

一个是supervisord，supervisord  是父进程，如果这个进程没有启动整个supervisor是无法使用的。

一个是supervisorctl ，这个是管理的命令工具，管理子进程都是用该工具进行管理。

#### supervisord的配置文件

supervisord的配置文件一般是在 `/etc/supervisord.conf`中，如果不知道配置文件位置，可以在/etc/init.d/supervisord中start函数中查看到。

```shell
start()
{
       echo -n $"Starting $prog: "
       touch $PIDFILE
       daemon $prog_bin -c /etc/supervisord.conf --pidfile $PIDFILE
       sleep 1
       [ -f $PIDFILE ] && success $"$prog startup" || failure $"$prog startup"
       echo
}
```

/etc/supervisord.conf 中可以将supervisord要监听的端口关闭，使得supervisord不占用端口。

```shell
;[inet_http_server]
;port=127.0.0.1:9001
```

/etc/supervisord.conf  也可以配置子进程的配置文件路径，这里默认的是在/etc/supervisord.conf.d/目录下

```shell
[include]
files = /etc/supervisord.conf.d/*.conf
```

#### supervisor子进程的配置文件

子进程的配置文件，位置是在/etc/supervisord.conf中设置的，具体看子进程的配置文件如下

```shell
[program:liveness] # 子进程的标识
directory= /opt/faceid/worker#子进程的工作目录
command= /opt/faceid/worker/start.sh#子进程的启动命令或脚本
autostart=true#是否自启
autorestart=true#是否自动重启
startretries= 10000#重试次数
startsecs=7#启动时间
numprocs= 1 # 启动这个进程的数量，一般不写，
process_name=%(program_name)s_%(process_num)s # 如果numprocs设置了多个，这里就可以区分多个名称liveness:liveness_0
stopasgroup=true
killasgroup=true
stdout_logfile_maxbytes=500MB
stdout_logfile_backups=10
stdout_logfile=/var/log/megvii/%(program_name)s_%(process_num)02d.log#日志路径
```

当然一个配置文件，可以写很多个进程启动配置。格式和上面一样。

#### supervisorctl 命令使用

- supervisorctl status 查看各个进程的状态
- supervisorctl reload 重启supervisor 下的所有进程
- supervisorctl stop name 停止指定进程
- supervisorctl start name 启动指定进程
- supervisorctl restart all  重启supervisor 下的所有进程

### 遇到过的问题

1. 遇到一个情况，明明已经通过supervisorctl stop all把服务全部停了，然后还是发现端口还在被占用，服务还正常提供

   这个是应为supervisord 除了当前用户在用外，还有其他用户在使用。导致这个问题的出现。解决办法是明确supervisord是有多个用户在使用。将其他用户的supervisord停止即可。

2. 给普通用户添加supervisorctl 管理权限

   - 修改supervisor.sock 文件的权限，该文件supervisorctl用XML_RPC和supervisord通信就是通过它进行的。如果不设置的话，supervisorctl也就不能用了

     如果需要给所有用户权限则修改/etc/supervisord.conf中[unix_http_server]
     ;chmod=0700
     去掉;， 修改为chmod=0777

   - 给普通用户添加supervisorctl  执行权限出现supervisorctl 命令不可用

     root用户下执行 **chmod 707 /usr/bin/supervisorctl**

     **chmod -R 707 /usr/lib/python2.7/sit-package**

   - 然后使用supervisorctl status 出现pkg_resources.DistributionNotFound:supervisor==3.2.0错误

     chmod -R 705 /usr/lib/python2.6/site-packages/*.egg

3. 找不到`unix:///tmp/supervisor.sock no such file`

   重启supervisord 服务，service 或者systemctl 命令都行

