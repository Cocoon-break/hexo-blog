---
title: Mac下打造流畅的终端使用环境
date: 2017-05-27 17:02:12
tags: 
- 开发效率
categories:
- 开发效率
---

作为程序员Mac上自带的终端是远远满足不了我们的，这时我们就要自己手动搭一个我们顺手的终端环境。下面我们就来搭建iterm2+ oh my zsh + tmux  的终端工具

## ITerm2 安装

1. 上Iterm2的官网下载iterm2,[iterm2下载](http://www.iterm2.com/)

2. 下载完成之后，发现iterm2 自带的配色我不太喜欢，安装完成之后进行item2的配色，这个纯属看个人喜好了。

   - 要进行配色，我们当然得要有配色的方案啦！什么你要自己配一个，嗯...... 这也是可以的，不过有很多现成的方案你要不要啊。github 上有超多的配色方案，我们先把这个库给clone下来。

     ```sh
     git clone git@github.com:mbadolato/iTerm2-Color-Schemes.git
     ```

     要是你没有git，那我觉得你没太必要继续下去了。当然你还想继续的话，你就上GitHub手动download 下来。

     <!-- more -->

   - 好了配色方案是有了接下来我们就是进行配色了,这个也是比较简单，打开iterm2的偏好设置，找到Profiles下的Colors，选中import，这里选择刚才git 克隆下来文件中的schemes文件夹下的文件，这些文件就是各种各样的配色方案，这就看个人喜好了哈，这样你的iterm2配色方案就完成了

     ![](https://cocoon-break.github.io/images/screenShot/iterm2_color.jpg)






## Mac下shell介绍

Shell 是LInux/Unix的一个外壳，你理解成衣服也行。它负责外界与Linux内核的交互，接收用户或其他应用程序的命令，然后把这些命令转化成内核能理解的语言，传给内核，内核是真正干活的，干完之后在把结果返回给用户或应用程序。



Linux/Unix 提供了很多种shell，常用的shell有这么几种，sh、bash、csh等。想知道系统有几种shell，可以通过以下命令查看。

```sh
cat /etc/shells
```



在 Linux 里执行这个命令和 Mac 略有不同，你会发现 Mac 多了一个 zsh，也就是说 OS X 系统预装了个 zsh，目前常用的 Linux 系统和 OS X 系统的默认 Shell 都是 bash，但是真正强大的 Shell 是深藏不露的 zsh，这货绝对是马车中的跑车，跑车中的飞行车，史称『终极 Shell』，但是由于配置过于复杂，所以初期无人问津，很多人跑过来看看 zsh 的配置指南，什么都不说转身就走了。直到有一天，国外有个穷极无聊的程序员开发出了一个能够让你快速上手的zsh项目，叫做「oh my zsh」



### 安装使用on-my-zsh

1. 通过git 把oh-my-zsh 下载下来

   ```sh
   git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
   ```

2. 添加配置文件并设置为默认的shell

   ```sh
   cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
   chsh -s /bin/zsh
   ```

   **注:** .zshrc 就相当于.bashrc了以后配置环境就在.zshrc中配置就行

3. 配置oh-my-zsh 的主题

   oh-my-zsh 提供了很多的主题，可以选择自己喜欢的风格。具体的主题效果可以参考[主题预览](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes)，选好主题了，接下来就是配置了。编辑~/.zshrc 文件

   ```sh
   vim ~/.zshrc
   #编辑主题，af-magic是我机器上的主题风格，具体可以设置为刚才主题预览中的
   ZSH_THEME="af-magic"
   ```

4. oh-my-zsh 就配置完了。这里只是简单的介绍oh-my-zsh。oh-my-zsh  功能还是很丰富的，更多oh-my-zsh请移步[oh-my-zsh GitHub地址](https://github.com/robbyrussell/oh-my-zsh)

## 安装tmux

Tmux 是一个工具，用于在一个终端窗口中运行多个终端会话。不仅如此，你还可以通过 Tmux 使终端会话运行于后台或是按需接入、断开会话，这个功能非常实用。

1. 先安装Homebrew，这个是Mac平台的包管理器。用来安装一些开发工具还是很方便。Mac系统自带了ruby的环境，我们通过ruby来安装Homebrew

   ```sh
   /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
   ```

2. 通过homebrew 安装tmux

   ```sh
   brew install tmux
   ```

3. 就是这么简单tmux就装完了



下一篇会讲具体的使用方法，包括iterm快捷键，tmux的使用