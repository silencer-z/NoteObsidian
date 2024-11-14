DMA，全称为：Direct Memory Access，即直接存储器访问。DMA 传输方式无需 CPU 直接 控制传输，也没有中断处理方式那样保留现场和恢复现场的过程，**通过硬件为RAM与I/O设备开辟一条直接传送数据的通路**，将数据从某一区域搬运到另一区域而不通过CPU，能使 CPU 的效率大为提高。

STM32 最多有 2 个 DMA 控制器（DMA2 仅存在大容量产品中），DMA1 有 7 个通道（TIM1~4、ADC1、SPI1、SPI/I2S2、I2C1~2和USART1~3）。DMA2 有 5 个通道。（TIM5~8、ADC3、SPI/I2S3、UART4、DAC通道1,2和SDIO）

**每个通道专门用来管理来自于一个或多个外设对存储器访问的请求**。还有一个仲裁器来协调各个 DMA 请求的优先权。一个通道对应着多个外设，他们的是或输入到DMA中，这就意味着同时只能有一个请求有效。

DMA1![[Pasted image 20240612140110.png]]

DMA2![[Pasted image 20240612142755.png]]

>DMA工作步骤：
>1. 接收DMA请求信号
>2. 根据优先级处理请求
>3. 给请求外设发送应答信号
>4. 从外设数据寄存器或者从当前外设/存储器地址寄存器指示的存储器地址取数据
>5. 存数据到外设数据寄存器或者当前外设/存储器地址寄存器指示的存储器地址
>6. 寄存器递减，继续下一数据传送
>7. DMA传送结束

## DMA 串口发送
以USART2 的DMA传输为例(实现通过DMA将数据从串口传出)
1. 配置USART2为接收模式并打开USART2全局中断
2. 添加USART2传输与接收对应DMA 的设置
3. 设置传输优先级为中等
4. 设置传输模式为正常模式
5. 设置DMA内存地址自增一个字节(Byte)

串口发送数据是将数据不断存进固定外设地址串口的发送数据寄存器(USARTx_TDR)。所以外设的地址是不递增。而内存储器存储的是要发送的数据，所以地址指针要递增，保证数据**依次被发出**

如果不开启串口中断，则程序只能发送一次数据,程序不能判断DMA传输是否完成，USART一直处于busy状态