# ARM Cortex-M 中断
ARM Cortex-M 内核的 MCU 具有一个用于中断管理的嵌套向量中断控制器(NVIC，全称：Nested vectored interrupt controller)。ARM Cortex-M 的 NVIC 最大可支持 256 个中断源，其中包括 16 个系统中断和 240 个外部中断。然而芯片厂商一般情况下都用不完这些资源，以STM32F103ZET6为例，就只用到了 10 个系统中断和 60 个外部中断。

ARM Cortex-M 使用 NVIC 对不同优先级的中断进行管理，用了 **8 位宽的寄存器**来配置中断的优先等级，这个寄存器就是中断优先级配置寄存器，因此最大中断的优先级配置范围位 0~255。但是芯片厂商一般用不完这些资源， **STM32只用到了中断优先级配置寄存器的高 4 位**[7:4 ]，低四位[3:0]取零处理，因此 STM32 提供了最大 2^4=16 级的中断优先等级。

中断优先级配置寄存器的值与对应的优先等级成反比，即中断优先级配置寄存器的值越小，中断的优先等级越高。STM32 的中断优先级可以分为抢占优先级和子优先级。
 - **抢占优先级**：抢占优先级高的中断可以打断正在执行但抢占优先级低的中断，实现中断嵌套。
 - **子优先级**：抢占优先级相同时，子优先级高的中断不能打断正在执行但子优先级低的中的中断，即子优先级不支持中断嵌套。
STM32使用中断优先级配置寄存器的高 4 位来配置抢占优先级和子优先级，一共有五种分配方式：
![[Pasted image 20241025110613.png]]
FreeRTOS官方建议使用中断优先分组4，用户就不用设置子优先级，只需要专注于抢占优先级即可。
```C
/* Set Interrupt Group Priority */
HAL_NVIC_SetPriorityGrouping(NVIC_PRIORITYGROUP_4);
```

# FreeRTOS 中断配置
FreeRTOSConfig.h 文件中有 6 个与中断相关的 FreeRTOS 配置项：

 - `configPRIO_BITS`:定义为 MCU 的 8 位优先级配置寄存器实际使用的位数，STM32中为4。

 - `configLIBRARY_LOWEST_INTERRUPT_PRIORITY`:定义为 MCU的最低优先等级，STM32只是用4位寄存器配置优先级，所以优先等级0~15，最低的是15。

 - `configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY`:定义为FreeRTOS 可管理的最高优先级的中断。我们暂且设为5，即优先级高于5的中断不受FreeRTOS管理。

 - `configKERNEL_INTERRUPT_PRIORITY`:定义为MCU 的最低优先级在中断优先级配置寄存器中的值，用于配置**SysTick** 和 **PenSV** 的中断优先级，可用其他宏表示为：**(configLIBRARY_LOWEST_INTERRUPT_PRIORITY<<(8-configPRIO_BITS))**

 - `configMAX_SYSCALL_INTERRUPT_PRIORITY(configMAX_API_CALL_INTERRUPT_PRIORITY)`:定义FreeRTOS可管理的最高优先级的中断的中断寄存器值，可用其他宏表示 **(configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY<<(8-configPRIO_BITS))**

# FreeRTOS 临界区
临界区是指那些必须完整运行的区域，在临界区中的代码必须完整运行，不能被打断。例如一些使用软件模拟的通信协议，通信协议在通信时，必须严格按照通信协议的时序进行，不能被打断。FreeRTOS 在进出临界区的时候，通过关闭和打开受 FreeRTOS 管理的中断，以保护临界区中的代码。

对于进出临界区，FreeRTOS 的源码中有四个相关的宏定义 ， 分别为`taskENTER_CRITICAL(), taskENTER_CRITICAL_FROM_ISR(),taskEXIT_CRITICAL(),taskEXIT_CRITICAL_FROM_ISR()`，这四个宏定义分别用于在中断和非中断中进出临界区。

- `taskENTER_CRITICAL()`通过关闭中断进入临界区，当然不受FreeRTOS管理的中断时不受影响的，且FreeRTOS通过这种方式进入临界区是**可以重复进入临界区**，类似于信息量，只需后续重复退出相同次数的临界区即可。通过这个宏进入临界区**不能从中断里面使用**，否则会报错。

- `taskENTER_CRITICAL_FROM_ISR()`用于从中断中进入临界区，将 BASEPRI寄存器设置为宏configMAX_SYSCALL_INTERRUPT_PRIORITY 的值，以达到关闭中断的效果，当然了，不受FreeRTOS 管理的中断是不受影响的，从中断中进入临界区时**不支持嵌套**的。