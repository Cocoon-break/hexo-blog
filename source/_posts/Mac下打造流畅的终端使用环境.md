---
title: Mac下打造流畅的终端使用环境
date: 2017-05-27 17:02:12
tags: 技巧
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






## 安装oh-my-zsh

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

4. oh-my-zsh 就配置完了

## 安装tmux

1. 先安装Homebrew，这个是Mac平台的包管理器。用来安装一些开发工具还是很方便。Mac系统自带了ruby的环境，我们通过ruby来安装Homebrew

   ```sh
   /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
   ```

2. 通过homebrew 安装oh-my-zsh

   ```sh
   brew install tmux
   ```

3. 就是这么简单tmux就装完了



下一篇会讲具体的使用方法，包括iterm快捷键，tmux的使用