} else if (!strcmp(argv[1], "rate")) {

pwm rate -c 12 -r 50 
	->
    ret = ioctl(fd, PWM_SERVO_SET_UPDATE_RATE, alt_rate);

１．　设置fmu,pwm的输出(直接cpu控制，　A1-A5)
src/drivers/px4fmu/fmu.cpp:952:	case PWM_SERVO_SET_UPDATE_RATE:

１．　设置fmu,io的输出（串口通信，发送数据到io板, M1-M8）
src/drivers/px4io/px4io.cpp:2408:	case PWM_SERVO_SET_UPDATE_RATE:


#define PWM_OUTPUT0_DEVICE_PATH	"/dev/pwm_output0"
int fd = open(dev, 0);

class PX4IO : public device::CDev
  ->virtual ssize_t		write(file *filp, const char *buffer, size_t len);
　　->virtual int		ioctl(file *filp, int cmd, unsigned long arg);

PX4IO::init() //CDev类中fop注册了
   -> ret = register_driver(PWM_OUTPUT0_DEVICE_PATH, &fops, 0666, (void *)this);

打开句柄
int fd = open(/dev/pwm_output0, 0);

//
int
PX4IO::ioctl(file *filep, int cmd, unsigned long arg)
{　
　　　　	case PWM_SERVO_SET_UPDATE_RATE:
		/* set the requested alternate rate */
		ret = io_reg_set(PX4IO_PAGE_SETUP, PX4IO_P_SETUP_PWM_ALTRATE, arg);
		break;

class PX4IO : public device::CDev
{
int			io_reg_set(uint8_t page, uint8_t offset, const uint16_t *values, unsigned num_values);
			－＞　int ret =  _interface->write((page << 8) | offset, (void *)values, num_values);
   			   ->PX4IO_serial::write(unsigned address, void *data, unsigned count)
                     －＞for (unsigned retries = 0; retries < 3; retries++) {　　　　　　　　　　　　　　
				　　　　　－＞PX4IO_serial::_wait_complete()
						-> stm32_dmasetup(
						->stm32_dmastart(_rx_dma, _dma_callback, this, false); 
							->ret = sem_timedwait(&_completion_semaphore, &abstime); //等待传输完成

device::Device		*_interface;
get_interface()
 ->interface = PX4IO_serial_interface();

class PX4IO_serial : public device::Device
{
->device::Device
*PX4IO_serial_interface()
{
return new PX4IO_serial();
}

//bootloader启动说明
static void
calculate_fw_crc(void)
{
#define APP_SIZE_MAX 0xf000
#define APP_LOAD_ADDRESS 0x08001000
        // compute CRC of the current firmware
        uint32_t sum = 0;

        for (unsigned p = 0; p < APP_SIZE_MAX; p += 4) {
                uint32_t bytes = *(uint32_t *)(p + APP_LOAD_ADDRESS);
                sum = crc32part((uint8_t *)&bytes, sizeof(bytes), sum);
        }

        r_page_setup[PX4IO_P_SETUP_CRC]   = sum & 0xFFFF;
        r_page_setup[PX4IO_P_SETUP_CRC + 1] = sum >> 16;
}

user_start(int argc, char *argv[])
{

 -》calculate_fw_crc(); 
先算出来》

