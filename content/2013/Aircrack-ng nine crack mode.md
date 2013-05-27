# aircrack-ng的九种攻击模式的介绍

- date: 2013-02-04
- category: Work
- tage: twitter, weibo

----

来源：[链接][1]

因为blogspot被墙，这篇译文在国内的朋友想看到就比较麻烦了，所以特意转来。

Aircrack-ng套装中，aireplay-ng模块提供了9种攻击模式，下面是对这9种攻击模式的介绍。

### 1.Attack 0: Deauthentication （解除认证）

该攻击会向指定AP发送结束认证包将一个或多个客户端与AP断开，断开客户端有如下的好处

*   找到隐藏的ESSID，这是个未开始广播的的ESSID
*   在强制客户端握手的时候捕获WPA/WPA2握手数据
*   生成ARP请求（windows客户端有时候会在断开连接时清空自己的ARP缓存）

### 2.Attack 1: Fake authentication（伪装客户端连接）

伪装客户端连接允许你执行两类WEP认证（开放系统和共享密钥），这对在需要连接到一个没有任何客户端连接的AP时是非常有用的。需要注意的是伪装客户端连接不产生任何ARP包，也不能用来认证和关联WPA/WPA2 AP。

### 3.Attack 2: Interactive packet replay（交互发包）

这个攻击允许注入一个特殊的包，可以从两个来源获取包，一个是从无线网卡获取包，另一个是从一个pcap文件。很多开源工具都可以抓取Pcap格式的文件，一般的做法是用packetforge-ng命令创建。

如果交互包发送成功，就意味着AP接受了交互包，并且生成一个新的inititialization vectory (初始向量，简称IV)。 为了实现这个功能，我们需要选择一个包，在这之前需要先弄清楚两个概念：

无线网络的广播MAC地址是FF:FF:FF:FF:FF:FF，当AP收到包含FF:FF:FF:FF:FF:FF特征的ARP请求时会循环发包，需要注意的是，这个ARP请求包的To DS标志必须设置成1

所以airreplay-ng过滤选项： -b：无线AP的MAC地址。  
-d：广播的MAC地址，也就是FF:FF:FF:FF:FF:FF。  
-t：1表示设置To Distribution System标志，0表示不设置To Distribution System标志。

为了使得注入包看上去更像是个原生的包，我们需要了解下面几个选项：  
-p 0841：设置祯控制字段，使得包看上去像是从客户端发到AP的一样，一定要把 "To DS"设置成1。  
-c FF:FF:FF:FF:FF:FF：设置广播的MAC地址，这会让AP重新发送包并且得到一个新的IV。 

### 4.Attack 3: ARP request replay attack(ARP请求攻击)

通过ARP请求生成新的IV是常用的可靠的攻击手段。程序监听一个ARP包并转发回AP，反过来，因为AP重复收到了包，会一次又一次的重复转发相同的包，然而每个重复的ARP包都在AP有一个新的IV，就是这些新的IV能够让你获取WEP密码。

ARP是address resolution protocol，TCP/IP协议使用ARP来把一个IP地址转换成一个物理地址，就像是以太网地址那样，一个主机希望获取一个物理地址就得向TCP/IP网络广播一个ARP请求，网络上的主机会回复一个物理地址。ARP是aircrack-ng提供的许多攻击的基础。

### 5.Attack 4: KoreK chopchop attack

该攻击如果成功的话会在不知道key的情况下解析WEP数据包。一般用于破解动态WEP，该攻击不能用于找回WEP密码，但是可以用来显示数据明文。然而一些AP可以抵御这类攻击，所以KoreK chopchop attack不是每次都能成功。 实际上AP返回的包长是60字节，如果不足42字节，aireplay会尝试猜测余下的缺失数据，而后会自动检查包头的校验和，KoreK chopchop attack需要至少一个WEP数据包。

### 6.Attack 5: Fragmentation attack (碎片攻击)

该攻击成功的话可以获取包含150字节的PRGA（pseudo random generation algorithm），也叫做伪随机码生成算法，该攻击只是用于获取PRGA而不能用来找回WEP密码，可以用packetforge-ng攻击使用这个PRGA生成用于各种注入攻击的数据包。至少需要从AP获得一个数据包。

基本上，程序会从数据包中获取少量的密钥信息，并向AP尝试发送包含少量密钥信息的ARP and/or LLC包。如果包可以成功返回，那么程序会从中获取更多的密钥信息，这个过程会不断的重复，直到获取了足够的1500字节PRGA数据。注意PRGA有时候也会少于1500字节。

### 7.Attack 6: Cafe-latte attack

Cafe-latte attack攻击允许你从一个客户端获取WEP密码，简单的说，就是从客户端抓取ARP包。首先使用airodump-ng抓取ARP包，随后使用aricrack-ng来分析出WEP密码

### 8.Attack 7: Client-oriented fragmentation attack

也叫做Hirte attack，是一种在客户端用IP或者ARP包攻击的方式，它增强了Cafe Latte attack，允许无限制的使用客户端的任何ARP包。

这个攻击需要一个来自客户端的ARP包或者IP包，我们需要生成一个ARP请求，该请求包的第33字位必须是客户端IP地址，并且目标MAC地址必须是全0，然而实际应用中，目标MAC地址可以是任意数字。

在包中，第23位（ARP包）或者第21位（IP包）是客户端的IP地址，如果包长是68字节或者86字节，包的最后附加有广播MAC地址，不满足这些条件的话，就认为这个包就是个IP包。

### 9.Attack 9: Injection test （注入测试）

如果你的无线网卡可以成功注入并且能够确定到AP的响应时间，并且你有两块无线网卡的话，就可以使用Injection test攻击。它可以确定哪个特别的注入测试能成功执行。

基本的injection test攻击提供附加的重要信息，首先它可以列出能响应广播所有AP，其次通过30次发包测试可以获得AP的通讯质量（以百分比的形式显示通讯质量），你可以任选AP的名字/MAC地址，你可以使用测试一个指定的AP还是测试一个隐藏的SSID。 如果实现呢？接下来将简单的描绘如何执行测试。 程序最初发送了广播请求，请求会询问AP“你是谁？”，然后收到请求的AP会返回一个关于自身的描述（并不是每个AP都会反馈这种类型的请求），只要程序可以收到AP的响应，那么这块无线网卡就可以成功注入。

较详细的内容请查看官网介绍：http://www.aircrack-ng.org/doku.php?id=aireplay-ng。

 [1]: http://jim19770812.blogspot.com/2012/05/aircrack-ng.html