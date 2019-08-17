@[TOC]
# 说明
使用链表实现1个队列

# 资源
> list_queue.md 

# 数据结构
```cpp
#define QUEUE_MAX_LEN  10
typedef struct  {
    int data;
    struct node *next;
}node_t;

typedef struct {
    int number;
    node_t *head;
    node_t *tail;
}queue_t;
```
# API接口
创建1个队列
queue_t *queue_create(void)

判断队列是否是空
bool queue_empty(queue_t *queue)

添加数据到队列中
int queue_add(queue_t *queue, int data)

删除队列中的数据
int queue_delete(queue_t *queue, int data)

写1个测试函数
int main(int argc, char **argv)


# queue_empty()判断队列中是否有数据

## 方案1
说明： 用head是否和tail相等
优点: 不需要独立的变量存储队列的长度
缺点: 最后1个节点不能释放，由于条件head和tail相等是成立的，但是节点也是存在
代码
```cpp
bool queue_empty(queue_t *queue) 
{
    return queue->head == queue->tail;
}
```

其他代码
```cpp
#include <stdio.h>
#include <stdbool.h>

#define QUEUE_MAX_LEN  10


typedef struct  {
    int data;
    struct node *next;
}node_t;

typedef struct {
    int size;
    node_t *head;
    node_t *tail;
}queue_t;


queue_t *queue_create(void)
{
    queue_t *queue;

    queue = malloc(sizeof(queue_t));
    if ( !queue) {
        return NULL;
    } 
    queue->size = 0;
    queue->head = NULL;
    queue->tail = NULL;
     
    return queue;
}

//队列是否是空
bool queue_empty(queue_t *queue) 
{
    return queue->head == queue->tail;
}

//添加1个节点的数据data
int queue_add(queue_t *queue, int data)
{
    node_t *node;

    node = malloc(sizeof(node_t));
    if (!node) {
        return -1;
    }
    printf("node address:0x%x\n", node);
    node->data = data;
    node->next = NULL;
    if (!queue->head) {
        queue->head = node;
    } else {
        queue->tail->next = node;
    }
    queue->tail = node;
    
    queue->size++;
}

//从队列的
int queue_delete(queue_t *queue)
{
    int data = 0;

    if (queue_empty(queue)) {
        return -1;
    }
    //获取节点的数据
    data = queue->head->data; 
    
    //释放节点
    node_t *node = queue->head;
    if ( queue->head ==  queue->tail) {
        queue->tail = NULL;
    }
    queue->head = node->next;
    free(node); 
    queue->size--;

    return data;
}

int main(){
	int data[10]={1,2,3,4,5,6,7,8,9,10};
	int i=0;
    int read_data[10] = {0};

    queue_t *queue = queue_create();
    while(i < 5){
        queue_add(queue, data[i]);
        printf("input data queue[%d]:%d\n", i, data[i]);
        i++;
    }
    while(!queue_empty(queue))
          printf("output data queue：%d\n",queue_delete(queue)); 
}
```

输出结果：
向对列输入5个数据，仅仅读出1个数据，方案2来解决方案1的问题
```
input data queue[0]:1
input data queue[1]:2
input data queue[2]:3
input data queue[3]:4
input data queue[4]:5
output data queue：1
output data queue：2
output data queue：3
output data queue：4
```

## 方案2 
说明: 使用长度size进行判断
优点: 解决不好判断方案1误判的问题
缺点: 多使用1个字节size进行判断
代码
```cpp
bool queue_empty(queue_t *queue) 
{
    return queue->size == 0;
}
```

其他代码
```cpp
#include <stdio.h>
#include <stdbool.h>

#define QUEUE_MAX_LEN  10

typedef struct  {
    int data;
    struct node *next;
}node_t;

typedef struct {
    int size;
    node_t *head;
    node_t *tail;
}queue_t;


queue_t *queue_create(void)
{
    queue_t *queue;

    queue = malloc(sizeof(queue_t));
    if ( !queue) {
        return NULL;
    } 
    queue->size = 0;
    queue->head = NULL;
    queue->tail = NULL;
     
    return queue;
}

//队列是否是空
bool queue_empty(queue_t *queue) 
{
    return queue->size == 0;
}

//添加1个节点的数据data
int queue_add(queue_t *queue, int data)
{
    node_t *node;

    node = malloc(sizeof(node_t));
    if (!node) {
        return -1;
    }
    printf("node address:0x%x\n", node);
    node->data = data;
    node->next = NULL;
    if (!queue->head) {
        queue->head = node;
    } else {
        queue->tail->next = node;
    }
    queue->tail = node;
    
    queue->size++;
}

//从队列的
int queue_delete(queue_t *queue)
{
    int data = 0;

    if (queue_empty(queue)) {
        return -1;
    }
    //获取节点的数据
    data = queue->head->data; 
    
    //释放节点
    node_t *node = queue->head;
    if ( queue->head ==  queue->tail) {
        queue->tail = NULL;
    }
    queue->head = node->next;
    free(node); 
    queue->size--;

    return data;
}

int main(){
	int data[10]={1,2,3,4,5,6,7,8,9,10};
	int i=0;
    int read_data[10] = {0};

    queue_t *queue = queue_create();
    while(i < 5){
        queue_add(queue, data[i]);
        printf("input data queue[%d]:%d\n", i, data[i]);
        i++;
    }

    while(!queue_empty(queue))
          printf("output data queue：%d\n",queue_delete(queue)); 
  
}
```

输出结果：
```
node address:0xa64f5280
input data queue[0]:1
node address:0xa64f56b0
input data queue[1]:2
node address:0xa64f56d0
input data queue[2]:3
node address:0xa64f56f0
input data queue[3]:4
node address:0xa64f5710
input data queue[4]:5
output data queue：1
output data queue：2
output data queue：3
output data queue：4
output data queue：5
```

