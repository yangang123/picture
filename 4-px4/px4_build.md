

# 1. Makefile 
> px4fmu-v5_default是1个目标，同时也是1个board的名字，同时是cmake的参数
> cmake-build是Makefile的函数
make px4fmu-v5_default upload
px4fmu-v5_default:
	$(call cmake-build,nuttx_px4fmu-v5_default) 


# 2.  cmake-build
```
define cmake-build
+@if [ $(PX4_CMAKE_GENERATOR) = "Ninja" ] && [ -e $(PWD)/build_$@/Makefile ]; then rm -rf $(PWD)/build_$@; fi
+@if [ ! -e $(PWD)/build_$@/CMakeCache.txt ]; then Tools/check_submodules.sh && mkdir -p $(PWD)/build_$@ && cd $(PWD)/build_$@ && cmake .. -G$(PX4_CMAKE_GENERATOR) -DCONFIG=$(1) || (cd .. && rm -rf $(PWD)/build_$@); fi
+@Tools/check_submodules.sh
+$(PX4_MAKE) -C $(PWD)/build_$@ $(PX4_MAKE_ARGS) $(ARGS)
```

# 3. 执行ninja过程

## 不带upload
1. 命令
```
ninja -C /home/yangang/work/theone_proj/firmware/build_px4fmu-v5_default 
```

## 2. 查看build.ninja文件
> all是默认的目标， “default all”表示all是默认的目标
```
build all: phony libmsg_gen.a src/firmware/nuttx/build_firmware_px4fmu-v5 src/firmware/nuttx/firmware_nuttx src/firmware/nuttx/libromfs.a src/modules/px4iofirmware/px4io-v2

default all
```

## ”make px4fmu-v5_default upload“的执行过程

> /home/yangang/work/theone_proj/firmware/build_px4fmu-v5_default是1个路径，默认找到build.ninja文件
> “upload”是ninja构建的目标，如果
```
ninja -C /home/yangang/work/theone_proj/firmware/build_px4fmu-v5_default  upload
```

```
build upload: phony src/firmware/nuttx/upload
```   

