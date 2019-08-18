# profile文件
linux的配置文件profile,bashrc的原理

## /etc/profile
/etc/profile是用户第一开机登陆，root权限给每个一个用户都创建的

## .profile
.profile是普通用户，每次登陆一次，就会执行的.profile的配置文件

## .bashrc
.bashrc是用户每次打开一个终端就会执行的的bashrc脚本

## 使用建议
.profile ：例如编辑器的路径最好配置在这里，如果配置在.bashrc中，打开一次控制台就会执行一次，就会导致控台启动很慢
