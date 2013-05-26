# 基于静态文件的新博客

- date: 2013-05-27
- tags: blog, python, liquidluck
- category: Life

------

周五浏览网页时偶然看到别人用 [Liquidluck][1] 搭建的静态博客站点：利用 [GitHub][2] 的webhook功能进行自动部署，同时支持 Markdown 文件，可将其自动输出为静态文件，此外还支持一些博客常见的功能，如标签、分类、归档等，非常吸引人。

于是，周末两天折腾，部署上之后，自己还做了套主题 [liquidluck-theme-moodpo][3] ，就是现在这个样子。期间遇到各种问题，趁还没忘记录下来。

## 一、安装过程

在安装 Liquidluck 之前需要安装 distribute 和 seuptools ，以便使用 pip 安装 Liquidluck。

````
$ wget http://pypi.python.org/packages/source/d/distribute/distribute-0.6.28.tar.gz

$ tar xvzf distribute-0.6.28.tar.gz

$ cd distribute-0.6.28

$ python setup.py install

````

````
$ wget https://pypi.python.org/packages/2.4/s/setuptools/setuptools-0.6c11-py2.4.egg

$ wget https://raw.github.com/pypa/pip/master/contrib/get-pip.py

$ python get-pip.py

$ pip -U install liquidluck

````

## 二、部署 Liquidluck

部署的过程相对简单，但还是有一些细节 [Liquidluck 文档][4]中没有写：

### 1. 在 GitHub 上建立博客的 repo

使用 webhook 的话一般直接把整个部署目录都放在 hook 范围之内，因此先使用 git 把目录下载下来，当然在此之前你应该在 GitHub 上先建立一个 repo，repo的大致内容如下：

````
blog
	-- content
		-- helloword.md
		-- media
			-- author.jpg
	-- settings.py # 你可以到 Liquidluck 的 GitHub 地址去下载一份 然后改一下相关配置
	-- README.md
````

然后 git 这个 repo:

````
$ git clone git://github.com/user/repo blog

````

### 2. 配置 webhook 

首先在 GitHub 上配置好 webhook，具体操作可以看[这里][4]，然后启动本地 webhook 服务：

````
$ cd blog

$ liquidluck webhook start -p 9876
````
这样就完成了 webhook 的配置，你可以先 push 一个文件到 GitHub 上，看有没有同步过来。

需要注意的一点是，如果你要安装主题，不要再 webhook 的目录内 git ，否则 webhook 功能将失效

### 3. 部署 Liquidluck

部署并启动服务：

````
$ cd blog

$ liquidluck bulid

$ liquidluck server -p 80

$ nohup liquidluck server -p 80 > access.log 2>&1 &             #后台运行

````

关于 Liquidluck 的安装与部署的细节就是这些，更详细的内容请看[官方文档][4]。

[1]: https://github.com/lepture/liquidluck
[2]: https://github.com
[3]: https://github.com/moodpo/liquidluck-theme-moodpo
[4]: http://liquidluck.readthedocs.org/en/latest/