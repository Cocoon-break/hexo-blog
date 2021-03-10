---
title: shell 笔记
date: 2017-05-25 11:14:12
index_img:
- /images/shell/004_shell_1.jpg
tags: 
- shell
categories:
- Linux
---

### xargs

一个例子来认识`xargs`这个Linux 自带的命令。一个文件夹下有十万的.log文件和.txt文件。这时候我们要删除所有的.log文件，很多脑子想到的就是`rm -rf *.log`。这时候你会发现报一个错误`Argument list too long`。这时你又不想写一个for 循环来删，我们就可以利用xargs 这个命令来解决。xargs 一般是和管道一起使用也就是这个`|`

```shell
ls *.log | xargs rm -rf 
```

ls 将当前目录下的log文件名标准输出，通过管道传递给xargs 过滤，然后进行删除。

**注：** 遇到过一个问题，Linux上一些以减号开头的文件，会将这个文件认为是一个参数，如：`rm -rf -xxxx.log`是无法执行的，无论是其他的什么命令去处理类似文件名的文件，都会有这个问题。这时候只要在文件名前加两个减号即可`rm -rf -- -xxx.log`

##### 参数

xargs  常用的几个参数也就那几个，其中一个是`-I`设置占位符。具体例子如下

```shell
ls *.log | xargs -I {} cp {}  /var/log/
```

 将log文件一个一个的复制到/var/log/目录下。

xargs还有一个-P的参数，是指定xargs同时处理多少个命令，也就是设置多少个并发。可以加快处理的速度

```shell
ls *.log | xargs -P 8 -i cp {} /var/log/
```

[更多参数使用](https://www.runoob.com/linux/linux-comm-xargs.html)

### awk

awk 是一种处理文本文件的语言，是一个强大的文本分析工具。相对于grep的查找，sed的编辑，awk在其对数据分析并生成报告时，显得尤为强大。awk一般是和管道一起使用。

具体例子我有一个a.csv文件，我想要第一列和第三列的数据生成一个b.csv

a.csv

```shell
a,b,c
d,e,f
h,i,j
```

```shell
cat a.csv | awk -F , '{print $1,$3 }' >> b.csv
```

b.csv

```shell
a,c
d,f
h,j
```

我们将awk 和 xargs统一利用起来。现在我用一个测试程序起了多个进程。想要一次性将测试的这些进程一次kill掉。思路是将进程号输出，然后利用xargs 一个一个进程kill，最后的命令如下

```shell
ps -ef | grep test-awk | awk '{print $2}'| xagrs kill -9 
```

Awk 很强大，功能远不止上面的两个例子。更多的参考[awk命令](https://www.runoob.com/linux/linux-comm-awk.html)

### curl

这个命令基本都很熟悉，这里记录几个实用的样例。

发送post json 数据

```shell
curl -H "Content-Type: application/json" -X POST -d '{"seq":"xxxxxxxx","policy":"free"}' "http://127.0.0.1:8080/reliablePhoto"
```

将报文json 写好在一个a.json文件，然后post，这样就不用每次在终端里拼写报文了

```shell
curl -H "Content-Type: application/json" -X POST --data-binary "@a.json" "http://127.0.0.1:8080/reliablePhoto"
```

发送post form-data数据

```shell
curl -F seq=xxxxxx -F img=@a.jpg http://127.0.0.1:8080/reliableImg
```

Curl 还有更多的参数，具体可以通过`man curl` 来获取帮助

### jq

通常后端开发使用curl 命令，请求的接口返回是json数据，默认的并没有对json 格式化，这时我们可以使用jq来格式化报文，当然也可以进行json数据解析。jq 命令并不是Linux系统自带的，所以我们需要格外安装。[下载的地址](https://stedolan.github.io/jq/download/)

jq的用法也很简单，而且官方的例子也很全。这里主要是记录下有这么一个好用的东西。[ 官方教程](https://stedolan.github.io/jq/tutorial/)

### shell 脚本读取命令行参数

shell 脚本中读取参数值

通常读取命令行中带过来的参数有两种方式，一直是直接根据参数位置直接获取，还有一种根据指定的key，去获取指定的值。第一种方式可以说是根本没有可读性，使用的人如果参数位置写错了，传入的值就全错了

#### 根据参数位置直接获取

1. 先定义一个脚本test.sh，打印位置参数

   ```shell
   #!/bin/bash
echo $1
   echo $2
   ```
   
2. 执行test.sh脚本

   ```shell
   ./test.sh haha hehehe
   ```

   执行的结果就是打印了两个参数，这个种方式的弊端就是，如果参数的位置传错了，那么$1 的值就不是haha了，当然可以使用这种方式写，就是可读性不太高 哈。


#### 指定key 获取参数

1. 一样的我们先定义脚本test.sh，一会儿我们在解读shell脚本中的含义

   ```shell
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