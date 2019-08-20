

1. ����ÿ��SPI����;

SPI1: ������
SPI2: �洢��

SPI4: ��ѹ��
SPI6: �ⲿ��չ��SPI

/* Define CS GPIO array */
static const uint32_t spi1selects_gpio[] = PX4_SENSOR_BUS_CS_GPIO;
static const uint32_t spi2selects_gpio[] = PX4_MEMORY_BUS_CS_GPIO;
#ifdef CONFIG_STM32F7_SPI3
static const uint32_t spi3selects_gpio[] = {FIXME};
#error Need to define SPI3 Usage
#endif
static const uint32_t spi4selects_gpio[] = PX4_BARO_BUS_CS_GPIO;
#ifdef CONFIG_STM32F7_SPI3
static const uint32_t spi5selects_gpio[] = {FIXME};;
#error Need to define SPI5 Usage
#endif
static const uint32_t spi6selects_gpio[] = PX4_EXTERNAL_BUS_CS_GPIO;


2. 
stm32_boardinitialize(void) //���ӳ�ʼ��
	 ->__EXPORT void stm32_spiinitialize(void)
		 -> stm32_configgpio(spi1selects_gpio[cs]);   //����spi����Ƭѡio���豸
		 -> stm32_gpiowrite(GPIO_SPI_CS_MPU9250, 1);  //
		_EXPORT void board_spi_reset(int ms)
			-> #ifdef CONFIG_STM32F7_SPI1             //����������SPI1�����IO�豸

__EXPORT int board_app_initialize(uintptr_t arg)      // ����APP����
	-> int ret = stm32_spi_bus_initialize();          // ����spi�����豸
			-> FAR struct spi_dev_s *stm32_spibus_initialize(int bus)
				-> stm32_configgpio(GPIO_SPI1_SCK);
				-> spi_bus_initialize(priv);           //spiӲ����ʼ��
					->spi_setfrequency((FAR struct spi_dev_s *)priv, 400000);
					                                   //����ʱ��
						->if (frequency >= priv->spiclock >> 1)
						  setbits = SPI_CR1_FPCLCKd2; /* 000: fPCLK/2 */
						  actual = priv->spiclock >> 1; //ʱ�Ӱ���ʵ�ʵĽ�������
						  ���磺 2��Ƶ��18MHZ, 4��Ƶ��9MHZ,���ʱ����10Mhz,��ôʵ�ʾ���9Mhz
    
//--------------------------------------------------------------------------------
stm32_boardinitialize(void)
	->stm32_spiinitialize();/* configure SPI interfaces */
	->up_ledinit();  /* configure LEDs */

	
theone�̼��еĴ��������ù���
__EXPORT int nsh_archinitialize(void)
    //-------------------------��ʼ����������Ƭѡio
    SPI_SELECT(spi1, PX4_SPIDEV_GYRO, false);
	SPI_SELECT(spi1, PX4_SPIDEV_ACCEL_MAG, false);
	SPI_SELECT(spi1, PX4_SPIDEV_BARO, false);
	SPI_SELECT(spi1, PX4_SPIDEV_MPU, false);
	
  ->spi1 = up_spiinitialize(1);
	->SPI_SETFREQUENCY(spi1, 10000000);
	->SPI_SETMODE(spi1, SPIDEV_MODE3);
  ->spi2 = up_spiinitialize(2);
    ->SPI_SETFREQUENCY(spi2, 12 * 1000 * 1000);
	->SPI_SETBITS(spi2, 8);
	->SPI_SETMODE(spi2, SPIDEV_MODE3);
  ->spi4 = up_spiinitialize(4);
    ->SPI_SETFREQUENCY(spi4, 10000000);
	->SPI_SETBITS(spi4, 8);
	->SPI_SETMODE(spi4, SPIDEV_MODE3);

//---------------------------------------------------------------------------------------
			
��ѹ�ƣ�
-------------------------------------------
MS5611_SPI on SPI bus 4 at 2 (11000 KHz)
ms5611: no device on bus 4
MS5611_SPI on SPI bus 1 at 3 (11000 KHz)
nsh> ms5611 test
ms5611: single read
ms5611: pressure:     1014.7300
ms5611: altitude:       -12.3124
ms5611: temperature:  33.6500
ms5611: time:        61172104
ms5611: periodic read 0
ms5611: pressure:     1014.7200
ms5611: altitude:       -12.2293
ms5611: temperature:  33.6400
ms5611: time:        61220267
ms5611: periodic read 1
ms5611: pressure:     1015.8200
ms5611: altitude:       -21.3711
ms5611: temperature:  33.6400
ms5611: time:        61723916
ms5611: periodic read 2
ms5611: pressure:     1016.0300
ms5611: altitude:       -23.1154
ms5611: temperature:  33.6400
ms5611: time:        62243489
ms5611: periodic read 3
ms5611: pressure:     1014.5900
ms5611: altitude:       -11.1484
ms5611: temperature:  33.0100
ms5611: time:        62757453
ms5611: periodic read 4
ms5611: pressure:     1014.6200
ms5611: altitude:       -11.3978
ms5611: temperature:  33.0100
ms5611: time:        63277232
ms5611: PASS


//����
mpu6000 test
mpu6000: single read
mpu6000: time:     231958753
mpu6000: acc  x:          0.3257        m/s^2
mpu6000: acc  y:          0.6594        m/s^2
mpu6000: acc  z:         -9.6141        m/s^2
mpu6000: acc  x:        140     raw 0x8c
mpu6000: acc  y:        278     raw 0x116
mpu6000: acc  z:        -4012   raw 0xf054
mpu6000: acc range:  78.4532 m/s^2 (  8.0000 g)
mpu6000: gyro x:          0.00247       rad/s
mpu6000: gyro y:          0.03431       rad/s
mpu6000: gyro z:          0.00302       rad/s
mpu6000: gyro x:        1       raw
mpu6000: gyro y:        34      raw
mpu6000: gyro z:        3       raw
mpu6000: gyro range:  34.9066 rad/s (2000 deg/s)
mpu6000: temp:           32.7091        deg celsius
mpu6000: temp:          -827    raw 0xfcc5


//����
l3gd20 test
l3gd20: gyro x:           0.00955       rad/s
l3gd20: gyro y:          -0.02599       rad/s
l3gd20: gyro z:          -0.01120       rad/s
l3gd20: temp:   48      C
l3gd20: gyro x:         7       raw
l3gd20: gyro y:         -22     raw
l3gd20: gyro z:         -6      raw
l3gd20: temp:   -8      raw
l3gd20: gyro range:  34.9066 rad/s (2000 deg/s)
l3gd20: PASS

//����
sm303d start
LSM303D on SPI bus 1 at 2 (11000 KHz)
nsh> lsm303d test
lsm303d: accel x:         1.44665       m/s^2
lsm303d: accel y:         1.88153       m/s^2
lsm303d: accel z:        -8.62554       m/s^2
lsm303d: accel x:       646     raw
lsm303d: accel y:       773     raw
lsm303d: accel z:       -3626   raw
lsm303d: accel range:  78.0000 m/s^2
lsm303d: accel antialias filter bandwidth: 30 Hz
lsm303d: mag device active: onboard
lsm303d: mag x:           0.60456       ga
lsm303d: mag y:          -0.03120       ga
lsm303d: mag z:           0.22112       ga
lsm303d: mag x:         7557    raw
lsm303d: mag y:         -390    raw
lsm303d: mag z:         2764    raw
lsm303d: mag range:   2.0000 ga

//-------------------------------------------------------------
ms5611 test
mpu6000 test
l3gd20 stop
lsm303d start
	-> LSM303D on SPI bus 1 at 2 (11000 KHz)
	-> L3GD20  on SPI bus 1 at 1 (11000 KHz)   
	-> MPU6000 on SPI bus 1 at 4 (1000 KHz)
	-> MS5611_SPI on SPI bus 4 at 2 (11000 KHz)  // ���ص�������1��.
	   ms5611: no device on bus 4
	   MS5611_SPI on SPI bus 1 at 3 (11000 KHz)
		
--------------------------------------------------------------------


theone��д�ľͲ���

// stm32_configgpio() ����IOģʽ
// stm32_gpiowrite(GPIO_SPI_CS_GYRO, 1); //IO����

__EXPORT void stm32_spiinitialize(void)
{
#ifdef CONFIG_STM32_SPI1
	stm32_configgpio(GPIO_SPI_CS_GYRO);
	stm32_configgpio(GPIO_SPI_CS_ACCEL_MAG);
	stm32_configgpio(GPIO_SPI_CS_BARO);
	stm32_configgpio(GPIO_SPI_CS_HMC);
	stm32_configgpio(GPIO_SPI_CS_MPU);

	/* De-activate all peripherals,
	 * required for some peripheral
	 * state machines
	 */
	stm32_gpiowrite(GPIO_SPI_CS_GYRO, 1);
	stm32_gpiowrite(GPIO_SPI_CS_ACCEL_MAG, 1);
	stm32_gpiowrite(GPIO_SPI_CS_BARO, 1);
	stm32_gpiowrite(GPIO_SPI_CS_HMC, 1);
	stm32_gpiowrite(GPIO_SPI_CS_MPU, 1);

	stm32_configgpio(GPIO_EXTI_GYRO_DRDY);
	stm32_configgpio(GPIO_EXTI_MAG_DRDY);
	stm32_configgpio(GPIO_EXTI_ACCEL_DRDY);
	stm32_configgpio(GPIO_EXTI_MPU_DRDY);
#endif

#ifdef CONFIG_STM32_SPI2
	stm32_configgpio(GPIO_SPI_CS_FRAM);
	stm32_gpiowrite(GPIO_SPI_CS_FRAM, 1);
#endif

#ifdef CONFIG_STM32_SPI4
	stm32_configgpio(GPIO_SPI_CS_EXT0);
	stm32_configgpio(GPIO_SPI_CS_EXT1);
	stm32_configgpio(GPIO_SPI_CS_EXT2);
	stm32_configgpio(GPIO_SPI_CS_EXT3);
	stm32_gpiowrite(GPIO_SPI_CS_EXT0, 1);
	stm32_gpiowrite(GPIO_SPI_CS_EXT1, 1);
	stm32_gpiowrite(GPIO_SPI_CS_EXT2, 1);
	stm32_gpiowrite(GPIO_SPI_CS_EXT3, 1);
#endif
}

mpu6000����������:  [����theone:: Firmware�̼�]
1. mpu6000���¹̼��ж������¶�����ͷ�ļ�����չ�������豸
2. 


//SPIƬѡIO����
#define GPIO_SPI_CS_GYRO	(GPIO_OUTPUT|GPIO_PUSHPULL|GPIO_SPEED_2MHz|GPIO_OUTPUT_SET|GPIO_PORTC|GPIO_PIN13)        ����PC13 L3GD20  ���� 
#define GPIO_SPI_CS_ACCEL_MAG	(GPIO_OUTPUT|GPIO_PUSHPULL|GPIO_SPEED_2MHz|GPIO_OUTPUT_SET|GPIO_PORTC|GPIO_PIN15)    ����PC15 LSM303D ���ټ�+������
#define GPIO_SPI_CS_BARO	(GPIO_OUTPUT|GPIO_PUSHPULL|GPIO_SPEED_2MHz|GPIO_OUTPUT_SET|GPIO_PORTD|GPIO_PIN7)         ����PD7  MS5611 ��ѹ��
#define GPIO_SPI_CS_MPU		(GPIO_OUTPUT|GPIO_PUSHPULL|GPIO_SPEED_2MHz|GPIO_OUTPUT_SET|GPIO_PORTC|GPIO_PIN2)         ����PC2  MPU6000��3�ᴫ����

��4��spi���豸
-------------------------------------------
#define PX4_SPIDEV_GYRO		    1 
#define PX4_SPIDEV_ACCEL_MAG	2  
#define PX4_SPIDEV_BARO		    3
#define PX4_SPIDEV_MPU		    4

------------------------------------------------------
#define SPI_SELECT(d,id,s) ((d)->ops->select(d,id,s))
spi1: 


// spi�ܹ�
struct spi_ops_s
{
  CODE int      (*lock)(FAR struct spi_dev_s *dev, bool lock);
  CODE void     (*select)(FAR struct spi_dev_s *dev, enum spi_dev_e devid,
                  bool selected);
  CODE uint32_t (*setfrequency)(FAR struct spi_dev_s *dev, uint32_t frequency);
#ifdef CONFIG_SPI_CS_DELAY_CONTROL
  CODE int      (*setdelay)(FAR struct spi_dev_s *dev, uint32_t a, uint32_t b,
                  uint32_t c);
#endif
  CODE void     (*setmode)(FAR struct spi_dev_s *dev, enum spi_mode_e mode);
  CODE void     (*setbits)(FAR struct spi_dev_s *dev, int nbits);
#ifdef CONFIG_SPI_HWFEATURES
  CODE int      (*hwfeatures)(FAR struct spi_dev_s *dev,
                  spi_hwfeatures_t features);
#endif
  CODE uint8_t  (*status)(FAR struct spi_dev_s *dev, enum spi_dev_e devid);
#ifdef CONFIG_SPI_CMDDATA
  CODE int      (*cmddata)(FAR struct spi_dev_s *dev, enum spi_dev_e devid,
                  bool cmd);
#endif
  CODE uint16_t (*send)(FAR struct spi_dev_s *dev, uint16_t wd);
#ifdef CONFIG_SPI_EXCHANGE
  CODE void     (*exchange)(FAR struct spi_dev_s *dev,
                  FAR const void *txbuffer, FAR void *rxbuffer,
                  size_t nwords);
#else
  CODE void     (*sndblock)(FAR struct spi_dev_s *dev,
                  FAR const void *buffer, size_t nwords);
  CODE void     (*recvblock)(FAR struct spi_dev_s *dev, FAR void *buffer,
                  size_t nwords);
#endif
  CODE int      (*registercallback)(FAR struct spi_dev_s *dev,
                  spi_mediachange_t callback, void *arg);
};


//spi�豸
static struct spi_dev_s *spi1;
static struct spi_dev_s *spi2;
static struct spi_dev_s *spi4;
static struct sdio_dev_s *sdio;

spi_sensors = stm32_spibus_initialize(PX4_SPI_BUS_SENSORS);
  -> //��ʼ��SPI�豸

  
  
//������������������:
#define PX4_SENSOR_BUS_CS_GPIO    
	 {GPIO_SPI_CS_ICM20689, GPIO_SPI_CS_ICM20602, GPIO_SPI_CS_BMI055_GYR, GPIO_SPI_CS_BMI055_ACC}


//nucleo����6���������� 
#define PX4_SPIDEV_GYRO			 PX4_MK_SPI_SEL(PX4_SPI_BUS_SENSORS,0)
#define PX4_SPIDEV_ACCEL_MAG	 PX4_MK_SPI_SEL(PX4_SPI_BUS_SENSORS,1)
#define PX4_SPIDEV_MPU			 PX4_MK_SPI_SEL(PX4_SPI_BUS_SENSORS,2)
#define PX4_SPIDEV_HMC			 PX4_MK_SPI_SEL(PX4_SPI_BUS_SENSORS,3)   �شŴ�����
#define PX4_SPIDEV_LIS           PX4_MK_SPI_SEL(PX4_SPI_BUS_SENSORS,4)
#define PX4_SPIDEV_BMI           PX4_MK_SPI_SEL(PX4_SPI_BUS_SENSORS,5)
#define PX4_SPIDEV_BMA           PX4_MK_SPI_SEL(PX4_SPI_BUS_SENSORS,6)

#define PX4_SENSOR_BUS_CS_GPIO   {0, 0, GPIO_SPI_CS_MPU9250, GPIO_SPI_CS_HMC5983, GPIO_SPI_CS_LIS3MDL, 0, 0} 
	 

	 
	 
#ifdef CONFIG_STM32F7_SPI1
static const struct spi_ops_s g_sp1iops =
{
  .lock              = spi_lock,
  .select            = stm32_spi1select,
  .setfrequency      = spi_setfrequency,
  .setmode           = spi_setmode,
  .setbits           = spi_setbits,
#ifdef CONFIG_SPI_HWFEATURES
  .hwfeatures        = spi_hwfeatures,
#endif
  .status            = stm32_spi1status,
#ifdef CONFIG_SPI_CMDDATA
  .cmddata           = stm32_spi1cmddata,
#endif
  .send              = spi_send,
#ifdef CONFIG_SPI_EXCHANGE
  .exchange          = spi_exchange,
#else
  .sndblock          = spi_sndblock,
  .recvblock         = spi_recvblock,
#endif
#ifdef CONFIG_SPI_CALLBACK
  .registercallback  = stm32_spi1register,  /* Provided externally */
#else
  .registercallback  = 0,                   /* Not implemented */
#endif
};  


#include <platforms/px4_getopt.h>

platforms�ļ������ض�����һЩnuttx�ĺ�


----------------------------------------------------
ms5611����
---------------------------------------------------
���¹̼�����  ms5611.cpp
�ɹ̼���:     ms5611_nuttx.cpp 

-----------------------------------------------------

1. ����1��MS5611����
	class MS5611 : public device::CDev{}

2. ʵ����ĳ�Ա����
	MS5611::MS5611(device::Device *interface, ms5611::prom_u &prom_buf, const char *path) :
	MS5611::~MS5611()

3. namespace ms5611
   {
	   struct ms5611_bus_option {
		enum MS5611_BUS busid;
		const char *devpath;
		MS5611_constructor interface_constructor;
		uint8_t busnum;
		MS5611 *dev;
	   }
   
   struct ms5611_bus_option bus_options[] = {} 
   
#if defined(PX4_SPIDEV_EXT_BARO) && defined(PX4_SPI_BUS_EXT)
	{ MS5611_BUS_SPI_EXTERNAL, "/dev/ms5611_spi_ext", &MS5611_spi_interface, PX4_SPI_BUS_EXT, NULL },
#endif
#ifdef PX4_SPIDEV_BARO
	{ MS5611_BUS_SPI_INTERNAL, "/dev/ms5611_spi_int", &MS5611_spi_interface, PX4_SPI_BUS_SENSORS, NULL },
#endif
#ifdef PX4_I2C_BUS_ONBOARD
	{ MS5611_BUS_I2C_INTERNAL, "/dev/ms5611_int", &MS5611_i2c_interface, PX4_I2C_BUS_ONBOARD, NULL },
#endif
#ifdef PX4_I2C_BUS_EXPANSION
	{ MS5611_BUS_I2C_EXTERNAL, "/dev/ms5611_ext", &MS5611_i2c_interface, PX4_I2C_BUS_EXPANSION, NULL },
#endif
};
   bus_options[]�ж��壺 �������Ǹ����ߣ��Ǹ�·�����Ǹ��ӿڣ���������
   
   3. 1 enum MS5611_BUS busid; ����id
		*devpath             : ·��
		MS5611_constructor interface_constructor;  ���캯��
		uint8_t busnum;                            ���ߺ�
		MS5611 *dev;                               �豸����
   
   start();
   test();
   reset();
   info();
   calibrate();
   usage();
   
   ����ms5611�ļ����豸������ʵ������֧��shell���ƣ�

4. ms5611����2���豸�ڵ�
   baro0  
   ms5611_spi_int
 
5.  enum MS5611_BUS {
		MS5611_BUS_ALL = 0,
		MS5611_BUS_I2C_INTERNAL,
		MS5611_BUS_I2C_EXTERNAL,
		MS5611_BUS_SPI_INTERNAL,
		MS5611_BUS_SPI_EXTERNAL,
		MS5611_BUS_SIM_EXTERNAL
	};
   
   �鿴ms5611���ص����Ǹ�������.

6. ����ms5611�ĺ����������ʽ, ֱ�������ڲ�spi����
   ms5611 -s start
   6.1 ����2���豸�ڵ㣺 ms5611_spi_int bar0
	   crw-rw-rw-        0 baro0
	   crw-rw-rw-        0 ms5611_spi_int
7.     
start(enum MS5611_BUS busid)
{
	uint8_t i;
	bool started = false;
        
        warnx("YG-> NUM_BUS_OPTIONS:        %d", NUM_BUS_OPTIONS);
        warnx("YG-> busid:        %d", busid);
   
	for (i = 0; i < NUM_BUS_OPTIONS; i++) {
		if (busid == MS5611_BUS_ALL && bus_options[i].dev != NULL) {
			// this device is already started
			continue;
		}

		if (busid != MS5611_BUS_ALL && bus_options[i].busid != busid) {
			// not the one that is asked for
			continue;
		}

		started = started || start_bus(bus_options[i]);
                warnx("YG-> i:        %d", i);
	}

	if (!started) {
		errx(1, "driver start failed");
	}

	// one or more drivers started OK
	exit(0);
}   

8. ���Է���
ms5611: YG-> NUM_BUS_OPTIONS:        4  
ms5611: YG-> busid:        3
MS5611_SPI on SPI bus 1 at 3 (11000 KHz)
ms5611: YG-> i:        1

ms5611 -s start
ms5611: YG-> NUM_BUS_OPTIONS:        4
ms5611: YG-> busid:        3
ms5611: bus option already started

src/modules/systemlib/err.h�ж�����
#define warnx(...) 					PX4_WARN(__VA_ARGS__)

src/platforms/px4_log.h�ж���������log�ĺ�

start();
 ->	start_bus(struct ms5611_bus_option &bus)  //��������1��ms5611���Ǹ�spibus��
   ->if (bus.dev != nullptr) {     //�Ѿ�����  
                                   MS5611_BUS_ALL //��������bus��ms5611������
        -> if (interface->init() != OK) {  //�����豸
			warnx("no device on bus %u", (unsigned)bus.busid);
            -> *interface = return new MS5611_SPI(busnum, (spi_dev_e)PX4_SPIDEV_BARO, prom_buf);    
                -> if (interface->init() != OK) { //��ʼ���ӿ�

  
   
