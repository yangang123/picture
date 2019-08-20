@[TOC]
# 介绍
>讲解linux的spi驱动架构, 包括用户空间和内核空间如何配合使用spi驱动。通过px4的DriverFramwork架构
实现imu传感器驱动架构作为案例进行讲解.

# 资源
> linux_spi.md
> 标题: 闫刚 linux平台实现IMU的DriverFramework

# 用户态

## spi包

<div align="center">
<p>  </p> 
<img src="https://github.com/yangang123/picture/raw/master/7-linux-kernel/resource/SPI_user_space.PNG" height="720" width="1280" > 
</div>

## 1. spidev的设备节点spidev0.3表示spi0的chip_select3
```cpp
#include <linux/spi/spidev.h>
#define IMU_DEVICE_ACC_GYRO "/dev/spidev0.3"
#define IMU_DEVICE_MAG "/dev/spidev0.2"

int do_test(unsigned int num_read_attempts)
   -> int ret = Framework::initialize();
   -> ImuTester pt; //创建1个局部对象
   -> DevMgr::getHandle(IMU_DEVICE_PATH, h);
   -> int SPIDevObj::start()
      -> m_dev_path = strdup(dev_path);
   -> ret = pt.run(num_read_attempts); /运行这个传感器
       -> ret = m_sensor.getSensorData(data, true);
```

# 内核态

<div align="center">
<p>  </p> 
<img src="https://github.com/yangang123/picture/raw/master/7-linux-kernel/resource/SPI_kernel_space.PNG" height="720" width="1280" > 
</div>

## 设备树
arch/arm/boot/dts/px4.dts设备树
```
&spi0 {
	is-decoded-cs = <1>;
	num-cs = <6>;
	status = "okay";

	/* ICM20689 */
	spidev@0 {
		compatible = "spidev";
		reg = <1>;
		spi-max-frequency = <20000000>;
		spi-cpol;
		spi-cpha;
	};

	/* MS5611 SUPPORT both MODE 0 and MODE 3 */
	spidev@1 {
		compatible = "spidev";
		reg = <2>;
		spi-max-frequency = <20000000>;
		spi-cpol;
		spi-cpha;		
	};
}
```

## 注册spidev设备
```cpp
spidev_ioctl(struct file *filp, unsigned int cmd, unsigned long arg)
result = ::ioctl(m_fd, SPI_IOC_MESSAGE(1), &spi_transfer);
  ->if (copy_from_user(tx_buf, (const u8 __user *)(uintptr_t) u_tmp->tx_buf,u_tmp->len)) //拷贝用户层数据
   //传输一个message
   -> static int spi_transfer_one_message(struct spi_master *master,
   -> status = spidev_sync(spidev, &msg);
		spi->cs_gpio = master->cs_gpios[spi->chip_select]; //chip_select是一个信号
```
