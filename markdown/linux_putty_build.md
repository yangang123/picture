# 一.源代码下载和编译

## 1. 源代码下载
https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html

## 2. 编译
```
$ .configure
$ make
```
https://the.earth.li/~sgtatham/putty/0.71/htmldoc/
# 二. 安装putty可执行文件
通过输出信息，我们可以知道安装软件主要分为2个部分:
1. 安装可以执行文件到/usr/local/bin
2. 安装文档说明到/usr/local/share/man/
```
$ sudo make install
[sudo] yangang 的密码： 
make  install-am
make[1]: 进入目录“/home/yangang/work/tools/putty-0.71”
make[2]: 进入目录“/home/yangang/work/tools/putty-0.71”
 /bin/mkdir -p '/usr/local/bin'
  /usr/bin/install -c plink pscp psftp puttygen pageant pterm putty puttytel '/usr/local/bin'
 /bin/mkdir -p '/usr/local/share/man/man1'
 /usr/bin/install -c -m 644 doc/plink.1 doc/pscp.1 doc/psftp.1 doc/puttygen.1 doc/pageant.1 doc/pterm.1 doc/putty.1 doc/puttytel.1 '/usr/local/share/man/man1'
make[2]: 离开目录“/home/yangang/work/tools/putty-0.71”
make[1]: 离开目录“/home/yangang/work/tools/putty-0.71”
```

# 三. 卸载putty可执行文件
```
yangang@yangang-ubuntu:~/work/tools/putty-0.71$ sudo make uninstall
 ( cd '/usr/local/bin' && rm -f plink pscp psftp puttygen pageant pterm putty puttytel )
 ( cd '/usr/local/share/man/man1' && rm -f plink.1 pscp.1 psftp.1 puttygen.1 pageant.1 pterm.1 putty.1 puttytel.1 )
```


