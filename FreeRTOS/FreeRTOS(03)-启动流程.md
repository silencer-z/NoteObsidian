一般来说，FreeRTOS需要在系统复位后完成硬件的初始化后，开始FreeRTOS初始化然后开始调度，关于如何启动任务也有两种方式：
- FreeRTOS初始化完成后直接完成**所有**任务的创建
- FreeRTOS初始化完成后先创建**一个**任务在这个任务中完成所有任务的创建
两种创建任务的方式并无优劣都可以。

关于FreeRTOS的初始化，其实是指在CMSIS-RTOS中 `osKernelInitialize` 。在函数中在非中断时将内核未激活状态调整为打开，如果FreeRTOS使用的是heap_5则还会对对应的堆栈区域进行初始化。

# FreeRTOS开启调度
前面的任务主要完成FreeRTOS的一些小配置，而在调度中FreeRTOS才会真正的展开，我们将以FreeRTOS CMSIS-RTOS V2为例进行讲解(官网版本大差不差只是CMSIS-RTOS对部分函数进行了统一封装，方便移植)。

```C
osStatus_t osKernelStart (void) {  
  osStatus_t stat;  
  
  if (IS_IRQ()) {  
    stat = osErrorISR;  
  }  
  else {  
    if (KernelState == osKernelReady) {  
      /* 复位SVC的中断，优先级为0*/  
      SVC_Setup();  
      /* Change state to enable IRQ masking check */  
      KernelState = osKernelRunning;  
      /* 开始FreeRTOS自身的调度*/  
      vTaskStartScheduler();  
      stat = osOK;  
    } else {  
      stat = osError;  
    }  
  }  
  return (stat);  
}
```

很明显 `osKernelStart` 也只是套壳程序，实质上还是调用 `vTaskStartScheduler` 开启自身的调度，下面将讲解其关键节点。

## 