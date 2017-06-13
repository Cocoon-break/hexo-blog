---
title: iterm,tmux,vim 的常用快捷键
date: 2017-06-02 13:52:12
tags: 技巧
---

上一篇博客我们搭建了Mac下流畅的终端使用环境，这篇博客我们讲一些常用的快捷键来提高开发效率。废话不多说我们开始吧



## iterm2 快捷

通常我们在使用时，一个窗口总是感觉不够用，这时我们就需要多窗口，或者一个窗口需要多个屏。打开Iterm2，我们按下`command + t`，来新建窗口，通过`commmand + w`来关闭当前窗口，如果我们想要切换不同的窗口时，我们可以通过`command + 方向键` 或者`command +  数字`来切换窗口。`command + q` 退出应用程序

**注：** `command + t`或者`command + w`或者`command +q` 在很多应用都是通用的,比如在chrom 上，这几个快捷键，分别表示新建tab页，关闭tab页，推出chrom。但是chrom 切换tab是使用`command + option +方向键` 这个就和iterm2有区别了。



在链接远程服务器时，我需要在同一窗口，查看不同服务器的状态，或者其他的一些信息。多窗口去查看就太麻烦 了。iterm 的分屏就能满足你的需要了。iterm2在分屏时，可以进行横向分屏和纵向分屏。纵向分屏使用`command + d`横向分屏使用`command + shift +d`，关闭当前分屏的快捷键也是使用`command + w`，切换各个分屏使用的是`command + option + 方向键`，以下是效果图

<!-- more -->

![](https://cocoon-break.github.io/images/screenShot/iterm2_keymap.jpg)



接下来就是在输入的时候，我们需要快速回到行首使用`ctrl + a`，快速回到行末使用`ctrl + e`，输入失误就需要删除错误了，`ctrl + u`是将当前行清空，使用 `ctrl + w`,删除光标之前的单词，使用 `ctrl + h`删除光标之前所有字符，使用`ctrl + k` 删除光标之后的所有字符。



总结下 ：

- 窗口和屏相关
  - 新建窗口：command + t
  - 关闭窗口或者分屏：command + w
  - 切换标签：command +  左右方向键
  - 切换全屏：command + enter
  - 水平分屏：command + shift + d 
  - 垂直分屏：command + d 
  - 切换分屏：command + option + 方向键
- 编辑相关
  - 清除当前行：ctrl + u
  - 删除光标之前的字符：ctrl + h
  - 删除光标之前的单词：ctrl + w
  - 删除光标到文本末尾：ctrl + k
  - 到行首： ctrl + a
  - 到行末：ctrl + e



## tmux 快捷键

 打开iterm2 输入`tmux` 就进入了tmux的交互模式了，tmux 也有窗口和屏的概念这里就不再说明了。直接进入主题，快捷键的使用。

在使用任何功能时我们都得先按下`ctrl + b` 然后**松开**，接着按下其他的键，比如我要新建一个窗口，先按下`ctrl + b`  然后在按下`c`,  这就完成了窗口的新建。

- 快捷键都是先按下`ctrl + b`然后在按一下键
  - c		新建窗口
  - &             关闭当前窗口
  - p              切换至上一个窗口
  - n              切换至下一个窗口
  - %              将当前窗口纵向分屏
  - "                将当前窗口横向分屏
  - 方向键      在多个分屏中切换
  - d                脱离当前tmux，在输入tmux attach 就能重新进入
  - z                使当前屏占满全屏，在次按下则恢复之前屏样式
  - x                关闭当前屏
  - alt + 方向键   调整当前屏大小

这里只将一些基础的快捷键，更多快捷键和配置，请自行谷歌，百度，以下为tmux  分屏之后的效果图

![](https://cocoon-break.github.io/images/screenShot/iterm2_tmux_keymap.jpg)



## vim 快捷键

vim 的功能很强大，这里也不会深入讲解介绍，只是收集了一些快捷键，以便提供一些效率。

- 查找
  - 从光标处向下搜索：/ + 要查找的词
  - 从光标处向上搜索：? + 要查找的词
  - 快速回到页首：{
  - 去页尾：}
  - 回行首：shift + ^
  - 回行末：shift + $
- 编辑
  - 删除光标出的字母：x
  - 删除光标所在行：dd
  - 删除单词包括空格：dw
  - 回撤上一次编辑：u 
  - 取消撤回功能(对u功取消)：control + r
  - 复制光标所在行：yy
  - 复制n(数字)行：nyy
  - 复制单词：yw
  - 复制n个单词：nyw
  - 复制光标所在位置到行末：y$
  - 复制光标所在位置到行首：y^
  - 粘贴：p

更多vim 操作快捷键：[vim快捷键](http://www.lcode.cc/2017/04/10/vim-shortcut-key.html)

这篇博客只是收集和介绍一些基础简单的快捷键，并没有很深入的去使用iterm2 ，tmux，vim，这些功能都很强大，想继续深入的同学可以去查查资料。