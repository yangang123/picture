# linux文件属性的问题


## 普通复制
丢失文件文件属性和执行权限, 文件的创建时间，文件的链接属性也丢失

1. 文件复制前
yangang@ubuntu:/sbin$ ll reboot
lrwxrwxrwx 1 root root 14 Mar  3 10:36 reboot -> /bin/systemctl*

2. 使用普通的复制命令
yangang@ubuntu:/sbin$ cp reboot ~/work/

3. 复制后的状态
yangang@ubuntu:/sbin$ ll ~/work/reboot
-rwxr-xr-x 1 yangang yangang 659856 Apr  7 19:51 /home/yangang/work/reboot*


## 高级复制
保留文件属性，文件执行权限， 保留文件创建时间

1. 使用普通的复制命令
yangang@ubuntu:/sbin$ **cp -p reboot ~/work/**


2. 复制后的状态
yangang@ubuntu:/sbin$ ll ~/work/reboot
-rwxr-xr-x 1 yangang yangang 659856 Mar  3 10:37 /home/yangang/work/reboot*


