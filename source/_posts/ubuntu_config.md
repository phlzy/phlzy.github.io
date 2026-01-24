---
title: Configuration of Ubuntu
date: 2021-05-09
tag: [ubuntu]
category: [Tutorial]
math: true
---

Ubuntu 配置

<!--more-->

# 准备工作

首先装好系统，记得在**软件和更新**里换源。

然后装 Chrome，搜狗拼音输入法，Clash，都照着教程来就好了。

这三个是一定要先装好的。

# 个性化

先安装 Gnome-Tweak-Tool：

``` shell
sudo apt-get update
sudo apt-get install gnome-tweak-tool
```

然后打开，可以在桌面一栏选择是否显示图标，在窗口一栏选择按钮位置等。我还是习惯默认的设置。

这时外观栏中的 Shell 应该是灰色的，需要先装一个扩展：

``` shell
sudo apt-get install gnome-shell-extensions
```

然后重新打开，在扩展栏中打开 User themes 按钮。

理论上 Shell 那边已经好了，但我实际操作中还是灰色的。这时需要再装一个浏览器扩展：

```shell
sudo apt install chrome-gnome-shell
```

然后 Chrome 打开 [这个网址](https://extensions.gnome.org/extension/19/user-themes/)，启用插件。再重新开一遍 Gnome-Tweak-Tool 就好了。

接下来就可以装主题了。

我这次装的是 [这个](https://www.opendesktop.org/s/Gnome/p/1241688)，进去在 File 标签下面下载压缩文件，通过以下两个命令解压：

``` shell
xz -d filename.tar.xz
tar xvf filename.tar
```

就会获得一个文件夹。在下载文件夹中打开终端，`sudo mv filename /usr/share/themes` 就好了。

现在打开 Tweaks 就可以选择刚才装好的主题。

图标的修改方法类似，[网址](https://www.opendesktop.org/s/Gnome/p/1102582/)，解压后放到 `/usr/share/icons` 里面。

修改 Shell：[网址](https://www.opendesktop.org/s/Gnome/p/1013741/)，解压后放到 `/usr/share/themes` 里面。

再去 Ubuntu 软件里安装 Dash to Dock 和 Coverflow Alt-Tab，在扩展里打开相应的开关，自定义设置即可。

[更深入的美化教程](https://www.cnblogs.com/feipeng8848/p/8970556.html)

# 软件安装

# 小工具

### caffeine

阻止休眠的小插件，非常好用

```shell
sudo add-apt-repository ppa:eugenesan/ppa
sudo apt-get update
sudo apt-get install caffeine -y
```

### copyq

评价很好的剪贴板管理软件

```shell
sudo add-apt-repository ppa:hluk/copyq 
sudo apt-get update
sudo apt-get install copyq
```

### neofetch

看系统状态

```shell
sudo add-apt-repository ppa:dawidd0811/neofetch 
sudo apt-get update 
sudo apt-get install neofetch
```

装完执行 `neofetch` 命令可以看到很多东西。当然还有更高级的用法。

### shutter

截图软件

```shell
sudo apt-get install shutter
```

应该可以设置快捷键，懒得搞了。

### simplescreenrecorder

录屏软件

```shell
sudo apt-get install simplescreenrecorder
```

### VLC

播放器，先要装一个 snap

```shell
sudo apt install snapd
sudo snap install vlc
```

### filezilla

FTP 客户端

```shell
sudo apt-get install filezilla
# sudo apt-get install filezilla-locales
```

## Git

```shell
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install git
```

## typora

打开 [官网](https://www.typora.io/) 照教程来就行了。

## vscode

去官网直接下载会非常慢并且很容易失败，找到下载链接后将网址：

```
https://az764295.vo.msecnd.net/stable/cfa2e218100323074ac1948c885448fdf4de2a7f/code_1.56.0-1620166262_amd64.deb
```

改成这样：

```
http://vscode.cdn.azure.cn/stable/cfa2e218100323074ac1948c885448fdf4de2a7f/code_1.56.0-1620166262_amd64.deb
```

就解决了。

**插件**：

- Chinese
- One Dark Pro
- C/C++
- C++ Intellisense
- Code Runner 记得开 run in terminal
- Bracket Pair Colorizer 2 
- Python
- TabOut

最主要的就这些。字体装一下 JetBrains Mono，大小 20

```
'JetBrains Mono', 'monospace', monospace, 'JetBrains Mono'
```

## shell

zsh、fish 之类的都很不错，等以后学会了再写。

# 环境配置

## C/C++

先看看有没有：

```shell
gcc --version
g++ --version
```

很遗憾，并没有：

```shell
sudo apt-get install gcc
sudo apt-get install g++
```

现在装的两个都是 7.5.0 版本的。

## Python

python3 已经有了，没有的话 `sudo apt install python3`。

## LaTex

有空再装。