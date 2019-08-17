@[TOC]

# px4架构的cmake移植到linux上
> px4的cmake架构非常棒，今天我一不小心移植到自己的代码中，从而后期全部按照px4的cmake提供的模块接口px4_add_module进行添加子模块非常酷。

# 仓库地址：
$git clone http://www.github.com/yangang123/cpp_test.git
$git checkout develop 

## 图一 PX4的源码cmake架构
<div >
<img src="https://github.com/yangang123/picture/raw/master/6-linux-app/resource/px4_cmake_1.png" height="720" width="1280" > 
</div>

## 图二 px4添加一个驱动模块的CMakeLists.txt文件

<div >
<img src="https://github.com/yangang123/picture/raw/master/6-linux-app/resource/px4_cmake_2.png" height="720" width="1280" > 
</div>

## 图三 openSTM的源码架构

<div >
<img src="https://github.com/yangang123/picture/raw/master/6-linux-app/resource/px4_cmake_3.png" height="720" width="1280" > 
</div>


## 图四 openSTM中添加子模块CMakeLists.txt文件
<div >
<img src="https://github.com/yangang123/picture/raw/master/6-linux-app/resource/px4_cmake_4.png" height="720" width="1280" > 
</div>
现在，在我们的软件工程中添加一个hello的模块，也是非常的方便

## make openSTM-v1
我们输入make openSTM-v1，然后的执行的流程过程如下图所示
```c
makefile 
  |
  v
CMakeLists.txt  -> cmake/configs/openSTM-v1.cmake -> cmake/common/px4_base.cmake 
    #调用出所有的子模块
    -> set(config_module_list)        
    
    #遍历所有的子模块
    -> foreach(module ${config_module_list}) 
   
    #添加子目录
    add_subdirectory(src/${module})

    #进入固件目录   
    -> add_subdirectory(src/firmware) -> add_executable(${FIRMWARE_NAME}) 
```

## 图五  openSTM编译
<div >
<img src="https://github.com/yangang123/picture/raw/master/6-linux-app/resource/px4_cmake_5.png" height="720" width="1280" > 

# Makefie基础

## 1. call是Makefile的函数
```
define hello
echo hello
endf
all:
 $(call target)
```
## 2 .定义个函数
>define 函数名（XXX）
函数体
endef
函数体中每行不需要加TAB键
函数体中可包含：命令  脚本  调用函数等
函数体中的$(n)为调用体传递进来的参数，$(0)为函数本来，$(1) 第一个参数，$(2)第二个参数
若实在其他函数中调用的，$(n)来自这个函数传递的；若在shell下调用的，$(n)来自于用户传递的。

## 3. makefile语法
   
在makefile中的命令使用@,表示不打印当前命令，否则执行当前命令之前，会把这条命令执行以下
```
@echo hello
hello
echo hello
echo hello
hello

下面，我讲一个makefile最复杂的地方，主要功能，管理build目录的功能
```

## 4 描述如何编译cmake 
```
define cmake-build
@if [ $(PX4_CMAKE_GENERATOR) = "Ninja" ] && [ -e $(PWD)/build_$@/Makefile ]; then rm -rf $(PWD)/build_$@; fi
@if [ ! -e $(PWD)/build_$@/CMakeCache.txt ]; then mkdir -p $(PWD)/build_$@ && cd $(PWD)/build_$@ && cmake .. -G$(PX4_CMAKE_GENERATOR) -DCONFIG=$(1) || (cd .. && rm -rf $(PWD)/build_$@); fi
@$(PX4_MAKE) -C $(PWD)/build_$@ $(PX4_MAKE_ARGS) $(ARGS)
endef
# create empty targets to avoid msgs for targets passed to cmake
define cmake-targ
$(1):
    @#
.PHONY: $(1)
endef
openSTM-v1:
    $(call cmake-build,openSTM-v1)
all: openSTM-v1
```

## 分析第一行
第一行 @if [ $(PX4_CMAKE_GENERATOR) = "Ninja" ] && [ -e $(PWD)/build_$@/Makefile ]; then rm -rf $(PWD)/build_$@; fi

分解开后
```
if [ $(PX4_CMAKE_GENERATOR) = "Ninja" ] && [ -e $(PWD)/build_$@/Makefile ];
then rm -rf $(PWD)/build_$@;
fi
```
说明： 当判断系统有ninja构建，同时在build文件中makefile这个文件，就会执行删除当前build文件，这个操作的主要目的是如果源码没有个ninja构建会影响ninja构建，直接删除旧的build构建文件

## 分析第二行
```
第二行@if [ ! -e $(PWD)/build_$@/CMakeCache.txt ]; then mkdir -p $(PWD)/build_$@ && cd $(PWD)/build_$@ && cmake .. -G$(PX4_CMAKE_GENERATOR) -DCONFIG=$(1) || (cd .. && rm -rf $(PWD)/build_$@); fi

分解开后
if [ ! -e $(PWD)/build_$@/CMakeCache.txt ]; 
then mkdir -p $(PWD)/build_$@ && cd $(PWD)/build_$@ 
&& cmake .. -G$(PX4_CMAKE_GENERATOR) -DCONFIG=$(1) || (cd .. && rm -rf $(PWD)/build_$@)
;fi
```

说明： 如果在build中文件不存在cmakecache.txt文件， 那么就创建build文件，同时打开build，然后cmake ../去构建项目， -G是执行构建的的方式，-DCONFIG, 会自动把CONFIG作为变量进行传递到各个cmake文件中。
PX4_CMAKE_GENERATOR ?= "Ninja" PX4_CMAKE_GENERATOR ?= "MSYS Makefiles" PX4_CMAKE_GENERATOR ?= "Unix Makefiles”

# 技巧：
1. 判断build中的Makefile判断是否ninja构建， 在ninja中构建的build目录，没有makefile文件build.ninja  CMakeCache.txt  CMakeFiles  cmake_install.cmake  rules.ninja  src2. 判断CMakeCache.txt是否存在，判断是否构建软件build目录