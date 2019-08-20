
uavcan start


/**
1. 波特率是1Mbps

2. gnss  初始化
3. mag   初始化 
4. baro  初始化
*/
uavcan: Node ID 1, bitrate 1000000
uavcan: SW version vcs_commit: 0x00000000
uavcan: sensor bridge 'gnss' init ok
uavcan: sensor bridge 'mag' init ok
uavcan: sensor bridge 'baro' init ok


static UavcanNode *instance() { return _instance; }

if (!std::strcmp(argv[1], "start")) {   //启动节点
	-> return UavcanNode::start(node_id, bitranode_idte);   //节点启动当前波特率 
	    ->const int can_init_res = can.init(bitrate);       //驱动初始化
		->_instance = new UavcanNode(can.driver, uavcan_stm32::SystemClock::instance());	                                                    //实例初始化
		->const int node_init_res = _instance->init(node_id); // 节点初始化
		     ->_node.setName("org.pixhawk.pixhawk");  //创建名字
             ->_node.setNodeID(node_id); //设置ID号
	         ->fill_node_info();         //填充节点信息
			    ->warnx("SW version vcs_commit: 0x%08x", unsigned(swver.vcs_commit));
				                        //打印节点信息
	          ->ret = _esc_controller.init();  //Actuators初始化
			  ->IUavcanSensorBridge::
			     make_all(_node, _sensor_bridges); // Sensor bridges 桥初始化
			      while (br != nullptr) {
					ret = br->init();
				    warnx("sensor bridge '%s' init ok", br->get_name()); //初始化3个桥
			  ->return _node.start();                                    //节点启动
		->_instance->_task = px4_task_spawn_cmd("uavcan",
 		                  SCHED_DEFAULT, SCHED_PRIORITY_ACTUATOR_OUTPUTS, StackSize,
					      static_cast<main_t>(run_trampoline), nullptr); //创建任务
		->int UavcanNode::run()                               //任务指定的函数 
		     (void)pthread_mutex_lock(&_node_mutex);          //锁定信号量
			 
             		
	
->(void)param_get(param_find("UAVCAN_NODE_ID"), &node_id);  //得到这个节点的状态信息
->(void)param_get(param_find("UAVCAN_BITRATE"), &bitrate);  //获取波特率
->warnx("Node ID %u, bitrate %u", node_id, bitrate);        //打印节点ID和波特率
->if (!std::strcmp(argv[1], "status")                       //输入status和info都输出系统信息
	|| !std::strcmp(argv[1], "info")) {                    
	
warnx("!!!!!!!!!send data......... ");


2. 参数信息的表头	
static const struct param_info_s 
   *param_info_base = (const struct param_info_s *) &px4_parameters;
   

	
1. 发现这个节点信息
param_find_internal(const char *name, bool notification)


BARO 气压信息：

int UavcanNode::start(uavcan::NodeID node_id, uint32_t bitrate)
   -> const int can_init_res = can.init(bitrate);   // 初始化成功

   
_instance = new UavcanNode(can.driver, uavcan_stm32::SystemClock::instance());// 创建1个node对象


static auto run_trampoline = [](int, char *[]) {return UavcanNode::_instance->run();};
_instance->_task = px4_task_spawn_cmd("uavcan", SCHED_DEFAULT, SCHED_PRIORITY_ACTUATOR_OUTPUTS, StackSize,

_instance->run()： 进程的入口，正常就不会退出了


./libuavcan/libuavcan/src/transport/uc_transfer.cpp  
./libuavcan/libuavcan/src/transport/uc_transfer_buffer.cpp 
./libuavcan/libuavcan/src/transport/uc_dispatcher.cpp 
./libuavcan/libuavcan/src/transport/uc_transfer_sender.cpp         //发送
./libuavcan/libuavcan/src/transport/uc_transfer_receiver.cpp       //接收
./libuavcan/libuavcan/src/transport/uc_transfer_listener.cpp       //监听


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


