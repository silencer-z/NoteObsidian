# 1. 任务状态
FreeRTOS 中任务存在四种任务状态，分别为**运行态**、**就绪态**、**阻塞态**和**挂起态**。FreeRTOS运行时，任务的状态一定是这四种状态中的一种。
## 运行态
如果一个任务得到 CPU 的使用权，即任务**被实际执行**时，那么这个任务处于运行态。一个处理器核心只允许有一个任务处于运行态。
## 就绪态
如果一个任务**已经能够被执行**（不处于阻塞态后挂起态），但当前**还未被执行**（具有相同优先级或更高优先级的任务正持有 CPU 使用权），那么这个任务就处于就绪态。
## 阻塞态
如果一个任务因延时一段时间或等待外部事件发生，那么这个任务就处理阻塞态。任务可以处于阻塞态以等待**队列、信号量、事件组、通知或信号量**等外部事件。通常情况下，处于阻塞态的任务都有一个阻塞的超时时间，在任务阻塞达到或超过这个**超时时间**后，即使任务等待的外部事件还没有发生，任务的阻塞态也会被解除。调用了延迟函数也是将任务置于阻塞态。
## 挂起态
任务一般通过函数 vTaskSuspend()和函数 vTaskResums()进入和退出挂起态与阻塞态一样，处于挂起态的任务也无法被运行。与阻塞态不同挂起态如果不释放则永远不会执行。

## 任务状态流转
![[Pasted image 20241108141441.png]]

# 2. 任务优先级
任务优先级是决定任务调度器如何分配 CPU 使用权的因素之一。每一个任务在创建时候都被分配一个`0~(configMAX_PRIORITIES-1)`的任务优先级。

如果在 FreeRTOSConfig.h文件中，将宏`configUSE_PORT_OPTIMISED_TASK_SELECTION`定义为 1，那么FreeRTOS则会使用使用**硬件计算前导零指令**计算下一个要运行的任务，对于 STM32 而言，硬件计算前导零的指令，最大支持32位的数，因此宏 `configMAX_PRIORITIES` 的值不能超过 32。系统支持的**优先级数量越多**，系统消耗的**资源也就越多**，应当合理地将宏 `configMAX_PRIORITIES` 定义为满足应用需求的最小值。

FreeRTOS 的任务优先级高低与其对应的优先级数值是**成正比**的，任务优先级数值为 0 的任务优先级是最低的任务优先级，任务优先级数值为`(configMAX_PRIORITIES-1)`的任务优先级是最高的任务优先级。FreeRTOS 的任务优先级高低与其对应数值的逻辑关系正好与STM32 的中断优先级高低与其对应数值的**逻辑关系相反**
# 3. 调度方式
FreeRTOS 一共支持三种任务调度方式，分别为抢占式调度、时间片调度和协程式调度。虽然协程式代码还存在，但是FreeRTOS官方已经不再开发了，其是为了**资源十分紧张**的设备开发的。
## 抢占式调度
抢占式调度主要时针对**优先级不同**的任务，每个任务都有一个优先级，**优先级高的任务可以抢占优先级低的任务**，只有当优先级高的任务发生**阻塞或者被挂起**，低优先级的任务才可以运行。
## 时间片调度
时间片调度主要针对**优先级相同**的任务，当多个任务的优先级相同时，任务调度器会在**每一次系统时钟节拍**到的时候切换任务，CPU轮流运行优先级相同的任务，每个任务运行的时间就是一个系统时钟节拍。
# 4. 任务控制块
在裸机系统中，程序的主体是CPU按照顺序执行的。而在多任务系统中，任务的执行是由系统调度的。系统为了调度任务为每个任务都额外定义了一个任务控制块，里面存有任务的所有信息，比如任务的栈指针，任务名称，任务的形参等。系统对任务的全部操作都可以通过这个任务控制块来实现。
```C
typedef struct tskTaskControlBlock
{
	/* 指向任务栈栈顶的指针 */
    volatile StackType_t   *pxTopOfStack; 
	/* MPU 相关设置 */
    #if ( portUSING_MPU_WRAPPERS == 1 )  
	    xMPU_SETTINGS  xMPUSettings;  
    #endif  
	/* 任务状态列表项 */
    ListItem_t           xStateListItem;
    /* 任务等待事件列表项 */
    ListItem_t           xEventListItem;
    /* 任务的任务优先级 */
	UBaseType_t          uxPriority;
	/* 任务栈的起始地址 */
	StackType_t          *pxStack;
	/* 任务的任务名 */
	char                 pcTaskName[ configMAX_TASK_NAME_LEN ];
	/* 指向任务栈栈底的指针 */
	#if ( ( portSTACK_GROWTH > 0 ) || ( configRECORD_STACK_HIGH_ADDRESS == 1 ) ) 
	    StackType_t           *pxEndOfStack;
	#endif  
    /* 记录任务独自的临界区嵌套次数 */
    #if ( portCRITICAL_NESTING_IN_TCB == 1 )  
        UBaseType_t           uxCriticalNesting;
	#endif  
	
	/* 由系统分配用于调试 */
	#if ( configUSE_TRACE_FACILITY == 1 )  
	    UBaseType_t           uxTCBNumber;
	    UBaseType_t           uxTaskNumber;
	#endif  
  
	#if ( configUSE_MUTEXES == 1 )  
		/* 保存任务原始优先级，用于互斥信号量的优先级翻转 */
	    UBaseType_t           uxBasePriority;
	    /* 记录任务获取的互斥信号量数量 */
	    UBaseType_t           uxMutexesHeld;  
	#endif  
	
	/* 用户可自定义任务的钩子函数用于调试 */
	#if ( configUSE_APPLICATION_TASK_TAG == 1 )  
		TaskHookFunction_t pxTaskTag;  
	#endif  
	
    /* 保存任务独有的数据 */
	#if( configNUM_THREAD_LOCAL_STORAGE_POINTERS > 0 )  
		void *pvThreadLocalStoragePointers [configNUM_THREAD_LOCAL_STORAGE_POINTERS];  
	#endif  
	
    /* 记录任务处于运行态的时间 */
	#if( configGENERATE_RUN_TIME_STATS == 1 )  
	    uint32_t  ulRunTimeCounter;
	#endif  
	
    /* 用于 Newlib */
	#if ( configUSE_NEWLIB_REENTRANT == 1 )  
		struct _reent xNewLib_reent;  
	#endif  
  
	#if( configUSE_TASK_NOTIFICATIONS == 1 )  
		/* 任务通知值 */
		volatile uint32_t ulNotifiedValue;  
		/* 任务通知状态 */
		volatile uint8_t ucNotifyState;  
	#endif  
	
    /* 任务静态创建标志 */
	#if( tskSTATIC_AND_DYNAMIC_ALLOCATION_POSSIBLE != 0 )
		uint8_t        ucStaticallyAllocated;
	#endif  
	
    /* 任务被中断延时标志 */
	#if( INCLUDE_xTaskAbortDelay == 1 )  
		uint8_t ucDelayAborted;  
	#endif  
	
	/* 用于 POSIX */
	#if( configUSE_POSIX_ERRNO == 1 )  
		int iTaskErrno;  
	#endif  
  
} tskTCB;
```

# 5. 任务栈
不论是裸机编程还是 RTOS 编程，栈空间的使用的非常重要。函数中的局部变量、函数调用时的现场保护和函数的返回地址等都是存放在栈空间中的。

对于 FreeRTOS，当使用**静态方式**创建任务时，需要用户自行分配一块内存，作为任务的栈空间。

```C
/**  
 * @param pxTaskCode     指向任务函数的指针
 * @param pcName         任务名，最大长度为 configMAX_TASK_NAME_LEN
 * @param usStackDepth   任务堆栈大小，单位：字（注意，单位不是字节）
 * @param pvParameters   传递给任务函数的参数
 * @param uxPriority     任务优先级，最大值为(configMAX_PRIORITIES-1)
 * @param puxStackBuffer 任务栈指针，内存由用户分配提供
 * @param pxTaskBuffer   任务控制块指针，内存由用户分配提供
 * @return   NULL        用户没有提供相应的内存，任务创建失败
 *           others      任务句柄，任务创建成功
 */  
TaskHandle_t xTaskCreateStatic( TaskFunction_t pxTaskCode,  
					            const char * const pcName,
					            const configSTACK_DEPTH_TYPE usStackDepth,  
					            void * const pvParameters,  
					            UBaseType_t uxPriority,  
					            StackType_t * const puxStackBuffer,
					            StaticTask_t * const pxTaskBuffer);
```

参数 usStackDepth 表示的任务栈大小，实际上是**以字为单位**的，并非以字节为单位。静态和动态创建任务时，任务栈的大小都与数据类型 StackType_t 有关，是**uint32_t**。