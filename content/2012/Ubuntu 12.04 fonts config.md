# Ubuntu 12.04 字体配置

- date: 2012-02-03
- category: work
- tags: ubuntu, font

----

如果安装Ubuntu的英文版，中文字体在其下就会显示的极其丑陋；当然，如果你安装的是中文版，对默认的字体不满意也可以重新配置一下。

### 1. 查看系统设置的字体存放目录

根据/etc/fonts/fonts.conf，默认的字体文件存放在如下几个位置： 

    <dir>/usr/share/fonts</dir>
    <dir>/usr/share/X11/fonts</dir>
    <dir>/usr/local/share/fonts</dir>
    <dir>~/.fonts</dir>
    

前三个字体存放位置需要root权限，而最后一个很自由，随便你怎么弄都不会有问题。如果在`~/`下没有`.fonts`目录，手动创建一个即可。

### 2. 安装字体

其实所谓的安装字体很简单，就是拷贝到上述`~/.fonts`目录啦。这里我推荐直接把M$系统的`fonts`文件夹下的字体都拷贝过来，这样，在浏览网页，写文档时，说不定什么时候就用到了；还有一种字体很值得安装：dejavu字体，Google找下下载吧。

拷贝完之后，还需要刷新下字体缓存，系统才会知道你又安装了那些字体：

    sudo fc-cache -f -v
    

### 3. 字体配置

以前配置字体，我都是直接修改或拷贝下别人的`.fonts.conf`文件，但是这种方法有时候不起作用，或者说这种方法只对你自己目录的字体起作用，而有些软件(其实是大部分软件)都不是在你的权限下安装的，而是在root权限下安装的，或者软件使用root权限在系统下先创建一个用户再使用此用户安装，这些情景下，只用`.fonts.conf`文件就不行。这是我的理解吧。

因此，这里推荐直接配置全局的字体：

ubuntu的全局字体配置在`/etc/fonts/`目录下：

    xiaoxie@xiaoxie-laptop:~$ cd /etc/fonts
    xiaoxie@xiaoxie-laptop:/etc/fonts$ ls -li
    总用量 24
    1049890 drwxr-xr-x 2 root root 4096 Jan 29 21:36 conf.avail
    1049891 drwxr-xr-x 2 root root 4096 Jan 29 21:37 conf.d
    1049892 -rw-r--r-- 1 root root 5287 Aug 22 13:43 fonts.conf
    1049893 -rw-r--r-- 1 root root 6961 Aug 22 13:43 fonts.dtd
    

可以看到，`/etc/fonts/`下一共有两个文件和两个文件夹(权限那列d表示directory)，这里解释下：  
conf.avail：存放了许许多多的字体配置文件，可在`conf.d/readme`中看到相关解释  
conf.d：存放的全都是对`conf.avail`中文件的软链接，这种方式在linux系统下很常见，比如apache的模块配置文件  
fonts.conf：描述字体文件存放目录位置  
fonts.dtd：`fonts.conf`是xml格式的，此文件就是此xml的规则约束文件

由于是设置中文字体，所以系统应该使用的是`69-language-selector-zh-cn.conf`这个文件，如果没有则在`conf.avail`中创建此文件：

    xiaoxie@xiaoxie-laptop:/etc/fonts/conf.avail$ sudo touch 69-language-selector-zh-cn.conf
    

对与字体设置，我们只需要对三种字体表现形式进行设置即可，分别是：Sans Serif、Serif和等宽字体。以下是我在`69-language-selector-zh-cn.conf`文件中的设置：

    <?xml version="1.0"?>
    <!DOCTYPE fontconfig SYSTEM "fonts.dtd">
    <fontconfig>
    
        <match target="pattern">
            <test qual="any" name="family">
                <string>serif</string>
            </test>
            <edit name="family" mode="prepend" binding="strong">
                <string>WenQuanYi Micro Hei</string>
                <string>DejaVu Serif</string>
                <string>Bitstream Vera Serif</string>
                <string>HYSong</string>
                <string>AR PL UMing CN</string>
                <string>AR PL UMing HK</string>
                <string>AR PL ShanHeiSun Uni</string>
                <string>AR PL New Sung</string>
                <string>WenQuanYi Bitmap Song</string>
                <string>AR PL UKai CN</string>
                <string>AR PL ZenKai Uni</string>
            </edit>
        </match> 
        <match target="pattern">
            <test qual="any" name="family">
                <string>sans-serif</string>
            </test>
            <edit name="family" mode="prepend" binding="strong">
                <string>WenQuanYi Micro Hei</string>
                <string>DejaVu Sans</string>
                <string>Bitstream Vera Sans</string>
                <string>WenQuanYi Zen Hei</string>
                <string>Droid Sans Fallback</string>
                <string>HYSong</string>
                <string>AR PL UMing CN</string>
                <string>AR PL UMing HK</string>
                <string>AR PL ShanHeiSun Uni</string>
                <string>AR PL New Sung</string>
                <string>AR PL UKai CN</string>
                <string>AR PL ZenKai Uni</string>
            </edit>
        </match> 
        <match target="pattern">
            <test qual="any" name="family">
                <string>monospace</string>
            </test>
            <edit name="family" mode="prepend" binding="strong">
                <string>WenQuanYi Micro Hei Mono</string>
                <string>DejaVu Sans Mono</string>
                <string>Bitstream Vera Sans Mono</string>
                <string>WenQuanYi Zen Hei Mono</string>
                <string>Droid Sans Fallback</string>
                <string>HYSong</string>
                <string>AR PL UMing CN</string>
                <string>AR PL UMing HK</string>
                <string>AR PL ShanHeiSun Uni</string>
                <string>AR PL New Sung</string>
                <string>AR PL UKai CN</string>
                <string>AR PL ZenKai Uni</string>
            </edit>
        </match> 
    
    </fontconfig>
    

PS：可以根据自己的爱好修改这三种字体。

最后，需要在`conf.d`中建立软链接：

    sudo ln -s ../conf.avail/69-language-selector-zh-cn.conf
    

### 4. 查看字体

上述过程完成后，注销系统重新登入，检查下当前字体吧。下面是我设置后的效果，大爱微米黑啊！

<a><img src="/media/2013/02/3296271792.png"></a>