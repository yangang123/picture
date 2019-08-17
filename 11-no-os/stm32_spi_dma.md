
# 资源
> stm32_spi_dma.md
> 闫刚 stm32-spi-DMA使用注意

# Stm32的spi的DMA过程
<div align="center">
<p>  安装 </p> 
<img src="https://github.com/yangang123/picture/raw/master/11-no-os/resource/stm32_spi_dma.png" height="720" width="1536" > 
</div>

1. DMA先把数据发送到SPI1->DR, TXE位被清除
2. 最后1个TXE,不会被DMA清除，发送完成数据，并不是接收最后1个字节发送完成
3. DMA为什么连续，不是TC能做到的，需要TXE把数据提前准备好
4. 发送DMA和DMA都需要配置, 仅仅DMA接收完成作为，作为完成表示，DMA发送完成标志需要手动清除.  无需管TXE, RNXE的标志

# spi死机的原因
出现spi死机的问题，我们需要打开DMA2时钟
<div >
<img src="https://github.com/yangang123/picture/raw/master/11-no-os/resource/spi_dma_init.png" height="720" width="1536" > 
</div>

