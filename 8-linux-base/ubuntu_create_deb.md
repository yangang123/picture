# 生成debhelloworld-1.0_amd64.deb的软件包
1. 创建debhelloworld文件
```
vi debhelloworld.c 
#include <stdio.h>
int main(int argc, char *argv)
{
    printf("hello world\r\n");
    return 0;
}
```

2. 编译c文件，生成可执行文件 
```
gcc debhelloworld.c -o debhelloworld_bin 
mkdir debhelloworld
cd debhelloworld
mkdir -p usr/bin/
cp ../debhelloworld_bin usr/bin/debhelloworld

```
3. 写1个control文件到debian下
```   
mkdir DEBIAN
vi DEBIAN/control
```
下面注意，Package的值一定要写成可执行文件的命令
```
Package: debhelloworld
Version: 1.0
Section: custom
Priority: optional
Architecture: all
Essential: no
Installed-Size: 1024
Maintainer: helloworld.org
Description: Print helloworld.org on the screen
```

4. 生成debhelloworld-1.0_amd64.deb文件 
```
dpkg-deb --build debhelloworld
mv debhelloworld.deb debhelloworld-1.0_amd64.deb
```
5.  验证debhelloworld-1.0_amd64.deb是否成功
```
yangang@ubuntu:~/work/tools$ sudo dpkg -i debhelloworld-1.0_amd64.deb
Selecting previously unselected package debhelloworld.
(Reading database ... 244879 files and directories currently installed.)
Preparing to unpack debhelloworld-1.0_amd64.deb ...
Unpacking debhelloworld (1.0) ...
Setting up debhelloworld (1.0) ...
```

6.   执行测试
```
yangang@ubuntu:~/work/tools$ debhelloworld
hello world
```

7.  卸载debhelloworld
```
yangang@ubuntu:~/work/tools$ sudo apt-get remove debhelloworld
[sudo] password for yangang:
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be REMOVED:
  debhelloworld
0 upgraded, 0 newly installed, 1 to remove and 11 not upgraded.
After this operation, 1,049 kB disk space will be freed.
Do you want to continue? [Y/n] y
(Reading database ... 244879 files and directories currently installed.)
Removing debhelloworld (1.0) ...
```    