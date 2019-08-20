sdlog2的格式:

    解析格式
    log_msg_format_t log_msg_format=
    {
    HEAD_BYTE1,
    HEAD_BYTE2,
    LOG_FORMAT_MSG,
    LOG_IMU_MSG,
    sizeof(struct log_IMU_s)+3,
    "IMU",
    "ffffff",
    "GyroX,GyroY,GyroZ,AccX,AccY,AccZ"
    };

    数据格式

struct imu_data_s{
uint8_t head1;
uint8_t head2;
uint8_t msg_type;
float acc_x;
float acc_y;
float acc_z;
float gyro_x;
float gyro_y;
float gyro_z;
} PACK_STRUCT_STRUCT;

注意: 格式中的长度: 数据+包头的总长度

#define SD_CMD_GO_IDLE_STATE ((uint8_t)0) /空闲/
#define SD_CMD_SEND_OP_COND ((uint8_t)1)
#define SD_CMD_ALL_SEND_CID ((uint8_t)2)
#define SD_CMD_SET_REL_ADDR ((uint8_t)3) /*!< SDIO_SEND_REL_ADDR for SD Card /
#define SD_CMD_SET_DSR ((uint8_t)4)
#define SD_CMD_SDIO_SEN_OP_COND ((uint8_t)5)
#define SD_CMD_HS_SWITCH ((uint8_t)6)
#define SD_CMD_SEL_DESEL_CARD ((uint8_t)7)
#define SD_CMD_HS_SEND_EXT_CSD ((uint8_t)8)
#define SD_CMD_SEND_CSD ((uint8_t)9)
#define SD_CMD_SEND_CID ((uint8_t)10)
#define SD_CMD_READ_DAT_UNTIL_STOP ((uint8_t)11) /!< SD Card doesn't support it /
#define SD_CMD_STOP_TRANSMISSION ((uint8_t)12)
#define SD_CMD_SEND_STATUS ((uint8_t)13)
#define SD_CMD_HS_BUSTEST_READ ((uint8_t)14)
#define SD_CMD_GO_INACTIVE_STATE ((uint8_t)15)
#define SD_CMD_SET_BLOCKLEN ((uint8_t)16)
#define SD_CMD_READ_SINGLE_BLOCK ((uint8_t)17)
#define SD_CMD_READ_MULT_BLOCK ((uint8_t)18)
#define SD_CMD_HS_BUSTEST_WRITE ((uint8_t)19)
#define SD_CMD_WRITE_DAT_UNTIL_STOP ((uint8_t)20) /!< SD Card doesn't support it /
#define SD_CMD_SET_BLOCK_COUNT ((uint8_t)23) /!< SD Card doesn't support it /
#define SD_CMD_WRITE_SINGLE_BLOCK ((uint8_t)24)
#define SD_CMD_WRITE_MULT_BLOCK ((uint8_t)25)
#define SD_CMD_PROG_CID ((uint8_t)26) /!< reserved for manufacturers /
#define SD_CMD_PROG_CSD ((uint8_t)27)
#define SD_CMD_SET_WRITE_PROT ((uint8_t)28)
#define SD_CMD_CLR_WRITE_PROT ((uint8_t)29)
#define SD_CMD_SEND_WRITE_PROT ((uint8_t)30)
#define SD_CMD_SD_ERASE_GRP_START ((uint8_t)32) /!< To set the address of the first write
block to be erased. (For SD card only) /
#define SD_CMD_SD_ERASE_GRP_END ((uint8_t)33) /!< To set the address of the last write block of the
continuous range to be erased. (For SD card only) /
#define SD_CMD_ERASE_GRP_START ((uint8_t)35) /!< To set the address of the first write block to be erased.
(For MMC card only spec 3.31) */

#define SD_CMD_ERASE_GRP_END ((uint8_t)36) /*!< To set the address of the last write block of the
continuous range to be erased. (For MMC card only spec 3.31) */

#define SD_CMD_ERASE ((uint8_t)38)
#define SD_CMD_FAST_IO ((uint8_t)39) /*!< SD Card doesn't support it /
#define SD_CMD_GO_IRQ_STATE ((uint8_t)40) /!< SD Card doesn't support it */
#define SD_CMD_LOCK_UNLOCK ((uint8_t)42)
#define SD_CMD_APP_CMD ((uint8_t)55)
#define SD_CMD_GEN_CMD ((uint8_t)56)
#define SD_CMD_NO_CMD ((uint8_t)64)




    sd卡存储数据的格式
    1. 陀螺校准后的数据
    log_msg.body.log_IMU.gyro_x = buf.sensor.gyro_rad_s[i * 3 + 0];
    log_msg.body.log_IMU.gyro_y = buf.sensor.gyro_rad_s[i * 3 + 1];
    log_msg.body.log_IMU.gyro_z = buf.sensor.gyro_rad_s[i * 3 + 2];

                                  2.  保存加速计的数据
      		log_msg.body.log_IMU.acc_x = buf.sensor.accelerometer_m_s2[i * 3 + 0];
      		log_msg.body.log_IMU.acc_y = buf.sensor.accelerometer_m_s2[i * 3 + 1];
      		log_msg.body.log_IMU.acc_z = buf.sensor.accelerometer_m_s2[i * 3 + 2];

                                 3. 保存的校准后的数据
      		log_msg.body.log_IMU.mag_x = buf.sensor.magnetometer_ga[i * 3 + 0];
      		log_msg.body.log_IMU.mag_y = buf.sensor.magnetometer_ga[i * 3 + 1];
      		log_msg.body.log_IMU.mag_z = buf.sensor.magnetometer_ga[i * 3 + 2];
      		//log_msg.body.log_IMU.temp_gyro = buf.sensor.gyro_temp[i * 3 + 0];


2.、 sensors中处理的数据
（1） 陀螺数据
math::Vector<3> vect(mag_report.x, mag_report.y, mag_report.z);
vect = _mag_rotation[i] * vect;
//校准后的数据
raw.magnetometer_ga[i * 3 + 0] = vect(0);
raw.magnetometer_ga[i * 3 + 1] = vect(1);
raw.magnetometer_ga[i * 3 + 2] = vect(2);

                   //原始数据
		raw.magnetometer_raw[i * 3 + 0] = mag_report.x_raw;
		raw.magnetometer_raw[i * 3 + 1] = mag_report.y_raw;
		raw.magnetometer_raw[i * 3 + 2] = mag_report.z_raw;
 （2） 加速计数据
		math::Vector<3> vect(accel_report.x, accel_report.y, accel_report.z);
		vect = _board_rotation * vect;
		raw.accelerometer_m_s2[i * 3 + 0] = vect(0);
		raw.accelerometer_m_s2[i * 3 + 1] = vect(1);
		raw.accelerometer_m_s2[i * 3 + 2] = vect(2);

定义的格式
#define LOG_DSTB_MSG 212
struct log_DSTB_s {
// add by rzy/RZY
float back_object_dist;
uint8_t id;
uint8_t type;
float current_distance;
};

这里面是订阅数据:

		if (copy_if_updated(ORB_ID(distance_sensor), &subs.distance_sensor_sub, &buf.distance_sensor)) {

			log_msg.msg_type = LOG_DSTF_MSG;
			log_msg.body.log_DSTF.front_object_dist = buf.distance_sensor.front_object_dist;
			log_msg.body.log_DSTF.front_dist_0 = buf.distance_sensor.front_dist_0;
			log_msg.body.log_DSTF.front_dist_1 = buf.distance_sensor.front_dist_1;
			log_msg.body.log_DSTF.front_dist_2 = buf.distance_sensor.front_dist_2;
			log_msg.body.log_DSTF.front_dist_3 = buf.distance_sensor.front_dist_3;
			log_msg.body.log_DSTF.front_dist_4 = buf.distance_sensor.front_dist_4;
			log_msg.body.log_DSTF.front_dist_5 = buf.distance_sensor.front_dist_5;
			log_msg.body.log_DSTF.front_dist_6 = buf.distance_sensor.front_dist_6;
			log_msg.body.log_DSTF.front_dist_7 = buf.distance_sensor.front_dist_7;
			log_msg.body.log_DSTF.front_dist_8 = buf.distance_sensor.front_dist_8;
			log_msg.body.log_DSTF.front_dist_9 = buf.distance_sensor.front_dist_9;
			log_msg.body.log_DSTF.front_cnt = buf.distance_sensor.front_cnt;
			LOGBUFFER_WRITE_AND_COUNT(DSTF);

			log_msg.msg_type = LOG_DSTD_MSG;
			log_msg.body.log_DSTD.down_object_dist = buf.distance_sensor.down_object_dist;
			log_msg.body.log_DSTD.down_x_dist_speed = buf.distance_sensor.down_x_dist_speed;
			log_msg.body.log_DSTD.down_y_dist_speed = buf.distance_sensor.down_y_dist_speed;
			log_msg.body.log_DSTD.down_z_dist_speed = buf.distance_sensor.down_z_dist_speed;
			log_msg.body.log_DSTD.down_valid_flag = buf.distance_sensor.down_valid_flag;
			log_msg.body.log_DSTD.down_cnt = buf.distance_sensor.down_cnt;
			LOGBUFFER_WRITE_AND_COUNT(DSTD);

			log_msg.msg_type = LOG_DSTB_MSG;
			log_msg.body.log_DSTB.back_object_dist = buf.distance_sensor.back_object_dist;
			log_msg.body.log_DSTB.id = buf.distance_sensor.id;
			log_msg.body.log_DSTB.type = buf.distance_sensor.type;
			log_msg.body.log_DSTB.current_distance = buf.distance_sensor.current_distance;
			LOGBUFFER_WRITE_AND_COUNT(DSTB);
		}

判断是否存在:
bool copy_if_updated_multi(orb_id_t topic, int multi_instance, int *handle, void *buffer)
{
bool updated = false;

if (*handle < 0) {
	if (OK == orb_exists(topic, multi_instance)) {
		*handle = orb_subscribe(topic);
		/* copy first data */
		if (*handle >= 0) {
			orb_copy(topic, *handle, buffer);
			updated = true;
		}
	}
} else {
	orb_check(*handle, &updated);

	if (updated) {
		orb_copy(topic, *handle, buffer);
	}
}

return updated;

}
