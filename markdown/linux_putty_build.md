# һ.Դ�������غͱ���

## 1. Դ��������
https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html

## 2. ����
```
$ .configure
$ make
```
https://the.earth.li/~sgtatham/putty/0.71/htmldoc/
# ��. ��װputty��ִ���ļ�
ͨ�������Ϣ�����ǿ���֪����װ�����Ҫ��Ϊ2������:
1. ��װ����ִ���ļ���/usr/local/bin
2. ��װ�ĵ�˵����/usr/local/share/man/
```
$ sudo make install
[sudo] yangang �����룺 
make  install-am
make[1]: ����Ŀ¼��/home/yangang/work/tools/putty-0.71��
make[2]: ����Ŀ¼��/home/yangang/work/tools/putty-0.71��
 /bin/mkdir -p '/usr/local/bin'
  /usr/bin/install -c plink pscp psftp puttygen pageant pterm putty puttytel '/usr/local/bin'
 /bin/mkdir -p '/usr/local/share/man/man1'
 /usr/bin/install -c -m 644 doc/plink.1 doc/pscp.1 doc/psftp.1 doc/puttygen.1 doc/pageant.1 doc/pterm.1 doc/putty.1 doc/puttytel.1 '/usr/local/share/man/man1'
make[2]: �뿪Ŀ¼��/home/yangang/work/tools/putty-0.71��
make[1]: �뿪Ŀ¼��/home/yangang/work/tools/putty-0.71��
```

# ��. ж��putty��ִ���ļ�
```
yangang@yangang-ubuntu:~/work/tools/putty-0.71$ sudo make uninstall
 ( cd '/usr/local/bin' && rm -f plink pscp psftp puttygen pageant pterm putty puttytel )
 ( cd '/usr/local/share/man/man1' && rm -f plink.1 pscp.1 psftp.1 puttygen.1 pageant.1 pterm.1 putty.1 puttytel.1 )
```


