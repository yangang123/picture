# 设置组的频率

nsh> pwm rate -d /dev/pwm_output0 -r 300 -g 0
nsh> pwm info 
device: /dev/pwm_output0
channel 1: 1100 us (alternative rate: 300 Hz failsafe: 0, disarmed: 0 us, min: 1000 us, max: 2000 us, trim:  0.00)
channel 2: 1100 us (alternative rate: 300 Hz failsafe: 0, disarmed: 0 us, min: 1000 us, max: 2000 us, trim:  0.00)
channel 3: 1100 us (default rate: 50 Hz failsafe: 0, disarmed: 0 us, min: 1000 us, max: 2000 us, trim:  0.00)
channel 4: 1100 us (default rate: 50 Hz failsafe: 0, disarmed: 0 us, min: 1000 us, max: 2000 us, trim:  0.00)
channel 5: 1100 us (default rate: 50 Hz failsafe: 0, disarmed: 0 us, min: 1000 us, max: 2000 us, trim:  0.00)
channel 6: 1100 us (default rate: 50 Hz failsafe: 0, disarmed: 0 us, min: 1000 us, max: 2000 us, trim:  0.00)
channel 7: 1100 us (default rate: 50 Hz failsafe: 0, disarmed: 0 us, min: 1000 us, max: 2000 us, trim:  0.00)
channel 8: 1100 us (default rate: 50 Hz failsafe: 0, disarmed: 0 us, min: 1000 us, max: 2000 us, trim:  0.00)
channel group 0: channels 1 2
channel group 1: channels 5 6 7 8
channel group 2: channels 3 4

