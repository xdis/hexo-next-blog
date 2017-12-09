---
title: Ubuntu配置完全教程
copyright: true
date: 2017-12-09 13:36:30
tags: [Ubuntu, 美化]
categories: Ubuntu
description: 最近将旧电脑换成了Ubuntu系统，在网上找了许多优化和配置教程，今天整理一份完整的教程给大家分享
---

# 前言

最近将旧电脑换成了Ubuntu系统，在网上找了许多优化和配置教程，今天整理一份完整的教程给大家分享

<!--more-->

# 系统清理

## 卸载LibreOffice

libreoffice事ubuntu自带的开源office软件，体验效果不如windows上的office，于是选择用WPS来替代（wps的安装后面会提到）

```bash
sudo apt-get remove libreoffice-common
```



## 删除Amazon的链接

```bash
sudo apt-get remove unity-webapps-common
```



## 删除不常用的软件

```bash
sudo apt-get remove thunderbird totem rhythmbox empathy brasero simple-scan gnome-mahjongg aisleriot
sudo apt-get remove gnome-mines cheese transmission-common gnome-orca webbrowser-app gnome-sudoku  landscape-client-ui-install
sudo apt-get remove onboard deja-dup
```



# 系统优化

## 切换软件源

在设置--软件和更新里--下载自--其他站点--中国--http://mirrors.aliyun.com/ubuntu



## 将所有软件源和软件更新到最新

```bash
sudo apt-get update
sudo apt-get upgrade
```



# 安装软件

## 安装GDebi

```bash
sudo apt-get install gdebi
```
安装完以后再安装ded包就可以右键打开方式--gdebi



## WPS

[WPS](http://community.wps.cn/download/) 官网下载即可



## 搜狗输入法

[搜狗拼音官网](https://pinyin.sogou.com/linux/?r=pinyin)下载安装。

在系统设置->语言中选择**fcitx**后重启即可使用搜狗拼音



## 网易云音乐

[网易云音乐官网](http://music.163.com/#/download)下载安装即可



## VIM编辑器

```bash
sudo apt-get install vim
```



## GIT

```shell
sudo apt-get intsall git
```

安装完成后进行GIT的设置

```shell
git config --global user.name "youname" # 设置GIT的账号
git config --global user.email "youeamil@email.com" # 设置GIT的邮箱
```

在设置完成后进行GIT的SSH设置

```shell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

在本机生成SSH密匙后将生成的密匙添加到GITHUB上

```shell
sudo apt-get install xclip
xclip -sel clip < ~/.ssh/id_rsa.pub
# 进入GITHUB密匙添加页进行密匙添加
```

最后测试是否SSH可以链接成功

```shell
ssh -T git@github.com
```

如果出现以下文字，代表操作成功

> Hi username! You've successfully authenticated, but GitHub does not
> provide shell access.



## Typora

```shell
# optional, but recommended
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE
# add Typora's repository
sudo add-apt-repository 'deb http://typora.io linux/'
sudo apt-get update
# install typora
sudo apt-get install typora
```



## Sublime Text 3

```shell
sudo add-apt-repository ppa:webupd8team/sublime-text-3
sudo apt-get update      
sudo apt-get install sublime-text
```

安装完后进行输入注册码进行激活

> —– BEGIN LICENSE —– 
> TwitterInc 
> 200 User License 
> EA7E-890007 
> 1D77F72E 390CDD93 4DCBA022 FAF60790 
> 61AA12C0 A37081C5 D0316412 4584D136 
> 94D7F7D4 95BC8C1C 527DA828 560BB037 
> D1EDDD8C AE7B379F 50C9D69D B35179EF 
> 2FE898C4 8E4277A8 555CE714 E1FB0E43 
> D5D52613 C3D12E98 BC49967F 7652EED2 
> 9D2D2E61 67610860 6D338B72 5CF95C69 
> E36B85CC 84991F19 7575D828 470A92AB 
>
> —— END LICENSE ——

Sublime插件推荐

> Package Control  功能：安装包管理
>
> Emmet			 功能：编码快捷键
>
> JSFormat		 功能：Javascript的代码格式化插件
>
> LESS			 功能：LESS高亮插件
>
> Less2CSS		 功能：编译Less
>
> Alignment		 功能：”=”号对齐
>
> sublime-autoprefixer	功能：CSS添加私有前缀
>
> Clipboard History	功能：粘贴板历史记录
>
> Bracket Highlighter	功能：代码匹配
>
> Git					功能：git管理
>
> jQuery			 功能：jQ函数提示
>
> DocBlockr		 功能：生成优美注释
>
> ColorPicker		 功能：调色板
>
> ConvertToUTF8	 功能：文件转码成utf-8
>
> AutoFileName	功能：快捷输入文件名
>
> Nodejs			功能：node代码提示
>
> Trailing spaces	功能：检测并一键去除代码中多余的空格
>
> FileDiffs		功能：强大的比较代码不同工具
>
> GBK Encoding Support 功能：中文识别
>
> All Autocomplete	搜索所有打开的文件来寻找匹配的提示词。
>
> SublimeCodeIntel	全功能的 Sublime Text 代码自动完成引擎 
>
> CTags			方法跳转
>
> Autoprefixer 	自动分析你的css文件，解析出新的css文件，可以配置你要兼容的浏览器，不过这个插件要在之前安装nodejs
>
> BracketHighlighter	配置文件的高亮设置，让你的代码有不同的颜色区分该插件提供配对标签，或大括号或字符引号的配对高亮显示，
>
> BufferScroll	你可以轻松书写一个文件多个位置了
>
> ChineseLocalization	语言包
>
> Color Highlighter  颜色功能还是很爽的，找了好久
>
> CSS Comments
>
> CSS Format
>
> CSS3
>
> HTML-CSS-JS Prettify
>
> JavaScript Completions
>
> Pretty JSON		格式化json
>
> SideBarEnhancements	 增强右键菜单文件操作功能
>
> SublimeLinter	代码校验插件，支持多种语言，这个是主插件，如果想检测特定的文件需要单独下载
>
> SublimeLinter-jshint	这个就是单独的插件，上面的一个分支
>
> SublimeTmpl	　创建常用文件初始模板，必须html,css,js模板
>
> Tag 		HTML/XML标签缩进、补全和校验
>
> Themr



# 主题美化



## unity-tweak-tool

```shell
sudo apt-get install unity-tweak-tool 
```



## Flatabulous主题

Flatabulous主题是一款ubuntu下扁平化主题

执行以下命令安装Flatabulous主题

```shell
sudo add-apt-repository ppa:noobslab/themes
sudo apt-get update
sudo apt-get install flatabulous-theme
```

该主题有配套的图标

```shell
sudo add-apt-repository ppa:noobslab/icons
sudo apt-get update
sudo apt-get install ultra-flat-icons
```

安装完成后，打开unity-tweak-tool软件，修改主题和图标

进入Theme，修改为Flatabulous

此界面下进入Icons栏，修改为Ultra-flat



## 终端

终端采用zsh和oh-my-zsh

首先，安装zsh

```shell
sudo apt-get install zsh
```

接下来我们需要下载 oh-my-zsh 项目来帮我们配置 zsh，采用wget安装

```shell
sudo wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh
```

zsh 基本已经配置完成,你需要一行命令就可以切换到 zsh 模式

```shell
chsh -s /usr/local/bin/zsh
```

如果显示无效，则可以

```shell
vi ~/.bashrc
# 在文件末尾加上bash -c zsh
```



# 开发环境配置

## NodeJS

在Node官网下载最新的稳定版并解压到一个文件夹

之后将其移动到通用的软件安装目录

```shell
sudo mv node-v4.4.4-linux-x64 /opt/
```

创建软链接npm 和 node 命令到系统命令 

```shell
sudo ln -s /opt/node-v4.4.4-linux-x64/bin/node /usr/local/bin/node
sudo ln -s /opt/node-v4.4.4-linux-x64/bin/npm /usr/local/bin/npm
```

### CNPM安装

```shell
npm install -g cnpm --registry=https://registry.npm.taobao.org
sudo ln -s /opt/node-v4.4.4-linux-x64/bin/cnpm /usr/local/bin/cnpm
```

### YARN安装

```shell
npm install -g yarn
sudo ln -s /opt/node-v4.4.4-linux-x64/bin/yarn /usr/local/bin/yarn
```



## JAVA

去官网下载JDK解压到文件夹中，并将其移动到/opt/下

接着配置JAVA的环境变量

```shell
sudo gedit  /etc/profile 打开 /etc/profile
```

然后在文件尾加上

```shell
export JAVA_HOME=/opt/jdk1.8.0_45
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
```

然后刷新环境变量

```shell
sudo source /etc/profile
```



## Pycharm

以Pycharm安装为例

首先先去官网下载最新的Pycharm

下载完成后解压并移动到/opt/下

最后为其创建快捷方式

```shell
cd /usr/share/applications/
sudo vim Pycharm.desktop
```

这里必须得用root权限sudo才能写入，然后在文件中写入以下内容

```shell
[Desktop Entry]
Type=Application
Name=Pycharm
GenericName=Pycharm3
Comment=Pycharm3:The Python IDE
Exec=sh /opt/pycharm/bin/pycharm.sh
Icon=/opt/pycharm/bin/pycharm.png
Terminal=pycharm
Categories=Pycharm
```

接着在将创建的快捷方式拖动到侧边栏即可