# ubuntu启动界面和ssh登录内部的占用情况

## 不登录系统是下面的现象 
<font color="green">
yangang@ubuntu:~/work/theone_proj/theone-heli$ free -h
              total        used        free      shared  buff/cache   available
Mem:           3.8G        451M        2.2G        6.5M        1.2G        3.1G
Swap:          4.0G          0B        4.0G
</font>

## 登录ubuntu系统后内存情况
<font color="green">
yangang@ubuntu:~$ free -h
              total        used        free      shared  buff/cache   available
Mem:           3.8G        1.2G        1.3G         21M        1.3G        2.3G
Swap:          4.0G          0B        4.0G
</font>

# 总结
<font color="green">
由于linux的界面占用很多内存情况，如果不启动界面的可以完成工作，最好不启动界面，通过ssh登录
服务器进行控制，通过FTP可以传输文件
</font>
