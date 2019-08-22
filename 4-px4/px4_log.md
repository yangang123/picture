@[TOC]
# 资源
> px4_log.md
> 标题： 闫刚 px4_log.h讲解

# err.h
```c
#define err(eval, ...)		do { \
		PX4_ERR(__VA_ARGS__); \
		PX4_ERR("Task exited with errno=%i\n", errno); \
		EXIT(eval); \
	} while(0)

#define errx(eval, ...)		do { \
		PX4_ERR(__VA_ARGS__); \
		EXIT(eval); \
	} while(0)

#define warn(...) 		PX4_WARN(__VA_ARGS__)
#define warnx(...) 		PX4_WARN(__VA_ARGS__)
```

# px4_log.h 
px4_log.h定义了log定义
```c
#if defined(__PX4_ROS)

#elif defined(__PX4_QURT)

    #if defined(TRACE_BUILD)
    #elif defined(DEBUG_BUILD)
    #elif defined(RELEASE_BUILD)

#else

    #include <inttypes.h>
    #include <stdint.h>
    #include <sys/cdefs.h>
    #include <stdio.h>
    #include <stdarg.h>

    #include <px4_defines.h>
    
    //log输出到printf接口
    #define __px4__log_printcond(cond, ...)	    if (cond) printf(__VA_ARGS__)
    #define __px4__log_printline(level, ...)    printf(__VA_ARGS__)

    #if defined(TRACE_BUILD)
        TRACE_BUILD

        /****************************************************************************
        * Extremely Verbose settings for a Trace build
        ****************************************************************************/
        #define PX4_PANIC(FMT, ...)	__px4_log_timestamp_thread_file_and_line(_PX4_LOG_LEVEL_PANIC, FMT, ##__VA_ARGS__)
        #define PX4_ERR(FMT, ...)	__px4_log_timestamp_thread_file_and_line(_PX4_LOG_LEVEL_ERROR, FMT, ##__VA_ARGS__)
        #define PX4_WARN(FMT, ...) 	__px4_log_timestamp_thread_file_and_line(_PX4_LOG_LEVEL_WARN,  FMT, ##__VA_ARGS__)
        #define PX4_DEBUG(FMT, ...) 	__px4_log_timestamp_thread(_PX4_LOG_LEVEL_DEBUG, FMT, ##__VA_ARGS__)

    #elif defined(DEBUG_BUILD)
        DEBUG_BUILD

        /****************************************************************************
        * Verbose settings for a Debug build
        ****************************************************************************/
        #define PX4_PANIC(FMT, ...)	__px4_log_timestamp_file_and_line(_PX4_LOG_LEVEL_PANIC, FMT, ##__VA_ARGS__)
        #define PX4_ERR(FMT, ...)	__px4_log_timestamp_file_and_line(_PX4_LOG_LEVEL_ERROR, FMT, ##__VA_ARGS__)
        #define PX4_WARN(FMT, ...) 	__px4_log_timestamp_file_and_line(_PX4_LOG_LEVEL_WARN,  FMT, ##__VA_ARGS__)
        #define PX4_DEBUG(FMT, ...) 	__px4_log_timestamp(_PX4_LOG_LEVEL_DEBUG, FMT, ##__VA_ARGS__)

    #elif defined(RELEASE_BUILD)
        RELEASE_BUILD

        /****************************************************************************
        * Non-verbose settings for a Release build to minimize strings in build
        ****************************************************************************/
        #define PX4_PANIC(FMT, ...)	__px4_log_modulename(_PX4_LOG_LEVEL_PANIC, FMT, ##__VA_ARGS__)
        #define PX4_ERR(FMT, ...)	__px4_log_modulename(_PX4_LOG_LEVEL_ERROR, FMT, ##__VA_ARGS__)
        #define PX4_WARN(FMT, ...) 	__px4_log_omit(_PX4_LOG_LEVEL_WARN,  FMT, ##__VA_ARGS__)
        #define PX4_DEBUG(FMT, ...) 	__px4_log_omit(_PX4_LOG_LEVEL_DEBUG, FMT, ##__VA_ARGS__)

    #else
        NULL_BUILD

        /****************************************************************************
        * Medium verbose settings for a default build
        ****************************************************************************/
        #define PX4_PANIC(FMT, ...)	__px4_log_modulename(_PX4_LOG_LEVEL_PANIC, FMT, ##__VA_ARGS__)
        #define PX4_ERR(FMT, ...)	__px4_log_modulename(_PX4_LOG_LEVEL_ERROR, FMT, ##__VA_ARGS__)
        #define PX4_WARN(FMT, ...) 	__px4_log_modulename(_PX4_LOG_LEVEL_WARN,  FMT, ##__VA_ARGS__)
        #define PX4_DEBUG(FMT, ...) 	__px4_log_omit(_PX4_LOG_LEVEL_DEBUG, FMT, ##__VA_ARGS__)

    #endif
```


# 在cmake中设置编译选项
>设置编译选项DTRACE_BUILD, DEBUG_BUILD, RELEASE_BUILD

```
// px4_base.cmake
set(cxx_compile_flags
    -g
    -fno-exceptions
    -fno-rtti
    -std=gnu++11
    -fno-threadsafe-statics
    -DCONFIG_WCHAR_BUILTIN
    -D__CUSTOM_FILE_IO__
    -DTRACE_BUILD
    )
```

# 编译测试
> 在px4_log.h中添加错误的代码，导致编译失败，然后分析，我们定义cxx的编译flag是否生效
> 注意查看-DTRACE_BUILD
```
[10/721] Building CXX object msg/CMakeFiles/uorb_msgs.dir/topics_sources/airspeed.cpp.obj
FAILED: /home/yangang/work/tools/gcc-arm-none-eabi-5_4-2016q2-20160622-linux/gcc-arm-none-eabi-5_4-2016q2/bin/arm-none-eabi-g++   -DCONFIG_ARCH_BOARD_PX4NUCLEOF767ZI_V1 -D__DF_NUTTX -D__PX4_NUTTX -D__STDC_FORMAT_MACROS -fno-common -ffunction-sections -fdata-sections -mcpu=cortex-m7 -mthumb -mfpu=fpv5-sp-d16 -mfloat-abi=hard -fno-common -ffunction-sections -fdata-sections -g -fno-exceptions -fno-rtti -std=gnu++11 -fno-threadsafe-statics -DCONFIG_WCHAR_BUILTIN -D__CUSTOM_FILE_IO__ -DTRACE_BUILD -fcheck-new -Wall -Warray-bounds -Wdisabled-optimization -Werror -Wextra -Wfatal-errors -Wfloat-equal -Wformat-security -Winit-self -Wlogical-op -Wmissing-declarations -Wmissing-field-initializers -Wpointer-arith -Wshadow -Wuninitialized -Wunknown-pragmas -Wunused-variable -Wno-implicit-fallthrough -Wno-unused-parameter -Wunused-but-set-variable -Wformat=1 -Wdouble-promotion -Wno-missing-field-initializers -Wno-overloaded-virtual -Wreorder -Wno-format-truncation -fvisibility=hidden -include visibility.h -fno-strict-aliasing -fomit-frame-pointer -fno-math-errno -funsafe-math-optimizations -ffunction-sections -fdata-sections -fno-strength-reduce -fno-builtin-printf -Os -DNDEBUG -isystem NuttX/nuttx/include/cxx -isystem NuttX/nuttx/include -I. -Isrc -Isrc/lib -Isrc/modules -I../../src -I../../src/drivers/boards/px4nucleoF767ZI-v1 -I../../src/include -I../../src/lib -I../../src/lib/DriverFramework/framework/include -I../../src/lib/matrix -I../../src/modules -I../../src/platforms -INuttX/nuttx/arch/arm/src/armv7-m -INuttX/nuttx/arch/arm/src/chip -INuttX/nuttx/arch/arm/src/common -INuttX/apps/include -MMD -MT msg/CMakeFiles/uorb_msgs.dir/topics_sources/airspeed.cpp.obj -MF msg/CMakeFiles/uorb_msgs.dir/topics_sources/airspeed.cpp.obj.d -o msg/CMakeFiles/uorb_msgs.dir/topics_sources/airspeed.cpp.obj -c msg/topics_sources/airspeed.cpp
<command-line>:0:13: error: expected unqualified-id before numeric constant
../../src/platforms/px4_log.h:404:1: note: in expansion of macro 'TRACE_BUILD'
 TRACE_BUILD
 ^
compilation terminated due to -Wfatal-errors.
ninja: build stopped: subcommand failed.
Makefile:154: recipe for target 'px4nucleoF767ZI-v1_default' failed
make: *** [px4nucleoF767ZI-v1_default] Error 1
```
