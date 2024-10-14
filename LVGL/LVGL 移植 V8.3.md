# LVGL库文件目录结构
需要从官网github中下载想要版本的源码
保留`example`(样例) `src`(源码) `lv_conf_template.h` `lvgl.h`并将其放入同一文件夹内命名为`LVGL`，将`example`中的`port`文件夹放在`example`同级，此时`example`重要存放样例程序，可以删去。将`port`中的文件重命名去掉`example,lv_conf_template.h`重命名为`lv_conf.h`。最后再在`CmakeList_template.txt`里面添加头文件和c文件,右键使用`STM32CubeMX`重新生成项目。
```
LVGL
|--example
	|-- ...
|--port
	|--lv_port_disp.c/h
	|--lv_port_fs.c/h
	|--lv_port_indev.c/h
|--src
	|-- ...
|--lv_conf.h
|--lvgl.h
```

```Cmake
include_directories(${includes}  
    System/sys  
    System/delay  
    Hardware/LCD           # 屏幕驱动头文件
    Hardware/LVGL          # lvgl.h位置
    Hardware/LVGL/port     # lv_port_**.h位置
    Hardware/LVGL/examples # 样例库
)

file(GLOB_RECURSE SOURCES ${sources}  
    "System/*.*"  
    "Hardware/LCD/*.*"     # LCD文件夹内所有文件
    "Hardware/LVGL/*.*"    # LVGL文件夹内所有文件
)
```
# STM32CubeMX设置
`STM32CubeMX`需要配置一个时钟供LVGL使用
这里就设置TIM6
```config
Prescaler                      72-1 // 系统时钟是72MHZ
Counter                        Mode Up
Counter Period                 1000-1
Auto-reload Preload            Enable
Trigger Event Selection        Reset

勾选 NVIC Settings -> global interrupt
```
# LVGL初始化
在main函数用户代码对应的用户代码段中加入对应的初始化程序
```C
HAL_TIM_Base_Start_IT(&htim6);  //开始计时器中断
lv_init();                      //lvgl初始化
lv_port_disp_init();            //显示驱动初始化
```
# LVGL接口设置
__心跳接口__:LVGL 需要一个系统滴答来了解动画和其他任务所用的时间
在中断的文件中`HAL_TIM_PeriodElapsedCallback`的函数中添加`lv_tick_inc(1)`产生滴答
在main函数的死循环中添加`lv_task_handler()`用于处理lv任务，每隔几毫秒调用一次

__显示接口__：修改`port`文件夹中的`lv_port_disp.c`与`lv_port_disp.h` 
1. 使能文件 
2. 修改相关头文件索引
3. 修改屏幕尺寸宏定义
4. `lv_port_disp_init`中的三种缓存形式选择一种其他注销
5. 将显示屏的初始化程序添加到`disp_init`中
6. 修改`disp_flush`函数，将其与画点函数对接，实现接口连接

__输入接口__:修改`port`文件夹中的`lv_port_indev.c`与`lv_port_indev.h` 
输入方式一共有五种：Touchpad、Mouse、Keypad、Encoder、Button。
1. 声明变量lv_indev_t，其中包含了设备类型和回调函数
2. 初始化设备
3. 将设备在LVGL中注册lv_indev_drv_init，lv_indev_drv_register
4. 实现回调函数，LVGL会重复调用回调函数实现物理设备的读取，在回调函数中完成的内容则是读取物理设备的信息，并将信息转化为事件包装起来，传递给LVGL供LVGL响应
