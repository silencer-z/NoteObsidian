# 1. 简介

操作系统是允许多个任务“同时运行”的，操作系统的这个特性被称为多任务。然而实际上，一个CPU 核心在某一时刻只能运行一个任务，而操作系统中任务调度器的责任就是决定在某一时刻 CPU究竟要运行哪一个任务，任务调度器使得 CPU 在各个任务之间来回切换并处理任务，由于切换处理任务的速度非常快，因此就给人造成了一种同一时刻有多个任务同时运行的错觉。

操作系统的分类方式可以由任务调度器的工作方式决定，比如有的操作系统给每个任务分配同样的运行时间，时间到了就切换到下一个任务，Unix 操作系统就是这样的。RTOS 的任务调度器被设计为可预测的，而这正是嵌入式实时操作系统所需要的。在实时环境中，要求操作系统必须实时地对某一个事件做出响应，因此任务调度器的行为必须是可预测的。像 FreeRTOS这种传统的 RTOS 类操作系统是由用户给每个任务分配一个任务优先级，任务调度器就可以根据此优先级来决定下一刻应该运行哪个任务。

FreeRTOS 是众多 RTOS 类操作系统中的一种，FreeRTOS 十分的小巧，可以在资源有限的
微控制器中运行，当然了，FreeRTOS 也不仅仅局限于在微控制器中使用。

# 2. 移植
## 2.1 文件解析

FreeRTOS 官网提供两个版本的 FreeRTOS 下载链接，分别为 FreeRTOS 和FreeRTOS LTS，其中 FreeRTOS LTS 是 FreeRTOS Long Time Support，这个版本的 FreeRTOS 会受官方长期的支持和维护。下载源码后其中主要文件便是三个文件夹中的内容。`FreeRTOS`文件夹中包含了FreeRTOS的内核程序，`FreeRTOS-Plus`中则为FreeRTOS的组件，`tool`文件夹中则为工具。我们只需关心内核程序即可。`FreeRTOS`文件夹下的`Source`文件夹内便是全部源码。下表便是文件的具体说明：
![[Pasted image 20241014161546.png]]
`Source` 文件夹中的 `portable` 内包含了 FreeRTOS 的移植文件，这些移植文件是针对不同芯片架构的。`portable` 文件夹里面的东西就是连接软件层面的 FreeRTOS 操作系统和硬件层面的芯片的桥梁。
- `ARMClang`:在 MDK 中使用 ARMClang 编译器（AC6）时使用的
- `GCC`:在GCC编译器下使用的
- `Keil/RVDS`:在 MDK 中使用 ARMCC 编译器（AC5）时使用的
- `MemMang`:FreeRTOS 提供的用于内存管理的文件
前三个文件夹内都包含了不同ARM架构下的移植文件，我们只需要将对应的文件添加到项目中即可。MemMang则是FreeRTOS不同的内存管理策略，也只需要添加我们选择的策略的对应文件到项目中即可
## 2.2 文件移植

不同的编译器环境或者开发环境对文件的处理大同小异，主要便是1. 将源码全部添加到项目中；2.针对自己的编译器环境和开发板型号确定`Portable`文件夹中的保留ARMClang，GCC还是RVDS的`port.c`。再根据内存管理策略保留对应的`heap_x.c`文件

此外FreeRTOS的配置文件也需要添加到对应的项目中去，`FreeRTOSConfig.h`可以从demo中获取并添加到项目中去

## 2.3 软件配置

嵌入式中程序最关键的便是时钟的配置，无论是操作系统还是图形库，时钟的配置无疑是放在第一位的。
**Config.h配置**：根据实际情况修改系统时钟频率configCPU_CLOCK_HZ,可以直接使用SystemCoreClock

**心跳配置**：在 Cortex-M 内核上，FreeRTOS 使用 Systick 定时器作为心跳时钟，一般默认心跳时钟为 1ms，进入 Systick 中断后，内核会进入处理模式进行处理。在 Systick 中断处理中，系统会在 ReadList 就绪链表从高优先级到低优先找需要执行的任务，进行调度。如果有任务的状态发生了变化，改变了状态链表，就会产生一个 PendSV 异常，进入 PendSV 异常，通过改变进程栈指针(PSP)切换到不同的任务。FreeRTOS便是通过Systick、PendSV、SVC完成任务调度。具体内容后续详细说明。

FreeRTOS在port.c文件中帮忙完成了`SVC_Handler`()、`PendSV_Handler`()、`SysTick_Handler`()三个中断处理函数，分别为`vPortSVCHandler`()、`xPortPendSVHandler`()、`xPortSysTickHandler`()，前两个中断处理函数直接用`vPortSVCHandler`()、`xPortPendSVHandler`()宏定义覆盖掉便可以了，但是`SysTick_Handler`则需要另外的配置。如果使用了HAL库，则HAL也会使用SysTick导致出现硬件错误，可以将HAL的系统时钟指定到其他的计时器上去，这个时候`xPortSysTickHandler`便可以直接代替`SysTick_Handler`了。(低版本的FreeRTOS中port也许没有xPortSysTickHandler的实现)