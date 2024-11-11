
```cmake
############################################################
# 1. 指定编译器和链接器，避免使用默认的 gcc
############################################################
# 设置编译器 C 的编译器
set(CMAKE_C_COMPILER armclang.exe)
set(CMAKE_C_COMPILER_WORKS TRUE)

# 即便用不到C++ 的编译器，还是要显示说明的,否则报错
set(CMAKE_CXX_COMPILER armclang.exe)
set(CMAKE_CXX_COMPILER_WORKS TRUE)

#设置 ASM 的编译器（不设置配合 -masm=auto 使用）
set(CMAKE_ASM_COMPILER armclang.exe)
#set(CMAKE_ASM_COMPILER armasm.exe)     # 指明 ASM 编译器，配合 第二种 CMAKE_ASM_FLAGS_INIT 方式使用
set(CMAKE_ASM_COMPILER_WORKS TRUE)

#设置链接器
set(CMAKE_C_LINK_EXECUTABLE armlink.exe)
set(CMAKE_ASM_LINK_EXECUTABLE armlink.exe)
set(CMAKE_CXX_LINK_EXECUTABLE armlink.exe)


############################################################
# 2. 获取芯片地址描述信息
# 获取当前 MCU 的 section 描述，及存储空间和起始地址的描述
# 类似于ARMGCC中的ld文件，是由keil产生的
set(SECTIONS_SCRIPT_PATH ${CMAKE_SOURCE_DIR}/${PROJECT_NAME}.sct)


############################################################
# 3. 设置与芯片对应的 --target 编译选项

set(C_TARGET_FLAG --target=arm-arm-none-eabi)      # MDK 的 link 分页中的配置
set(ASM_TARGET_FLAG --target=arm-arm-none-eabi)    # MDK 的 link 分页中的配置(不支持显示指定 ASM 编译器的方式)

set(LINKER_TARGET_FLAG --cpu=${mcpu})
set(COMPILE_RULE_FLAG "-mcpu=${mcpu}")

# 设置 C 编译器选项(这里就把MDK中的 C/C++ 分页里最下边一栏的属性贴进来)
# 参数 -w 表示忽略所有警告，不然要配具体忽略哪些警告，尽管贴过来也行，但是太乱
# 优化选项 -O 有 1~3   -Os 是平衡  -Oz 是最小体积
set(CMAKE_C_FLAGS_INIT "${C_TARGET_FLAG} ${COMPILE_RULE_FLAG} \
    -xc -std=c99 -fno-rtti -funsigned-char -fshort-enums -fshort-wchar \
    -gdwarf-3 -Oz -ffunction-sections -w")

# 设置 C++ 编译器选项(没有用到 c++ 所以不用配置)
#set(CMAKE_CXX_FLAGS_INIT ${CMAKE_C_FLAGS_INIT})

# 设置ASM编译器选项
# 注意： -masm=auto 选项是 MDK 的 link 分页里没有的参数，需要加上
set(CMAKE_ASM_FLAGS_INIT "${ASM_TARGET_FLAG} ${COMPILE_RULE_FLAG} \
        -masm=auto -c -gdwarf-3 ")
# 第二种方式 配套显示执行 armasm.exe 为 ASM 编译器的方法，看上起更清晰一些
#set(CMAKE_ASM_FLAGS_INIT "${ASM_TARGET_FLAG} --cpu=${mcpu}")



# 设置链接器选项
# 这些参数再 ARMCC 文档里么有，但 MDK 的 link 分页有，
# 使用忽略所有警告的配置时没有 --map 及其之后的内容， 这里根据需要保留了一些信息，在 demo.map 中可以看到
set(CMAKE_EXE_LINKER_FLAGS_INIT  " \
            ${LINKER_TARGET_FLAG} \
            --strict \
            --scatter ${SECTIONS_SCRIPT_PATH} \
            --info sizes --info totals --info unused --info veneers \
            --summary_stderr \
            --info summarysizes"
        )


############################################################
# 项目设置
############################################################ 
# 指定项目名称和语言标准C++17与C11
project(${projectName} C CXX ASM)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_C_STANDARD 99)

############################################################
# 编译选项
############################################################ 
#可选的设置硬件浮点运算的编译和链接选项，可在使用FreeRTOS编译报错时设置
#add_compile_definitions(ARM_MATH_CM4;ARM_MATH_MATRIX_CHECK;ARM_MATH_ROUNDING)
#add_compile_options(-mfloat-abi=hard -mfpu=fpv4-sp-d16)
#add_link_options(-mfloat-abi=hard -mfpu=fpv4-sp-d16)
#可选的设置软件浮点运算的编译
#add_compile_options(-mfloat-abi=soft)

############################################################
# 设置项目结构，包含文件
############################################################
# 设置头文件目录
include_directories(${includes})
# 添加预处理器定义
add_definitions(${defines})
# 递归获取所有源文件
file(GLOB_RECURSE SOURCES ${sources})

# 对于混合兼容的环境，需要屏蔽各种编译环境引起的文件“干扰”,通过 list(REMOVE_ITEM) 命令移除不同编译环境下的干扰文件
# 在原来 CubeMX 自动生成的 gcc 编译环境目录上,附加 ARMCC 编译需要的文件
file(GLOB_RECURSE SOURCES ${SOURCES} "MDK-ARM/startup_stm32f103xb.s")
# 将由 CubeMX 生成的 GCC 编译环境中的会干扰ARMCC环境的文件，放在 EXCLUDE_SRCS 自定义列表中
file(GLOB_RECURSE EXCLUDE_SRCS
        "Middlewares/Third_Party/RealThread_RTOS/libcpu/arm/cortex-m3/context_gcc.S"
        "startup/*.*"
        "Core/Src/syscalls.c"
        "STM32F103C8Tx_FLASH.ld"
        )
# 从源文件列表(SOURCES)中剔除干扰文件(EXCLUDE_SRCS)
list(REMOVE_ITEM SOURCES ${EXCLUDE_SRCS})


add_executable(${PROJECT_NAME} ${SOURCES} ${LINKER_SCRIPT})

set(HEX_FILE $${PROJECT_BINARY_DIR}/$${PROJECT_NAME}.hex)
set(BIN_FILE $${PROJECT_BINARY_DIR}/$${PROJECT_NAME}.bin)
set(ELF_FILE $${PROJECT_BINARY_DIR}/$${PROJECT_NAME}.elf)

# 使用 armclang 自带的 fromelf 工具，实现 elf 转 hex
set(ARMCC_fromelf fromelf.exe)
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        # 相当于fromelf.exe" --i32combined --output="xxx/demo.hex" "xxx/demo.elf"
        COMMAND ${ARMCC_fromelf} --i32combined --output="${HEX_FILE}" "${ELF_FILE}"
        COMMENT "Building ${HEX_FILE}"
        )

```

具体参数是由keil提供的，主要差别主要有两个，一个启动文件，一个链接脚本。