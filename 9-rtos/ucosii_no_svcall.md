# svcall
svcall是CPU的异常处理，主要负责非特权级别的应用程序对特权级别程序的访问。在linux中就是应用程序通过svc调用内核接口，如果open(....), read(....), write(....)

## ucosii中svcall
所以uCOS操作系统的实现不是典型的Client/Server模型，没有使用svcall中断
所以应用程序完全可以破坏OS的堆栈，导致系统崩溃，各个应用程序也可以相互破坏各自的堆栈。


## linux下的svcall中断
linux 1.0内核中用到的系统调用入下
```
static inline _syscall0(int,idle)
static inline _syscall0(int,fork)
static inline _syscall0(int,pause)
static inline _syscall1(int,setup,void *,BIOS)
static inline _syscall0(int,sync)
static inline _syscall0(pid_t,setsid)
static inline _syscall3(int,write,int,fd,const char *,buf,off_t,count)
static inline _syscall1(int,dup,int,fd)
static inline _syscall3(int,execve,const char *,file,char **,argv,char **,envp)
static inline _syscall3(int,open,const char *,file,int,flag,int,mode)
static inline _syscall1(int,close,int,fd)
static inline _syscall1(int,_exit,int,exitcode)
static inline _syscall3(pid_t,waitpid,pid_t,pid,int *,wait_stat,int,options)
```