
uavcan start


/**
1. ��������1Mbps

2. gnss  ��ʼ��
3. mag   ��ʼ�� 
4. baro  ��ʼ��
*/
uavcan: Node ID 1, bitrate 1000000
uavcan: SW version vcs_commit: 0x00000000
uavcan: sensor bridge 'gnss' init ok
uavcan: sensor bridge 'mag' init ok
uavcan: sensor bridge 'baro' init ok


static UavcanNode *instance() { return _instance; }

if (!std::strcmp(argv[1], "start")) {   //�����ڵ�
	-> return UavcanNode::start(node_id, bitranode_idte);   //�ڵ�������ǰ������ 
	    ->const int can_init_res = can.init(bitrate);       //������ʼ��
		->_instance = new UavcanNode(can.driver, uavcan_stm32::SystemClock::instance());	                                                    //ʵ����ʼ��
		->const int node_init_res = _instance->init(node_id); // �ڵ��ʼ��
		     ->_node.setName("org.pixhawk.pixhawk");  //��������
             ->_node.setNodeID(node_id); //����ID��
	         ->fill_node_info();         //���ڵ���Ϣ
			    ->warnx("SW version vcs_commit: 0x%08x", unsigned(swver.vcs_commit));
				                        //��ӡ�ڵ���Ϣ
	          ->ret = _esc_controller.init();  //Actuators��ʼ��
			  ->IUavcanSensorBridge::
			     make_all(_node, _sensor_bridges); // Sensor bridges �ų�ʼ��
			      while (br != nullptr) {
					ret = br->init();
				    warnx("sensor bridge '%s' init ok", br->get_name()); //��ʼ��3����
			  ->return _node.start();                                    //�ڵ�����
		->_instance->_task = px4_task_spawn_cmd("uavcan",
 		                  SCHED_DEFAULT, SCHED_PRIORITY_ACTUATOR_OUTPUTS, StackSize,
					      static_cast<main_t>(run_trampoline), nullptr); //��������
		->int UavcanNode::run()                               //����ָ���ĺ��� 
		     (void)pthread_mutex_lock(&_node_mutex);          //�����ź���
			 
             		
	
->(void)param_get(param_find("UAVCAN_NODE_ID"), &node_id);  //�õ�����ڵ��״̬��Ϣ
->(void)param_get(param_find("UAVCAN_BITRATE"), &bitrate);  //��ȡ������
->warnx("Node ID %u, bitrate %u", node_id, bitrate);        //��ӡ�ڵ�ID�Ͳ�����
->if (!std::strcmp(argv[1], "status")                       //����status��info�����ϵͳ��Ϣ
	|| !std::strcmp(argv[1], "info")) {                    
	
warnx("!!!!!!!!!send data......... ");


2. ������Ϣ�ı�ͷ	
static const struct param_info_s 
   *param_info_base = (const struct param_info_s *) &px4_parameters;
   

	
1. ��������ڵ���Ϣ
param_find_internal(const char *name, bool notification)


BARO ��ѹ��Ϣ��

int UavcanNode::start(uavcan::NodeID node_id, uint32_t bitrate)
   -> const int can_init_res = can.init(bitrate);   // ��ʼ���ɹ�

   
_instance = new UavcanNode(can.driver, uavcan_stm32::SystemClock::instance());// ����1��node����


static auto run_trampoline = [](int, char *[]) {return UavcanNode::_instance->run();};
_instance->_task = px4_task_spawn_cmd("uavcan", SCHED_DEFAULT, SCHED_PRIORITY_ACTUATOR_OUTPUTS, StackSize,

_instance->run()�� ���̵���ڣ������Ͳ����˳���


./libuavcan/libuavcan/src/transport/uc_transfer.cpp  
./libuavcan/libuavcan/src/transport/uc_transfer_buffer.cpp 
./libuavcan/libuavcan/src/transport/uc_dispatcher.cpp 
./libuavcan/libuavcan/src/transport/uc_transfer_sender.cpp         //����
./libuavcan/libuavcan/src/transport/uc_transfer_receiver.cpp       //����
./libuavcan/libuavcan/src/transport/uc_transfer_listener.cpp       //����


# if __GNUC__
__attribute__ ((format(printf, 2, 3)))
# endif
static void UAVCAN_TRACE(const char* src, const char* fmt, ...)
{
    va_list args;
    (void)std::printf("UAVCAN: %s: ", src);
    va_start(args, fmt);
    (void)std::vprintf(fmt, args);
    va_end(args);
    (void)std::puts("");
}


