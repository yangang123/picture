# syslog

nuttx系统log输出接口有三种方式:字符设备，/dev/console，ramlog

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
# 总结
syslog和printf一样有1个流的设计，他们最大区别，syslog是内核使用的log输出，syslog输出也可以是ramlog输出， printf是应用程序调用调试输出的


