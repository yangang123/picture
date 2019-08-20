@[TOC]
# 资源
> 文件名: px4_ppm.md
> 标题  : 闫刚 ppm定时+单通道捕获中断

## 1. px4 io板的PPM配置
```c
board/px4io/board_config.h

//配置PPM采集的定时器
#define HRT_TIMER		1	/* use timer1 for the HRT */
//配置PPM采集定时器的通道，通常是CCR
#define HRT_PPM_CHANNEL		1	/* use capture/compare channel 1 PA8 */
//配置PPM的输入管脚
#define GPIO_PPM_IN		(GPIO_ALT|GPIO_CNF_INPULLUP|GPIO_PORTA|GPIO_PIN8)
```

## 2. 采集过程
```c
/** PPM decoder state machine */
struct {
	uint16_t	last_edge;	/**< last capture time */
	uint16_t	last_mark;	/**< last significant edge */
	uint16_t	frame_start;	/**< the frame width */
	unsigned	next_channel;	/**< next channel index */
	 
          enum {
		UNSYNCH = 0,
		ARM,
		ACTIVE,
		INACTIVE
	} phase;
} ppm;

//采集状态机

enum {
		UNSYNCH = 0, 
		ARM, //已经捕获到PPM的信息
		ACTIVE,
		INACTIVE
	} phase
```
## 3.  函数调用

```
hrt_tim_isr(int irq, void *context, void *arg)
   -> hrt_ppm_decode(status);
      -> 	uint16_t count = rCCR_PPM; //读取寄存器的值
      -> if (width >= PPM_MIN_START) { 
        ppm.phase = ARM;
        ppm_buffer[i] = ppm_temp_buffer[i]; //pwm值
        }
       -> case case UNSYNCH:

        case ARM:
        case INACTIVE:

controls.c 
      static bool

ppm_input(uint16_t *values, uint16_t *num_values, uint16_t *frame_len)
{       
        //原子操作，对和中断共享的数据，进行保护
        irqstate_t state = px4_enter_critical_section();
        //外部做超时判断
        if (hrt_elapsed_time(&ppm_last_valid_decode) < 200000) { 
            values[i] = ppm_buffer[i];

        }
        px4_leave_critical_section(state);

}
```