# 资源
> ros_node.md
> 闫刚 ros node讲解

# 运行ros python的订阅和发布的例子

## 运行roscore
$ roscore

## 运行1个发布者，3个监听者
./talker
.listener
.listener
.listener

## 查看node list
```shell
$ rosnode list
/listener_42936_1590332118904
/listener_43059_1590332129428
/listener_43180_1590332136028
/rosout
/rqt_gui_py_node_43199
/talker_42923_1590332112048
```
   
## 查看node图形
rosrun rqt_graph  rqt_graph

<div align="center">
<img src="https://github.com/yangang123/yangang123.github.io/tree/master/13-ros/resource/rosgraph.png" height="720" width="500" >
</div>

## 001_talker_listener文件内容分析
yangang@ubuntu:~/work/ros/ros_tutorials-0.10.1/rospy_tutorials$ tree -L 2 001_talker_listener/
001_talker_listener/
├── listener -> listener.py
├── listener.py
├── README
├── talker -> talker.py
├── talker_listener.launch
├── talker.py
└── talker_timer.py

0 directories, 7 files

# 源码分支
创建节点，订阅主题chatter
```py
rospy.init_node('listener', anonymous=True)
rospy.Subscriber('chatter', String, callback)
````


