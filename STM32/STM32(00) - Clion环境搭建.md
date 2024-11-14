## 工具链配置
1. 库文件生成依赖STMcubeMX 生成项目格式为 STMcubeMXIDE debug口为串口
2. 编译工具 mingw使用Clion自带的 arm-none-eabi-gcc 从arm官网下载注意区别，arm-none-eabi-gcc应该加入环境变量PATH中去
3. 安装烧录工具 openocd
4. 将openocd地址和STM32CubeMX的程序位置填入Clion相关设置中
# STM32cubeMX配置
## 外部时钟设置
芯片需要一个频率来进行工作，通常选用的是晶振来提供工作频率。虽然芯片内部也能够提供时钟，但是其不够稳定，只能用在一些不是需要很精准的模块中，所以便需要一个外部时钟为我们提供更加稳定的时钟频率。外部时钟包括：
- HSE高速外部时钟
- LSE低速外部时钟
前者是系统CPU时钟的最**主要**的时钟，后者则是**供给RTC和WDG**使用的，当不食用这两个外设的时候，是没有必要为LSE使能的。在STM32cubeMX中关于时钟RCC的设置中，外部时钟有三个选择：
- Corerystal/Ceramic Resonator
- BYPASS ClockSource
- Disable
最后一个选项**Disable**显而易见，指不开启该时钟源，**Corerystal/Ceramic Resonator**是指使用晶体/陶瓷谐振器提供时钟源，更广义上是指无源晶振，无需外部供电。**BYPASS Clock Resource**是指旁路时钟源，更广义是指有源晶振，需要外部供电，输出固定频率波形。一般大多数情况都是使用Corerystal/Ceramic Resonator。
![[Pasted image 20240601183339.png]]
Master Clock Output选项则会将设置的时钟源通过引脚输出。
![[Pasted image 20240601183535.png]]
在RCC的配置只需要注意HSI(内部高速时钟)的频率设置，与时钟引脚的设置，其实一般情况也无需考虑。
## 时钟树配置
STM32CubeMX中将时钟的配置图形化，可以更加方便的配置时钟源，以下就以f103的时钟为例：![[Pasted image 20240601184311.png]]
可以看到，来自是时钟源的时钟信号，经过处理设置为合适的频率送入不同的外设当中去，其中两个部分的设置尤为重要，一个是**System Clock Mux** 另一个是**PLL Source Mux**。前者选择将不同的时钟源作为系统时钟，一般选择有HSI、HSE、PLLCLK。HSI根据温度和环境的情况频率会飘逸不稳定，而PLLCLK相对于HSE可以达到更高的频率。PLL Source Mux则是将输出时钟信号倍频，通过这两者的设置，便可为系统提供更高、更稳定的时钟信号，具体的设置可以参考外设提示的最大频率设置。
## 代码生成配置
### 项目配置

![[Pasted image 20240703165822.png]]

在项目配置这里着重需要注意的是工具链设置，可以选择STM32cubeIDE，Cmake，makefile，MDK-ARM分别是不同IDE管理项目的形式，我们使用CLion则需要设置为STM32cubeIDE，若是使用keil则需要设置为MDK-ARM，并指定版本。

堆栈的大小根据具体的项目要求完成设置。

### 代码生成配置

![[Pasted image 20240703170058.png]]
对于我们没有用到的组件，我们可以选择不将其置于项目中，只添加必要的文件，这样可以提高编译速度和降低项目所占存储大小。
选择将.c/.h文件分开，可以使得整个项目结构更加的清晰。
## 调试串口配置
在调试中，我们经常需要通过串口打印出一些我们需要的数据来辅助调试，使用STM32cubeMX之后，也需要进行这方面的配置方便我们调试。
首先我们需要使能串口
![[Pasted image 20240601191537.png]]
将其配置好，然后在生成代码之后，添加关于printf和scanf的重定向文件，将其加入cmakelist中便可以使用scanf和printf了。之后会说明具体目录结构和推荐位置。
``` c
// retarget.h
#ifndef _HAL_UART_RETARGET_H__  
#define _HAL_UART_RETARGET_H__  
  
#include <stdio.h>  
#include "stm32f1xx_hal.h"  //根据芯片型号改变
  
#ifdef __GNUC__  
/* With GCC, small printf (option LD Linker->Libraries->Small printf  
   set to 'Yes') calls __io_putchar() */
	#define PUTCHAR_PROTOTYPE int __io_putchar(int ch)  
#else  
	#define PUTCHAR_PROTOTYPE int fputc(int ch, FILE *f)  
#endif 

/* __GNUC__ */  
#ifdef __GNUC__  
	#define GETCHAR_PROTOTYPE int __io_getchar (void)  
#else  
	#define GETCHAR_PROTOTYPE int fgetc(FILE * f)  
#endif 
/* __GNUC__ */  
  
/**  
 * @brief 注册重定向串口  
 * @param usartx 需要重定向输入输出的串口  
 */
void RetargetInit(UART_HandleTypeDef *huart);  
#endif //_HAL_UART_RETARGET_H__
```

``` c
// retarget.c
#include "retarget.h"  
  
/* 告知连接器不从C库链接使用半主机的函数 */
#pragma import(__use_no_semihosting)  
  
/* 定义USART端口，用于注册重定向的串口 */
static UART_HandleTypeDef *sg_hUart;  
  
/**  
 * @brief 注册重定向串口  
 * @param usartx 需要重定向输入输出的串口句柄  
 */
 void RetargetInit(UART_HandleTypeDef *huart) {  
    /* 注册串口 */    
    sg_hUart = huart;  
  
    /* Disable I/O buffering for STDOUT stream, so that  
     * chars are sent out as soon as they are printed. */
    setvbuf(stdout, NULL, _IONBF, 0);  
    
    /* Disable I/O buffering for STDIN stream, so that  
     * chars are received in as soon as they are scanned. */    
    setvbuf(stdin, NULL, _IONBF, 0);  
}  
  
/**  
  * @brief  Retargets the C library printf function to the USART.  
  * @param  None  
  * @retval None  */
PUTCHAR_PROTOTYPE {  
	/* 发送一个字节数据到串口 */    
	/* 很简单，直接调用 HAL 库中的 串口发送数据函数 */
	HAL_UART_Transmit(sg_hUart, (uint8_t *)&ch, 1, 0xFFFF);  
    /* 返回发送的字符 */
    return ch;  
}  
  
/**  
  * @brief  Retargets the C library scanf, getchar function to the USART.  
  * @param  None  
  * @retval None  */
  GETCHAR_PROTOTYPE {  
    /* 用于接收数据 */    
    uint8_t ch;  
    /* 调用 HAL 库中的接收函数 */    
    HAL_UART_Receive(sg_hUart, (uint8_t *) &ch, 1, 0xFFFF);  
    /* 直接返回接收到的字符 */    
    return (int) ch;  
}
```

>[!WARNING] 使用scanf时需要主要，字符串需要**以空格结尾或者回车结尾**。否则无法发送，且回车与空格将会被当做下一次输入的一部分
# Clion配置
## 编译配置
一般来说，通过Clion产生的项目，Clion会自动生成一个Cmake应用程序，可以通过它完成程序的编译工作，但是无法烧录程序。
![[Pasted image 20240706200512.png]]
## 驱动程序库添加

1. 将驱动文件加入项目目录
2. 编辑CMake_template.txt (自己理解是CLion依据这个文件结合STMcubeMX生成Cmake)
3. 右键选中CMake_template.txt,选择使用STM32CubeMX更新CMake项目 

>[!NOTE] 头文件需要添加.h文件上一级文件夹位置，源文件不必每一个都加进去，只需要加一个总的源文件就可以

```Cmake
include_directories(${includes} 文件夹1/ 文件夹2/ 文件夹3/)
add_definitions(${defines})
file(GLOB_RECURES SOURCES ${sources} "文件夹1/*.*" "文件夹2/*.*" "文件夹3/*.*")
```
我们以正点原子提供的Sys.h和之前串口配置文件为例
![[Pasted image 20240706195800.png]]![[Pasted image 20240706195918.png]]