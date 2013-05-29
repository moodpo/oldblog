# Window 7 安装 Apache 2.4 和 PHP 5.4 过程

- date: 2013-05-28
- category: Work
- tags: apache, php

-------------------

现在使用 UI 向导安装 AMP(Apache,MySQL 和 PHP)已经变得非常容易，比如XAMPP，WAMP。但是使用这些集成的包安装你只能获取很少的知识，对于其配置却一无所知，安装完之后也无法按照自己的方式去修改。因此我推荐手动去安装配置它们。

## 一、下载地址

- Apache 5.4 —— [httpd-2.4.4-win32.zip][1]

- PHP 5.4 ——  [php-5.4.15-Win32-VC9-x86.zip][2]

注意，VC9 线程安全版本中已经包含了 PHP 和 Apache connector DLL，因此无需下载此 DLL。

## 二、配置

### 1. Apache

使用任意编辑器打开 apache2.4/conf/httpd.conf 文件开始配置。

#### 1.1 设置 Apache 位置

````
ServerRoot "D:/Program Files/apache2.4"
````
#### 1.2 启用使用的模块

我只去掉了 mod_rewrite 模块的注释。

#### 1.3 在模块内容下增加以下内容

````
LoadModule php5_module "D:/Program Files/PHP5.4/php5apache2_4.dll"
AddHandler application/x-httpd-php .php
PHPIniDir "D:/Program Files/PHP5.4"
````
#### 1.4 修改服务器管理员邮件地址

ServerAdmin info@yoursite.com

#### 1.5 修改文档根目录

````
DocumentRoot "E:/www"
<Directory "E:/www">
````
#### 1.6 找到一下内容替换实际的路径

````
ScriptAlias /cgi-bin/ "D:/Program Files/apache2.4/cgi-bin/"
<Directory "D:/Program Files/apache2.4/cgi-bin">
````
#### 1.7 如果你想启用 `.htaccess` 请修改 `<Directory “D:/www”>` 下内容

````
AllowOverride All
````
#### 1.8 添加 index.php 到 index 目录中

````
DirectoryIndex index.html index.php
````

### 1. PHP

#### 1.1 重命名 php.ini-development 为 php.ini

#### 1.2 修改扩展路径

````
extension_dir = "D:/Program Files/PHP5.4/ext"
````
#### 1.3 取消以下行的注释

````
extension=php_curl.dll
extension=php_mysql.dll
extension=php_mysqli.dll
extension=php_pdo_mysql.dll
extension=php_soap.dll
````
#### 1.4 如果你使用 PHP 的邮件功能请修改下面内容

````
SMTP = smtp.yoursite.com
smtp_port = 25
sendmail_from = youremail@sender.com
````
#### 1.5 最后设置下时区

````
date.timezone = PRC
````

## 三、安装

需要将 Apache 2.4 的服务安装到系统服务中，使用以下命令(需要管理员权限)：

````
cd D:/Program Files/apache2.4/bin
httpd -k install
````

编写一个 index.php 文件，内容为 `<?php phpinfo() ?>`， 启动apache服务，访问以下 http://localhost/吧

[1]: http://www.apachelounge.com/download/win32/binaries/httpd-2.4.4-win32.zip
[2]: http://windows.php.net/downloads/releases/php-5.4.15-Win32-VC9-x86.zip