# Ubuntu 13.04 安装后要做的几件事

- date: 2013-06-23
- category: Work
- tags: ubuntu, ATI, eclipse, sublime

----

系统安装已经变得很简单，但是安装完后离在其上开始进行开发却还很远；因为有很多事情还没有做，比如驱动、开发工具设置、系统设置等等，然而正是这些事情让进度卡壳。这次安装 Ubuntu 13.04其实是继续之前，之前安装失败了，重装也没有搞好。现在有点时间了，终于在周末搞的差不多了，先记录先来。

## 一、ATI驱动安装

在 Ubuntu 12.04 之前，一直使用这篇文章的方法安装即可，但是从 Ubuntu 12.10 开始，由于xorg的版本更新，而 ATI 驱动跟进太慢，AMD 最终放弃对 ATI Mobility Radeon HD 2xxx-4xxx系列显卡的支持，于是我就悲催鸟！

我的显卡是 ATI Mobility Radeon HD 4650，按照以前的方法安装失败，显卡不能驱动，风扇呼呼的扇，但温度还是很高。可通过一下命令查看显卡类型：

	lspci -vvnn | grep VGA

运行命令后我的显示内容：

	ping- SERR- FastB2B- DisINTx-
	02:00.0 VGA compatible controller [0300]: Advanced Micro Devices [AMD] nee ATI RV730/M96 [Mobility Radeon HD 4650/5165] [1002:9480] (prog-if 00 [VGA controller])

幸亏有老外开发者专门做了 PPA 源，可以装打好补丁的 fglrx ，运行以下命令：

	sudo add-apt-repository ppa:makson96/fglrx
	sudo apt-get update && sudo apt-get upgrade
	sudo apt-get install fglrx-legacy

## 二、安装 Eclipse 及插件

安装 Eclipse 其实不能说叫安装，应该叫下载解压，从 www.eclipse.org 下载后解压到相应目录就可以运行了，当然需要JDK的支持，可以直接在软件中心安装 OpenJDK 。但是如果使用 Unity 环境就需要修复全局菜单bug，以及配置下 Unity 上的快捷方式。

### 1. 全局菜单配置

#### 1.1 备份要修改的文件

	sudo cp /usr/lib/gtk-2.0/2.10.0/menuproxies/libappmenu.so /usr/lib/gtk-2.0/2.10.0/menuproxies/libappmenu.so.bak

Ubuntu 13.04 中目录变更为以下：
	
	# 64位操作系统
	/usr/lib/x86_64-linux-gnu/gtk-2.0/2.10.0/menuproxies/libappmenu.so
	# 32位操作系统
	/usr/lib/i386-linux-gnu/gtk-2.0/2.10.0/menuproxies/libappmenu.so

#### 1.2 用vim打开文件并修改
	
	sudo vim /usr/lib/gtk-2.0/2.10.0/menuproxies/libappmenu.so
	# 搜索Eclipse并修改为Xclipse
	/Eclipse
	rX
	# 保存并退出
	ZZ

#### 1.3 重新载入LD配置(可选)
	
	sudo ldconfig

重新打开 Eclipse ，看是不是菜单已变成全局的了^_^

### 2. 创建 Unity 快捷图标

	sudo gedit /usr/share/applications/eclipse.desktop

添加以下内容：

	#!/usr/bin/env xdg-open

	[Desktop Entry]

	Version=1.0
	Name=Eclipse
	GenericName=Integrated Development Application
	Comment=Eclipse Juno
	Exec=/home/username/[path]/eclipse
	TryExec=/home/username/[path]/eclipse
	Icon=eclipse
	Terminal=false
	Type=Application
	Categories=Development;IDE;

注意，Icon的路径无需指向刚下载解压的 Eclipse 里的图标，使用系统自己的即可。

好了，保存之后看一下 HUD 中是不是已经有了 Eclipse 了。

### 3. 修改 Eclipse 的外观

Eclipse 在 Win 下比较美观，然而在 Ubuntu 下可完全走了样，尤其是 Juno 版本，在 Ubuntu 和 Eclipse 论坛上一片 "Very ugly!" 之声。那巨大的 Tab,撑烂的 Toolbar，间距超大的的 TreeView 简直让人难以忍受。下面就带你一步步改变这些状况，拯救程序员本来就可怜的眼睛。

之所以会有上述糟糕的表现，不是 Eclipse 的问题，是因为 GTK 2.0 中的控件间距太大原因导致的，而谷歌搜索的大部分解决方法也正是着力于修改 GTK 2.0 控件的表现形式，重新定义一些样式。大多数方法是在 `/home/username/` 下新建 `.gtkrc-2.0` 文件，并在其中添加CSS。然而这种方法会把系统全局的样式都改变，比如 Chrome 的，甚至系统的菜单。哪有没有一种方法只修改 Eclipse 的呢？答案是肯定的！我们可以专门在启动 Eclipse 时指定 GTK 2.0 使用的 gtkrc 文件，而别的应用直接启动就不会受影响了。

自己定义实在是麻烦，好在已经有人为我们写好了，链接在[这里][4]。你可以把它 git clone 到一个目录，然后修改，eclicp 脚本，比如我把它放在了 Public 下，eclipse 脚本内容如下：

	#!/bin/bash

	myPATH="$(dirname $0)"
	PATH=${PATH/$myPATH/}
	eclipse=/home/xiaoxie/Work/eclipse/eclipse
	GTK2_RC_FILES=$HOME/Public/clearlooks.gtkrc exec $eclipse

你也可以修改 clearlooks.gtkrc 文件的内容，默认定义的字体实在太小，可以试着改大些，内容无非就是一些 CSS 样式，这里就不多说了。

此外我还想推荐一个 Eclipse 主题 —— [Eclipse Chrome theme][1],用起来实在是很爽！直接在 Eclipse Marketplace 里搜索安装即可。

ToolTip 背景色的问题直接安装 gnome-color 工具在里面修改即可。

最后展示下我的 Eclipse 最终效果吧！

![Eclipse截屏][2]

### 4. 安装 EGit 插件

安装 git :

	sudo apt-get install git

JavaEE 的项目在 GitHub 上，这个插件就是必须的。可以直接在 Eclipse Marketplace 里搜索安装，非常方便。

启用 Package Control 功能的方法（Packages/ 目录在 ～/.config/sublime-text-3/Packages/ 下）：

	cd Packages/
	git clone https://github.com/wbond/sublime_package_control.git "Package Control"
	cd "Package Control"
	git checkout python3

重启后 Ctrl + Shift + p 如果出现“没有可用包”的情况，那估计是伟大的天朝的问题。解决方法有两种，一种是修改 Preferences -> Package setting -> Package Control -> Setting-Default 文件中的 channels 属性，将 https 改为 http；另一种方法是直接在 Setting-Default 文件中配置代理。

还有很多地址可能被墙，可以使用简单有效的方法——修改 hosts ，这里是 hosts [更新列表地址][3]，将内容添加到 /etc/hosts文件中即可。

[1]: https://github.com/jeeeyul/eclipse-themes
[2]: /media/2013/06/002.png "Eclipse 截屏"
[3]: https://smarthosts.googlecode.com/svn/trunk/hosts
[4]: https://github.com/nailgun/eclipse-gtkrc
