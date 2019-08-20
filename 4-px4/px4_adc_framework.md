rc.sensors:10:if adc start

adc_main(int argc, char *argv[])
	-> 	g_adc = new ADC(ADC_CHANNELS);
		-> CDev("adc", ADC0_DEVICE_PATH),
		-> 	_sample_perf(perf_alloc(PC_ELAPSED, "adc_samples")),
		->  channels |= 1 << 16; /* always enable the temperature sensor */
		
			struct adc_msg_s
			{
			  uint8_t      am_channel;               /* The 8-bit ADC Channel */
			  int32_t      am_data;                  /* ADC convert result (4 bytes) */
			} packed_struct;
			
			_samples = new adc_msg_s[_channel_count]; //����ռ�
	-> ADC::_tick()
		-> update_adc_report(now);	//�����8��ͨ��
			-> if (max_num > (sizeof(adc.channel_id) / sizeof(adc.channel_id[0]))) {
		-> update_system_power(now);
			#  define px4_enter_critical_section()       enter_critical_section()
			/**
				�رտ����жϣ���ֹ������ȣ��жϴ���
			**/
			#  define px4_leave_critical_section(flags) 	leave_critical_section(flags)
			
