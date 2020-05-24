# 资

## 设置source.list
```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```

## 设置key
```
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
```

## 更新
```
sudo apt-get update
```

## 更新ros系统
```
sudo apt-get install ros-kinetic-desktop-full
```

## 设置环境变量
```
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

## 安装依赖包
```
sudo apt install python-rosdep python-rosinstall python-rosinstall-generator python-wstool build-essential
```

sudo apt install python-rosdep

# 注意
> 如何是笔记本，那么手机开热点，电脑连接手机热点，否则会出现
```shell
$ sudo rosdep init
```
<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/tree/master/13-ros/resource/rosdep_init.png" height="720" width="500" >
</div>

```shell
$ rosdep update
```
<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/tree/master/13-ros/resource/rosdep_update.png" height="720" width="500" >
</div>

## 失败的图片
<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/tree/master/13-ros/resource/rosdep_init_failed.png" height="720" width="500" >
</div>

# 测试
>3个窗口分别执行下面命令, 小乌龟就出现了
```c
$ roscore
$ rosrun turtlesim turtlesim_node
$ rosrun turtlesim turtle_teleop_key
```
<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/tree/master/13-ros/resource/turtlesim.png" height="720" width="500" >
</div>