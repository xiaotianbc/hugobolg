---
title: "使用windows+Clion开发树莓派pico RP2040简易入门指南"
date: 2022-03-25T10:15:51+08:00
draft: false
tags: ["pico","嵌入式"]
---



## Pico 规格参数

- 双核 Arm Cortex-M0 + @ 133MHz
- 芯片内置 264KB SRAM 和 2MB 的板载闪存
- 通过专用 QSPI 总线支持最高 16MB 的片外闪存
- DMA 控制器
- 30 个 GPIO 引脚，其中 4 个可用作模拟输入

- 2 个 UART、2 个 SPI 控制器和 2 个 I2C 控制器
- 16 个 PWM 通道
- USB 1.1 主机和设备支持
- 8 个树莓派可编程 I/O（PIO）状态机，用于自定义外围设备支持
- 支持 UF2 的 USB 大容量存储启动模式，用于拖放式编程

## 在 Pico 上使用 C/C++前置工具

树莓派官方提供了一个大而全的上手指南：https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf

如果你想要的是简单直接的上手指南，本文可以提供一个最小化的建议。

首先你需要安装以下前置软件，**并将其加入Path环境变量**：

* arm none eabi gcc ，用于编译Cortex-M系列单片机的编译器，下载地址：https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads

* Clion ， 核心灵魂 （不要忘记安装官方的简体中文语言包哦）
* Python 3
* Git

另外，在官方的教程中，还需要安装来自microsoft的Build Tools for Visual Studio 2019 ，主要是用于nmake生成工具，但是此工具至少需要占用6g的硬盘空间，而且完全不是必须的，这里推荐使用mingw64的生成器，只需要几百MB的硬盘空间，而且更加好用：https://www.msys2.org/，请根据官网的[Getting Started](https://www.msys2.org/)教程，至少安装完成 `pacman -S --needed base-devel mingw-w64-x86_64-toolchain`。

## 获取SDK和样例

在合适的位置，使用类似以下的代码即可获取官方的sdk和一些例子：

```bash
C:\Users\pico\Downloads> git clone -b master https://github.com/raspberrypi/pico-sdk.git
C:\Users\pico\Downloads> cd pico-sdk
C:\Users\pico\Downloads\pico-sdk> git submodule update --init
C:\Users\pico\Downloads\pico-sdk> cd ..
C:\Users\pico\Downloads> git clone -b master https://github.com/raspberrypi/pico-examples.git
```

## 使用Clion编译官方例程

打开Clion，在`设置`-`构建，执行，部署`里选择`工具链`，添加一个新的`mingw工具链`，工具集选择刚刚安装的`msys2`的位置，`CMake`选择`Clion`自带的即可，make会自动侦测到，`C/C++编译器`选择刚刚安装的Arm gcc的位置。

使用Clion打开文件夹 `pico-examples`，如果你遇到了错误，不要慌张，先在``设置`-`构建，执行，部署``-CMake里把当前的工具链切换成刚刚添加的`mingw工具链`，生成器选择为 `MinGW Makefiles`，最下面的环境选项中添加一条 `PICO_SDK_PATH=你的pico-sdk所在的位置\pico-sdk`，此时在点击确认，如果不出意外的话，应该已经可以成功加载cmake项目了。

在屏幕上方，运行目标的下拉菜单里，可以切换官方的例程，选择想要编译的例程，点击锤子按钮即可编译，编译完成的uf2文件在 `cmake-build-工具链名称`文件夹中，直接将其拖放到RP2040的bootloader里即可运行。

## 使用Clion开发自己的项目

在熟悉了官方例程之后，我们可以开始写自己的项目了。

1. 在合适的位置创建一个名为blinks的文件夹

2. 将pico-sdk\external\pico_sdk_import.cmake复制到项目文件夹中

3. 创建User/main.c，将以下内容复制粘贴过去。

   ```c
   #include "pico/stdlib.h"
   #include <stdio.h>
   const uint LED_PIN = 25;

   int main() {
       stdio_init_all();
       uint32_t cnt=0;
       gpio_init(LED_PIN);
       gpio_set_dir(LED_PIN, GPIO_OUT);
       while (true) {
           printf("Hello, world! %lu\n",cnt++);
           gpio_put(LED_PIN, 1);
           sleep_ms(1000);
           gpio_put(LED_PIN, 0);
           sleep_ms(1000);
       }
   }
   ```

   4. 创建CMakeLists.txt，将以下内容复制粘贴过去即可，此Cmakelists的格式稍微参考了STM32的格式，有需要也可以自己修改

      ```cmake
      cmake_minimum_required(VERSION 3.12)

      # 设置pico-sdk的路径
      set(PICO_SDK_PATH "E:/mcu_projects/RP2040/picoCmyself/pico-sdk")

      # 引入SDK (必须写在下面的project行前)
      include(pico_sdk_import.cmake)

      # 工程配置
      project(Blinks)
      set(CMAKE_CXX_STANDARD 17)
      set(CMAKE_C_STANDARD 11)

      # 初始化SDK
      pico_sdk_init()

      # 自定义优化等级
      add_compile_options(-Os)

      #预处理器搜索头文件的路径
      include_directories(
              User
      )

      #所有需要编译的源文件
      file(GLOB_RECURSE SOURCES
              "User/*.*"
              )

      #添加项目的源文件
      add_executable(${PROJECT_NAME}
              ${SOURCES}
              )

      # 引入pico_stdlib库
      target_link_libraries(${PROJECT_NAME} pico_stdlib)

      # enable usb output, disable uart output
      pico_enable_stdio_usb(${PROJECT_NAME} 1)
      pico_enable_stdio_uart(${PROJECT_NAME} 0)

      # create map/bin/hex/uf2 file etc.
      pico_add_extra_outputs(${PROJECT_NAME})
      ```

4. 用Clion打开此文件夹，编译即可。

