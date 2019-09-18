#git tag使用

<font color="blue">
我们的软件开发到某个阶段，可以创建发行版本，现在gitee和github上的可以自动显示你tag打包后的
发行版本，很方便。
</font>

1. 创建tag
>git tag -a cct_v1.1.0_beta -m "新硬件V2, 色温模式"

2. 查看tag
>git tag

3. 推送tag到远程分支
> git push origin cct_v1.1.0_beta
       
4. 删除本地分支
>git tag -d cct_v1.1.0_beta

5. 删除远程分支
>git push origin :cct_v1.1.0_beta

6. 同步仓库的方法
>git submodule sync --recursive                                  
git submodule update --init --recursive      

7. 子模块submodule1内容被删除，回复内容的方式
> 7.1和7.2是重要的方法
7.1 rm submodule1 -rf
> 删除文件夹
7.2 git clone xxxx  submodule1
> 手动克隆仓库
7.3 git submodule update --init --recursive 
> 以后就可以正常管理子模块
