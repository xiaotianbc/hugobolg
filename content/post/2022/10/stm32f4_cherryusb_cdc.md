---
title: "STM32F4标准库-移植cherryusb并测试cdc"
date: 2022-10-06T10:15:51+08:00
draft: false
tags: ["嵌入式"]
---

usb cdc是一种非常常用的通信工具，可以在不需要外接串口设备的情况下实现连接到电脑的串口输出。
cherryusb是一款国产的小而美的usb协议栈：https://github.com/sakumisu/CherryUSB

本文主要介绍使用STM32F401+标准外设库，通过cherryusb实现usb cdc通信的过程。
本文主要参考以下文章，只是将里面的Hal库的内容替换为标准库的内容：https://cherryusb.readthedocs.io/zh_CN/latest/quick_start/stm32.html
移植要点
● 首先需要准备一个调通了的STM32F4的标准库模板程序，完成对外置晶振的初始化，如果有调试串口的初始化更好。
● 添加USB底层初始化程序：

/**
 * 初始化USB-FS-Device的gpio和时钟电源
 */
void mcu_usb_device_init(void) {
    GPIO_InitTypeDef GPIO_InitStructure;
    RCC_AHB2PeriphClockCmd(RCC_AHB2Periph_OTG_FS, ENABLE);
    RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOA, ENABLE);


    /* Configure DM DP Pins */
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_11 | GPIO_Pin_12;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_100MHz;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF;
    GPIO_InitStructure.GPIO_OType = GPIO_OType_PP;
    GPIO_InitStructure.GPIO_PuPd = GPIO_PuPd_NOPULL;
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    GPIO_PinAFConfig(GPIOA, GPIO_PinSource11, GPIO_AF_OTG1_FS);
    GPIO_PinAFConfig(GPIOA, GPIO_PinSource12, GPIO_AF_OTG1_FS);

    RCC_APB2PeriphClockCmd(RCC_APB2Periph_SYSCFG, ENABLE);

    NVIC_InitTypeDef NVIC_InitStructure;

    //开启 OTG_FS_IRQ中断，此中断的回调函数，已经在协议栈中实现，我们无需关注，非常轻松
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_4);
    NVIC_InitStructure.NVIC_IRQChannel = OTG_FS_IRQn;
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 3;
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    NVIC_Init(&NVIC_InitStructure);
}
● 把CherryUSB的release 代码拉下来放到项目里：
●
● STM32F4的USB ip叫dwc2，我们这次要测试的是cdc 功能，那就把对应的头文件和源文件添加到项目中，顺便把cdc_acm_template加入项目，我们需要编写业务代码时就参考此文件里面函数的调用方法：
●
●
● 复制一份 cherryusb_config_template.h，放到 Core/Inc 目录下，并命名为 usb_config.h
●
● 添加dwc2ip使用全速usb的宏定义
●
● 在main文件中重新实现usb_dc_low_level_init函数，这个函数是个弱定义，我们在函数中添加底层驱动的初始化：
void usb_dc_low_level_init(void) {
    extern void mcu_usb_device_init();
    mcu_usb_device_init();
}
● 参考cdc_acm_template文件里面的cdc_acm_data_send_with_dtr_test函数，我们可以大概知道阻塞发送usb cdc 的形式：
void cdc_acm_data_send_with_dtr_test(void) {
    //此处要判断dtr_enable的状态，是因为在usb尚未初始化成功时，dtr永远不可能enable.
    //如果在这里不做判断，有可能导致主机尚未完全初始化，写ep失败，就进入下面的死循环。
    if (dtr_enable) {
        memset(&write_buffer[10], 'a', 2038);
        ep_tx_busy_flag = true;
        usbd_ep_start_write(CDC_IN_EP, write_buffer, 2048);
        while (ep_tx_busy_flag) {
        }
    }
}
下面可以实现我们自己的测试函数和printf函数：

USB_NOCACHE_RAM_SECTION USB_MEM_ALIGNX uint8_t my_write_buffer[2048];
volatile uint32_t err_cnt = 0;


/**
 * 随机初始化发送buffer
 */
void cdc_acm_data_ready() {
    for (int i = 0; i < 2048; ++i) {
        my_write_buffer[i] = (i % 60) + 30;
    }
}

/**
 * 一次发送2048字节的测试
 */
void cdc_acm_data_send_test(void) {
    ep_tx_busy_flag = true;
    //如果返回值为0，则说明写ep成功，才能在此等待发送完成后回调函数把ep_tx_busy_flag置0
    if (usbd_ep_start_write(CDC_IN_EP, my_write_buffer, 2048) == 0) {
        while (ep_tx_busy_flag) {
        }
    } else {
        err_cnt++;
    }
}

#include "stdarg.h"

/**
 * 自定义cherryusb_cdc版本的printf
 */
void cherryusb_cdc_printf(int8_t *fmt, ...) {
    int32_t length = 0;
    va_list ap;
    int8_t *pt;

    va_start(ap, fmt);
    vsprintf((char *) my_write_buffer, (const char *) fmt, ap);
    pt = &my_write_buffer[0];
    while (*pt != '\0') {
        length++;
        pt++;
    }

    ep_tx_busy_flag = true;
    //如果返回值为0，则说明写ep成功，才能在此等待发送完成后回调函数把ep_tx_busy_flag置0
    if (usbd_ep_start_write(CDC_IN_EP, my_write_buffer, 2048) == 0) {
        while (ep_tx_busy_flag) {
        }
    } else {
        err_cnt++;
    }

    va_end(ap);
}

● 然后在主程序中就可以调用了：

RCC_ClocksTypeDef RCC_Clocks;

int main(void) {
    /* SysTick end of count event each ms */
    RCC_GetClocksFreq(&RCC_Clocks);
    SysTick_Config(RCC_Clocks.HCLK_Frequency / 1000);
    Delay(50); /* Insert 50 ms delay */
    mcu_uart_open(PC_PORT); //初始化串口，完成基本的调试打印
    printf("hello cherryusb test\n");
    extern void cdc_acm_init();
    cdc_acm_init();
    //此处暂停1秒钟，主要目的是为了让USB主机完成对设备的初始化，枚举等工作，不然后面发送数据会发生错误
    Delay(1000);
    extern void cdc_acm_data_ready();
    cdc_acm_data_ready();
    extern void cdc_acm_data_send_test();
    extern volatile uint32_t err_cnt;
    while (1) {
        for (int i = 0; i < 100; ++i) {
            cdc_acm_data_send_test();
        }
        cherryusb_cdc_printf("\n\n\n\nerr cnt=%d\n\n\n\n", err_cnt);
        Delay(1000);
    }
}
