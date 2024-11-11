OpenOCD是一个用来调试、烧录嵌入式SOC的软件，需要搭配debug adapter（比如JLink，ST-Link，DAP-Link）和GDB（或Telnet）一起使用。
![[Pasted image 20240703145014.png]]
OpenOCD**开源**免费，支持的SOC和debug adapter**非常众多**。可以使用**命令行**交互，好定制，并且支持Windows，Linux等**多平台**，还**稳定**。
# OpenOCD 命令行
OpenOCD可以通过命令行交互，以下是基本命令， `openocd -h `获得安装OpenOCD的基本信息，包括版本和支持的操作。
`openocd -c`后接具体操作，完成指定任务。但是通常后只接一个操作，多个操作还需再次`-c`。
openocd自己也提供了一些固有的配置文件，不用使用者每次一个个输入，此时就可以通过`openocd -f`直接读取配置文件，完成里面写好的固定操作。
`openocd -d`修改openocd的输出调试信息等级，后面不接数字等级的话，则会设置最高等级3，此时输出信息最多，包括调试消息。默认输出等级为2，此时只输出信息性消息、警告和错误。

```shell
%% 获取openocd帮助信息 %%
openocd -h 
%% 通过openocd执行操作 %%
openocd -c <command>
%% 通过openocd打开配置文件执行操作 %%
openocd -f <config>
%% 调整openocd 的调试信息等级 %%
openocd -d <level>
```

# 自定义配置
openocd自己提供了一些配置在自己安装位置下\share\openocd\scripts，但是这绝对不能满足全部的需要，并且也许我们需要导入多次配置，所以我们需要针对自己的情况，自定义配置。

最简单的配置便是将多个配置结合起来。在openocd提供的众多配置中其中interface和target中提供的配置是最为重要的，前者指定了我们将会使用什么debug-adepter，后者指定我们会将目标芯片是什么，所以最简单的配置便是根据实际情况，将interface中的配置和target中的配置结合起来，适配我们的情况。
```TCL
%% 指定使用stlink为调试器，stm32f4xx为目标芯片 %%
source [find interface/stlink.cfg] 
source [find target/stm32f4x.cfg]
```

>[!NOTE] openocd 将会搜寻配置文件位置：
> 1. 当前目录
> 2. 使用 `-s`选项在命令行上指定的任何搜索目录
> 3. 使用 `add_script_search_dir`命令指定的任何搜索目录
> 4. OPENOCD_SCRIPTS环境变量中的目录
> 5. `%APPDATA%/OpenOCD`(win)
> 6. `$HOME/Library/Preferences/org.openocd`(Darwin)
> 7. `$XDG_CONFIG_HOME/openocd ($XDG_CONFIG_HOME defaults to $HOME/.config)`
> 8. `$HOME/.openocd`
> 9. openocd安装地址下的script

保存这个到一个新的`cfg`文件，则构成了一个新的配置文件。

整个cfg文件是使用TCL语言写的，我们没必要重头学习，只需要必要的指令便可
```
%% [find] 查找配置，并返回完整路径 %%

adapter driver cmsis-dap
transport select swd
adapter_khz 500
```