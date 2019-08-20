@[TOC]
# 资源
> 文件名: px4_uavcan_dsdl.md
> 标题 ： 闫刚 px4_uavcan_dsdl的原理

# 简介
> DSDL包含了UAVCAN的所有数据结构， 它是python写的自动化生成头文件hpp的工具。学会它，我们基本可以了解到UAVCAN中最核心的通信结构。
> UAVCAN如何生成头文件hpp ?带着疑问，我们找到一些突破点

# 1. cannode构建分析

## 1.1  找到uavcan的单板
>查找nuttx-config下的cannode的板子，找到单板cannode-v1, 由于px4cannode-v1是基于UAVCAN典型的1个例子，
>编译整个make px4cannode-v1, 注意，px4cannode-v1这个单板的构建是通过ninja构建，一般的makefile可能编译不过

<div >
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/4-px4/resource/uavcan_1.png" height="200" width="1280" > 
</div>

## 1.2   分析一下UAVCAN下的CAMKE
>$ vi ./libuavcan/CMakeLists.txt

<div >
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/4-px4/resource/uavcan_2.png" height="300" width="1280" > 
</div>

- DSDLC_INPUTS = test/dsdl_test/root_ns_a" "test/dsdl_test/root_ns_b" "${CMAKE_CURRENT_SOURCE_DIR}/../dsdl/uavcan
- DSDLC_OUTPUT  = "include/dsdlc_generated"
- 从这里，我们基本就可以知道，这个DSDL基本的工作就是把输入文件转换成头文件，我也知道了输入文件的路劲，保存到了${DSDLC_INPUTS} 这个环境变量中， 输出文件保存到了${DSDLC_OUTPUT }文件
  
<div >
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/4-px4/resource/uavcan_3.png" height="200" width="1280" > 
</div>

- 如何解析这些文件，从下面图片，我基本可以知道python解析了${CMAKE_CURRENT_SOURCE_DIR}/dsdl_compiler/libuavcan_dsdlc 这个脚本文件

<div >
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/4-px4/resource/uavcan_3.png" height="200" width="1280" > 
</div>

- 从上面我们了解到，工具是libuavcan_dsdlc， 输入文件是在dsdl下的所有.uavcan文件，输出文件是在include/dsdlc_generated下，我们下面查看这2个文件夹

### 1.3.1 查看dsdl下的文件
<div >
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/4-px4/resource/uavcan_3.png" height="200" width="1280" > 
</div>

### 1.3.2. 查看输出路径下的include/dsdlc_generated
<div >
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/4-px4/resource/uavcan_3.png" height="200" width="1280" > 
</div>

# 2.我们添加自己的UAVCAN数据如何添加vitual_button文件夹
>文件下面有1个文件命名为1500.VitualButton.uavcan
## 2.1 创建文件
```
$ touch ./libuavcan/dsdl/uavcan/equipmen./libuavcan/dsdl/uavcan/equipmen/vitual_button
 
$ cat ./libuavcan/dsdl/uavcan/equipmen/vitual_button/1500.VitualButton.uavcan
 
uint8 value  # range 0 to 255
```

## 2.2 预期结果
>我们如何编译成功，会在include/dsdlc_generatd中生成1个文件夹是include/dsdlc_generated,文件名字应该是VitualButton.hpp文件

## 2.3 执行编译命令
```
/usr/bin/python /home/yangang/work/px4/Firmware/src/modules/uavcan/libuavcan/libuavcan/dsdl_compiler/libuavcan_dsdlc 
/home/yangang/work/px4/Firmware/src/modules/uavcan/libuavcan/libuavcan/../dsdl/uavcan -Oinclude/dsdlc_generated
```

## 2.4 查看输出文件
<div >
<img src="https://github.com/yangang123/yangang123.github.io/raw/master/4-px4/resource/uavcan_3.png" height="200" width="1280" > 
</div>

# 3. 错误的文件的例子
## 3.1   1500.camerafd.uavcan文件内容
```
uint8 id
```
## 3.2 添加文件,编译出错
```bash
FAILED: cd /home/yangang/work/px4_proj/Firmware/src/modules/uavcan/libuavcan/libuavcan && /usr/bin/python /home/yangang/work/px4_proj/Firmware/src/modules/uavcan/libuavcan/libuavcan/dsdl_compiler/libuavcan_dsdlc test/dsdl_test/root_ns_a test/dsdl_test/root_ns_b /home/yangang/work/px4_proj/Firmware/src/modules/uavcan/libuavcan/libuavcan/../dsdl/uavcan -Oinclude/dsdlc_generated && /opt/cmake-3.3.2/bin/cmake -E touch /home/yangang/work/px4_proj/Firmware/build/px4cannode-v1_default/libuavcan_dsdlc_run.stamp
Internal error
Traceback (most recent call last):
  File "/home/yangang/work/px4_proj/Firmware/src/modules/uavcan/libuavcan/libuavcan/dsdl_compiler/pyuavcan/uavcan/dsdl/parser.py", line 606, in parse
    return self.parse_source(filename, source_text)
  File "/home/yangang/work/px4_proj/Firmware/src/modules/uavcan/libuavcan/libuavcan/dsdl_compiler/pyuavcan/uavcan/dsdl/parser.py", line 598, in parse_source
    raise ex
DsdlException: /home/yangang/work/px4_proj/Firmware/src/modules/uavcan/libuavcan/dsdl/uavcan/equipment/camera_gimbal/1500.camerafd.uavcan: 
Invalid type name [uavcan.equipment.camera_gimbal.camerafd]
```
## 3.3 解决方法
> 文件名字，除了前面的数字外，首字母需要大写。修改文件名为1500.Camerafd.uavcan

## 3.4 调试过程
> 根据编译信息，我们找到出现的异常地方是parser.py", line 606, in parse, 跟踪代码如下，可以看出，字母camerafd的首字母要打大小写

```
enforce(re.match(r'[A-Z][A-Za-z0-9_]*$', short_name), 'Invalid type name [%s]', name)
 
def enforce(cond, fmt, *args):
    if not cond:
        error(fmt, *args)
```
