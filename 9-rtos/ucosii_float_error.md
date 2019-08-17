
# 资源
> ucosii_float_error.md
> 闫刚 ucosii打印浮点数据错误


# 出现的现象
浮点数据打印很不正常，如下图所示，字符串打印的特别长
```
voltage_value:0.000
voltage_value:-2.000
voltage_value:0.000
voltage_value:26815622264063340000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000.000
voltage_value:-2.000
voltage_value:-26815622264087052000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000.000
```
#  查找代码
```
INFO_DEBUG("voltage_value:%.3f", voltage_value[0]);
INFO_DEBUG("voltage_value:%.3f", voltage_value[1]);
```
#  解决问题可能出现位置
```
堆栈没有设置8字节对其，导致float数据出错
OS_STK INPUT_TASK_STK[INPUT_STK_SIZE];
```

# 重新查看打印效果
```
voltage_value:6.897
voltage_value:0.156
voltage_value:6.882
voltage_value:0.152
voltage_value:6.837
```