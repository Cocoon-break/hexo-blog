---
title: shell 脚本中读取参数值
date: 2017-05-25 11:14:12
tags: Linux
---

shell 脚本中读取参数值

通常读取命令行中带过来的参数有两种方式，一直是直接根据参数位置直接获取，还有一种根据指定的key，去获取指定的值。第一种方式可以说是根本没有可读性，使用的人如果参数位置写错了，传入的值就全错了

## 根据参数位置直接获取

1. 先定义一个脚本test.sh，打印位置参数

   ```ssh
   #!/bin/bash

   echo $1
   echo $2
   ```

2. 执行test.sh脚本

   ```ssh
   ./test.sh haha hehehe
   ```

   执行的结果就是打印了两个参数，这个种方式的弊端就是，如果参数的位置传错了，那么$1 的值就不是haha了，当然可以使用这种方式写，就是可读性不太高 哈。

   <!-- more -->

## 指定key 获取参数

1. 一样的我们先定义脚本test.sh，一会儿我们在解读shell脚本中的含义

   ```ssh
   #!/bin/bash

   #先定义接收各个参数值的变量
   IP_VALUE=""; NODE_VALUE="";VIP_VALUE="";

   while true;do 
   	if [ ! $2 ]; then break; fi
   	case $1 in
   		-i|--ip)
   			case $2 in
   			"")IP_VALUE="";shift 2;;
   			*)IP_VALUE=$2;shift 2;;
   			esac;;
   		-n|--node)
   			case $2 in
   			"")NODE_VALUE="";shift 2;;
   			*)NODE_VALUE=$2;shift 2;;
   			esac;;
   		-v|--vip)
   			case $2 in
   			"")NODE_VALUE="";shift 2;;
   			*)NODE_VALUE=$2;shift 2;;
   			esac;;
   		 --) shift ;break ;;
   		 *) break;;
   		esac
   done

   if [ -z "${IP_VALUE}" ];    then echo "error: ip is not found.";             exit 1; fi
   if [ -z "${NODE_VALUE}" ];  then echo "error: node is not found.";           exit 1; fi
   if [ -z "${VIP_VALUE}" ];   then echo "error: vip is not found.";            exit 1; fi
   ```

   - 通过while 循环参数的$1 和$2 取key 和 value。
   - case $1取参数的key，case $2取其中的值，如果取到了，就通过shift 2，把最前面的两个位置给移除了比如:`./test.sh -v kkk -n bbbb` 读取到如果执行了 shift 2,它就会把-v kkk 给扔了
   - 底下的if 是判断有没有读取到想要的值

2. 其他的方式去根据key获取参数

   - 一个是getopts 但是getopts不支持长选项，具体可以参考[getopts命令行参数处理](http://www.cnblogs.com/xiangzi888/archive/2012/04/03/2430736.html)
   - 另一个是getopt，[shell中使用getopts 和 getopt](http://blog.csdn.net/wh211212/article/details/53750366)

个人建议使用第一种方式去处理shell参数，不会有太多的局限性，唯一的问题就是代码会多一些哈！