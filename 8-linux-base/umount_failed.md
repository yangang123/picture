# umount时候出现，提示失败
多进程访问了设备，是否有其他进程打开了sd文件系统，或者打开了sd卡的文件
准备通过命令eject取出u盘的时候提示 ''device is busy"
>[root@delphi /]# umount umount /home/yangang/work/raspbain/mnt
umount: /mnt/cdrom: device is busy.
        (In some cases useful info about processes that use
         the device is found by lsof(8) or fuser(1))
# 处理方法
杀死这个进程，重新取消挂载，不再提示错误信息
>yangang@ubuntu:~/work$ lsof|grep 'raspbain/mnt'
bash      3546               yangang  cwd       DIR               8,18      4096        198 /home/yangang/work/raspbain/mnt/etc/network
yangang@ubuntu:~/work$ kill -9 3546
yangang@ubuntu:~/work$ sudo umount /home/yangang/work/raspbain/mnt


