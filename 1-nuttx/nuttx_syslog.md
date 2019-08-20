@[TOC]
# syslog

# 总结
syslog和printf一样有1个流的设计，他们最大区别，syslog是内核使用的log输出，syslog输出也可以是ramlog输出， printf是应用程序调用调试输出的

# 控制台/dev/console
> 所有进程都是依赖/dev/console所以至少有1个控制台，下面的控制台都是独立，usb作为/dev/ttyACM0和/dev/console
> 没有任何关系
- USB
- serial
- RAMLOG
- syslog
  
```c
ret = syslog_dev_initialize("/dev/console", OPEN_FLAGS, OPEN_MODE);
return register_driver("/dev/console", &g_ramlogfops, 0666, priv);
(void)uart_register("/dev/console", &uart_devs[CONSOLE_UART - 1]->dev);
ret = uart_register("/dev/yangang", &priv->serdev);
```

# cpu刚上电
# 内核空间
# 用户空间

# cpu刚上电
> 在os_start调用os_bringup上电的时候之前建议使用up_putc进行调试

```
DEBUGVERIFY(os_bringup());
```

## up_putc
> up_putc是依赖CONSOLE_UART，所以要使用串口控制台
```
int up_putc(int ch)
{
#if CONSOLE_UART > 0
  struct up_dev_s *priv = uart_devs[CONSOLE_UART - 1];
  uint16_t ie;

  up_disableusartint(priv, &ie);

  /* Check for LF */

  if (ch == '\n')
    {
      /* Add CR */

      up_lowputc('\r');
    }

  up_lowputc(ch);
  up_restoreusartint(priv, ie);
#endif
  return ch;
}
```

## up_lowput
>up_lowputc输出1个字符
```
void up_lowputc(char ch)
{
#ifdef HAVE_CONSOLE
  /* Wait until the TX data register is empty */

  while ((getreg32(STM32_CONSOLE_BASE + STM32_USART_ISR_OFFSET) & USART_ISR_TXE) == 0);
#ifdef STM32_CONSOLE_RS485_DIR
  stm32_gpiowrite(STM32_CONSOLE_RS485_DIR, STM32_CONSOLE_RS485_DIR_POLARITY);
#endif

  /* Then send the character */

  putreg32((uint32_t)ch, STM32_CONSOLE_BASE + STM32_USART_TDR_OFFSET);

#ifdef STM32_CONSOLE_RS485_DIR
  while ((getreg32(STM32_CONSOLE_BASE + STM32_USART_ISR_OFFSET) & USART_ISR_TC) == 0);
  stm32_gpiowrite(STM32_CONSOLE_RS485_DIR, !STM32_CONSOLE_RS485_DIR_POLARITY);
#endif

#endif /* HAVE_CONSOLE */
}
```

## 输出信息
```
static int uart_open(FAR struct file *filep)
{
  FAR struct inode *inode = filep->f_inode;
  FAR uart_dev_t   *dev   = inode->i_private;
  uint8_t           tmp;
  int               ret;
  up_lowputc('d');
  up_lowputc('s');
```

```
ds                                                                                                                                                       
  ds                                                                                                                                                     
    ds                                                                                                                                                   
      ds                                                                                                                                                   
```

# 内核空间
>nuttx系统log输出接口有三种方式:字符设备，/dev/console，ramlog
> 全局变量g_default_channel

## 默认的全局变量g_default_channel是syslog的输出通道
```
#if defined(CONFIG_RAMLOG_SYSLOG)
const struct syslog_channel_s g_default_channel =
{
  ramlog_putc,
  ramlog_putc,
  syslog_default_flush
};
#elif defined(HAVE_LOWPUTC)
const struct syslog_channel_s g_default_channel =
{
  up_putc,
  up_putc,
  syslog_default_flush
};
#else
const struct syslog_channel_s g_default_channel =
{
  syslog_default_putc,
  syslog_default_putc,
  syslog_default_flush
};
#endif
```
```
int syslog_initialize(enum syslog_init_e phase)
{
  int ret = OK;

#if defined(CONFIG_SYSLOG_CHAR)
  if (phase == SYSLOG_INIT_LATE)
      ret = syslog_dev_channel();


#elif defined(CONFIG_RAMLOG_SYSLOG)
      ret = ramlog_syslog_channel();
    }

#elif defined(CONFIG_SYSLOG_CONSOLE)
      ret = syslog_console_channel();
    }

#endif
```

## syslog调试fatfs

## make menuconfig进行配置

>CONFIG_DEBUG_ERROR=y
>CONFIG_DEBUG_WARN=y
>CONFIG_DEBUG_INFO=y
>CONFIG_DEBUG_ASSERTIONS=y

```
#ifdef CONFIG_DEBUG_FS_ERROR
#  define ferr(format, ...)     _err(format, ##__VA_ARGS__)
#else
#  define ferr(x...)
#endif

#ifdef CONFIG_DEBUG_FS_WARN
#  define fwarn(format, ...)   _warn(format, ##__VA_ARGS__)
#else
#  define fwarn(x...)
#endif

#ifdef CONFIG_DEBUG_FS_INFO
#  define finfo(format, ...)   _info(format, ##__VA_ARGS__)
#else
```

## dmesg 输出配置
```
CONFIG_RAMLOG=y             : Enable the RAM-based logging feature.
CONFIG_RAMLOG_CONSOLE=n     : (We don't use the RAMLOG console)
CONFIG_RAMLOG_SYSLOG=y      : This enables the RAM-based logger as the
                              system logger.
CONFIG_RAMLOG_NONBLOCKING=y : Needs to be non-blocking for dmesg
CONFIG_RAMLOG_BUFSIZE=16384 : Buffer size is 16KiB
```

## 字符设备输出

```
# CONFIG_USART3_SERIAL_CONSOLE is not set
CONFIG_USART6_SERIAL_CONSOLE=y

-CONFIG_SYSLOG_SERIAL_CONSOLE=y
-# CONFIG_SYSLOG_CHAR is not set
# CONFIG_SYSLOG_INTBUFFER is not set
# CONFIG_SYSLOG_TIMESTAMP is not set
# CONFIG_SYSLOG_SERIAL_CONSOLE is not set
CONFIG_SYSLOG_CHAR=y
# CONFIG_RAMLOG_SYSLOG is not set
# CONFIG_SYSLOG_CONSOLE is not set
# CONFIG_SYSLOG_NONE is not set
# CONFIG_SYSLOG_FILE is not set
# CONFIG_CONSOLE_SYSLOG is not set
CONFIG_SYSLOG_CHAR_CRLF=y
CONFIG_SYSLOG_DEVPATH="/dev/ttyS2"
```


## mkfatfs代码通过syslog调试

>CONFIG_DEBUG_FEATURES=y
>CONFIG_DEBUG_FS_ERROR=y
>CONFIG_DEBUG_FS_WARN=y
>CONFIG_DEBUG_FS_INFO=y

```c
int mkfatfs(FAR const char *pathname, FAR struct fat_format_s *fmt)
{
  struct fat_var_s var;
  int ret;

  /* Initialize */

  memset(&var, 0, sizeof(struct fat_var_s));

  /* Get the filesystem creation time */

  var.fv_createtime = fat_systime2fattime();

  /* Verify format options (only when DEBUG enabled) */

#ifdef CONFIG_DEBUG_FEATURES
  if (!pathname)
    {
      ferr("ERROR: No block driver path\n");
      ret = -EINVAL;
      goto errout;
    }

  if (fmt->ff_nfats < 1 || fmt->ff_nfats > 4)
    {
      ferr("ERROR: Invalid number of fats: %d\n", fmt->ff_nfats);
      ret = -EINVAL;
      goto errout;
    }

  if (fmt->ff_fattype != 0  && fmt->ff_fattype != 12 &&
      fmt->ff_fattype != 16 && fmt->ff_fattype != 32)
    {
      ferr("ERROR: Invalid FAT size: %d\n", fmt->ff_fattype);
      ret = -EINVAL;
      goto errout;
    }
#endif
```

# 调试输出方案

## 调试最优方案1
>os_start之前: up_putc
>kernel+driver: /dev/tty2
>user space: /dev/console

## 调试方案2: 
>os_start之前: up_putc
>kernel+driver: /dev/console
>user space: /dev/console

## 程序稳定
>os_start之前: 没有
>kernel+driver: 没有
>user space: /dev/console