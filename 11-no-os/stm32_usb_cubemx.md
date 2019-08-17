﻿
# 资源
> stm32_usb_cubemx.md
> 闫刚 stm32的usb的hal软件架构原理讲解

# 一. usb基础
stm32的usb也是很多公司都在使用的接口，usb全速可以达到12M/s, 作为虚拟串口接口，还是不错的.usb整个协议还是很复杂的，整理了下我这几年对usb的使用，讲解下usb软件架构，和一些重要的知识点

# 二. cubemx生成usb虚拟串口的demo工程

[usb_vcp_demo仓库链接](https://github.com/yangang123/usb_vcp_demo)

下载仓库: 
```
git clone https://github.com/yangang123/usb_vcp_demo.git
```

## 2.1 步骤：
1. 选择板子是stm32f4 discovery
2. 选择外部晶振
3. 设置系统时钟是168MHz
4. 设置USB是device
5. 设置USB的从设备的类是vcp
6. 设置堆栈是0x600[注意]
7. 设置复位
8. 查看端口
9. 查看结果
  

<div align="center">
<p>  1. 选择板子是stm32f4 discovery </p> 
<img src="https://github.com/yangang123/usb_vcp_demo/raw/master/picture/1.png" height="720" width="1280" > 
</div>

<div align="center"> 
<p>  2. 选择外部晶振 </p> 
<img src="https://github.com/yangang123/usb_vcp_demo/raw/master/picture/2.png" height="720" width="1280" > 
</div>

<div align="center"> 
<p>  3. 设置系统时钟是168MHz </p> 
<img src="https://github.com/yangang123/usb_vcp_demo/raw/master/picture/3.png" height="720" width="1280"> 

</div>

<div align="center"> 
<p>  4. 设置USB是device </p> 
<img src="https://github.com/yangang123/usb_vcp_demo/raw/master/picture/4.png" height="720" width="1280"height="720" width="1280"> 
</div>

<div align="center"> 
<p>  5. 设置USB的从设备的类是vcp </p> 
<img src="https://github.com/yangang123/usb_vcp_demo/raw/master/picture/5.png" height="720" width="1280"> 
</div>

<div align="center"> 
<p>  6. 设置堆栈是0x600 </p> 
<img src="https://github.com/yangang123/usb_vcp_demo/raw/master/picture/6.png" height="720" width="1280"> 
</div>

<div align="center"> 
<p>  7. 设置复位 </p> 
<img src="https://github.com/yangang123/usb_vcp_demo/raw/master/picture/7.png" height="720" width="1280"> 
</div>

<div align="center"> 
<p>  8. 查看端口 </p> 
<img src="https://github.com/yangang123/usb_vcp_demo/raw/master/picture/8.png" height="720" width="1280"> 
</div>

<div align="center"> 
<p>  9. 查看结果 </p> 
<img src="https://github.com/yangang123/usb_vcp_demo/raw/master/picture/9.png" height="720" width="1280"> 
</div>


## 2.2 测试工作： 
PC ： WIN10 
软件：友善之臂串口助手
Keil: MDK5
单板：stm32f4 discory

## 2.3 测试过程：
1. KEIL中要设置下载完成后，自动复位
2. 在电脑上查看COM口是哪个
3. 程序中每秒发送1个字符'a', 
4. 串口助手每秒发送1个字符'a'

## 整个工程保存已经提交到了github中
地址:

# 三. 如何处理发送和接收

接收和发送的接口都是usbd_cdc_if.c中

## 3.1 数据缓冲大小配置是下面
```
#define APP_RX_DATA_SIZE  4
#define APP_TX_DATA_SIZE  4
```

## 3.2 发送数据

发送数据用下面这个api接可以了
uint8_t CDC_Transmit_FS(uint8_t* Buf, uint16_t Len)
下面是1s周期发送"a"数据
```
HAL_Delay(1000);
char *string = "a";
CDC_Transmit_FS((uint8_t*)string, 1);
```

## 3.3 接收数据

接收缓冲区会用到 uint8_t UserRxBufferFS[APP_RX_DATA_SIZE];
接收数据的长度   int recv_len = 0;
接收数据的接口是 static int8_t CDC_Receive_FS (uint8_t* Buf, uint32_t *Len), 接收数据
```
static int8_t CDC_Receive_FS (uint8_t* Buf, uint32_t *Len)
{ 
  recv_len = *Len;
  USBD_CDC_SetRxBuffer(&hUsbDeviceFS, &Buf[0]);
  USBD_CDC_ReceivePacket(&hUsbDeviceFS);
  return (USBD_OK);
}
```

# 四. 发送和接收原理

| 层次 | 功能 |  主要文件|
| :------| ------: | :------: |
| APP |  应用层| usbd_cdc_if.c |
| calss | 不同的设备不同的数据 | usb_cdc.c |
| core | USB数据缓冲的处理 | usb_core.c |
| HAL  | 硬件中断处理 | stm32f4_ll_usb.c |

## 4.1 发送数据

USB_EPStartXfer: 启动一个端点传输
USBx_INEP: 使能EP
USB_WritePacket:数据写入fifo
```
CDC_Transmit_FS((uint8_t*)string, 15); 
    -> USBD_CDC_TransmitPacket(&hUsbDeviceFS);
        ->   USBD_LL_Transmit(pdev,  CDC_IN_EP, hcdc->TxBuffer,hcdc->TxLength);
            ->   hal_status = HAL_PCD_EP_Transmit(pdev->pData, ep_addr, pbuf, size);
                -> HAL_StatusTypeDef USB_EPStartXfer(USB_OTG_GlobalTypeDef *USBx , USB_OTG_EPTypeDef *ep, uint8_t dma)
                    ->USBx_INEP(ep->num)->DIEPCTL |= (USB_OTG_DIEPCTL_CNAK | USB_OTG_DIEPCTL_EPENA);
                    ->USB_WritePacket(USBx, ep->xfer_buff, ep->num, ep->xfer_len, dma);
```           

## 4.2 接收数据
```
HAL_PCD_IRQHandler(&hpcd_USB_OTG_FS);
    -> HAL_PCD_DataOutStageCallback(hpcd, epnum);
        -> USBD_LL_DataOutStage((USBD_HandleTypeDef*)hpcd->pData, epnum, hpcd->OUT_ep[epnum].xfer_buff);
            -> pdev->pClass->DataOut(pdev, epnum); 
                ->  ((USBD_CDC_ItfTypeDef *)pdev->pUserData)->Receive(hcdc->RxBuffer, &hcdc->RxLength);
```
### 4.2.1 接口 DataOut
接口结构体
```
  typedef struct _Dev   ice_cb{
  uint8_t  (*DataOut)(struct _USBD_HandleTypeDef *pdev , uint8_t epnum); 
```
注册DataOut这个接口
```
USBD_ClassTypeDef  USBD_CDC = {
  USBD_CDC_DataOut,
```
### 4.2.2 接口 Receive

接口结构体
```
typedef struct _USBD_CDC_Itf
{
  int8_t (* Init)          (void);
  int8_t (* DeInit)        (void);
  int8_t (* Control)       (uint8_t, uint8_t * , uint16_t);   
  int8_t (* Receive)       (uint8_t *, uint32_t *);  
}USBD_CDC_ItfTypeDef;
```

注册(* Receive)这个接口函数
```
USBD_CDC_ItfTypeDef USBD_Interface_fops_FS = 
{
  CDC_Init_FS,
  CDC_DeInit_FS,
  CDC_Control_FS,  
  CDC_Receive_FS
};
```
