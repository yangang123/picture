@[TOC]
# 资源
> nuttx_fread_fwrite.md

# 简介
> fwrite， write，printf的实现原理
```c
#include 	<stdio.h>
#include	<unistd.h>
#include	<string.h>
#include	<errno.h>

int main(int argc,char**argv){
    char buf[]="hello,world\n";
    
    //使用'printf' 进行打印
    printf("%s",buf);
     
    //使用'fwrite' 进行打印
    fwrite(buf,strlen(buf), 1,stdout);
    
    //use 'write' function to print char 
    write(STDOUT_FILENO,&buf,strlen(buf));
    
    return(0);
}
```
执行结果
```c
hello,world
hello,world
hello,world 
```

# fread, fwrite基础知识
>在unix下对文件的操作，分别是fopen, fread, fwrite， 另一个是open, read, write， 它们的区别是， fopen 系列是标准的C库函数；open系列是 POSIX 定义的，是UNIX系统里的system call文件句柄(file handles) ，也称文件结构指针， 文件描述符（file descriptors)是一个整形变量, 我们常知道的， stdout(标准输出), stdin(标准输入), stderr（标准错误），其实都是文件句柄，每一个文件句柄都会对应1个文件描述符，stdout， stdin， stderr，正好对应文件描述0，1，2fread和fwrite接口NAMEfread, fwrite - binary stream input/output
SYNOPSIS
```c
#include <stdio.h>
size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);
size_t fwrite(const void *ptr, size_t size, size_t nmemb,
              FILE *stream);
```

# 3.1 nuttx的配置选项
```
# CONFIG_STDIO_DISABLE_BUFFERING is not set
CONFIG_STDIO_BUFFER_SIZE=32   #配置文件的缓冲区大小
CONFIG_STDIO_LINEBUFFER=y     #是否打开文件缓冲
```

# 3.2 nuttx的文件句柄和文件描述符的定义
## 3.2.1 file_struct文件结构
>文件结构体文件位置NuttX/nuttx/include/nuttx/fs/fs.h， 文件结构中，包含了描述文件的缓冲区，这个是文件结构的很大的特点
```c
#if CONFIG_NFILE_STREAMS > 0
struct file_struct
{
  int                fs_fd;        //绑定的文件描述符
#ifndef CONFIG_STDIO_DISABLE_BUFFERING
  sem_t              fs_sem;       /* For thread safety */
  pid_t              fs_holder;    /* Holder of sem */
  int                fs_counts;    /* Number of times sem is held */
  FAR unsigned char *fs_bufstart;  //执行缓冲区的头部，缓冲是f_open动态分配的
  FAR unsigned char *fs_bufend;    /* Pointer to 1 past end of buffer */
  FAR unsigned char *fs_bufpos;    /* Current position in buffer */
  FAR unsigned char *fs_bufread;   /* Pointer to 1 past last buffered read char. */
#endif
  uint16_t           fs_oflags;    /* Open mode flags */
  uint8_t            fs_flags;     /* Stream flags */
#if CONFIG_NUNGET_CHARS > 0
  uint8_t            fs_nungotten; /* The number of characters buffered for ungetc */
  unsigned char      fs_ungotten[CONFIG_NUNGET_CHARS];
#endif
};
```
## 3.2.2 file_opreation框架
>文件描述符的定义linux的每个文件描述符，都绑定了1个静态的file opreation的文件接口，绑定了f_priv驱动的对象，操作底层的寄存器
```c
struct file
{
  int               f_oflags;   /* Open mode flags */
  off_t             f_pos;      /* File position */
  FAR struct inode *f_inode;    /* Driver or file system interface */
  void             *f_priv;     /* Per file driver private data */
};

/* This defines a list of files indexed by the file descriptor */

#if CONFIG_NFILE_DESCRIPTORS > 0
struct filelist
{
  sem_t   fl_sem;               /* Manage access to the file list */
  struct file fl_files[CONFIG_NFILE_DESCRIPTORS];
};
```

## 3.2.3 标准输入，标准输出，标准错误
> 标准输入输出的流这个文件句柄，是每个线程创建的时候，自动创建的，自动绑定到文件描述符0，1，2上
```c
#define stdin      (&sched_getstreams()->sl_streams[0])
#define stdout     (&sched_getstreams()->sl_streams[1])
#define stderr     (&sched_getstreams()->sl_streams[2])
```

## 3.2.4 posix的标准接口定义
```c
struct file_operations
{
  /* The device driver open method differs from the mountpoint open method */

  int     (*open)(FAR struct file *filep);

  /* The following methods must be identical in signature and position because
   * the struct file_operations and struct mountp_operations are treated like
   * unions.
   */

  int     (*close)(FAR struct file *filep);
  ssize_t (*read)(FAR struct file *filep, FAR char *buffer, size_t buflen);
  ssize_t (*write)(FAR struct file *filep, FAR const char *buffer, size_t buflen);
  off_t   (*seek)(FAR struct file *filep, off_t offset, int whence);
  int     (*ioctl)(FAR struct file *filep, int cmd, unsigned long arg);

  /* The two structures need not be common after this point */

#ifndef CONFIG_DISABLE_POLL
  int     (*poll)(FAR struct file *filep, struct pollfd *fds, bool setup);
#endif
#ifndef CONFIG_DISABLE_PSEUDOFS_OPERATIONS
  int     (*unlink)(FAR struct inode *inode);
#endif
```

# 3.3 fopen接口实现过程
```c
FAR FILE *fopen(FAR const char *path, FAR const char *mode)
    //打开文件描述符
    fd = open(path, oflags, 0666);
        
        //把文件句柄绑定到os中
        -> ret = fs_fdopen(fd, oflags, NULL); 
        
        //动态文件缓冲区, 确定start和end的位置, 
        stream->fs_bufstart =	group_malloc(tcb->group, CONFIG_STDIO_BUFFER_SIZE);
        stream->fs_bufend  = &stream->fs_bufstart[CONFIG_STDIO_BUFFER_SIZE];
        stream->fs_bufpos  = stream->fs_bufstart;
        stream->fs_bufread = stream->fs_bufstart;
        
        //绑定文件描述到os的流中
        stream->fs_fd      = fd;  
};
```

# 3.4 fwrite接口的实现
```c
size_t fwrite(FAR const void *ptr, size_t size, size_t n_items, FAR FILE *stream)
    ->  size_t  full_size = n_items * (size_t)size;
	  ->  bytes_written = lib_fwrite(ptr, full_size, stream); //写入全部字节

      //先数据写到文件缓存区中
      for (dest = stream->fs_bufpos; gulp_size > 0; gulp_size--)
        *dest++ = *src++;
      
      //缓冲区满了，才写入到写到真正的设备上
      if (dest >= stream->fs_bufend)
          int bytes_buffered = lib_fflush(stream, false);
            ->   bytes_written = write(stream->fs_fd, src, nbuffer);3.3.3 fread实现过程  size_t fread(FAR void *ptr, size_t size, size_t n_items, FAR FILE *stream)
         bytes_read = lib_fread(ptr, full_size, stream);
        
        //如果缓冲区有数据，直接先从缓冲区拿到数据
        while ((count > 0) && (stream->fs_bufpos < stream->fs_bufread))
            *dest++ = *stream->fs_bufpos++;
            count--;
          }
        
          buffer_available = stream->fs_bufend - stream->fs_bufread;

        //如果数据不够,就直接从文件中读取
        -> if (count > buffer_available)
            bytes_read = read(stream->fs_fd, dest, count);

        //如果需要的数据，小于缓冲区容纳的大小，那么读取整个缓冲区的大小
        ->   if (count 《 buffer_available)
          bytes_read = read(stream->fs_fd, dest, buffer_available);3.4 printf接口实现printf的最底层函数是up_putc() 这个函数，就是smt32阻塞等待发送完成，而且，这个up_putc可以被任何地方调用int printf(FAR const IPTR char *fmt, ...)
          int vfprintf(FAR FILE *stream, FAR const IPTR char *fmt, va_list ap)
                ->lib_stdoutstream(&stdoutstream, stream);
                  ->  outstream->public.put = stdoutstream_putc;
                    ->  result = fputc(ch, sthis->stream);
                        -> ret = lib_fwrite(&buf, 1, stream); // 下面就和fwrite的实现就一样了
                          ->  if (dev->isconsole)
                          -> ret = uart_irqwrite(dev, buffer, buflen);
                            ->uart_putc('\r'); //  #define uart_putc(ch) up_putc(ch)     
                                  -> void up_lowputc(char ch)
                                      -> while ((getreg32(CONSOLE_BASE+A1X_UART_LSR_OFFSET) & UART_LSR_THRE) == 0);
                                      putreg32((uint32_t)ch, CONSOLE_BASE+A1X_UART_THR_OFFSET);
                -> n = lib_vsprintf(&stdoutstream.public, fmt, ap);
                      ->   obj->put(obj, FMT_CHAR);
                  if (FMT_CHAR == '\n')
                      (void)obj->flush(obj); 
```              
# 3. 5 文件系统原理
>文件系统挂载文件系统的挂载是我一直疑惑的问题，这此终于明白了文件系统的挂载的问题，块设备和字符设备的区别在于，字符设备驱动注册设备节点，同时open, write可以直接操作字符设备， 而块设备的注册，并不是直接给write用的，而是给文件系统用的，文件系统注册到挂载点上，挂载点去操作文件，块设备，不像字符设备驱动一样，可以单个字节处理，而flash的特性是扇区处理，一般SD卡扇区是512字节
```c
union inode_ops_u
{
  FAR const struct file_operations     *i_ops;    /* Driver operations for inode *//
  FAR const struct mountpt_operations  *i_mops;   /* Operations on a mountpoint */
};

//挂载文件系统
if mount -t vfat /dev/mmcsd0 /fs/microsd

//mount.c 
mountpt_inode->u.i_mops  = mops; //文件系统和设备节点挂载好
```
# 3.6 sd卡文件的write操作这个sd卡文件的操作流程
> posix的write调用文件系统fat_write，再调用驱动的mmcsd_writessize_t write(int fd, FAR const void *buf, size_t nbytes)
```c
//fat文件系统接口的write
    ret = fat_hwwrite(fs, userbuffer, ff->ff_currentsector, nsectors);
      //调用mmcsd卡的驱动接口
      ->  ssize_t nSectorsWritten =
            inode->u.i_bops->write(inode, buffer, sector, nsectors);
            ->mmcsd_write,/* mmcsd 写*/
```
# 总结
通过以上的分析，我们的fread和fwrite，还用printf都是带缓冲，fwrite写的内容被刷新到sd卡的条件是，缓冲区满了，才会写到物理sd, fread是每次从物理扇区读取缓冲区的数据，比如，缓冲是32字节,fread要获取10字节，fread也直接读取32字节，缓冲到文件的缓冲区了，剩下的22字节，下次如果用户调用fread的时候，读取10字节，10字节小于缓冲区剩余的22字节，可以直接从缓冲区读走，这样可以提高缓冲区的大小。printf具有字节独立的特性，由于printf处理的都是字符，那么高效的处理方法就是判断是否接收到回车符号，就可以刷新缓冲区，而fgetc是，等待接收的数据中有了回车符号，才进行处理。块设备  【fwrite【c库】->write【系统调用】->fat_write【文件系统】->mmcsd_write【SD驱动】】字符设备 【fwrite【c库】->write【系统调用】->serial_write【串口驱动】】