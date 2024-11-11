# 任务创建
FreeRTOS 有四种不同的创建任务函数和一种删除任务函数：
![[Pasted image 20241028104922.png]]
## xTaskCreate()
此函数用于使用动态的方式创建任务，任务的任务控制块以及任务的栈空间所需的内存，均由 FreeRTOS **从 FreeRTOS 管理的堆中分配**，若使用此函数，需要在 FreeRTOSConfig.h 文件中将宏 `configSUPPORT_DYNAMIC_ALLOCATION` 配置为 1。此函数创建的任务会立刻进入就绪态，由任务调度器调度运行。
```C
/**  
 * @param pxTaskCode     指向任务函数的指针
 * @param pcName         任务名，最大长度为 configMAX_TASK_NAME_LEN
 * @param usStackDepth   任务堆栈大小，单位：字（注意，单位不是字节）
 * @param pvParameters   传递给任务函数的参数
 * @param uxPriority     任务优先级，最大值为(configMAX_PRIORITIES-1)
 * @param pxCreatedTask  任务句柄,也就是任务的任务控制块
 * @return   pdPASS      任务创建成功
 *           errCOULD_NOT_ALLOCATE_REQUIRED_MEMORY 内存不足，任务创建失败
 */  
BaseType_t xTaskCreate( TaskFunction_t pxTaskCode,  
			            const char * const pcName,
			            const configSTACK_DEPTH_TYPE usStackDepth,  
			            void * const pvParameters,  
			            UBaseType_t uxPriority,  
			            TaskHandle_t * const pxCreatedTask );
```
## xTaskCreateStatic()
此函数用于使用静态的方式创建任务，任务的任务控制块以及任务的栈空间所需的内存，需要由**用户分配提供**，若使用此函数，需要在FreeRTOSConfig.h文件中将宏 `configSUPPORT_STATIC_ALLOCATION` 配置为 1。此函数创建的任务会立刻进入就绪态，由任务调度器调度运行。
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
## xTaskCreateRestricted()
此函数用于使用动态的方式创建**受 MPU 保护**的任务，任务的任务控制块以及任务的栈空间所需的内存，均由 FreeRTOS**从FreeRTOS 管理的堆中分配**，若使用此函数，需要将宏`configSUPPORT_DYNAMIC_ALLOCATION` 和宏 `portUSING_MPU_WRAPPERS` 同时配置为 1。此函数创建的任务会立刻进入就绪态，由任务调度器调度运行。
```C
/**  
 * @param pxTaskDefinition   指向任务参数结构体的指针
 * @param pxCreatedTask      任务句柄,也就是任务的任务控制块
 * @return   pdPASS          任务创建成功
 *           errCOULD_NOT_ALLOCATE_REQUIRED_MEMORY 内存不足，任务创建失败
 */  
BaseType_t xTaskCreateRestricted(
					const TaskParameters_t * const pxTaskDefinition,  
					TaskHandle_t * const pxCreatedTask );
```
## xTaskCreateRestrictedStatic()
此函数用于使用静态的方式创建**受 MPU 保护**的任务，此函数创建的任务的任务控制块以及任务的栈空间所需的内存，需要由**用户自行分配提供**，若使用此函数，需要将宏`configSUPPORT_STATIC_ALLOCATION` 和宏 `portUSING_MPU_WRAPPERS` 同时配置为 1。此函数创建的任务会立刻进入就绪态，由任务调度器调度运行。
```C
/**  
 * @param pxTaskDefinition   指向任务参数结构体的指针
 * @param pxCreatedTask      任务句柄,也就是任务的任务控制块
 * @return   pdPASS          任务创建成功
 *           errCOULD_NOT_ALLOCATE_REQUIRED_MEMORY 内存不足，任务创建失败
 */  
BaseType_t xTaskCreateRestrictedStatic (
					const TaskParameters_t * const pxTaskDefinition,  
					TaskHandle_t * const pxCreatedTask );
```
# 任务删除
## vTaskDelete()
此函数用于删除已被创建的任务，被删除的任务将被**从就绪态任务列表、阻塞态任务列表、挂起态任务列表和事件列表中移除**，要注意的是，空闲任务会负责释放被删除任务中由系统分配的内存，但是由用户在任务删除前申请的内存，则需要由用户在任务被**删除前提前释放**，否则将导致**内存泄露**。若使用此函数，需要在FreeRTOSConfig.h文件中将宏`INCLUDE_vTaskDelete`配置为 1。
```C
/**  
 * @param xTaskToDelete      待删除任务的任务句柄
 */  
void vTaskDelete(TaskHandle_t xTaskToDelete);
```
# 任务挂起与恢复
## vTaskSuspend()
此函数用于挂起任务，无论优先级如何，**被挂起的任务都将不再被执行，直到任务被恢复**。此函数并不支持嵌套，不论使用此函数重复挂起任务多少次，只需调用一次恢复任务的函数，那么任务就不再被挂起。若使用此函数，需要在FreeRTOSConfig.h文件中将宏`INCLUDE_vTaskSuspend` 配置为 1。
```C
/**  
 * @param xTaskToSuspend      待挂起任务的任务句柄
 */  
void vTaskSuspend(TaskHandle_t xTaskToSuspend);
```

## vTaskResume()
此函数用于在**任务中**恢复被挂起的任务，不论一个任务被函数`vTaskSuspend()`挂起多少次，只需要使用函数`vTaskResume()`恢复一次，就可以继续运行。若使用此函数，需要在 FreeRTOSConfig.h 文件中将宏 `INCLUDE_vTaskSuspend` 配置为 1。
```C
/**  
 * @param xTaskToResume      待恢复任务的任务句柄
 */  
void vTaskResume(TaskHandle_t xTaskToResume);
```

## xTaskResumeFromISR()
此函数用于在**中断中**恢复被挂起的任务，若使用此函数，需要在 FreeRTOSConfig.h 文件中将宏 `INCLUDE_xTaskResumeFromISR` 配置为 1。不论一个任务被函数`vTaskSuspend()`挂起多少次，只需要使用函数`vTakResumeFromISR()`恢复一次，就可以继续运行。
```C
/**  
 * @param xTaskToResume      待恢复任务的任务句柄
 */  
void xTaskResumeFromISR(TaskHandle_t xTaskToResume);
```