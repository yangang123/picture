# svcall
svcall��CPU���쳣������Ҫ�������Ȩ�����Ӧ�ó������Ȩ�������ķ��ʡ���linux�о���Ӧ�ó���ͨ��svc�����ں˽ӿڣ����open(....), read(....), write(....)

## ucosii��svcall
����uCOS����ϵͳ��ʵ�ֲ��ǵ��͵�Client/Serverģ�ͣ�û��ʹ��svcall�ж�
����Ӧ�ó�����ȫ�����ƻ�OS�Ķ�ջ������ϵͳ����������Ӧ�ó���Ҳ�����໥�ƻ����ԵĶ�ջ��


## linux�µ�svcall�ж�
linux 1.0�ں����õ���ϵͳ��������
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