---
title: xargs 和 jq
date: 2017-07-05 13:45:12
tags: Linux
---

​	xargs是Linux系统上自带的命令，xargs命令是给其他命令传递参数的一个过滤器，也是组合多个命令的一个工具。jq 不是Linux系统自带的命令，需要自行安装。jq主要在Linux上解析json。

## xargs

​	举一个例子，我们常用的一个命令`rm -rf *.log` 如果文件log文件太多我们会得到一个异常的信/bin/rm Argument list too long，也就是说rm能接受的参数是有限的。这时我们就可以利用xargs 过滤。

```sh
ls *.log | xargs rm -rf 
```

ls 将当前目录下的log文件名标准输出，通过管道传递给xargs 过滤，然后进行删除。

**题外话：** Linux上一些以减号开头的文件，会将这个文件认为是一个参数，如：`rm -rf -xxxx.log`是无法执行的，无论是其他的什么命令去处理类似文件名的文件，都会有这个问题。可以通过在文件名前加两个减号解决。

如`rm -rf -- -xxxx.log`

​	xargs 的具体用法，还是一个文件夹中有一大堆文件，通过`ls`可以将文件多行输出。但是我想单行或者指定行数输出呢！通过给xargs指定-n 参数。

```sh
ls *.log | xargs -n2 #-n2 就是指定两行输出，-n3就是三行输出，以此类推
```

前面都是一些简易的操作，感觉不到xargs 到底是怎么处理管道出来的文件。现在我的一个需求是将所有的log文件复制到指定目录下。普通复制命令`cp xxx.log /var/log`。我们先忽略cp 命令有-r参数。先看看以下写法是否正确

<!-- more -->

```sh
ls *.log | xargs cp /var/log
```

你一看立马就觉得不对，cp 不是接受两个参数吗？一个文件，一个目标文件或路径。可是又不知道怎么去解决这个事。在我们上面的例子都真正没有去理解xargs，如`ls *.log | xargs rm -rf ` 为什么能好使？ls出来的文件通过管道传递给xargs 过滤，然后xargs 过滤出的文件默认追加到命令行的最后。但是在这个例子中，我们不想将xargs吐出来的放到命令行的最后。xargs的占位符就发挥作用了，将上面的命令修改

 ```sh
ls *.log | xargs -I {} cp {}  /var/log/
 ```

通过`-I`参数指定大括号为占位符，cp后的大括号就是xargs 吐出来的文件名。这里不一定使用大括号，也可指定其他的符号。`-i`默认的占位符就是大括号

```sh
ls *.log | xargs -i cp {} /var/log/
```

xargs还有一个-P的参数，是指定xargs同时处理多少个命令。但是感觉没什么效果。例子`ls *.log | xargs -P 8 -i cp {} /var/log/`

xargs 的使用就讲到这里

## jq

jq 命令并不是Linux系统自带的，所以我们需要格外安装。[下载的地址](https://stedolan.github.io/jq/download/)

jq 的官方说法是jq is like `sed` for JSON data - you can use it to slice and filter and map and transform structured data with the same ease that `sed`, `awk`, `grep` and friends let you play with text。使用jq在Linux系统上去处理json是非常方便的。比如在终端上curl一个接口，返回一个json，这个json可能很长，我只想看其中部分数据，就可以用上jq了。

jq的用法也很简单，而且官方的例子也很全。这里主要是记录下有这么一个好用的东西。哈哈哈

[ 官方教程](https://stedolan.github.io/jq/tutorial/)



## 自己实践的例子

前段时间有需要将两万张的照片去发起请求，然后处理curl返回的结果。利用了xargs和jq。将命令贴出来记录一下。

```sh
ls TP | xargs -i -n1 curl -s -F img=@TP/{} http://127.0.0.1:9001/faceid/v1/liveness_cfg                                                                 | jq -c -r '[.liveness.mask,.liveness.replay,.liveness.graphics]' > TP.txt &
```

TP就是有两万张照片的文件夹，将请求的结果通过jq处理，然后输出到TP.txt文件 中