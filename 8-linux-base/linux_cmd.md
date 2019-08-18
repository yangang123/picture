# 查看文件大小的命令
我们在linux下经常想查看文件夹的大小，或者文件的大小, 通过命令du，我们可以轻松看到文件的大小 -h表示，按照Gb/Mb/kb表示， --max-depth=1 表示仅仅查看当前文件下第一层子目录文件/文件夹的磁盘占用空间
<font color="red">
输入命令
yangang@ubuntu:~$ sudo du -h  --max-depth=1 work/ <font>

<font color="blue"> 控制台输出结果
1.6G    work/github_proj
6.3G    work/raspbain
4.0K    work/java_test
4.0G    work/tools
4.0K    work/mnt
4.9G    work/linux_proj
3.6G    work/theone_proj
21G     work/
<font>

# 统计代码行数
```
yangang@ubuntu:~/work/github_proj/qxwz_rtk_track/KeilIDE$ find -name *.h -or -name *.c| xargs cat |wc -l
```

# wget获取代码
> 下载包的路径是通过浏览器的代码中看到
linux下通过wget进行下载：
注意： wget URL
地址： https://mirrors.edge.kernel.org/pub/linux/kernel/v4.x/ 可以通过浏览器获取到
手动输入的地址：linux-4.11.2.tar.gz
合并后下载的命令是： wget https://mirrors.edge.kernel.org/pub/linux/kernel/v4.x/

# 总结

从上面，就可以我的工作目录下的文件夹的占用空间情况. github_proj文件夹占用1.6G的大小，其他也是一样
