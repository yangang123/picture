# ����debhelloworld-1.0_amd64.deb�������
1. ����debhelloworld�ļ�
```
vi debhelloworld.c 
#include <stdio.h>
int main(int argc, char *argv)
{
    printf("hello world\r\n");
    return 0;
}
```

2. ����c�ļ������ɿ�ִ���ļ� 
```
gcc debhelloworld.c -o debhelloworld_bin 
mkdir debhelloworld
cd debhelloworld
mkdir -p usr/bin/
cp ../debhelloworld_bin usr/bin/debhelloworld

```
3. д1��control�ļ���debian��
```   
mkdir DEBIAN
vi DEBIAN/control
```
����ע�⣬Package��ֵһ��Ҫд�ɿ�ִ���ļ�������
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

4. ����debhelloworld-1.0_amd64.deb�ļ� 
```
dpkg-deb --build debhelloworld
mv debhelloworld.deb debhelloworld-1.0_amd64.deb
```
5.  ��֤debhelloworld-1.0_amd64.deb�Ƿ�ɹ�
```
yangang@ubuntu:~/work/tools$ sudo dpkg -i debhelloworld-1.0_amd64.deb
Selecting previously unselected package debhelloworld.
(Reading database ... 244879 files and directories currently installed.)
Preparing to unpack debhelloworld-1.0_amd64.deb ...
Unpacking debhelloworld (1.0) ...
Setting up debhelloworld (1.0) ...
```

6.   ִ�в���
```
yangang@ubuntu:~/work/tools$ debhelloworld
hello world
```

7.  ж��debhelloworld
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