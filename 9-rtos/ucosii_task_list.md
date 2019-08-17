
# UCOSII TCB 
TCB是UCOSii的任务控制块,是一块静态的内存空间，通过双向链表进行管理

## TCB结构
> TCB的第一个成员是栈指针[好设计]
> TCB是一个双向链表

typedef struct os_tcb {
    OS_STK          *OSTCBStkPtr;           /* Pointer to current top of stack                         */
    struct os_tcb   *OSTCBNext;             /* Pointer to next     TCB in the TCB list                 */
    struct os_tcb   *OSTCBPrev;             /* Pointer to previous TCB in the TCB list                 */

## TCB的内存空间
> TCB的空间是任务的数量OS_MAX_TASKS+系统任务的数量OS_N_SYS_TASKS
OS_EXT  OS_TCB            OSTCBTbl[OS_MAX_TASKS + OS_N_SYS_TASKS];   /* Table of TCBs                  */


## TCB链表的初始化
>  OSTCBList指向空指针
>  OSTCBFreeList指向第一个TCB
static  void  OS_InitTCBList (void)
   for (ix = 0u; ix < (OS_MAX_TASKS + OS_N_SYS_TASKS - 1u); ix++) {    /* Init. list of free TCBs     */
        ix_next =  ix + 1u;
        ptcb1   = &OSTCBTbl[ix];
        ptcb2   = &OSTCBTbl[ix_next];
        ptcb1->OSTCBNext = ptcb2;
    } 
    OSTCBList               = (OS_TCB *)0;            
    OSTCBFreeList           = &OSTCBTbl[0];

## TCB重要的指针
#define OS_LOWEST_PRIO 63
OS_EXT  OS_TCB           *OSTCBCur;           //指向当前任务            
OS_EXT  OS_TCB           *OSTCBFreeList;      //指向空闲TCB的头             
OS_EXT  OS_TCB           *OSTCBHighRdy;       //指向最高等待的任务             
OS_EXT  OS_TCB           *OSTCBList;          //指向最后1个被创建的任务           
OS_EXT  OS_TCB            OSTCBTbl[OS_MAX_TASKS + OS_N_SYS_TASKS];  //任务的内存空间
OS_EXT  OS_TCB           *OSTCBPrioTbl[OS_LOWEST_PRIO + 1u];  //通过优先级去找到指针的地址