
    下载
    https://bitbucket.org/nuttx/nuttx/downloads
    https://sourceforge.net/projects/nuttx/files/nuttx/

注意： 我建议直接进入网站，查看源码的历史版本，选择其中1个版本进行下载，直接git,
可能理解不直观

    下载3个源码包【版本无所谓】
    https://bitbucket.org/nuttx/nuttx/downloads
    https://bitbucket.org/nuttx/apps
    https://bitbucket.org/nuttx/tools
    -> kconfig

    执行配置
    ./configure.sh stm32f4discovery/usbnsh
    cd ..

注意: 这个操作必须nuttx/tools的目录下，由于这个脚本是和config路径绑定的

    执行make menuconfig，发现错误
    Never do 'make menuconfig' on a configuration that has
    not been converted to use the kconfig-frontends tools! This will
    damage your configuration

    安装kconfig-frontends
    cd ../tools
    cd kconfig-frontends
    sudo ./configure --enable-mconf --disable-nconf --disable-gconf --disable-qconf
    最重要的是 : --enable-mconf
    可能出现错误： 没有找到依赖gperf
    解决方法 ： sudo apt-get install gperf 安装
    sudo make
    sudo make install

    继续make menuconfig，发现错误
    kconfig-mconf: error while loading shared libraries: libkconfig-parser-3.12.0.so: cannot open shared object

解决方法：
1)静态编译: sudo ./configure --disable-shared --enable-static --enable-mconf
2)创建连接: sudo ln -s /usr/local/lib/libkconfig-parser-3.12.0.so /lib/libkconfig-parser-3.12.0.so
@1: 在/lib/下是没有这个动态库文件的，创建链接后，自动在/lib目录下生成1个libkconfig-parser-3.12.0.so
@2: 查看libkconfig-parser-3.12.0.so的属性
$ cd /lib
$ ls -l libkconfig-parser-3.12.0.so
lrwxrwxrwx 1 root root 42 5月 17 14:17 libkconfig-parser-3.12.0.so -> /usr/local/lib/libkconfig-parser-3.12.0.so
【libkconfig-parser-3.12.0.so这个链接，链接到实际的/usr/local/lib/libkconfig-parser-3.12.0.so】

注意： 其他方法很多，但是经过尝试，不成功，不知道为什么，以上2种方法，经过自己验证

    继续make menuconfig，发现一切正常.

    执行make ,发现错误:
    no found debug.h文件
    解决方法： 查看系统配置build使用的是windows, LinuxBuild Setup=>Build host platform: Linux

最后保存退出，输入make，则开始编译，编译得到一个nuttx.bin文件，这个可以烧入STM32F4 Discovery板来运行。刚才配置的是usbnsh，因此如果正常运行起来，通过usb连接STM32F4 Discovery到PC会在PC上找到一个serail设备，用终端程序连接打开，就可以看到Nuttx的nsh界面了。

    使用usb串口进行连接
    默认是：串口2 115200 PA2(TX) PA3(RX)
主要使用到的调试方法：

    仔细查看README

    仔细查看执行出错的地方提示

    ubuntu依赖库的添加

    动态库share lib, 静态库 static lib的作用

    创建链接的ln -s的方法

    执行make之前，先确定，指定的的交叉编译工具链和执行文件的平台

    ./configure make make install都需要sudo权限

    环境变量的执行，需要. ./setenv.sh,和其他不同是多个.

    查看README的方法：启动有个CONTEX的目录

    在youtube中查找一些nuttx的编译


可能出现错误： 没有找到依赖gperf-3.0.4
安装
./configure
make
sudo make install


如果始终解决不了，就继续make menuconfig，发现错误
kconfig-mconf: error while loading shared libraries: libkconfig-parser-3.12.0.so: cannot open shared object

就把这个包libkconfig-parser-3.12.0.so拷贝到lib目录下

sudo cp /usr/local/lib/libkconfig-parser-4.11.0.so /lib
